# Domain 4: Manage GitHub Actions for the enterprise (24%)

This domain is Actions administration at organization and enterprise scale: the policies that decide where workflows run, which actions they may use and what their token can do; how standard automation is distributed or enforced; secrets and variables at every scope; and the runners that execute jobs, including self-hosted fleets, runner groups, networking and installed software. Questions usually state one requirement (a compromised action, reserved runners, an IP allow list, a credential that must stay gated) and ask for the setting, pattern, flag or API call that meets it.

## 4.1 Govern actions, workflows, secrets and variables

- **Layering:** enterprise → organization → repository. Each level can only **narrow** the level above. A repository cannot enable what its org disabled; an org cannot loosen the enterprise.
- **Three separate policies:** enablement (where workflows run at all), allowed actions (which actions/reusable workflows they use), token permissions (what they may change). Only enablement stops execution.

| Level        | Enablement choices                                                                        |
| ------------ | ----------------------------------------------------------------------------------------- |
| Enterprise   | All orgs / specific orgs / disable                                                        |
| Organization | All repos / selected repos / disable (left-off repos cannot run even a plain `run:` step) |
| Repository   | Enable or disable within what the org permits                                             |

- **Allowed actions:**

| Option                                    | What runs                                      | Watch out                      |
| ----------------------------------------- | ---------------------------------------------- | ------------------------------ |
| Allow all                                 | Anything                                       | No allow/block list            |
| Allow owner (org/enterprise) actions only | Only your own repositories' actions            | Also blocks `actions/checkout` |
| Allow owner, and select non-owner         | Owner actions plus ticked sources and patterns | The only option with a list    |

- Under "select": **Allow actions created by GitHub** (`actions` and `github` orgs), **Allow Marketplace actions by verified creators** (a broad switch), and **specified patterns**.
- Local `./` actions are never restricted by these policies.
- **Patterns:**

| Pattern                        | Matches                            |
| ------------------------------ | ---------------------------------- |
| `OWNER/REPO@ref`               | One action at a tag, branch or SHA |
| `OWNER/REPO/PATH/FILENAME@ref` | A reusable workflow file           |
| `OWNER/*`, `@*`                | Every repo of an owner; any ref    |
| `!OWNER/REPO@*`                | Block entry                        |

- Block one bad action, keep everything else: `*, !example-org/format-action@*`. Requires moving off "Allow all" first.
- **SHA-pinning policy** (enterprise, org or repo): refuses action references by tag or branch; requires full 40-char SHAs for actions in **steps**, including GitHub's own. Reusable workflows called by a job may keep tags. Local actions untouched. It controls format, not source.
- **Default GITHUB_TOKEN permissions:**

| Setting                                 | Default access                                  |
| --------------------------------------- | ----------------------------------------------- |
| Restricted (read contents and packages) | Read contents and packages (plus metadata) only |
| Permissive (read and write)             | Read and write across most scopes               |

- The default applies only with **no** `permissions` key. A declared key replaces it entirely; unlisted scopes become `none` (so `pull-requests: write` alone breaks a tag push).
- A restricted enterprise default cannot be loosened below; jobs declare the writes they need.
- **Allow GitHub Actions to create and approve pull requests** is off by default; while off, the token gets **403** opening or approving a PR even with `pull-requests: write`.
- **Fork PR policies:**
  - Public repos: choose how much approval outside contributors need (first-time contributors new to GitHub, all first-time contributors, all external contributors). `pull_request_target` workflows always run (base context), which is why they must never execute fork code.
  - Private/internal repos, four independent boxes: run workflows from fork PRs (read-only token, no secrets); send write tokens; send secrets and variables; require approval. The two "send" boxes belong only where every forker is trusted.
- **Distributing vs enforcing:**

| Mechanism                                          | Files in each repo? | Enforced? |
| -------------------------------------------------- | ------------------- | --------- |
| Workflow template                                  | Yes, a copy         | No        |
| Reusable workflow / shared action                  | Yes, a caller       | No        |
| Ruleset "Require workflows to pass before merging" | **No**              | **Yes**   |

