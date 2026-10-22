# Domain 5: Secure and optimize automation (15%)

The smallest domain, but most of its material reappears inside the others. It covers hardening workflows (approval gates, script injection, token scope, OIDC, pinning, allow lists, attestations) and cutting wasted minutes and storage. Questions are built around one deciding constraint, such as "no long-lived cloud keys stored in GitHub" or "a second person must approve", and the wrong options are real features applied in the wrong place.

## 5.1 Secure and optimize automation

- **Environment protections** (apply to every job with `environment: <name>`, before it starts and before it can read environment secrets):

| Protection                        | What it does                                            | Use when                                            |
| --------------------------------- | ------------------------------------------------------- | --------------------------------------------------- |
| Required reviewers                | Job waits for one listed person/team (up to six listed) | A human must sign off                               |
| Prevent self-review               | The run's triggerer cannot approve it                   | A **second** person must approve                    |
| Wait timer                        | Holds the job N minutes, then continues automatically   | Soak period, no approval                            |
| Deployment branches and tags      | Only selected refs may deploy                           | Feature branches must never reach production        |
| Custom deployment protection rule | A GitHub App approves or rejects                        | Decision depends on an external system              |
| Allow administrators to bypass    | On by default                                           | Turn **off** when even admin hotfixes need approval |

- Required reviewers alone allow self-approval. A wait timer is a delay, not `timeout-minutes`. A pre-deploy "check" job is weaker than a custom rule because it lives in the editable workflow file.
- Plans: public repos get all rules on every current plan. Private/internal: environments and branch rules need Pro, Team or Enterprise; required reviewers, wait timers and custom rules need Enterprise.
- Referencing an environment changes the job's OIDC subject (see below).
- **Script injection:** `${{ }}` is evaluated and pasted into script text before the shell runs, so attacker text can close a quote and add commands.

| Untrusted (contributor sets it)                    | Not contributor-shaped                              |
| -------------------------------------------------- | --------------------------------------------------- |
| `github.head_ref` (branch name)                    | `github.event.pull_request.number`, `github.run_id` |
| PR/issue title and body; comment and review bodies | `github.repository`                                 |
| Commit messages, author/committer names and emails | `github.event.head_commit.id` (computed SHA)        |
| Artifacts uploaded by fork runs                    | GitHub usernames                                    |

- Commit metadata stays untrusted after merging to a protected branch. Branch names can contain `"`, `;`, `$`, `|`, and `${IFS}` expands to whitespace, so "no spaces in branch names" is no defence.

| Approach                                                          | Safe?                                      |
| ----------------------------------------------------------------- | ------------------------------------------ |
| `env: TITLE: ${{ ... }}` then `"$TITLE"` in the script            | Yes                                        |
| `"${{ env.TITLE }}"` in `run:`                                    | No (still an expression)                   |
| Single quotes, backticks, `toJSON()` around the expression        | No                                         |
| `::add-mask::` on the value                                       | No (hides output, does not stop execution) |
| JavaScript action reading it with `core.getInput()`               | Yes                                        |
| Composite or Docker `sh -c` with `${{ inputs.x }}` in the command | No (moves the injection)                   |

- `actions/github-script`: pass through `env` and read `process.env.BODY`, never `${{ }}` inside `script`.
- `workflow_run` reading a fork run's artifact: validate against the exact format (e.g. `^[0-9]+$` for a PR number), then pass through `env:` quoted. No expression function makes spliced text safe.
- In public repos, fork `pull_request` runs always get a read-only token and no secrets whatever `permissions` asks. Private/internal: same by default; admins can opt in to sending write tokens/secrets, which hands that power to untrusted code.
- Also: validate formatted inputs, least-privilege permissions, prefer vetted actions over long inline scripts.
- **GITHUB_TOKEN lifecycle:** a GitHub App installation token for the Actions app, created **per job**, scoped to **one repository**, expires when the job ends (or its max lifetime). Attributed to `github-actions[bot]`.
  - Handed to an external service that calls later → **401 Bad credentials**. Pushing to another private repo → 403 or not found; no permission setting changes that.
