# Domain 1: Author and manage workflows (24%)

This domain covers writing a workflow from the trigger down: which events start it, how jobs and steps are arranged and gated, which contexts each key can read, and how data, files, caches, summaries and badges come out of a run. Most questions show a short YAML snippet and ask what it does, why it never ran or a value arrived empty, or which change fixes it. Wrong options are usually real features used in the wrong place.

## 1.1 Workflow triggers, events, execution and outputs

- **Trigger families** under `on:`:

| Kind              | Trigger                                                    | Started by                                           |
| ----------------- | ---------------------------------------------------------- | ---------------------------------------------------- |
| Repository events | `push`, `pull_request`, `issues`, `release`, `create`, ... | Activity in the repository                           |
| Scheduled         | `schedule` with cron entries                               | The clock                                            |
| Manual            | `workflow_dispatch`                                        | Run workflow button, REST API, `gh workflow run`     |
| External webhook  | `repository_dispatch`                                      | Any outside system calling the REST API              |
| Chaining          | `workflow_run`                                             | Another workflow requested, in progress or completed |
| Reuse             | `workflow_call`                                            | A caller job using this file as a reusable workflow  |

- **Default-branch rule:** `schedule`, `workflow_dispatch`, `repository_dispatch` and `workflow_run` only consider workflow files on the **default branch**. A file that exists only on a feature branch never fires for them. This explains most "my workflow never ran" items.
- **Schedules:**
  - POSIX cron, evaluated in **UTC** unless a `timezone` key (IANA name such as `Europe/London`) is set beside the cron entry; with a timezone the schedule follows daylight saving.
  - Shortest interval is every 5 minutes. Runs can be delayed under load (top of the hour is busiest), and very delayed runs may be dropped.
  - Public repositories disable scheduled workflows after **60 days** with no repository activity. Symptom: no runs at all. Re-enable in the Actions tab or via the API; new activity alone does not.
