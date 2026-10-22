# GH-200 Commonly Confused Services and Concepts

Side-by-side comparisons of the GH-200 services and ideas that exam questions most often set against each other. Each table ends with the one thing to remember.

- [Domain 1: Author and manage workflows](#domain-1-author-and-manage-workflows)
- [Domain 2: Consume and troubleshoot workflows](#domain-2-consume-and-troubleshoot-workflows)
- [Domain 3: Author and maintain actions](#domain-3-author-and-maintain-actions)
- [Domain 4: Manage GitHub Actions for the enterprise](#domain-4-manage-github-actions-for-the-enterprise)
- [Domain 5: Secure and optimize automation](#domain-5-secure-and-optimize-automation)

## Domain 1: Author and manage workflows

### workflow_dispatch vs repository_dispatch

|                | workflow_dispatch                           | repository_dispatch                                          |
| -------------- | ------------------------------------------- | ------------------------------------------------------------ |
| Targets        | One specific workflow file, on a chosen ref | The repository; every workflow whose `types` match runs      |
| Data           | Declared, typed `inputs`                    | Free-form `client_payload` JSON plus an `event_type`         |
| Read data from | `inputs.<name>`                             | `github.event.client_payload.<field>`, `github.event.action` |
| Typical caller | People, scripts, `gh workflow run`          | External systems via `POST /repos/{owner}/{repo}/dispatches` |
| Bad input      | API returns 422, no run                     | 204 even if no workflow matches                              |

**Remember:** dispatch a file with typed inputs, or broadcast an event with a payload. Both need the workflow file on the default branch; a `workflow_dispatch` can then run any ref, using that ref's copy of the file.

### pull_request vs pull_request_target vs workflow_run

|                 | pull_request                  | pull_request_target                        | workflow_run                                 |
| --------------- | ----------------------------- | ------------------------------------------ | -------------------------------------------- |
| Code it runs    | The PR merge commit           | Base repository's default branch           | Base repository's default branch             |
| Fork PR token   | Read-only                     | Can write                                  | Can write                                    |
| Fork PR secrets | Not passed                    | Available                                  | Available                                    |
| Safe use        | Build and test untrusted code | Label, comment, triage (never run PR code) | Privileged follow-up that consumes artifacts |

**Remember:** test fork code under pull_request; do privileged work in workflow_run, and never check out and run the fork head in a privileged trigger.

### GITHUB_ENV vs GITHUB_OUTPUT vs job outputs vs artifacts

|                | GITHUB_ENV                  | GITHUB_OUTPUT                          | Job outputs                      | Artifacts                      |
| -------------- | --------------------------- | -------------------------------------- | -------------------------------- | ------------------------------ |
| Carries        | Environment variables       | Named strings from a step with an `id` | Strings mapped from step outputs | Files                          |
| Reach          | Later steps of the same job | Later steps via `steps.<id>.outputs`   | Jobs that `needs` it             | Other jobs, other runs, people |
| Available from | The next step               | The next step                          | Downstream jobs                  | After upload                   |

**Remember:** env for later steps, outputs for named values, needs to cross jobs, artifacts for files.

### Cache vs artifact

|                 | Cache (`actions/cache`)                            | Artifact (`upload-artifact`)                     |
| --------------- | -------------------------------------------------- | ------------------------------------------------ |
| Purpose         | Reuse files that rarely change, to run faster      | Keep files a run produced                        |
| Typical content | Dependencies                                       | Binaries, test reports                           |
| Lifetime        | Evicted after 7 days unused or over the size limit | Retention period (`retention-days` can shorten)  |
| Can it change?  | No, entries are immutable; change the key          | Names unique per run; `overwrite: true` replaces |

**Remember:** caches speed up inputs, artifacts keep outputs.

### Wait timer vs required reviewers vs timeout-minutes vs concurrency

|                 | Wait timer                  | Required reviewers         | timeout-minutes                  | concurrency                         |
| --------------- | --------------------------- | -------------------------- | -------------------------------- | ----------------------------------- |
| Where set       | Environment                 | Environment                | Job or step                      | Workflow or job                     |
| Effect          | Fixed delay, then continues | Waits for a human approval | Cancels a job that runs too long | One running (one pending) per group |
| Needs a person? | No                          | Yes                        | No                               | No                                  |

**Remember:** only required reviewers involve a person; only the wait timer is a planned pause.

### success() vs failure() vs always() vs !cancelled()

|             | success()                                | failure()                                | always()                   | !cancelled()                      |
| ----------- | ---------------------------------------- | ---------------------------------------- | -------------------------- | --------------------------------- |
| True when   | Nothing upstream failed or was cancelled | Any previous step or ancestor job failed | Every time, even on cancel | Anything except cancellation      |
| Default?    | Implied when `if` has no status function | No                                       | No                         | No                                |
| Typical use | Normal flow                              | Notify or clean up after failure         | Must-run work              | Upload reports after pass or fail |

**Remember:** to react to one job, pair a status function with `needs.<job>.result`.

### fail-fast vs continue-on-error vs max-parallel

|           | fail-fast                                   | continue-on-error                                              | max-parallel              |
| --------- | ------------------------------------------- | -------------------------------------------------------------- | ------------------------- |
| Set on    | `strategy`                                  | The job (often from a matrix value)                            | `strategy`                |
| Effect    | One failure cancels the matrix's other jobs | That job may fail without cancelling others or failing the run | Caps jobs running at once |
| Job count | Unchanged                                   | Unchanged                                                      | Unchanged                 |

**Remember:** none of these reduce how many matrix jobs exist; only `exclude` does.

### outcome vs conclusion (step results)

|                         | outcome                               | conclusion |
| ----------------------- | ------------------------------------- | ---------- |
| Measured                | Before `continue-on-error` is applied | After it   |
| Tolerated failure shows | `failure`                             | `success`  |

**Remember:** to react to a failure you chose to tolerate, test `outcome`.

## Domain 2: Consume and troubleshoot workflows

### Workflow template vs reusable workflow vs composite action

|                           | Workflow template                   | Reusable workflow                | Composite action                       |
| ------------------------- | ----------------------------------- | -------------------------------- | -------------------------------------- |
| Shares                    | A whole workflow file as a scaffold | Complete jobs                    | A sequence of steps                    |
| Used with                 | New workflow page (copied once)     | `jobs.<id>.uses`                 | `steps[*].uses`                        |
| Later updates reach users | No                                  | Yes, at the ref the caller names | Yes, at the ref the step names         |
| Runner                    | Whatever the copy says              | Its own jobs choose              | The calling job's runner and workspace |
| Secrets                   | Normal workflow secrets             | `secrets:` or `secrets: inherit` | Passed in as inputs                    |
| Marketplace               | No                                  | No                               | Yes                                    |

**Remember:** a template is a copy, a reusable workflow is a call to jobs, a composite action is a call to steps.

### Skipped workflow vs skipped job

|                | Workflow skipped by filter or `[skip ci]` | Job skipped by `if:`                          |
| -------------- | ----------------------------------------- | --------------------------------------------- |
| Run created?   | No                                        | Yes                                           |
| Required check | Pending, blocks the merge                 | Success                                       |
| Use it to      | Avoid running anything                    | Let a required check pass while skipping work |

**Remember:** to make a required check skippable, skip the job, not the workflow.

### ACTIONS_STEP_DEBUG vs ACTIONS_RUNNER_DEBUG vs RUNNER_DEBUG

|             | ACTIONS_STEP_DEBUG                  | ACTIONS_RUNNER_DEBUG                      | RUNNER_DEBUG                  |
| ----------- | ----------------------------------- | ----------------------------------------- | ----------------------------- |
| Who sets it | You (secret or variable)            | You (secret or variable)                  | The runner, when debug is on  |
| Adds        | Debug lines in step logs            | Runner diagnostic logs in the log archive | Nothing; scripts can read it  |
| Use         | An action or step needs more detail | The runner or job setup misbehaves        | Detect debug mode in a script |

**Remember:** "Re-run with debug logging" turns on both for one re-run only.

### Re-run all jobs vs re-run failed jobs vs re-run one job

|            | Re-run all jobs            | Re-run failed jobs                     | Re-run one job                 |
| ---------- | -------------------------- | -------------------------------------- | ------------------------------ |
| Runs again | Every job                  | Every failed job plus dependents       | That job plus dependents       |
| CLI        | `gh run rerun <run-id>`    | `gh run rerun <run-id> --failed`       | `gh run rerun --job <job-id>`  |
| REST       | `POST .../runs/{id}/rerun` | `POST .../runs/{id}/rerun-failed-jobs` | `POST .../jobs/{job_id}/rerun` |

**Remember:** every re-run uses the same commit and ref and the original actor; a fix needs a new run.

### Disable a workflow vs delete the workflow file

|               | Disable      | Delete the file                                 |
| ------------- | ------------ | ----------------------------------------------- |
| Scope         | Every branch | Only the branch where the deletion is committed |
| Commit needed | No           | Yes                                             |
| Undo          | Enable again | Restore the file in a new commit                |
| Run history   | Kept         | Kept                                            |

**Remember:** disabling is the only way to stop a workflow everywhere without changing any branch.

### github.ref vs github.ref_name vs github.head_ref vs github.base_ref (pull request run)

|                 | github.ref            | github.ref_name | github.head_ref | github.base_ref |
| --------------- | --------------------- | --------------- | --------------- | --------------- |
| Value on a PR   | `refs/pull/<n>/merge` | `<n>/merge`     | Source branch   | Target branch   |
| Value on a push | `refs/heads/main`     | `main`          | Empty           | Empty           |

**Remember:** a PR run builds the merge of head into the current base, not the branch tip.

### download-artifact in the same run vs from another run

|             | Same run                      | Another run                                              |
| ----------- | ----------------------------- | -------------------------------------------------------- |
| Inputs      | `name` (and `path`)           | `name`, `run-id` **and** `github-token`                  |
| Ordering    | Job must `needs` the uploader | Triggering run's id, e.g. `github.event.workflow_run.id` |
| Permissions | Default                       | Token with `actions: read`                               |

**Remember:** without `github-token`, `run-id` is ignored and the artifact is "not found".

## Domain 3: Author and maintain actions

### JavaScript vs Docker container vs composite actions

|                      | JavaScript                       | Docker container                          | Composite                                   |
| -------------------- | -------------------------------- | ----------------------------------------- | ------------------------------------------- |
| `runs.using`         | `node24`                         | `docker`                                  | `composite`                                 |
| Runner OS            | Linux, Windows, macOS            | Linux only                                | Whatever the calling job uses               |
| Start-up             | Fastest                          | Build or pull the image every job         | Depends on its steps                        |
| Brings its own tools | Only bundled packages            | Yes, the whole image                      | No; installs through its steps              |
| Inputs arrive as     | `INPUT_<NAME>` / `core.getInput` | `INPUT_<NAME>` env vars                   | `${{ inputs.<id> }}`                        |
| Exam cue             | "every OS", "fast", "toolkit"    | "fixed tool versions", "native libraries" | "package these steps", "call other actions" |

**Remember:** need Windows or macOS? Not Docker. Need independence from the runner image? Docker.

### Docker args vs entrypoint vs CMD

|                    | `runs.args`       | `runs.entrypoint` | Dockerfile `CMD`            |
| ------------------ | ----------------- | ----------------- | --------------------------- |
| Replaces           | `CMD`, as a whole | `ENTRYPOINT`      | Nothing                     |
| When used          | Always when set   | Always when set   | Only when `args` is not set |
| Changes the image? | No                | No                | Not applicable              |

**Remember:** args never merge with CMD; they replace it. With an exec-form `ENTRYPOINT`, `$VAR` in `args` arrives literally, so use `${{ }}` (resolved by the runner) or start through a shell.

### pre/post vs main (JavaScript action scripts)

|                   | pre                           | main                 | post                                 |
| ----------------- | ----------------------------- | -------------------- | ------------------------------------ |
| When              | Start of the job              | At the action's step | End of the job, reverse action order |
| Condition key     | `pre-if` (default `always()`) | The step's `if`      | `post-if` (default `always()`)       |
| Local `./` action | Not supported                 | Supported            | Supported                            |

**Remember:** status functions in `pre-if`/`post-if` read the whole job's status, and state passes between scripts only through `saveState`/`getState`.

### core.error vs core.setFailed vs process.exit(1)

|                     | core.error | core.setFailed             | process.exit(1)  |
| ------------------- | ---------- | -------------------------- | ---------------- |
| Annotation          | Yes        | Yes                        | No               |
| Step fails?         | No         | Yes, when the process ends | Yes, immediately |
| Code after it runs? | Yes        | Yes                        | No               |

**Remember:** setFailed is the clean way to fail a step and still run cleanup.

### Public vs private/internal vs Marketplace distribution

|                 | Public repository     | Private / internal repository             | Marketplace listing                     |
| --------------- | --------------------- | ----------------------------------------- | --------------------------------------- |
| Who can run it  | Any workflow          | Callers the Access option admits          | Anyone                                  |
| Code visible to | Everyone              | Repository members (internal: enterprise) | Everyone                                |
| Switch          | Repository visibility | Settings > Actions > General > Access     | Release with the Marketplace box ticked |

**Remember:** Marketplace adds discovery, not access; any public action is already usable.

### Private action vs internal action (Access set to share)

|                    | Private repository                      | Internal repository                             |
| ------------------ | --------------------------------------- | ----------------------------------------------- |
| Admits             | Private callers only                    | Private or internal callers                     |
| Public callers     | Never                                   | Never                                           |
| Reach several orgs | Enterprise option, private callers only | Enterprise option, private and internal callers |

**Remember:** public repositories can use neither; make the action public if they need it.

### Major tag vs exact tag vs full SHA vs branch (consumer refs)

|            | `@v2`                      | `@v2.3.1`                                      | Full 40-char SHA              | `@main`                      |
| ---------- | -------------------------- | ---------------------------------------------- | ----------------------------- | ---------------------------- |
| Gets fixes | Yes                        | No                                             | No                            | Every push                   |
| Can change | Yes, by design             | Only if force-moved (not if immutable release) | No                            | Yes, including breaking work |
| Use when   | Want patches automatically | Want a fixed version                           | Need identical code every run | Rarely                       |

**Remember:** a short SHA is not a valid pin.

### Release tag vs moving major tag (with immutable releases)

|                      | Release tag (`v1.4.2`)                   | Major tag (`v1`)                             |
| -------------------- | ---------------------------------------- | -------------------------------------------- |
| Has a GitHub release | Yes                                      | No, keep it a plain tag                      |
| Movable              | No, locked once the release is published | Yes, force-pushed to each compatible release |
| Assets               | Upload while draft, then publish         | Not applicable                               |

**Remember:** attach releases only to specific versions, or the major tag becomes impossible to move.

## Domain 4: Manage GitHub Actions for the enterprise

### Enablement vs allowed-actions policy vs token permissions

|          | Enablement                        | Allowed-actions policy                          | Token permissions                      |
| -------- | --------------------------------- | ----------------------------------------------- | -------------------------------------- |
| Decides  | Whether workflows run at all      | Which actions and reusable workflows they use   | What `GITHUB_TOKEN` may change         |
| Levels   | Enterprise, org, repository       | Enterprise, org, repository                     | Default setting plus `permissions` key |
| Exam cue | "teams must not run any workflow" | "only GitHub-owned actions", "block one action" | "release job cannot push a tag"        |

**Remember:** each level only narrows the one above; loosening is always a change at the parent.

### Allow owner only vs select non-owner vs allow all

|                         | Owner only   | Owner and select non-owner               | Allow all |
| ----------------------- | ------------ | ---------------------------------------- | --------- |
| GitHub's own actions    | Blocked      | Allowed if "created by GitHub" is ticked | Allowed   |
| Patterns and `!` blocks | No           | Yes                                      | No        |
| REST value              | `local_only` | `selected`                               | `all`     |

**Remember:** to block one action and keep the rest, use select with `*, !owner/repo@*`.

### Workflow template vs Access sharing vs ruleset-required workflow

|                         | Workflow template | Access sharing (reusable workflow/action) | Ruleset "Require workflows to pass" |
| ----------------------- | ----------------- | ----------------------------------------- | ----------------------------------- |
| File in each repo       | Yes, a copy       | Yes, a caller                             | No                                  |
| Enforced                | No                | No                                        | Yes, blocks merging                 |
| Honours trigger filters | Yes               | Yes                                       | No, runs on every targeted PR       |

**Remember:** only a ruleset can require a workflow without touching the repositories.

### Organization vs repository vs environment secrets

|                 | Organization                      | Repository                 | Environment                                   |
| --------------- | --------------------------------- | -------------------------- | --------------------------------------------- |
| Who gets it     | Repos chosen by its access policy | Every workflow in the repo | Jobs naming the environment, after rules pass |
| Precedence      | Lowest                            | Middle                     | Highest                                       |
| Gates the value | No                                | No                         | Yes (reviewers, branch policy)                |
| Count limit     | 1,000                             | 100                        | 100                                           |

**Remember:** the lowest scope wins per name, and only environment secrets are protected by approval.

### Secret vs configuration variable

|                | Secret                                 | Variable                            |
| -------------- | -------------------------------------- | ----------------------------------- |
| Stored         | Encrypted                              | Plain text                          |
| Logs           | Redacted                               | Not redacted                        |
| Context        | `secrets.X`                            | `vars.X`                            |
| Usable in `if` | No (map to env first)                  | Yes                                 |
| REST write     | `PUT` with libsodium `encrypted_value` | `POST` to create, `PATCH` to update |
| Read back      | Never                                  | Yes                                 |

**Remember:** anything that grants access is a secret.

### Runner labels vs runner groups

|                | Labels                           | Runner groups                                           |
| -------------- | -------------------------------- | ------------------------------------------------------- |
| Purpose        | Pick a suitable runner           | Decide which repositories and workflows may use runners |
| Access control | None                             | Yes                                                     |
| Per runner     | Many                             | Exactly one                                             |
| In `runs-on`   | Array of labels (all must match) | `group:` key                                            |

**Remember:** to reserve runners, use a group; labels restrict nothing.

### Standard hosted vs larger vs self-hosted runners

|                  | Standard hosted | Larger runners       | Self-hosted              |
| ---------------- | --------------- | -------------------- | ------------------------ |
| Maintenance      | GitHub          | GitHub               | You                      |
| Static IPs       | No              | Yes (Linux, Windows) | Your network's egress IP |
| Runner groups    | No              | Yes                  | Yes                      |
| Public repo cost | Free            | Billed               | Your infrastructure      |

**Remember:** for an IP allow list with hosted runners, use larger runners with static IP ranges.

### Ephemeral runner vs job-completed hook

|                        | Ephemeral runner                     | ACTIONS_RUNNER_HOOK_JOB_COMPLETED    |
| ---------------------- | ------------------------------------ | ------------------------------------ |
| What it is             | A runner de-registered after one job | A script run at the end of every job |
| Can delete the machine | Yes, via your controller             | No, it runs before the job completes |
| Use                    | Autoscaling, clean machine per job   | Per-job cleanup or reporting         |

**Remember:** autoscale with ephemeral runners and the `workflow_job` queued/completed events.

### Usage metrics vs performance metrics

|         | Usage metrics        | Performance metrics                 |
| ------- | -------------------- | ----------------------------------- |
| Reports | Minutes consumed     | Run time, queue time, failure rates |
| Answers | "What does it cost?" | "Why do jobs wait or fail?"         |

**Remember:** long queue times are a performance-metrics question.

## Domain 5: Secure and optimize automation

### GITHUB_TOKEN vs PAT vs GitHub App token

|                               | GITHUB_TOKEN             | Personal access token                                       | GitHub App installation token       |
| ----------------------------- | ------------------------ | ----------------------------------------------------------- | ----------------------------------- |
| Owner                         | The Actions app          | A user                                                      | The app installation (organization) |
| Repositories                  | The workflow's repo only | Selected (fine-grained) or all the user can reach (classic) | Where installed, narrowed per token |
| Lifetime                      | Ends with the job        | Long-lived                                                  | One hour                            |
| Survives a leaver             | Yes                      | No                                                          | Yes                                 |
| Narrowed by `permissions` key | Yes                      | No                                                          | No (use `permission-*` inputs)      |

**Remember:** cross-repository automation that must outlive any employee uses a GitHub App token.

### OIDC federation vs stored cloud secret

|                          | OIDC federation                                         | Stored cloud key (any secret scope) |
| ------------------------ | ------------------------------------------------------- | ----------------------------------- |
| Long-lived key in GitHub | No                                                      | Yes                                 |
| Credential lifetime      | Short, expires on its own                               | Until rotated                       |
| Needs                    | `id-token: write` and a cloud trust condition on claims | A secret                            |
| Can tie to approval      | Yes, trust `...:environment:production`                 | Only if it is an environment secret |

**Remember:** moving a key to a "safer" secret scope still stores a long-lived key; only federation removes it.

### Required reviewers vs prevent self-review vs custom protection rule vs wait timer

|                     | Required reviewers      | Prevent self-review                      | Custom protection rule           | Wait timer                 |
| ------------------- | ----------------------- | ---------------------------------------- | -------------------------------- | -------------------------- |
| Decision by         | A listed person or team | Any listed reviewer except the triggerer | A GitHub App                     | The clock                  |
| Stops self-approval | No                      | Yes                                      | Not applicable                   | Not applicable             |
| Exam cue            | "a human must approve"  | "a second person must approve"           | "external change system decides" | "soak period, no approval" |

**Remember:** admins can bypass all of these unless "Allow administrators to bypass" is cleared.

### Safe vs unsafe handling of untrusted input

|                            | `env:` then `"$VAR"` | `"${{ env.VAR }}"` | `::add-mask::`         | `toJSON()` / quoting the expression |
| -------------------------- | -------------------- | ------------------ | ---------------------- | ----------------------------------- |
| Value reaches the shell as | Data                 | Script text        | Script text            | Script text                         |
| Stops injection            | Yes                  | No                 | No (only hides output) | No                                  |

**Remember:** any `${{ }}` inside `run:` is pasted in before the shell starts.

### Workflow-level vs job-level permissions

|                          | Workflow-level block            | Job-level block                      |
| ------------------------ | ------------------------------- | ------------------------------------ |
| Applies to               | Every job without its own block | That job only                        |
| Combined with the other? | No                              | Replaces the workflow block entirely |
| Unlisted scopes          | `none`                          | `none`                               |

**Remember:** a job that lists only `pull-requests: write` has no `contents` access.

### SHA pin vs tag vs immutable release tag

|                                    | Full commit SHA         | Plain tag (`@v4`)             | Immutable release tag       |
| ---------------------------------- | ----------------------- | ----------------------------- | --------------------------- |
| Can be moved                       | No                      | Yes                           | No                          |
| Freezes nested `uses` / image tags | No                      | No                            | No                          |
| Gets fixes by                      | Reviewed Dependabot PRs | The maintainer moving the tag | Changing to a newer release |

**Remember:** take the SHA from the action's own repository, not a fork, and check its nested references.

### gh attestation verify --owner vs --repo vs --signer-workflow

|               | `--owner`                      | `--repo`                     | `--signer-workflow`                    |
| ------------- | ------------------------------ | ---------------------------- | -------------------------------------- |
| Requires      | Built in any repo of the owner | Built in that one repository | Signed by that one (reusable) workflow |
| Required flag | One of `--owner` / `--repo`    | One of `--owner` / `--repo`  | Optional, added to a source flag       |

**Remember:** attestations prove where and how an artifact was built, not that it is safe.

### SLSA Build Level 2 vs Level 3

|               | Build Level 2                               | Build Level 3                                  |
| ------------- | ------------------------------------------- | ---------------------------------------------- |
| How on GitHub | `actions/attest` in the repo's own workflow | Build and attest in a shared reusable workflow |
| Adds          | Signed provenance from a hosted build       | Build isolated from the caller's workflow code |

**Remember:** a composite action or self-hosted runner does not get you to Level 3.

### Cache key choices

|         | `hashFiles('package-lock.json')` | `github.sha` / `github.run_id`       | `hashFiles('**/node_modules/**')` |
| ------- | -------------------------------- | ------------------------------------ | --------------------------------- |
| Hits    | Until the lockfile changes       | Never                                | Always the same constant key      |
| Problem | None                             | New entry every run, fills the limit | Never refreshes                   |

**Remember:** key the cache on the file that defines the dependencies.

[← Back to the study guide](README.md)