- Templates: `.github` repo → `workflow-templates/`, `.yml` + `.properties.json` (`name`, `description` required). `$default-branch` becomes the literal branch at creation. Visibility: public `.github` → all repos; internal → internal and private; private → private only; users need Read on non-public. Templates do not cross orgs.
- Access sharing: private repo → same org (or same user); internal → enterprise; in enterprise orgs a private repo can also go enterprise-wide (callers must be private) and an internal one can be limited to its org. GitHub Team orgs (no internal visibility) share private repos at org level. The allowed-actions policy still applies on top.
- Ruleset-required workflows: configured once in an org or enterprise ruleset (source repo, file, ref; target repos by name or property). Run on `pull_request`, `pull_request_target`, `merge_group` with default types, and **ignore** `branches`, `paths`, `types` filters. Source visibility must suit targets, and a private/internal source needs its Access setting open or it never starts. Most CodeQL actions are blocked there (only `upload-sarif` allowed); use default setup + "Require code scanning results".
- **Delegation with custom org roles:** fine-grained permissions "Manage organization Actions policies", "...Actions secrets", "...Actions variables", "...runners and runner groups". Grant only what is needed; repository Admin does not reach org settings.
- **Secrets vs variables:** secrets are encrypted and redacted; variables are plain text. Anything granting access (token, password, connection string, webhook URL with key) is a secret. Read with `secrets.X` / `vars.X`.
  - Undefined variable → empty string (handy for optional flags). Unset/unreachable secret → empty string. Secrets cannot be used in `if:`.
  - Actions never read secrets on their own: pass via `with:` or `env:`. Composite actions cannot use the secrets context.
- **Scopes and precedence:**

| Scope        | Who receives it                                                         |
| ------------ | ----------------------------------------------------------------------- |
| Organization | Repos chosen by policy: all, private (private + internal), or selected  |
| Repository   | Every workflow and job, any branch                                      |
| Environment  | Only jobs with `environment:` that name it, after protection rules pass |

- Same name at several levels: **environment > repository > organization**, evaluated name by name.
- GitHub Free: org secrets and variables are **not** available to private repos.
- Environment secrets are the only scope that gates the value itself. A job that merely `needs:` a deploy job gets nothing. Widen deployers by changing the environment's branch policy, not by copying the secret to repo level.
- **Naming and limits:**
  - Names: alphanumerics and underscores; no `GITHUB_` prefix; cannot start with a number; case-insensitive (stored upper-case); unique per level.

| Limit                                  | Value                                   |
| -------------------------------------- | --------------------------------------- |
| One secret                             | 48 KB                                   |
| Organization secrets                   | Up to 1,000                             |
| Repository / environment secrets       | Up to 100 each                          |
| Org secrets a repo's workflows can use | First 100 shared, sorted alphabetically |

- Larger than 48 KB: `gpg --symmetric --cipher-algo AES256` the file, commit the encrypted file, store the passphrase as a secret, decrypt at run time. Decrypted contents are not redacted.
- **REST for secrets:** `GET .../actions/secrets/public-key` (key + `key_id` of that scope) → encrypt with **libsodium** sealed box, Base64 → `PUT .../actions/secrets/{name}` with `encrypted_value` and `key_id` (**201** created, **204** updated). No plain-text form; values can never be read back.
  - Org secrets add `visibility` (`all`, `private`, `selected` with `selected_repository_ids`). `PUT .../secrets/{name}/repositories` replaces the list; `PUT .../repositories/{repository_id}` adds one (204, no value needed); `DELETE` removes one. Single-repo calls return **409** unless visibility is `selected`.
- **REST for variables:** `POST .../actions/variables` (create), `PATCH .../variables/{name}` (update, 204), `GET` (returns the value), `DELETE`. Org variables: one `POST /orgs/{org}/actions/variables` with `name`, `value`, `visibility`.
- **CLI:** `gh secret set X --repo o/r`; `--env qa`; `--org o --visibility selected --repos a,b`; `gh variable set X --body v --org o --visibility all`. Org secret without `--visibility` defaults to private; needs `admin:org` scope.

