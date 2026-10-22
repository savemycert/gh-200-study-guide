# GH-200 Study Guide: GitHub Actions

A free, open study guide for the **GitHub Actions (GH-200)** exam: revision notes for every domain, side-by-side comparisons of commonly confused services, a glossary, 20 worked sample questions, and the official syllabus as a checklist with a full free lesson for every topic.

Maintained by [SaveMyCert](https://www.savemycert.com/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide), where you can read every lesson free, [practice with explained questions](https://www.savemycert.com/practice/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide) and [take timed mock exams](https://www.savemycert.com/mocks/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide).

## Contents

- [Exam at a glance](#exam-at-a-glance)
- [Exam domains](#exam-domains)
- [What is in this repo](#what-is-in-this-repo)
- [Syllabus checklist](#syllabus-checklist)
  - [Domain 1: Author and manage workflows](#domain-1-author-and-manage-workflows)
  - [Domain 2: Consume and troubleshoot workflows](#domain-2-consume-and-troubleshoot-workflows)
  - [Domain 3: Author and maintain actions](#domain-3-author-and-maintain-actions)
  - [Domain 4: Manage GitHub Actions for the enterprise](#domain-4-manage-github-actions-for-the-enterprise)
  - [Domain 5: Secure and optimize automation](#domain-5-secure-and-optimize-automation)
- [How to study for GH-200](#how-to-study-for-gh-200)
- [Sample questions](sample-questions.md)
- [Free resources](#free-resources)

## Exam at a glance

| | |
|---|---|
| Exam code | GH-200 |
| Level | Associate |
| Questions | 40–60 |
| Time limit | 100 min |
| Passing score | 700 / 1000 |
| Format | Multiple choice & more |
| Exam fee |  |
| Valid for | 2 years |

Exam details change. Always confirm them in the official [Study guide for Exam GH-200: GitHub Actions (Microsoft Learn)](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-200) from Microsoft.

## Exam domains

| # | Domain | Weight | Topics |
|---|---|---|---|
| 1 | [Author and manage workflows](#domain-1-author-and-manage-workflows) | 24% | 2 |
| 2 | [Consume and troubleshoot workflows](#domain-2-consume-and-troubleshoot-workflows) | 19% | 2 |
| 3 | [Author and maintain actions](#domain-3-author-and-maintain-actions) | 18% | 2 |
| 4 | [Manage GitHub Actions for the enterprise](#domain-4-manage-github-actions-for-the-enterprise) | 24% | 2 |
| 5 | [Secure and optimize automation](#domain-5-secure-and-optimize-automation) | 15% | 1 |

That is 5 domains and 9 topics. Spend your time in proportion to the weights: the heaviest domain decides more of your score than the lightest.

## What is in this repo

| File | What it gives you |
|---|---|
| [Domain 1 notes](notes/domain-1-author-and-manage-workflows.md) | Author and manage workflows: condensed revision notes per topic |
| [Domain 2 notes](notes/domain-2-consume-and-troubleshoot-workflows.md) | Consume and troubleshoot workflows: condensed revision notes per topic |
| [Domain 3 notes](notes/domain-3-author-and-maintain-actions.md) | Author and maintain actions: condensed revision notes per topic |
| [Domain 4 notes](notes/domain-4-manage-github-actions-for-the-enterprise.md) | Manage GitHub Actions for the enterprise: condensed revision notes per topic |
| [Domain 5 notes](notes/domain-5-secure-and-optimize-automation.md) | Secure and optimize automation: condensed revision notes per topic |
| [Commonly confused services](comparisons.md) | Side-by-side tables of the services questions set against each other |
| [Glossary](glossary.md) | Every in-scope term and service in one sentence |
| [Sample questions](sample-questions.md) | 20 worked questions with answers and reasoning |
| [Exam-day guide](exam-day-guide.md) | Booking, testing options, scoring, results and retakes |

## Syllabus checklist

Tick each topic off once you can explain it without notes. The "Must know" facts are the ones questions turn on. Each lesson link goes to the complete, free lesson.

### Domain 1: Author and manage workflows

**Weight: 24%.** Triggers and events, workflow structure (jobs, steps, conditions, matrices, service containers, contexts and expressions, YAML anchors), and execution outputs: caching, artifacts, job summaries and badges. Official weight range 20–25%.

📝 Revision notes: [Domain 1: Author and manage workflows](notes/domain-1-author-and-manage-workflows.md)

- [ ] **Workflow triggers, events, execution and outputs**
  <br>"Configure workflow triggers and events": Configure workflows to run for scheduled, manual, webhook, and repository events; Choose appropriate scope, permissions, and events for workflow automation; Define and validate workflow_dispatch inputs (types, required, defaults) and pass inputs to reusable workflows via workflow_call with inputs and secrets mapping. "Manage workflow execution and outputs": Configure caching and artifact management; apply retention policies via REST APIs (logs, artifacts, workflow runs) at org/repo level; Pass data between jobs and steps (artifacts, outputs, environment files via GITHUB_ENV and GITHUB_OUTPUT, reusable workflow outputs); Generate job summaries using GITHUB_STEP_SUMMARY for rich Markdown reports (test results, coverage, links); Add workflow status badges and environment protections.
  - 📖 Lesson: [GitHub Actions Triggers, Events, Caching, Artifacts and Outputs (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-triggers-events-outputs/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
  - Must know: schedule, workflow_dispatch, repository_dispatch and workflow_run only use workflow files on the default branch; a dispatch can then target any ref.
  - Must know: Cron is UTC unless a timezone key is set, runs can be delayed, and public repositories disable schedules after 60 days without activity.
- [ ] **Design and implement workflow structure**
  <br>"Design and implement workflow structure": Use jobs, steps, and conditional logic; Implement dependencies between jobs; Use workflow commands and environment variables; Use service containers (services:) for dependent services (databases, queues); configure ports, health checks, and container options; Use strategy and matrix to generate job variations (OS, language/runtime versions); apply include/exclude; control fail-fast and max-parallel; optimize matrix size for cost and performance; account for runner image changes (Ubuntu 20.04 deprecation, Windows; Implement YAML anchors and aliases (&, * and merge <<) to reuse repeated mappings/steps within a single workflow file; Use predefined contexts (github, runner, env, vars, secrets, inputs, matrix, needs, strategy, job, steps, github.event, github.ref) to access workflow, repository, and runtime metadata; understand immutable actions behavior and version pinning requirements; Evaluate expressions with ${{ }} referencing contexts; distinguish static (workflow parse) vs runtime evaluation; prevent secret leakage in logs and expressions; Leverage editor tooling (GitHub Actions VS Code extension / YAML schema completion, metadata IntelliSense, validation) to author and maintain workflows efficiently.
  - 📖 Lesson: [GitHub Actions Workflow Structure: Jobs, Conditions, Matrices and Contexts (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-structure-jobs-matrix-contexts/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
  - Must know: An if without a status function implies success(); use always() or !cancelled() plus needs..result to run after an upstream failure or skip, and remember && binds tighter than ||.
  - Must know: A leading ! in an if must be inside ${{ }} (or quoted), because YAML reads an unquoted ! as a tag.

### Domain 2: Consume and troubleshoot workflows

**Weight: 19%.** Reading what a workflow did and why it failed: triggers, logs and run history, matrix expansions and reruns, artifacts via UI and API, and starter vs reusable workflows vs composite actions. Official weight range 15–20%.

📝 Revision notes: [Domain 2: Consume and troubleshoot workflows](notes/domain-2-consume-and-troubleshoot-workflows.md)

- [ ] **Interpret workflow results, artifacts and logs**
  <br>"Interpret workflow behavior and results": Identify workflow triggers and effects from configuration and logs; Diagnose failed workflow runs using logs and run history; Expand and interpret YAML anchors, aliases, and merged mappings when analyzing workflow configuration; Interpret matrix expansions, correlate job names to matrix axes, analyze failures across variants, and selectively rerun individual matrix jobs. "Access workflow artifacts and logs": Locate workflows, logs, and artifacts in the UI and via API; Download and manage workflow artifacts.
  - 📖 Lesson: [Troubleshoot GitHub Actions Workflow Runs: Logs, Matrix Re-runs and Artifacts (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-troubleshoot-workflow-runs-logs-artifacts/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
  - Must know: A run starts only when the event, its activity type and every filter match; branches plus paths must both pass, and branches-only push filters ignore tags.
  - Must know: pull_request defaults to opened, synchronize and reopened; merging a pull request is a push to the base branch.
- [ ] **Use and manage workflow templates**
  <br>"Use and manage workflow templates": Consume organization-level and reusable workflows; Consume non-public organization workflow templates; Use starter workflows (public and private/non-public templates); customize and adapt; distinguish from reusable workflows and composite actions; Differentiate starter workflows (copy scaffold, independent after creation) vs reusable workflows (central versioned definition invoked via workflow_call) vs composite actions (encapsulated step logic); Contrast disabling and deleting workflows.
  - 📖 Lesson: [GitHub Actions Workflow Templates, Reusable Workflows and Composite Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-templates-reusable-workflows/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
  - Must know: Template = a one-time copy; reusable workflow = central jobs called by a job; composite action = central steps run inside a job.
  - Must know: A workflow created from a template is independent: customise it in its own repository, and later template edits never reach it.

### Domain 3: Author and maintain actions

**Weight: 18%.** JavaScript, Docker and composite actions: metadata and directory structure, workflow commands, troubleshooting, immutable actions and version pinning, and publishing to the Marketplace. Official weight range 15–20%.

📝 Revision notes: [Domain 3: Author and maintain actions](notes/domain-3-author-and-maintain-actions.md)

- [ ] **Create, structure and troubleshoot custom actions**
  <br>"Create and troubleshoot custom actions": Identify and implement action types (JavaScript, Docker, composite); understand immutable actions rollout on hosted runners and implications for version pinning and registry sources; Troubleshoot action execution and errors. "Define action structure and metadata": Specify required files, directory structure, and metadata; Implement workflow commands within actions.
  - 📖 Lesson: [Create Custom GitHub Actions: JavaScript, Docker and Composite Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-custom-actions-javascript-docker-composite/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
  - Must know: JavaScript actions run on every runner OS and start fastest; Docker actions, docker:// steps and job containers are Linux-only; composite actions package steps (or existing scripts) and can call other actions.
  - Must know: action.yml or action.yaml must contain name, description and runs; required: true on an input is not enforced, so validate in code.
- [ ] **Distribute and maintain actions**
  <br>"Distribute and maintain actions": Select distribution models (public, private, marketplace); Publish actions to the GitHub Marketplace; Apply versioning and release strategies.
  - 📖 Lesson: [Share, Publish and Version GitHub Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-publish-marketplace-versioning/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
  - Must know: Any workflow can use an action in a public repository with owner/repo@ref; Marketplace only adds discovery.
  - Must know: Private and internal actions are shared through Settings > Actions > General > Access on the action's repository — not people's permissions, caller tokens or allow lists.

### Domain 4: Manage GitHub Actions for the enterprise

**Weight: 24%.** Actions at organization and enterprise scale: governance policies and reusable components, GitHub-hosted and self-hosted runners and runner groups, networking, and secrets and variables at every scope. Official weight range 20–25%.

📝 Revision notes: [Domain 4: Manage GitHub Actions for the enterprise](notes/domain-4-manage-github-actions-for-the-enterprise.md)

- [ ] **Govern actions, workflows, secrets and variables**
  <br>"Distribute and govern actions and workflows": Define and manage reusable components and templates; Control access to actions and workflows within the enterprise; Configure organizational use policies. "Manage encrypted secrets and variables": Define and scope encrypted secrets and variables at the organization, repository, and environment levels; Access and use secrets and variables in workflows and actions; manage secrets and variables programmatically via REST APIs.
  - 📖 Lesson: [Govern GitHub Actions in the Enterprise: Policies, Templates, Secrets and Variables (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-enterprise-governance-secrets-variables/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
  - Must know: Each policy level can only narrow the level above; organizations cannot loosen an enterprise policy, and repositories cannot override their organization.
  - Must know: Enablement decides where workflows run at all; the allowed-actions policy decides which actions they use; token permissions decide what they may change.
- [ ] **Manage runners at scale**
  <br>"Manage runners at scale": Configure and monitor GitHub-hosted and self-hosted runners; Apply IP allow lists and networking settings; Manage runner groups and troubleshoot runner issues; Identify preinstalled software/tool versions on GitHub-hosted runners (image release notes, toolcache) and install additional software at runtime (setup-* actions, package managers, caching, container images, custom self-hosted images).
  - 📖 Lesson: [Manage GitHub Actions Runners at Scale: Self-Hosted, Larger Runners and Runner Groups (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-runners-self-hosted-runner-groups/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
  - Must know: runs-on as an array needs a runner with every label; labels are case-insensitive and choose among runners, they never restrict access.
  - Must know: runs-on with group plus labels requires the runner to be in that group and carry every label.

### Domain 5: Secure and optimize automation

**Weight: 15%.** Hardening and tuning automation: environment protections, script-injection defences, GITHUB_TOKEN permissions vs PATs, OIDC federation, SHA pinning, action allow lists, artifact attestations, and caching and retention for cost. Official weight range 10–15%.

📝 Revision notes: [Domain 5: Secure and optimize automation](notes/domain-5-secure-and-optimize-automation.md)

- [ ] **Secure and optimize automation**
  <br>"Implement security best practices": Use environment protections and approval gates; Identify and use trustworthy actions from the Marketplace; Mitigate script injection (sanitize/validate inputs, least-privilege permissions, avoid untrusted data in run:, proper shell quoting, prefer vetted actions over inline scripts); Understand GITHUB_TOKEN lifecycle (ephemeral, scoped), configure granular permissions, contrast with PAT; restrict write scopes; Use OIDC token (id-token permission) for cloud provider federation to eliminate long-lived cloud secrets; Pin third-party actions to full commit SHAs; align with immutable actions enforcement on hosted runners; avoid floating @main/@v* without justification; Enforce action usage policies (organization/repository allow/deny lists, required reviewers for unverified actions); Generate and verify artifact attestations / provenance (e.g., SLSA, build metadata) and integrate into deployment verification. "Optimize workflow performance and cost": Configure caching and artifact retention for efficiency; apply retention policies programmatically via REST APIs; Recommend strategies for scaling and optimizing workflows.
  - 📖 Lesson: [Secure and Optimize GitHub Actions: Permissions, OIDC, Pinning, Attestations and Caching (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-security-oidc-permissions-caching/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
  - Must know: Required reviewers let the deployer approve their own run unless Prevent self-review is on, and admins can bypass the rules unless that option is cleared.
  - Must know: ${{ }} is pasted into script text before it runs, so untrusted values belong in env: and are read as "$VAR" or process.env, never as ${{ env.VAR }}.

## How to study for GH-200

1. **Read the lesson for each topic** in the checklist above, starting with the heaviest domain. Every lesson is free on the [GH-200 revision notes](https://www.savemycert.com/revision/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide).
2. **Practice straight after reading.** Answer [GH-200 practice questions](https://www.savemycert.com/practice/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide) on the topic you just read. Each option comes with an explanation of why it is right or wrong.
3. **Review what you got wrong**, re-read that lesson section, and tick the topic off only when you get its questions right.
4. **Take a full-length [GH-200 mock exam](https://www.savemycert.com/mocks/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)** under the real time limit. Aim to pass mocks comfortably before you book.
5. **On the last day**, skim the [GH-200 cheat sheet](https://www.savemycert.com/cheat-sheet/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide) instead of starting anything new.

## Sample questions

[sample-questions.md](sample-questions.md) has 20 worked GH-200 questions with the answer, why each option is right or wrong, and the reasoning steps.

## Free resources

- [Study guide for Exam GH-200: GitHub Actions (Microsoft Learn)](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-200): the official source (Microsoft)
- [GH-200 certification overview](https://www.savemycert.com/certifications/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)
- [GH-200 revision notes](https://www.savemycert.com/revision/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide): every lesson, free to read
- [GH-200 practice questions](https://www.savemycert.com/practice/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide): with an explanation on every option
- [GH-200 mock exams](https://www.savemycert.com/mocks/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide): full-length and timed
- [GH-200 cheat sheet](https://www.savemycert.com/cheat-sheet/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide): the key facts on one page
- [All certification study guides](https://github.com/savemycert/certification-study-guides)

## Contributing

Spotted an error or an out-of-date fact? [Open an issue](../../issues) with the topic and a link to the official source. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License and disclaimer

This guide is licensed under [CC BY 4.0](LICENSE). You can reuse and adapt it, including commercially, as long as you credit **SaveMyCert** with a link to https://www.savemycert.com/.

This is an independent study resource. It is not affiliated with or endorsed by Microsoft. GitHub Actions and GH-200 are trademarks of their respective owner. Exam domains and weights are taken from the official exam guide linked above.
