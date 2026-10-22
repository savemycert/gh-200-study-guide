# Domain 2: Consume and troubleshoot workflows (19%)

This domain is about reading what a workflow did and why: predicting runs from triggers, diagnosing failures from logs and run history, expanding anchors and matrices, re-running the right jobs, fetching logs and artifacts, and consuming shared automation (templates, reusable workflows, composite actions). Expect a YAML snippet or a run-page description and a question about what happened or what to do next, often under a constraint such as "no new commit" or "retry only one variant".

## 2.1 Interpret workflow results, artifacts and logs

- **Reading a trigger:** a run starts only when the event occurs **and** its activity type **and** every filter match. If any one fails, no run is created.
  - Branch filter + path filter: both must pass.
  - `paths` passes when at least one changed file matches.
  - Branches-only push filters ignore tag pushes (tags use `tags`/`tags-ignore`).
  - `pull_request` defaults: `opened`, `synchronize`, `reopened`. `closed`, `labeled` and others must be listed. `branches` matches the base branch.
  - **Merging a PR is a push** to the base branch, so `push: branches: [main]` runs for the merge (if any path filter still passes on the merge commit).
  - The run list in the Actions tab names each run's event; inside a run, `github.event_name` shows it.
- **Skipped workflow vs skipped job:**

| What is skipped                              | Run exists? | Required check shows   |
| -------------------------------------------- | ----------- | ---------------------- |
| Whole workflow, by branches/paths filter     | No          | Pending (blocks merge) |
| Whole workflow, by a commit skip instruction | No          | Pending (blocks merge) |
| One job, by an `if:` that is false           | Yes         | Success                |

- **Skip instructions:** `[skip ci]`, `[ci skip]`, `[no ci]`, `[skip actions]`, `[actions skip]`, or a `skip-checks: true` trailer. Apply only to `push` and `pull_request`; for a PR the head commit's message counts. `pull_request_target`, `workflow_dispatch` and other events still run.
- **Path-scoped required checks** wait forever on PRs that touch no matching file; `paths-ignore` has the same problem and branch protection cannot require a check per path. Fix: always run the workflow, skip the expensive job via an `if:` on a detect job's output.
- **What a pull_request run builds:**

| Event                 | `GITHUB_REF`          | `GITHUB_SHA`                            |
| --------------------- | --------------------- | --------------------------------------- |
| `push`                | Branch or tag pushed  | The pushed commit                       |
| `pull_request`        | `refs/pull/<n>/merge` | Merge of head into the **current** base |
| `pull_request_target` | The PR's base branch  | Last commit on the base branch          |

- Classic symptom: the branch passes locally but its PR run fails in a file the PR never touched, because base moved. Reproduce by merging current base into the branch locally.
- **Reading a failed run:** failed steps expand automatically; annotations at the top of the summary point at errors. **Search logs only covers expanded steps.** Secret values show as `***`.
- **Shell defaults (Linux/macOS):**

| Step setting  | Command run                                | Early pipeline failure |
| ------------- | ------------------------------------------ | ---------------------- |
| No `shell:`   | `bash -e {0}`                              | Hidden                 |
| `shell: bash` | `bash --noprofile --norc -eo pipefail {0}` | Fails the step         |
| `shell: sh`   | `sh -e {0}`                                | Hidden                 |

- So `pytest | tee report.log` passes with the default shell even if pytest fails. Windows default shell is `pwsh`.
- **Other clues:** a job that hit `timeout-minutes` (default **360** per job) shows **cancelled** with an "exceeded the maximum execution time" message. Cancelled matrix jobs may have been stopped by `fail-fast`. Run history shows whether earlier commits passed.
- **Debug logging:**

|         | `ACTIONS_STEP_DEBUG` = true                          | `ACTIONS_RUNNER_DEBUG` = true                      |
| ------- | ---------------------------------------------------- | -------------------------------------------------- |
| You get | Debug lines in step logs (`core.debug`, `::debug::`) | Runner diagnostic logs (`Runner_*`, `Worker_*`)    |
| Where   | Step logs on the run page                            | `runner-diagnostic-logs` folder in the log archive |
| Use     | An action or step needs more detail                  | The runner or job setup misbehaves                 |

