# Domain 3: Author and maintain actions (18%)

This domain covers building your own actions and getting them to other people: the three action types, the metadata file, how inputs, outputs, state and workflow commands work from inside an action, the errors custom actions most often produce, and how to distribute, publish and version them. Many items show a short `action.yml`, Dockerfile or `uses:` line and ask which change is valid or what the action actually receives.

## 3.1 Create, structure and troubleshoot custom actions

- **Three types:**

|                     | JavaScript                       | Docker container              | Composite                               |
| ------------------- | -------------------------------- | ----------------------------- | --------------------------------------- |
| `runs.using`        | `node24` (`node20` still parses) | `docker`                      | `composite`                             |
| Runner OS           | Linux, Windows, macOS            | **Linux only**                | Any the job uses (steps must suit it)   |
| Start-up            | Fastest; Node is on the runner   | Image build or pull every job | Whatever its steps do                   |
| Environment         | Runner's, plus bundled packages  | Self-contained image          | Runner's; extras installed by its steps |
| Calls other actions | No                               | No                            | Yes, via `uses` steps                   |

- **Choose by the hard constraint:** must run on Windows/macOS → not Docker. Must not depend on preinstalled runner software → Docker. "These steps everywhere, no build" → composite. Wrapping existing Python/shell scripts on all three OSes → composite with `actions/setup-python` plus run steps calling files in the action folder.
- **Linux rule covers every container feature:** container actions, `docker://` steps, job containers and service containers. To keep most of a job on Windows/macOS, move the container step into its own Linux job and pass files as an artifact. Installing Docker on Windows/macOS does not lift it. Self-hosted runners need Linux **and** Docker; labels add no capabilities.
- **Composite action vs reusable workflow:** composite = one step inside the caller's job (same runner, workspace); reusable workflow = called at job level, runs as its own job.
- **Metadata file:** `action.yml` or `action.yaml` at the action's root. Required: **`name`, `description`, `runs`**. Optional: `author`, `inputs`, `outputs`, `branding`.
- **Inputs:**
  - `required: true` is **not enforced**; the action gets an empty value. Validate in code: `core.getInput('token', { required: true })`.
  - `default` can be an expression, e.g. `${{ github.token }}`. The job token is not in the step environment unless mapped, but an action can always reach `github.token`; callers override with a PAT or App token for wider access.
  - `deprecationMessage` warns whenever the input is set; the value still arrives.
  - JS and Docker actions receive `INPUT_<NAME>` env vars (upper-cased, spaces → underscores). Composite actions get **no** `INPUT_` variables; use `${{ inputs.<id> }}`.
- **References and layout:**

| Where the action lives                  | `uses:`               | Notes                                  |
| --------------------------------------- | --------------------- | -------------------------------------- |
| Root of a public repository             | `owner/repo@ref`      | Ref always last                        |
| Subfolder (several actions in one repo) | `owner/repo/path@ref` | Each folder has its own `action.yml`   |
| Same repository                         | `./path/to/action`    | Needs `actions/checkout` first; no ref |
| Prebuilt image                          | `docker://image:tag`  | Runs the image as a step               |

- A `./` action before checkout fails: runner can't find `action.yml`, `action.yaml` or `Dockerfile`. In a reusable workflow, checkout fetches the **caller's** repository, so `./` points at the wrong repo; use `owner/shared-repo/path@ref`.
- **JavaScript actions:** `runs.using: node24`, `runs.main`, optional `pre`/`post`. Per the lesson, Node 20 was removed from runners in September 2026, so use `node24`. `javascript` is not a valid `using`; `entrypoint`/`image` are Docker keys.
  - The runner **never runs `npm install`**. Commit `node_modules`, or bundle with `@vercel/ncc` (or rollup) and point `main` at the committed bundle (`dist/index.js`).
  - Symptom: `Cannot find module '@actions/core'`. Installing in a `pre` script is fragile and adds network work to every job.
- **Composite actions:**
  - `shell` is **required on every run step** (no defaults reach inside the action).
  - Each output needs `value: ${{ steps.<inner-id>.outputs.<name> }}`; without it the caller reads empty. The caller reads `steps.<its-own-step-id>.outputs.<id>`.
  - Run steps start in the **caller's workspace**. Files shipped with the action: `${{ github.action_path }}` or `$GITHUB_ACTION_PATH` (composite only). `working-directory` does not help.
  - Map inputs through `env:` and quote `"$VAR"` to avoid script injection.
- **Docker actions:** `runs.image` required.

| `runs` key                           | Effect                                                          |
| ------------------------------------ | --------------------------------------------------------------- |
| `image`                              | Dockerfile path (built every job) or `docker://` (pulled)       |
| `entrypoint`                         | Overrides Dockerfile `ENTRYPOINT`                               |
| `args`                               | Overrides `CMD` **as a whole** (not merged); no args → CMD used |
| `env`                                | Variables in the container                                      |
| `pre-entrypoint` / `post-entrypoint` | Run before/after the main entrypoint                            |