- **permissions rules:**
  - Listing any scope sets every unlisted scope to `none`.
  - A job-level block **replaces** the workflow-level block (not merged). A job listing only `pull-requests: write` loses `contents: read`, so checkout of a private repo fails.
  - No block → org/repo default (restricted read, or permissive read/write).
  - Tags, branches, commits and releases are all `contents`. `id-token` has only `write` or `none`; write allows requesting an OIDC token and grants no repo access.
  - Reusable workflows can only reduce permissions; grant `id-token: write` on the calling job.
- **Token choices:**

|                            | GITHUB_TOKEN         | Fine-grained PAT        | Classic PAT                   | GitHub App installation token       |
| -------------------------- | -------------------- | ----------------------- | ----------------------------- | ----------------------------------- |
| Owned by                   | Actions app          | A user                  | A user                        | The installation (organization)     |
| Repositories               | Workflow's repo only | Selected                | Everything the user can reach | Where installed, narrowed per token |
| Lifetime                   | Ends with the job    | Until expiry/revocation | Can be set to never expire    | **One hour** after minting          |
| Breaks when someone leaves | No                   | Yes                     | Yes                           | No                                  |

- `actions/create-github-app-token`: `client-id`, `private-key`; with neither `owner` nor `repositories`, only the current repo; `owner` alone = every installed repo of that owner; `permission-<scope>` inputs narrow it (otherwise it inherits all installation permissions). The job's `permissions` block never affects App tokens or PATs.
- **OIDC federation:** exchange a short-lived GitHub-signed token for short-lived cloud credentials, so no long-lived cloud key is stored at all (moving a key to an environment or org secret still stores it).
  - Job needs `permissions: id-token: write` (plus `contents: read` for checkout). Issuer: `https://token.actions.githubusercontent.com`. Claims include `sub`, `aud`, `repository`, `ref`, `environment`, `job_workflow_ref`.
  - Cloud trusts the issuer with conditions on claims (AWS IAM role trust policy, Azure federated credential, Google workload identity pool provider); login actions (`aws-actions/configure-aws-credentials`, `azure/login`, `google-github-actions/auth`) present the token.

| Job context                    | Default `sub`                                              |
| ------------------------------ | ---------------------------------------------------------- |
| References an environment      | `repo:ORG/REPO:environment:NAME`                           |
| `pull_request`, no environment | `repo:ORG/REPO:pull_request`                               |
| Anything else                  | `repo:ORG/REPO:ref:refs/heads/BRANCH` (or `refs/tags/TAG`) |

- Adding `environment:` changes the subject, so a trust condition on `...:ref:refs/heads/main` stops matching.
- Tie cloud access to approval: trust only `...:environment:production` **and** put required reviewers on production. Either alone leaves a gap. Avoid `repo:org/web:*`.
- `aud` is not an identity check (same for every job).
- Trusting one central reusable workflow: customise the subject with `PUT /orgs/{org}/actions/oidc/customization/sub` and `include_claim_keys: ["job_workflow_ref"]`, then match it in the cloud.
- **Trustworthy actions:** GitHub does **not** security-review Marketplace actions. The verified creator badge confirms the publisher, not the code, and does not pin versions; stars and listings confirm nothing. Read the source and `action.yml`, run with least privilege, prefer GitHub-owned or verified creators and enforce it with policy.
- **Pinning:** `@main` moves on every push; `@v4`/`@v4.2.1` can be moved by anyone with write access (or stolen credentials); immutable release/action versions cannot change, but most third-party tags are not immutable; a **full 40-char SHA** cannot change. Short SHAs are not accepted.
  - Take the SHA from the action's **own** repository: forks share an object network, so a fork-only commit can resolve.
  - A pin freezes only that action's files. Nested `uses: other/tool@v2` in a composite `action.yml`, or a Docker image by tag, still resolves at run time. Check nested pins.
  - Workflow dependency lockfiles (public preview since October 2026, per the lesson) record exact commits including transitive actions; SHA pins remain the baseline.
  - Keep pins current with **Dependabot** version updates (`package-ecosystem: "github-actions"`), which opens reviewable PRs. Auto-rewriting SHAs and pushing to main skips review.
- **Allowed-actions policy (REST):**

