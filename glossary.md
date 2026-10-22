# GH-200 Glossary

162 GH-200 terms and services, each in one plain-English sentence.

[#](#) · [A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [I](#i) · [J](#j) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [R](#r) · [S](#s) · [T](#t) · [U](#u) · [V](#v) · [W](#w) · [Y](#y)

## #

- **--disableupdate**: A registration flag that stops self-updates and obliges you to apply each runner release within 30 days.
- **.github repository**: An organization repository that holds shared files such as workflow templates.
- **@vercel/ncc**: A compiler that bundles an action's code and dependencies into one committed file.
- **\_diag directory**: The runner folder holding `Runner_` and `Worker_` logs and network-check results.
- **$default-branch**: A template placeholder replaced once with the repository's default branch name when a workflow is created.

## A

- **Action**: A reusable unit of work a workflow step runs with `uses:`.
- **action.yml**: The metadata file (also `action.yaml`) at an action's root that requires `name`, `description` and `runs`.
- **Actions Access setting**: A repository setting that lets other repositories' workflows use its actions and reusable workflows.
- **Actions enablement policy**: The enterprise, organization or repository setting that decides whether workflows may run at all.
- **Actions Runner Controller (ARC)**: A Kubernetes operator that autoscales ephemeral runners with runner scale sets.
- **Actions toolkit**: The `@actions/core` and related npm packages that wrap inputs, outputs and workflow commands for JavaScript actions.
- **Activity type**: A sub-type of an event, such as `opened` or `labeled` for `pull_request`, filtered with `types:`.
- **Administrator bypass**: The default environment option that lets repository admins force a deployment past its rules.
- **Allow/block pattern**: An entry such as `owner/repo@ref` or `!owner/repo@*` in the allowed-actions list.
- **Allowed actions policy**: The setting that decides which actions and reusable workflows workflows may reference.
- **Annotation**: A notice, warning or error message attached to a run, optionally to a file and line.
- **Approval gate**: An environment protection rule that holds a deployment job until it is approved.
- **Artifact**: A set of files uploaded from a run so later jobs, other runs or people can download them.
- **Artifact attestation**: A signed statement binding an artifact's digest to the repository, commit and workflow that built it.
- **artifact-url output**: An `upload-artifact` output giving a link to the uploaded artifact.
- **Attempt**: One execution of a run; each re-run adds an attempt with a higher `run_attempt`.
- **Attestation bundle**: The downloaded attestation file used with `--bundle` for offline verification.
- **Audience claim (aud)**: The OIDC claim naming the intended recipient, the same for every job.
- **Azure private networking**: Deploying hosted runners into a subnet of your Azure virtual network.

## B

- **branding**: Optional metadata that sets an action's Marketplace icon and color.

## C

- **Cache**: Stored files, usually dependencies, restored by key in later runs to make them faster.
- **Cache eviction**: Removal of caches unused for 7 days or beyond the repository's cache storage limit.
- **Calling job**: A job whose `uses` key calls a reusable workflow and accepts only a limited set of keys.
- **cancel-in-progress**: A concurrency setting that cancels a running job or run when a newer one joins the group.
- **Composite action**: An action that bundles several steps and runs them as one step inside the calling job.
- **Configuration variable**: A plain-text value read through the `vars` context and not redacted.
- **Context**: An object such as `github`, `env`, `matrix` or `steps` that expressions read workflow and run data from.
- **core.setFailed**: A toolkit call that logs an error annotation and sets a failing exit code without stopping the code.
- **create-github-app-token**: The action that mints a narrowed GitHub App installation token inside a job.
- **Custom deployment protection rule**: An environment rule where a GitHub App approves or rejects each deployment.
- **Custom image**: A larger-runner image built from a job using the `snapshot` keyword.
- **Custom organization role**: A role built from fine-grained permissions, such as managing Actions secrets, to delegate administration.

## D

- **Default group**: The runner group new runners join unless registered or moved elsewhere.
- **Dependabot version updates**: Automated pull requests that propose newer versions of actions, including SHA-pinned ones.
- **deprecationMessage**: An input setting that logs a warning whenever a workflow sets that input.
- **Disabled workflow**: A workflow switched off in the Actions tab, CLI or API so no trigger starts it on any branch.
- **disabled_inactivity**: The workflow state GitHub sets when it disables a public repository's schedules after 60 days without activity.
- **Distribution model**: How an action reaches users: a public repository, a shared private or internal repository, or a Marketplace listing.
- **Docker container action**: An action that runs in a container built from a Dockerfile or pulled from a registry, on Linux runners only.
- **docker:// reference**: A `uses` or `runs.image` value that pulls a prebuilt container image instead of building a Dockerfile.
- **Draft release**: An unpublished release that can still be edited and have assets uploaded.

## E

- **Encrypted secret**: A value GitHub stores encrypted and redacts from logs, read through the `secrets` context.
- **Environment**: A named deployment target whose protection rules and secrets apply to jobs that reference it.
- **Environment secret**: A secret released only to jobs that reference its environment, after protection rules pass.
- **Ephemeral runner**: A self-hosted runner that takes exactly one job and is then de-registered.
- **Event (trigger)**: An activity named under `on:` that starts a workflow run.
- **Exec-form ENTRYPOINT**: A Dockerfile entrypoint written as a JSON array, which starts without a shell and does not expand variables.
- **Expression**: A `${{ }}` value GitHub evaluates and substitutes before the key or script that contains it runs.

## F

- **fail-fast**: A matrix setting, true by default, that cancels the remaining matrix jobs when one fails.
- **filePatterns**: Regular expressions in template metadata that decide which repositories a template is suggested for.
- **Fork pull request policy**: Settings that decide whether and how workflows run, with which token and secrets, for pull requests from forks.

## G

- **gh run download**: The GitHub CLI command that downloads a run's artifacts, or one with `-n`.
- **gh run view**: The GitHub CLI command that shows a run and, with `--log` or `--log-failed`, its logs.
- **GitHub App installation token**: A one-hour token minted from a GitHub App installation, owned by the organization.
- **GitHub Marketplace**: A public listing where people discover actions, published from releases of public repositories.
- **GITHUB_API_URL**: The environment variable holding the REST API base URL for the GitHub platform the job runs on.
- **GITHUB_ENV**: An environment file whose `NAME=value` lines become environment variables for later steps of the same job.
- **GITHUB_OUTPUT**: An environment file where a step writes named outputs for later steps and jobs.
- **GITHUB_PATH**: The environment file `core.addPath` writes to prepend a directory to PATH for later steps.
- **GITHUB_STATE**: The environment file `core.saveState` writes so an action's later scripts can read private values.
- **GITHUB_STEP_SUMMARY**: An environment file where a step writes Markdown shown on the run's summary page.
- **GITHUB_TOKEN**: The short-lived token GitHub creates for each job to call the API on the workflow's repository.
- **GitHub-hosted runner**: A fresh GitHub-managed virtual machine selected by a label such as `ubuntu-latest`.
- **github.action_path**: The directory where a composite action was downloaded, used to reach files shipped with it.

## I

- **id-token permission**: The token scope (`write` or `none`) that lets a job request an OIDC token.
- **Immutable action**: An action version published so it cannot change, which hosted runners download in preference to Git.
- **Immutable release**: A published release whose tag is locked to its commit and whose assets cannot change.
- **include / exclude**: Matrix keys that add keys or extra combinations, and remove combinations, respectively.
- **INPUT\_ variable**: The environment variable (`INPUT_<NAME>`) through which JavaScript and Docker actions receive each input.
- **IP allow list**: An Enterprise Cloud setting that refuses requests from addresses not on the list.

## J

- **JavaScript action**: An action whose Node.js code runs directly on the runner on any operating system.
- **Job**: A set of steps that run in order on one runner; jobs run in parallel unless ordered with `needs`.
- **Job hook script**: A script named by `ACTIONS_RUNNER_HOOK_JOB_STARTED` or `..._JOB_COMPLETED` that runs around every job.
- **Job ID**: The numeric identifier of one job (or matrix variant), used to re-run or fetch logs for it.
- **Job outputs**: Values a job maps from its step outputs so dependent jobs can read them through `needs`.
- **job_workflow_ref**: An OIDC claim naming the reusable workflow and ref that ran the job.
- **Just-in-time (JIT) runner**: An ephemeral runner configuration created through the REST API.

## K

- **key_id**: The identifier of the public key used to encrypt a secret, sent with the encrypted value.

## L

- **Larger runner**: A GitHub-managed runner with more resources and features such as static IPs and runner groups.
- **Least privilege**: Granting each job only the token scopes and secrets it needs.
- **libsodium sealed box**: The encryption a REST client applies with the scope's public key before writing a secret.
- **Local action**: An action referenced with a `./` path in the same repository, available only after checkout.
- **Log archive**: The zip of all logs for a run or attempt, downloadable from the run page or the REST API.

## M

- **Major version tag**: A tag such as `v1` that maintainers move to the latest compatible release.
- **Marketplace Developer Agreement**: The agreement the owning account must accept before releases can be published to Marketplace.
- **Matrix expansion**: The list of jobs a matrix produces after the cross product, exclude and include are applied.
- **Matrix strategy**: A job setting that creates one job per combination of the listed variables.
- **Merge key**: The YAML `<<` key that merges an anchored mapping into another mapping, with local keys overriding.
- **Merge ref**: `refs/pull/<n>/merge`, the merge of a pull request's head into its base that pull_request runs build.

## N

- **needs**: A job key that makes a job wait for other jobs and gives it access to their outputs and results.

## O

- **OpenID Connect (OIDC)**: A standard that lets a job exchange a signed GitHub identity token for short-lived cloud credentials.
- **Organization secret access policy**: The choice of all, private or selected repositories that may use an organization secret.

## P

- **paths filter**: A trigger filter that runs the workflow only when at least one changed file matches a pattern.
- **permissions key**: Workflow or job YAML that sets each token scope to read, write or none; unlisted scopes become none.
- **Personal access token (PAT)**: A user-owned token, fine-grained or classic, that carries that user's identity.
- **pipefail**: A bash option that makes a pipeline fail when any command in it fails; added by `shell: bash`.
- **post-if**: The condition for a post script, `always()` by default and evaluated against the job's status.
- **pre / post**: Optional scripts that run at the start and end of the job around an action's main script.
- **Prevent self-review**: An environment option that stops the person who triggered a run from approving it.
- **properties.json**: The metadata file beside an organization template that gives its name, description and suggestion rules.
- **Provenance**: Verifiable information about where and how an artifact was built.
- **pull_request_target**: A pull request trigger that runs in the base repository's context with secrets and a write-capable token.

## R

- **Re-run**: Running jobs of an existing run again on the same commit and ref, as a new attempt.
- **Registration token**: A short-lived token used once by `config.sh` to register a self-hosted runner.
- **Release branch**: A branch such as `release/v1` used to develop and validate one major version line.
- **repository_dispatch**: A trigger fired by an external system through the REST API with an event type and a JSON payload.
- **Required status check**: A branch protection check that must report success before a pull request can merge.
- **restore-keys**: Prefix keys `actions/cache` tries on a miss, restoring the newest matching cache.
- **Retention period**: How long a repository keeps artifacts and logs before they expire.
- **retention-days**: An upload-artifact input that shortens one artifact's lifetime, never extending it past the repository setting.
- **Reusable workflow**: A workflow with an `on: workflow_call` trigger that other workflows call from a job.
- **Ruleset-required workflow**: A central workflow that an organization or enterprise ruleset runs on targeted pull requests and requires to pass.
- **Run history**: The list of past runs for a workflow in the Actions tab or the REST API.
- **Runner diagnostic logging**: The runner's own `Runner_` and `Worker_` logs, enabled by `ACTIONS_RUNNER_DEBUG`.
- **Runner group**: A set of self-hosted or larger runners with access rules for repositories and workflows.
- **Runner image**: The operating system image of a hosted runner, with its preinstalled software listed per version.
- **Runner image label**: A `runs-on` value such as `ubuntu-24.04` or `windows-latest` that selects a hosted runner image.
- **Runner label**: A tag on a runner that `runs-on` uses to select it; labels are case-insensitive.
- **runs.using**: The metadata key that sets the action type: a Node runtime, `docker` or `composite`.

## S

- **schedule**: A trigger that runs a workflow on a POSIX cron expression, evaluated in UTC unless a timezone is set.
- **Script injection**: Untrusted text substituted into a script through an expression and run as code.
- **Secret redaction**: The runner replacing exact registered secret values with `***` in logs.
- **secrets: inherit**: A calling-job setting that passes all of the caller's secrets to a reusable workflow in the same org or enterprise.
- **Self-hosted runner**: A machine you provide that runs GitHub's open-source runner application.
- **Semantic versioning**: Release tags in the form `vMAJOR.MINOR.PATCH`, where a new major signals a breaking change.
- **Service container**: A Docker container declared under `services:` that provides a dependency such as a database to a job.
- **SHA pinning**: Referencing an action by its full 40-character commit SHA so its code cannot change.
- **SHA-pinning policy**: An Actions setting that requires actions in steps to be referenced by a full 40-character commit SHA.
- **Sigstore**: The signing infrastructure used for attestations; public repositories use its public-good instance.
- **Skip instruction**: A commit-message marker such as `[skip ci]` that stops push and pull_request workflows from starting.
- **SLSA**: Supply-chain Levels for Software Artifacts, a framework whose build levels describe provenance guarantees.
- **Starter workflow**: The former name for a workflow template.
- **Static IP range**: A fixed public address range assigned to Linux or Windows larger runners.
- **Status badge**: An SVG image, addressed by workflow file name, that shows a workflow's latest result.
- **Status check function**: An expression function such as `success()`, `failure()`, `always()` or `cancelled()` used in `if`.
- **Step**: One unit of a job that either runs a shell command (`run`) or calls an action (`uses`).
- **Step debug logging**: Extra debug lines in step logs, enabled by `ACTIONS_STEP_DEBUG`.
- **Subject claim (sub)**: The OIDC claim describing the job's repository and ref, environment or pull request context, matched by cloud trust conditions.
- **Supply-chain attack**: Compromising a dependency, such as moving an action's tag to malicious code, to reach its users.

## T

- **timeout-minutes**: A job or step limit after which the job is cancelled; the job default is 360 minutes.
- **Tool cache**: The `RUNNER_TOOL_CACHE` directory of preinstalled runtime versions that setup actions resolve from.
- **Trust condition**: A cloud-side rule, such as an IAM role trust policy, that accepts only tokens with matching claims.
- **Trusted root**: The Sigstore certificate authorities and log keys, exported with `gh attestation trusted-root`, used for offline verification.

## U

- **Untrusted input**: Context values contributors control, such as branch names, titles, bodies and commit messages.

## V

- **Verified creator**: A Marketplace publisher GitHub has verified, admitted by the "verified creators" option.
- **Verified creator badge**: A Marketplace mark for actions from partner organizations GitHub has verified.

## W

- **Workflow**: A YAML file in `.github/workflows` that defines events, jobs and steps for GitHub Actions to run.
- **Workflow command**: A specially formatted `::command::` line printed to stdout that the runner acts on.
- **Workflow dependency lockfile**: A preview feature that records the exact commit of every action a workflow uses, including transitive ones.
- **Workflow permissions setting**: The default `GITHUB_TOKEN` scopes (restricted or permissive) for workflows without a `permissions` key.
- **Workflow run**: One execution of a workflow, identified by a run ID and run number.
- **Workflow template**: A workflow file GitHub copies into a repository as a starting point, after which the copy is independent.
- **workflow_call**: A trigger that makes a workflow reusable, so a job in another workflow can call it.
- **workflow_dispatch**: A trigger that lets people, the REST API or the CLI start one workflow on demand with typed inputs.
- **workflow_job webhook**: A webhook event, with queued, in_progress and completed actions, used to drive per-job autoscaling.
- **workflow_run**: A trigger that starts a workflow when another named workflow is requested, in progress or completed.

## Y

- **YAML anchor and alias**: `&name` marks a node and `*name` inserts a copy of it within the same file at parse time.

[← Back to the study guide](README.md)