- **workflow_dispatch inputs:** each has `description`, `required`, `default`, `type`.
  - Types: `string` (default), `boolean`, `number`, `choice` (needs `options`), `environment` (all the repository's environments; you cannot narrow the list).
  - Validation happens on every dispatch, not only in the form. An API or CLI dispatch that omits a required input with no default, sends a value outside a `choice` list, or names an undeclared input gets **422** and no run.
  - Run workflow only appears for files on the default branch. Once there, a dispatch can target **any ref**, and that run uses the workflow file and code from that ref. To test a new manual workflow, merge a stub, then dispatch the feature branch.
  - Read `inputs.<name>` (typed). `github.event.inputs.<name>` makes every value a string, so `github.event.inputs.verbose == false` is never true.
- **repository_dispatch:** the external system POSTs `/repos/{owner}/{repo}/dispatches` with an `event_type` and optional `client_payload`. GitHub replies **204**, which only means the event was created.
  - `types:` filters on `event_type`; without it every dispatch runs the workflow.
  - Event type is `github.event.action`; payload fields are `github.event.client_payload.<field>`, never `inputs`.
- **Filters:**
  - `pull_request` with no `types` runs for `opened`, `synchronize`, `reopened`. Setting `types` **replaces** the defaults, so list them again when adding `labeled`.
  - For `pull_request`, `branches` matches the **base** branch.
  - A push trigger with only `tags` ignores branch pushes, and only `branches` ignores tags. `push` + `tags: ['v*']` is the trigger for version tags; `release` fires only for GitHub releases.
  - `paths` runs when **at least one** changed file matches. `paths-ignore` skips only when **every** changed file is ignored.
  - Branch (or tag) filter plus path filter: **both** must pass.
- **Choosing events safely:**

| Trigger               | Runs in context of               | Fork PR token | Fork PR secrets | Safe use                                          |
| --------------------- | -------------------------------- | ------------- | --------------- | ------------------------------------------------- |
| `pull_request`        | The PR merge commit              | Read-only     | Not passed      | Build and test untrusted code                     |
| `pull_request_target` | Base repository's default branch | Can write     | Available       | Label, comment, triage without running PR code    |
| `workflow_run`        | Base repository's default branch | Can write     | Available       | Privileged follow-up that only consumes artifacts |

- Dangerous combination: `pull_request_target` + checkout of `github.event.pull_request.head.sha` + build/test. Fork-controlled scripts then run with secrets and a write token. Safe split: test under `pull_request`, upload results, post them from a `workflow_run` workflow that never executes fork code.
- **Events caused by GITHUB_TOKEN:** most start **no** new runs (tag push, commit push outside a PR, release, issue). Exceptions: `workflow_dispatch` and `repository_dispatch`. A PR created or updated with the token creates `pull_request` runs that wait for approval by someone with write access. Fix for follow-on runs: a GitHub App installation token or a PAT. Extra token permissions do not change the rule.
- **Token scope:** org/repo default is permissive or restricted (new orgs and repos default to restricted). A `permissions:` key at workflow or job level overrides it, and **listing any scope sets every unlisted scope to `none`**. Pick the scope named after what changes: `issues`, `contents` (commits, tags, releases), `pull-requests`, `actions` (runs, artifacts).
- **workflow_run:** `workflows: [CI]`, `types: [completed]`.
  - `completed` fires for success, failure and cancelled, so check `github.event.workflow_run.conclusion`.
  - Chains stop after **three levels**: A → B → C → D → E → F never runs E or F.
  - It runs on the latest default-branch commit, so a plain checkout gives the default branch. To build what the triggering run tested, check out `github.event.workflow_run.head_sha` (`head_branch` may have moved).
- **workflow_call (reusable workflows):**
  - Inputs accept only `boolean`, `number`, `string`. No `choice` or `environment`; validate inside.
  - The caller's `env` is not propagated, and a calling job cannot set `env` or `runs-on`. Per-caller values travel as inputs.
  - Secrets: map them one by one, or `secrets: inherit` (same org or enterprise). A `required: true` secret not supplied fails the call.
  - Outputs take three hops out: step writes `GITHUB_OUTPUT` → `jobs.<job>.outputs` → `on.workflow_call.outputs.<name>.value: ${{ jobs.<job>.outputs.<x> }}`. Workflow-level outputs cannot reference steps. The caller reads `needs.<calling-job>.outputs.<name>`.
- **Passing data:**

| Need                    | Mechanism                                          | Reach                                        |
| ----------------------- | -------------------------------------------------- | -------------------------------------------- |
| Env var for later steps | `NAME=value` >> `$GITHUB_ENV`                      | Later steps of the same job only             |
| Named string value      | `name=value` >> `$GITHUB_OUTPUT` (step needs `id`) | `steps.<id>.outputs`; other jobs once mapped |
| String for another job  | Job `outputs:` mapped from a step output           | Jobs that `needs` it                         |
| Files                   | `upload-artifact` / `download-artifact`            | Other jobs, other runs, people               |

- `GITHUB_ENV` takes effect from the **next** step; use a shell variable in the writing step. Multi-line values use a `name<<EOF` delimiter. `::set-output` is deprecated.
- **Cache vs artifact:** cache = reuse files that rarely change (dependencies) to go faster; artifact = keep files a run produced (binaries, reports).
- **actions/cache:**
  - Exact `key` hit → `cache-hit` true. Miss → each `restore-keys` prefix, newest match restored, `cache-hit` false.
  - On a primary miss the post step saves a new entry, by default only if the job succeeds. Caches are **immutable**; a changing key (lockfile hash) refreshes content.
  - Evicted after **7 days** unused; repository size limit evicts oldest.
  - A run can restore caches from its own branch and the default branch; a PR also from its base. Sibling branches and other PRs are isolated.
- **Artifacts (v4):** names are unique and immutable per run; several jobs cannot upload into one artifact (put `matrix.os` in the name). `overwrite: true` replaces, it does not merge. `retention-days` sets one artifact's life within the repository maximum. From another run, `download-artifact` needs `run-id` (`github.event.workflow_run.id`, not `github.run_id`) **and** `github-token` with `actions: read`.
- **Retention via REST:**

| Goal                                | Call                                                                              |
| ----------------------------------- | --------------------------------------------------------------------------------- |
| Org-wide artifact and log retention | `PUT /orgs/{org}/actions/permissions/artifact-and-log-retention`                  |
| One repository                      | `PUT /repos/{owner}/{repo}/actions/permissions/artifact-and-log-retention`        |
| Delete one run's logs, keep the run | `DELETE /repos/{owner}/{repo}/actions/runs/{run_id}/logs`                         |
| Delete a whole run                  | `DELETE /repos/{owner}/{repo}/actions/runs/{run_id}`                              |
| Delete one artifact                 | `DELETE /repos/{owner}/{repo}/actions/artifacts/{artifact_id}`                    |
| Delete caches                       | `DELETE /repos/{owner}/{repo}/actions/caches` (by key) or `.../caches/{cache_id}` |

- A repository cannot keep data longer than its org or enterprise allows. Retention endpoints change policy for **future** runs; delete endpoints clean up what exists.
- **Job summaries:** append Markdown to `$GITHUB_STEP_SUMMARY`. Each step has its own file; `>` overwrites only the current step's content. Limit **1 MiB** per step; secrets are masked.
- **Status badges:** addressed by workflow **file name**: `.../actions/workflows/deploy.yml/badge.svg`, optional `?branch=` and `?event=`. Default is the default branch. Private repo badges are visible only to people with access.
- **Environment protections** (job opts in with `environment: <name>`): required reviewers, wait timer (fixed delay, then automatic), deployment branches and tags, custom protection rules (a GitHub App decides). Admins can bypass unless that is disallowed. Environment secrets reach only jobs that reference the environment, after the rules pass.
- **Confusions:**
  - Branch protection governs merges, not deployments. `timeout-minutes` cancels long jobs. `concurrency` limits runs per group but sets no fixed wait. None of them is a wait timer.
  - `workflow_dispatch` targets one file on a chosen ref with typed inputs. `repository_dispatch` targets the repository; every matching workflow runs.

📖 Full lesson: [GitHub Actions Triggers, Events, Caching, Artifacts and Outputs (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-triggers-events-outputs/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

## 1.2 Design and implement workflow structure

- **Shape:** a workflow has jobs; jobs run in **parallel** by default; steps inside a job run in order on one runner and share its file system. A step is `run:` (shell) or `uses:` (action).
- **if conditions:**
  - Evaluated as an expression even without `${{ }}`. A skipped job or step reports `skipped`, not failure.
  - A leading `!` must be inside `${{ }}` or quoted, because YAML reads an unquoted `!` as a tag. There is no `not` keyword.
  - `&&` binds tighter than `||`: `a && b || c` is `(a && b) || c`. Add parentheses.
  - String literals use single quotes; string comparison ignores case.
  - Loose equality converts mismatched types to numbers. Outputs are strings, so `needs.scan.outputs.found == true` is never true. Compare with `'true'` or use `fromJSON()`.
- **Which ref field:**

| Field             | Push to main      | PR into main          |
| ----------------- | ----------------- | --------------------- |
| `github.ref`      | `refs/heads/main` | `refs/pull/<n>/merge` |
| `github.ref_name` | `main`            | `<n>/merge`           |
| `github.base_ref` | empty             | `main`                |
| `github.head_ref` | empty             | the source branch     |

- **needs and status functions:**
  - `needs: [build, test]` waits for both to succeed. A failed or skipped needed job skips its dependents, down the chain.
  - An `if` without a status function gets an implied `success()`.

| Function       | True when                                                          |
| -------------- | ------------------------------------------------------------------ |
| `success()`    | Nothing earlier (or any ancestor job) failed or was cancelled      |
| `failure()`    | A previous step failed; for jobs, **any ancestor** failed          |
| `cancelled()`  | The run was cancelled                                              |
| `always()`     | Always, even on cancel                                             |
| `!cancelled()` | Anything except cancellation; GitHub recommends it over `always()` |

- React to one job: `always() && needs.migrate.result == 'failure'`. `needs.<job>.result` is `success`, `failure`, `cancelled` or `skipped`.
- `needs` context holds **direct** dependencies only. To read a grandparent's outputs, list it in `needs` too.
- **Required checks:** a job skipped by its own `if` reports **Success**. A workflow that never starts (filtered by branches/paths or a skip instruction) leaves the check **Pending**. To let some PRs skip a required job, keep the workflow triggered and skip the job based on an earlier detect job's output.
- **Workflow commands** (printed to stdout):

| Command                                  | Effect                                                                              |
| ---------------------------------------- | ----------------------------------------------------------------------------------- |
| `::notice::`, `::warning::`, `::error::` | Annotation; `file`, `line`, `endLine`, `col`, `endColumn`, `title` place it on code |
| `::debug::`                              | Shown only with `ACTIONS_STEP_DEBUG` = true                                         |
| `::group::title` ... `::endgroup::`      | Collapsible log group                                                               |
| `::add-mask::value`                      | Replace value with `***` for the rest of the job                                    |
| `::stop-commands::token` ... `::token::` | Stop processing commands so untrusted `::` lines print as text                      |

- **Environment variables:** `env:` at workflow, job and step level; the most specific wins. Default variables (`GITHUB_SHA`, `RUNNER_OS`, `RUNNER_TEMP`) live in the shell but **not** in the `env` context, so `env.RUNNER_OS` is empty in an expression. Use `github.sha`, `runner.os` in expressions; `$GITHUB_SHA` in scripts.
- **Service containers** (`jobs.<id>.services`): Linux runners only (hosted Ubuntu or self-hosted Linux with Docker). The runner creates a network, starts services before steps, removes them after.

| Steps run...                  | Host name                   | Ports                                                                                               |
| ----------------------------- | --------------------------- | --------------------------------------------------------------------------------------------------- |
| In a container (`container:`) | The service label (`redis`) | All ports reachable; no mapping; `localhost` is the job's own container                             |
| Directly on the runner        | `localhost`                 | Must map with `ports:`; `'6379'` alone picks a host port, read from `job.services.<id>.ports[6379]` |

- Health checks via `options: --health-cmd ... --health-interval --health-timeout --health-retries` make the runner wait until healthy. Better than a fixed `sleep`.
- Other keys: `image`, `env`, `ports`, `volumes`, `credentials`, `options`. `--network` is not supported; `needs` does not apply to services.
- `container:` runs the job's own steps in an image; `services:` runs extra containers beside it.
- **Matrix:**
  - One job per combination; at most **256** jobs per run.
  - `exclude` first, partial match removes all matching combinations. Excluded combinations never get a runner; a job-level `if` cannot read `matrix`.
  - `include` next: adds keys to every matching combination without overwriting original values; an entry that would overwrite an original value becomes a **new job**. Include-only matrix = one job per entry.
  - Adding a value to an axis multiplies jobs; for one extra combination use one full `include` entry.

| Setting                      | Effect                                                         |
| ---------------------------- | -------------------------------------------------------------- |
| `fail-fast` (default `true`) | One failure cancels in-progress and queued matrix jobs         |
| `continue-on-error` (job)    | That job may fail without cancelling others or failing the run |
| `max-parallel`               | Caps how many run at once; all still run                       |
| `concurrency`                | One running and one pending per group; not "N at a time"       |

- Matrix job outputs: all jobs share one `outputs:` map, so the same name is overwritten by whichever job ran last (order not guaranteed). Give each variant its own output name.
- **Runner image changes:** `-latest` labels move. `windows-latest` moved to Windows Server 2025; per the lesson, `ubuntu-latest` moves from 24.04 to 26.04 in a gradual rollout (19 October to 19 November 2026), so the same label can resolve to either image during it. Ubuntu 20.04 and Windows Server 2019 images were retired in 2025, so `ubuntu-20.04` jobs get no hosted runner and re-running does not help. Pin a version label (`ubuntu-24.04`) to control upgrades; branch on `runner.os`, not the label.
- **YAML anchors:** `&name` marks a node, `*name` copies it. Resolved at **parse time**, same file only, anchor must come **before** the alias. Merge key `<<: *defaults` merges a mapping with local keys overriding; GitHub's docs describe anchors and aliases but not merge keys, so validate before relying on `<<`.
- **Context availability:**

| Key                                        | Contexts                                                                |
| ------------------------------------------ | ----------------------------------------------------------------------- |
| Workflow `env`                             | github, secrets, inputs, vars                                           |
| `jobs.<id>.if`                             | github, needs, vars, inputs (no secrets, env, matrix, runner)           |
| `jobs.<id>.env`                            | github, needs, strategy, matrix, vars, secrets, inputs (no runner, env) |
| `runs-on`, `continue-on-error`, `services` | github, needs, strategy, matrix, vars, inputs                           |
| `steps.if`                                 | adds job, runner, env, steps (no secrets)                               |
| `steps.env`, `with`, `run`                 | everything including secrets                                            |

- Secrets are never allowed in `if`. Map the secret to `env` and test `env.X != ''` in a **step** condition.
- `steps.<id>.outcome` is before `continue-on-error`; `conclusion` is after. A tolerated failure has outcome `failure`, conclusion `success`.
- `strategy.job-index` is the natural shard number; `github.run_number` is the same for every job.
- **Evaluation and leakage:**
  - Parse/validation errors fail before any job. Job-level keys are evaluated when the job is scheduled; step expressions just before each step on the runner.
  - `${{ }}` in `run:` is pasted into the script **before** the shell starts, so `${{ env.X }}` written in the same step reads the old value, and attacker text becomes code. Pass untrusted values through `env:` and use `"$VAR"`, or as an action `with:` input.
  - Redaction matches exact strings. Not masked: encoded forms (Base64), derived values (a token minted with the secret), single fields of a JSON/YAML secret. Use `::add-mask::` and one secret per value.
- **Version pinning:** `@main` moves on every push; tags can be moved or re-created; a full 40-character SHA or `docker://image@sha256:` is fixed; `./local` comes from the same commit. Immutable releases lock a release's tag. An admin policy can **require full-SHA pins** for every action in a step (including GitHub's own); reusable workflows called by a job can still use tags.
- **VS Code extension:** syntax highlighting, schema/expression/context validation and IntelliSense (including job and step outputs), hover docs, parses inputs/outputs of referenced actions and reusable workflows, lists runs and logs, manages secrets/variables/environments. It does **not** run jobs locally, rewrite refs to SHAs or estimate minutes.

📖 Full lesson: [GitHub Actions Workflow Structure: Jobs, Conditions, Matrices and Contexts (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-structure-jobs-matrix-contexts/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

[← Back to the study guide](../README.md)