| UI setting                    | `allowed_actions` |
| ----------------------------- | ----------------- |
| Allow all                     | `all`             |
| Owner (enterprise/org) only   | `local_only`      |
| Owner plus selected non-owner | `selected`        |

- Selected sub-settings: `github_owned_allowed`, `verified_allowed`, `patterns_allowed`. A `!` entry blocks and wins over a matching allow.
- Two calls in order: `PUT /orgs/{org}/actions/permissions` (`allowed_actions: selected`), then `PUT /orgs/{org}/actions/permissions/selected-actions`. `.../permissions/workflow` sets default token permissions, not actions.
- Block one internal action: `octo-org/*, !octo-org/legacy-deploy@*`.
- "Required reviewers for unverified actions" is not one setting: allow only GitHub-owned and verified plus explicit patterns, and require code-owner review on `.github/workflows/`.
- **Artifact attestations:** signed statement binding an artifact digest to repo, commit, workflow, event and runner (SLSA provenance), signed via the job's OIDC identity.
  - Permissions: `id-token: write`, `contents: read`, `attestations: write`; plus `packages: write` for registry images. Action: `actions/attest` (`subject-path`, or `subject-name` + `subject-digest` + `push-to-registry: true` for images). `actions/attest-build-provenance` remains as a wrapper.
  - Availability: public repos on all current plans; private/internal need Enterprise Cloud. Public repos sign via Sigstore public-good; private via GitHub's own Sigstore instance.
  - **SLSA Build L2** = `actions/attest` in the repo's own workflow. **L3** = build and attest inside a shared reusable workflow, isolating the build from caller code. Composite actions and self-hosted runners do not give L3.

| Requirement                   | Command                                                                                      |
| ----------------------------- | -------------------------------------------------------------------------------------------- |
| Built anywhere in the org     | `gh attestation verify <file> --owner octo-org`                                              |
| Built in one repo             | `gh attestation verify oci://IMAGE --repo octo-org/payments`                                 |
| Built by one central workflow | `... --owner octo-org --signer-workflow octo-org/build-platform/.github/workflows/build.yml` |

- `--owner`/`--repo` (one is required) scope the source; `--signer-workflow`/`--signer-repo` pin the signer. Run it before deploying and fail on failure, or enforce with an admission controller.
- Offline: carry two files: the bundle from `gh attestation download` (`--bundle`) and the root from `gh attestation trusted-root` (`--custom-trusted-root`). Source flag still required. Verify on the machine that deploys.
- Proves origin and integrity, **not** absence of vulnerabilities, code review or a human signer.
- **Caching for cost:** key on the dependency file (`hashFiles('package-lock.json')`). `github.sha`/`run_id` keys never hit and flood storage; hashing `node_modules` (absent at key time) gives a constant key. 7-day eviction; repo cache limit **10 GB by default** (can be raised). Free space with `gh cache delete <key>` / `--all` or the caches REST endpoint. Retention settings never touch caches.
- **Artifact retention:** `retention-days` can only **shorten** (a larger value is reduced to the max and the upload still succeeds). Use `retention-days: 1` for hand-off artifacts. Set org/repo retention with the `artifact-and-log-retention` endpoints.
- **Cutting wasted minutes:**

| Waste                                            | Fix                                                       |
| ------------------------------------------------ | --------------------------------------------------------- |
| Superseded runs on the same PR keep going        | `concurrency` by workflow + ref with `cancel-in-progress` |
| Hung job runs to the long default limit          | `timeout-minutes` above a healthy run                     |
| Push and PR both run for every PR push           | `push` only for `main`, plus `pull_request`               |
| CI for docs-only changes                         | `paths` / `paths-ignore`                                  |
| Matrix job that gives the same result everywhere | Run once on Linux                                         |

- `group: ${{ github.workflow }}-${{ github.ref }}` with `cancel-in-progress: ${{ github.event_name == 'pull_request' }}` cancels superseded PR runs but lets main finish. A group on `github.sha` never collects pushes.
- Linux runners cost less per minute than Windows or macOS; `max-parallel` and `fail-fast` change scheduling, not job count.

📖 Full lesson: [Secure and Optimize GitHub Actions: Permissions, OIDC, Pinning, Attestations and Caching (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-security-oidc-permissions-caching/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

[← Back to the study guide](../README.md)