- Set either as a repository/org **secret or variable**; a secret wins over a variable of the same name.
- `RUNNER_DEBUG` is set **by** the runner (to 1) when debug is on; setting it yourself enables nothing.
- One-off: **Re-run jobs → Enable debug logging**, `gh run rerun <id> --debug`, or `enable_debug_logging` in REST. Turns on both, for that re-run only. Cannot add debug to a finished attempt.
- **Re-runs and attempts:**
  - A re-run adds an **attempt** to the same run: `run_id` and `run_number` stay, `run_attempt` increments. The **Latest** dropdown lists attempts.
  - It uses the **same `GITHUB_SHA` and `GITHUB_REF`**, so a later fix (even to the workflow file) is not picked up. Test a fix with a new push or `workflow_dispatch`.
  - Needs write access; allowed up to **30 days** after the initial run; at most **50** re-runs.
  - `github.actor` stays the original triggerer (and their privileges); `github.triggering_actor` is who started this attempt.
  - Each attempt keeps its own logs. After "Re-run failed jobs", the latest attempt only ran those jobs; get attempt 1 for the full original picture: `GET .../runs/{run_id}/attempts/{n}/logs`.
  - Run history via REST: `GET .../actions/workflows/{workflow_id}/runs` (ID or file name, filters `branch`, `event`, `status`). `GET .../actions/runs` covers all workflows with no workflow filter. `GET .../runs/{run_id}/jobs` returns metadata, not log text.
- **Expanding anchors:** an aliased step is an exact copy including `with:`; an aliased job copies the whole mapping including `needs`, `if`, `timeout-minutes`, but keeps its own job key. Aliases copy configuration, not files or results. Merge keys (`<<`) are undocumented in GitHub's workflow docs; expand them the YAML way (anchored mapping, then local keys on top).
- **Expanding a matrix:** cross product → `exclude` (partial match) → `include`.
  - Each include entry joins every combination it can without overwriting an **original** value; values added by an earlier include **can** be overwritten by a later one; an entry that joins nothing becomes a new job.
  - Because include runs after exclude, it can add back a single excluded combination.
  - Default job name: job name plus values in parentheses in declaration order, e.g. `test (ubuntu-latest, 22)`; included values appear too.
- **Analysing variant failures:** find the shared axis (all Windows jobs fail → OS cause; all Node 24 jobs fail → runtime cause). Cancelled jobs have unknown results; one red plus several cancelled = one known failure. Set `fail-fast: false` to let every variant report.
- **Re-run options:**

| Option             | What runs again                            | UI / CLI / REST                                                                  |
| ------------------ | ------------------------------------------ | -------------------------------------------------------------------------------- |
| Re-run all jobs    | Every job                                  | `gh run rerun <run-id>`; `POST .../runs/{run_id}/rerun`                          |
| Re-run failed jobs | All failed jobs **plus their dependents**  | `gh run rerun <run-id> --failed`; `POST .../runs/{run_id}/rerun-failed-jobs`     |
| Re-run one job     | That job (one variant) plus its dependents | Job's re-run icon; `gh run rerun --job <job-id>`; `POST .../jobs/{job_id}/rerun` |

- `--job` and the job endpoint take the **numeric job ID**, not the name. To retry only some failures, list jobs, take their IDs, re-run each.
- **Logs and artifacts via REST** (paths start `/repos/{owner}/{repo}`):