📖 Full lesson: [Govern GitHub Actions in the Enterprise: Policies, Templates, Secrets and Variables (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-enterprise-governance-secrets-variables/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

## 4.2 Manage runners at scale

- **Runner kinds:**

|                       | Standard GitHub-hosted                | Larger runners                                                     | Self-hosted                                |
| --------------------- | ------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------ |
| Maintained by         | GitHub                                | GitHub                                                             | You                                        |
| Clean machine per job | Yes                                   | Yes                                                                | Only if ephemeral                          |
| Groups / static IPs   | No                                    | Yes (static IPs and Azure private networking on Linux and Windows) | Groups yes; IP is your network's           |
| Custom image          | No                                    | Yes                                                                | You build it                               |
| Cost                  | Per minute on private; free on public | Always billed, even public repos                                   | No Actions minutes; you pay infrastructure |

- Larger runners need GitHub Team or Enterprise Cloud. GitHub recommends self-hosted runners only for **private** repos (fork PRs can run code on your machine).
- **Registration:** one level only (repository, organization or enterprise). `config.sh` / `config.cmd` with `--url` and a short-lived `--token`. Flags: `--labels`, `--runnergroup`, `--ephemeral`, `--disableupdate`, `--replace`, `--unattended`. Run as a service: `svc.sh install` and `svc.sh start` (Linux/macOS) or a Windows service.
- **Routing:**
  - A job goes to an online, idle runner whose labels **and** group match `runs-on`. An array needs **every** label.
  - Default self-hosted labels: `self-hosted`, OS (`linux`, `windows`, `macOS`), arch (`x64`, `ARM`, `ARM64`). Labels are case-insensitive.
  - `runs-on: { group: build-secure, labels: [linux, arm64] }` = in that group and carrying every label. A group name inside a label array is just a label nobody has.
  - Labels select among runners; they are **not access control**.
  - Change labels on GitHub (runner settings page or REST add/set/remove); running a job or editing `.env` does not relabel.
- **Queue behaviour:** no matching idle runner → job stays queued; fails after **24 hours**. Assigned but not picked up in **60 seconds** → re-queued. No fallback from self-hosted to hosted. GitHub does not inspect installed software; a `docker` label is just a label.
- **Status:** Idle, Active, Offline (machine down, network lost, or service not running). A label/group mismatch still shows Idle.
- **Updates and removal:** the runner self-updates. With `--disableupdate` you must apply every release within **30 days** or jobs stop being queued to it; critical security updates can block jobs until applied. Not connected for more than **14 days** → removed automatically (ephemeral: **1 day**); re-register with a new token.
- **Ephemeral and autoscaling:** ephemeral runner takes one job, then is de-registered (`--ephemeral` or a JIT config via REST). Recommended for autoscaling; persistent autoscaling is not.

| Option                                 | How                                                         | When                                 |
| -------------------------------------- | ----------------------------------------------------------- | ------------------------------------ |
| Actions Runner Controller (scale sets) | Kubernetes operator; ephemeral pods via JIT; scales to zero | You run Kubernetes                   |
| Runner scale set client                | Library over the same APIs                                  | ARC-style scaling outside Kubernetes |
| `workflow_job` webhooks                | Your service provisions/removes machines                    | Your own autoscaler                  |

- Webhook autoscaler: act on `workflow_job` **queued** (start a runner) and **completed** (remove it). `in_progress` is too late; `workflow_run` is per run, not per job.
- **Pre/post-job scripts:** `ACTIONS_RUNNER_HOOK_JOB_STARTED` / `..._JOB_COMPLETED` = absolute path to a `.sh` or `.ps1`, set in the OS environment or the runner's `.env` (restart after editing). Non-zero exit from the started script fails the job; `continue-on-error` does not apply. Output under "Set up runner" / "Complete runner". The completed script runs **before** the job completes, so do not use it to delete the machine.
- **Runner groups** (the access boundary for self-hosted and larger runners): each runner in exactly one group; new runners go to **Default** unless `--runnergroup`.
  - Repository access: All or Selected. **Public repos excluded** until "Allow public repositories" is on.
  - Enterprise groups: shared with selected orgs by an enterprise owner, then an **org owner** must grant repositories.
  - Workflow access (Enterprise Cloud): selected workflows by full path at a ref (`.../deploy.yml@refs/heads/main`). Only jobs **directly defined** in a listed workflow qualify. Listing a reusable workflow admits its jobs from every caller; listing only the caller does not admit the called workflow's jobs.
  - For a public open-source repo, standard hosted runners are usually the safe and free choice.
- **Networking:** self-hosted runners need **outbound HTTPS 443** only, no inbound rules (they long-poll). Allow `github.com`, `api.github.com`, `*.actions.githubusercontent.com`, `codeload.github.com`, `results-receiver.actions.githubusercontent.com`, `*.blob.core.windows.net`, plus registries used.
- **Proxies:** `https_proxy`, `http_proxy`, `no_proxy` read **at start**. Put them in the runner's `.env` (a shell export never reaches a service), restart, and use **lowercase** on Linux/macOS (lowercase wins). Workflow `env:` does not affect the runner's own connection. On Azure VMs add `168.63.129.16` and `169.254.169.254` to `no_proxy`. Docker daemon and client need their **own** proxy config. `GITHUB_ACTIONS_RUNNER_TLS_NO_VERIFY` only skips TLS verification.
- **IP allow lists** (Enterprise Cloud): unlisted addresses are refused, including runner requests with `GITHUB_TOKEN`.
  - Self-hosted: add each runner's public egress IP.
  - Hosted: standard runners use large changing pools; use larger runners with **static IP ranges** or Azure private networking, or self-hosted. `GET /meta` ranges are not a supported allowlist.
  - The "allow IP allow list configuration for installed GitHub Apps" setting admits apps' declared IPs, not runners.

| Option                   | Runner type                                            | Best when                                     |
| ------------------------ | ------------------------------------------------------ | --------------------------------------------- |
| Static public IP ranges  | Larger runners, Linux/Windows (not macOS)              | Target firewall accepts listed public ranges  |
| Azure private networking | Hosted runners in your Azure subnet (not macOS larger) | Target has no public endpoint                 |
| API gateway with OIDC    | Standard runners                                       | You can run a gateway trusting the OIDC token |
| WireGuard overlay        | Standard runners                                       | You can run a peer in the private network     |

- **Troubleshooting self-hosted:** an offline runner produces no job log, so start on the machine.
  - `config.sh --check --url <url> --pat <token>` tests each service, PASS/FAIL, logs to `_diag`.
  - `_diag`: `Runner_` logs (connecting, receiving jobs) and `Worker_` logs (each job).
  - Linux service `actions.runner.<org>-<repo>.<name>.service`: `journalctl -u <name> -f`, `svc.sh status`, `systemctl show -p User <name>`. macOS: `svc.sh status`. Windows: `Get-Service actions.runner.*`, Event Viewer.
  - `permission denied` on `/var/run/docker.sock` → add the runner account to the docker group, restart. A snap-installed Docker can break container actions.
- **Monitoring:** Runners page (status, labels; hosted Active jobs and All jobs usage vs concurrency max). Actions metrics in Insights: **usage** = minutes; **performance** = run time, queue time, failure rates (diagnose long waits here). Each larger runner has its own max concurrency, which can queue jobs even when the org is below its overall limit. Orgs can disable standard hosted runners entirely.
- **Preinstalled software:** images update weekly and roll out over days. The job log's **Set up job → Runner Image** shows image name/version with links to Included Software and the release notes. The `actions/runner-images` readme updates only after a rollout finishes. Compare Runner Image versions between a passing and failing run. Runner version there is the runner app, not the image.
- **Installing more:**
  - `setup-*` actions resolve versions from the tool cache (`RUNNER_TOOL_CACHE`), downloading if missing; they keep your version stable across image changes. Pinning the OS label does not freeze tool versions. `check-latest: true` fetches the newest matching patch. `cache: npm|pip|...` caches the package manager download cache.
  - Self-hosted: set `AGENT_TOOLSDIRECTORY` in `.env`, give the runner account write access, restart.
  - Linux: `sudo apt-get update` then `sudo apt-get install -y <pkg>`. macOS: `brew install`. Windows: `choco install`.
  - Job `container:` for a prebuilt toolchain; `services:` are side containers.
  - Custom images for larger runners: image-generation runner + job with the `snapshot` keyword; GitHub recommends regenerating weekly.

📖 Full lesson: [Manage GitHub Actions Runners at Scale: Self-Hosted, Larger Runners and Runner Groups (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-runners-self-hosted-runner-groups/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

[← Back to the study guide](../README.md)