- Exec-form `ENTRYPOINT` has no shell, so `$GITHUB_REF` in `args` arrives literally. Fix: `${{ github.ref }}` in `args`, or start through a shell.
- Dockerfile rules: **no `USER`** (must run as root, else permission denied writing the workspace); **no `WORKDIR`** (workspace is mounted over `/github/workspace`, hiding baked files; store tools elsewhere, e.g. `/opt/tool/`); entry script must be **executable in Git** (`git update-index --chmod=+x entrypoint.sh`).
- A prebuilt `docker://ghcr.io/owner/tool:1.4` image removes per-job build time.
- **pre and post:**
  - `pre` runs at job start; `post` at job end, in reverse order of actions. `pre-if`/`post-if` default to `always()`.
  - Status functions in them read the **job's** status. `post-if: success()` skips the post script if any job step failed, even if the action's own main succeeded. Keep `always()` for cleanup.
  - `pre` is **not supported for local `./` actions** (files appear only after checkout).
  - main and post are separate processes. Pass private values with `core.saveState` (writes `GITHUB_STATE`, exposed as `STATE_<name>`) and `core.getState`. `exportVariable` would leak to every later step.
- **Toolkit calls:**

| Call                        | Mechanism                 | Effect                                 |
| --------------------------- | ------------------------- | -------------------------------------- |
| `core.setOutput`            | `GITHUB_OUTPUT`           | Step output for the caller             |
| `core.exportVariable`       | `GITHUB_ENV`              | Env var for later steps                |
| `core.addPath`              | `GITHUB_PATH`             | Prepends to PATH for later steps       |
| `core.saveState`            | `GITHUB_STATE`            | This action's later scripts only       |
| `core.setSecret`            | `::add-mask::`            | Mask in logs                           |
| `core.error/warning/notice` | `::error::` etc.          | Annotation only; step result unchanged |
| `core.startGroup/endGroup`  | `::group::`               | Collapsible log group                  |
| `core.setFailed`            | `::error::` + exit code 1 | Fails the step when the process ends   |

- Setting `process.env.PATH` affects only the action itself; use `addPath`.
- `setFailed` does not stop the code, so cleanup after the catch still runs. `process.exit(1)` stops immediately.
- **Immutable actions:** per GitHub's Dec 2024 / Feb 2025 notices, hosted runners download an immutable version for `owner/action@v4` if one exists, else fall back to the Git repository. Downloaded from `pkg.actions.githubusercontent.com` (covered by `*.actions.githubusercontent.com`). Self-hosted runners whose allow lists blocked the host were opted out and kept using Git. Existing tags were not converted, so a full SHA is still the only general guarantee. The work has since folded into immutable releases.
- **Troubleshooting map:**

| Symptom                                   | Fix                                                              |
| ----------------------------------------- | ---------------------------------------------------------------- |
| Can't find `action.yml` for a `./` action | Check out first; use `owner/repo/path@ref` in reusable workflows |
| `Cannot find module '@actions/core'`      | Commit `node_modules` or an ncc bundle                           |
| Container step fails on Windows/macOS     | JS action, or a separate Linux job + artifact                    |
| Composite "missing required property"     | Add `shell` to the run step                                      |
| Composite output always empty             | Add `value:` mapping                                             |
| Composite can't find its own script       | `$GITHUB_ACTION_PATH`                                            |
| Post script skipped after a failed step   | Keep `post-if: always()`                                         |
| `$INPUT_X` empty in a composite           | Use `${{ inputs.x }}` via env                                    |
| `exec /entrypoint.sh: permission denied`  | Set the executable bit in Git                                    |
| Permission denied writing the workspace   | Remove `USER`                                                    |
| Literal `$VAR` received                   | `${{ }}` in `args`                                               |
| Error annotation but step passes          | `core.setFailed`                                                 |

- For more detail: `ACTIONS_STEP_DEBUG`, `ACTIONS_RUNNER_DEBUG`, or re-run with debug logging; "Set up job" shows which action versions were downloaded.