| Need                            | Request                                        | Returns                   |
| ------------------------------- | ---------------------------------------------- | ------------------------- |
| All logs, latest attempt        | `GET /actions/runs/{run_id}/logs`              | 302 to a zip              |
| All logs of one attempt         | `GET /actions/runs/{run_id}/attempts/{n}/logs` | 302 to a zip              |
| One job's log                   | `GET /actions/jobs/{job_id}/logs`              | 302 to plain text         |
| Delete a run's logs             | `DELETE /actions/runs/{run_id}/logs`           | 204, run kept             |
| A run's artifacts               | `GET /actions/runs/{run_id}/artifacts?name=`   | List with IDs             |
| Artifacts across the repository | `GET /actions/artifacts?name=`                 | Same names from every run |
| Download an artifact            | `GET /actions/artifacts/{artifact_id}/zip`     | 302 to the zip            |
| Delete an artifact              | `DELETE /actions/artifacts/{artifact_id}`      | 204, run and logs kept    |

- Redirect URLs expire after **one minute**. No download-by-name endpoint; zip is the only format; to fetch a named artifact from one run, filter that run's list and use the ID.
- **GitHub CLI:** `gh run view <id> --log` (all steps), `--log-failed` (failed steps only). `gh run download <id>` (all artifacts), `-n <name>` (one).
- **download-artifact:**
  - `name` → one artifact extracted straight into `path`. No name → every artifact, each in its own directory. `pattern` + `merge-multiple: true` → matching artifacts into one directory.
  - `download-path` output belongs to download-artifact; upload-artifact returns `artifact-id`, `artifact-url`, `artifact-digest` (link it in the job summary).
  - From another run: `run-id` **plus** `github-token`. Without the token, `run-id` is ignored and you get "artifact not found". With `GITHUB_TOKEN` and a `permissions` block, include `actions: read`.
- **Managing artifacts:** deleting needs write access; deleting one artifact keeps the run; deleting a run deletes logs and artifacts. Retention changes apply to new artifacts. Zips do not keep file modes (dirs 755, files 644), so binaries lose the execute bit; tar them and upload with `archive: false` (upload-artifact v7+).