📖 Full lesson: [Create Custom GitHub Actions: JavaScript, Docker and Composite Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-custom-actions-javascript-docker-composite/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

## 3.2 Distribute and maintain actions

- **Distribution models:**

|                | Public repository              | Private / internal repository                          | GitHub Marketplace                                |
| -------------- | ------------------------------ | ------------------------------------------------------ | ------------------------------------------------- |
| Who can run it | Any workflow, `owner/repo@ref` | Only callers the Access option admits                  | Anyone (it is a public repo)                      |
| Who sees code  | Everyone                       | People with repo access (internal: enterprise members) | Everyone                                          |
| How to enable  | Make the repo public           | Settings > Actions > General > Access                  | Publish a release with the Marketplace box ticked |

- Marketplace is a **listing** for discovery, not a hosting location. Copying files or forking freezes the code, so consumers never get fixes.
- **References:** root `owner/repo@ref`; subfolder `owner/repo/path@ref` (path is the folder, ref last); `./path` needs a checkout and cannot reach another repo; `$/path` uses the running commit without a checkout (GitHub.com only, not GHES). `my-org/app/...@main` runs whatever main holds at job start. `.github/actions/setup` with no prefix is invalid.
- **Private/internal sharing (Access setting):**

| Action repository          | Options                                    | Callers admitted          |
| -------------------------- | ------------------------------------------ | ------------------------- |
| Private, user-owned        | Not accessible / same user                 | That user's private repos |
| Private, org-owned         | Not accessible / organization / enterprise | **Private** repos only    |
| Internal (enterprise orgs) | Not accessible / organization / enterprise | Private or internal repos |

- Only the Access setting on the **action's** repository opens it. Not people's roles, not caller tokens, not allowed-actions policies.
- A **public repo can use neither** private nor internal actions, even in the same org with "allow all actions".
- To reach several orgs: internal repo + "Accessible from repositories in the enterprise". A user-owned private repo can never admit an org; transfer it first.
- Outside collaborators on a consuming repo can see the action's behaviour through run logs.
- **Marketplace requirements:**
  - Public repository with a **single metadata file at the root**; one listing per repository (subfolder actions stay usable but unlisted).
  - Listing name comes from `name` in the metadata, and must not match an existing Marketplace action, a GitHub user or org (unless that account publishes), a Marketplace category, or a reserved GitHub feature name. Containing a word is fine; fix clashes by changing `name`.
- **Publishing:** open the metadata file → Draft a release → tick **Publish this Action to the GitHub Marketplace** → fix warnings → choose primary (optional secondary) category → tag (e.g. `v1.0.0`) and title → publish.
  - The owning account must accept the **Marketplace Developer Agreement** (for an org, only an **owner** can); until then the box is disabled. Publishing needs **2FA**.
  - No pre-listing review. Pushing a tag alone lists nothing. The verified creator badge is optional and for partner organizations.
- **Branding:** `branding.icon` (Feather icon name) and `branding.color` (fixed set). Cosmetic only.
- **Delisting:** clear the Marketplace box on **every** published release; repo, tags and releases stay so consumers keep working. Deleting tags or making the repo private breaks them.
- **Transfer:** releases, tags and listing move; old `uses:` references redirect. An unverified new owner loses the verified badge.
- **Release strategy:**
  - Semantic release tags (`v1.4.2`) plus a moving **major tag** (`v1`) pointing at the latest compatible release.
  - Compatible change → `v1.4.3`/`v1.5.0`, then force-move `v1`. Breaking change → `v2.0.0` and a new `v2`; leave `v1` alone. Never delete a referenced major tag.
  - Order: validate on a release branch (`release/v1`), create the release, move the major tag **last**.
  - Automating the move: `permissions: contents: write` (tags are contents; `actions` scope is for runs) and `git push --force origin v1`. Typical trigger: `release` type `published`.
  - Parallel majors: fix each line on its branch, release each (`v1.8.3`, `v2.1.1`), move each major tag only within its line.
- **Consumer refs:**

| Reference              | Gets updates?            | Code can change?                                              |
| ---------------------- | ------------------------ | ------------------------------------------------------------- |
| `@v2`                  | Yes, compatible releases | Yes, by design                                                |
| `@v2.3.1`              | No                       | Only if the tag is force-moved (not for an immutable release) |
| Full 40-char SHA       | No                       | No                                                            |
| `@main`, `@release/v2` | Every push               | Yes, including unreleased work                                |

- Short SHAs are not accepted as pins.
- **Immutable releases:** a repo/org setting; each newly published release locks its tag to its commit and its assets. Title, notes and pre-release/latest flags stay editable. A deleted release's tag name can never be reused. Applies to **future** releases only.
  - Workflow: create as **draft**, upload all assets, then publish. Uploading after publishing fails whatever the token.
  - Keep moving tags (`v1`, `v1.1`) as **plain tags with no release attached**; once a release locks a major tag it cannot be moved again.
- **Packaging:** commit the bundled JS (`dist/index.js`, not git-ignored) in the tagged commit; changing `runs.using` never installs packages. For Docker, publish a versioned image per release and use `docker://`. Read `GITHUB_API_URL` / `GITHUB_GRAPHQL_URL` instead of hard-coding `api.github.com` so the action works on GHE.com and GHES; `GITHUB_SERVER_URL` is the web address.

📖 Full lesson: [Share, Publish and Version GitHub Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-publish-marketplace-versioning/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

[← Back to the study guide](../README.md)