📖 Full lesson: [Troubleshoot GitHub Actions Workflow Runs: Logs, Matrix Re-runs and Artifacts (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-troubleshoot-workflow-runs-logs-artifacts/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

## 2.2 Use and manage workflow templates

- **Three sharing mechanisms:**

|                             | Workflow template (starter)          | Reusable workflow                        | Composite action                       |
| --------------------------- | ------------------------------------ | ---------------------------------------- | -------------------------------------- |
| Shared unit                 | Whole workflow file, as a scaffold   | One or more complete jobs                | A sequence of steps                    |
| How used                    | Copied into `.github/workflows` once | `jobs.<id>.uses`                         | `steps[*].uses`                        |
| Source updates reach users? | **No**, copy is independent          | Yes, next run of the named ref           | Yes, next run of the named ref         |
| Runner                      | Whatever the copy says               | Each called job sets its own `runs-on`   | The calling job's runner and workspace |
| Secrets                     | Normal workflow secrets              | `secrets:` or `secrets: inherit`         | No secrets context; pass as inputs     |
| Logs                        | Normal                               | Every job and step separately, real time | One step of the calling job            |
| Marketplace                 | No                                   | No                                       | Yes                                    |

- **Choosing:** a starting point teams change freely → template. A central fix must reach every repository without edits, or the work needs its own runner → reusable workflow. Shared logic must run inside an existing job on files earlier steps made → composite action.
- **Using a template:** Actions tab → New workflow → pick → Configure → edit → commit. Public templates come from `actions/starter-workflows`; org templates appear in their own section.
  - The result is an ordinary file you own. Customise the copy, not the template.
  - Later template edits **never** propagate. There are no inputs, no `with:`, no version ref.
- **Organization templates:** in the org's `.github` repository, `workflow-templates/` directory (not `.github/workflows`). Each needs `<name>.yml` and `<name>.properties.json` with the same base name; without the JSON it is not offered.

| Key            | Required? | Purpose                                                             |
| -------------- | --------- | ------------------------------------------------------------------- |
| `name`         | Yes       | Title in the list                                                   |
| `description`  | Yes       | Short description                                                   |
| `iconName`     | No        | SVG in `workflow-templates` or `octicon <name>`                     |
| `categories`   | No        | Group and filter                                                    |
| `filePatterns` | No        | Regexes matched against root files; suggest the template on a match |

- `filePatterns` decides where a template is **suggested**; a trigger's `paths` decides when a created workflow **runs**.
- `$default-branch` is replaced **once**, at creation. The copy holds the literal name and does not follow a later rename. In an ordinary workflow file it is just text; it is not an expression.
- **Non-public templates:**

| `.github` repo is... | Templates offered to                         |
| -------------------- | -------------------------------------------- |
| Public               | All repositories (public, internal, private) |
| Internal             | Internal and private                         |
| Private              | Private only                                 |

- For internal/private `.github`, the person creating the workflow also needs **Read** access to it. The Actions Access setting is a different control (workflows calling code), not template visibility. Templates are per organization.
- **Calling a reusable workflow** (job-level `uses`, never step-level):
  - Other repo: `{owner}/{repo}/.github/workflows/{file}@{ref}`. Full path and ref required.
  - Same repo: `$/.github/workflows/{file}` (recommended; not on GHES) or `./.github/workflows/{file}`, both **without** a ref; the file comes from the caller's commit.
  - No expressions in `uses`.
  - Refs: full SHA = immutable and safest; release tag = readable but movable; major tag or branch = moves by design. A tag beats a branch of the same name.
- **Calling job keywords allowed:** `name`, `uses`, `with`, `secrets` (incl. `inherit`), `strategy`, `needs`, `if`, `concurrency`, `permissions`. Anything else (`runs-on`, `steps`, `env`, `environment`, `timeout-minutes`, `container`, `services`) makes the workflow invalid.
  - Matrix fan-out is allowed: `strategy.matrix` on the calling job, pass `matrix.x` through `with:`.
  - Environments go on the **called** job: declare an input and set `environment: ${{ inputs.target }}`.
- **Access and nesting:**
  - Public repo workflows: callable from anywhere. Private/internal: holder repo admin sets **Settings > Actions > General > Access**. That is the only switch; people's Read access and tokens do not grant it.
  - Private shares only with private; internal with internal and private; a public repo can never call a private workflow.
  - Max **ten levels** (caller + nine nested); no loops; every workflow in the chain must be accessible; at most **50** unique reusable workflows per file, counting nested trees.
  - `GITHUB_TOKEN` permissions can only stay the same or **decrease** down the chain; asking for more fails the run. Grant on the caller.
  - In the called workflow, `github` context describes the **caller's** run (`github.repository` is the caller). Caller env is not propagated either way.
- **Composite action as a consumer:** used as a step; same runner and workspace (no artifact upload needed); inputs via `with:`; **no secrets context inside it**, so pass secrets as inputs; one step in the log.
- **Disable vs delete:**

|                 | Disable                    | Delete the file                                    |
| --------------- | -------------------------- | -------------------------------------------------- |
| File            | Unchanged on every branch  | Removed from that branch by a commit               |
| Which runs stop | All triggers, all branches | Only events evaluated on branches without the file |
| Commit needed   | No                         | Yes                                                |
| Coming back     | Enable again               | Restore in a new commit                            |
| History         | Kept                       | Past runs not removed                              |

- UI: Actions tab → workflow → `...` → Disable workflow. CLI: `gh workflow disable|enable <file>`. REST: `PUT .../actions/workflows/{workflow_id}/disable` and `/enable` (ID or file name).
- Not the same as turning Actions off for the repository (`PUT .../actions/permissions` with `enabled: false`) or cancelling one run.
- API states: `active`, `disabled_manually`, `disabled_inactivity`, `disabled_fork`, `deleted`.
- **Automatic disabling:** scheduled workflows in public repos after **60 days** with no activity; scheduled workflows in forks of public repos are disabled by default. Re-enable explicitly; new activity does not.

📖 Full lesson: [GitHub Actions Workflow Templates, Reusable Workflows and Composite Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-templates-reusable-workflows/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

[← Back to the study guide](../README.md)
