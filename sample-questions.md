# GH-200 Sample Questions with Answers

20 worked practice questions for the **GitHub Actions (GH-200)** exam. Try each one before you open the answer.

For many more, with the same explanation on every option, use the [GH-200 practice questions](https://www.savemycert.com/practice/github-actions/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide) on SaveMyCert.

## Question 1

*Author and manage workflows*

**A developer adds a nightly integration-test workflow with a valid cron expression to the branch feature/nightly and pushes it. A push-triggered workflow on the same branch runs normally, but after two days the nightly workflow has never started. The repository's default branch is main. What explains the missing runs?** Choose one.

- **a.** Scheduled runs use only the workflow files that exist on the default branch
- **b.** Scheduled runs need an open pull request from the branch into main
- **c.** Scheduled runs wait until a repository administrator approves the new cron entry
- **d.** Scheduled runs start only after the workflow has been started once by hand

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Correct. GitHub evaluates schedule triggers only from workflow files on the default branch, so a cron defined only on feature/nightly is never picked up until it is merged to main.
- **b** ❌ An open pull request does not make a schedule fire. Only the copy of the workflow on the default branch is scheduled.
- **c** ❌ Cron entries need no approval. Approval gates exist for environments and for some fork pull request runs, not for schedules.
- **d** ❌ There is no such prerequisite for schedules. A manual first run is never required; the cron starts working as soon as the file is on the default branch.

**The concept.** Scheduled workflows run only from the default branch.

**Why this is correct.** The schedule event reads the workflow file on the repository's default branch. A cron that exists only on a feature branch is ignored, even though push and pull_request triggers on that branch work normally. Merging the workflow to main makes the schedule active. Manual first runs, approvals and open pull requests have no effect on schedules.

**How to reason it out**

1. Note which trigger works (push) and which does not (schedule).
2. Recall where GitHub reads schedule triggers from: the default branch only.
3. Check the branch the file was pushed to: a feature branch.
4. Conclude the schedule is never registered until the file reaches main.

> **Exam tip:** A cron trigger counts only when the workflow file is on the default branch.

</details>

📖 Learn this topic: [GitHub Actions Triggers, Events, Caching, Artifacts and Outputs (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-triggers-events-outputs/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 2

*Author and manage workflows*

**A team in Sydney wants a cleanup workflow to start at 02:00 local time every day. The run must stay at 02:00 local time when daylight saving time starts and ends, and the team does not want to edit the workflow twice a year. How should the trigger be configured?** Choose one.

- **a.** Keep the cron at 02:00 and set a TZ variable in the workflow env block
- **b.** Convert 02:00 Sydney time to UTC once and use that hour in the cron
- **c.** Keep the cron at 02:00 and add a timezone key set to Australia/Sydney
- **d.** Add two cron entries, one for standard time and one for daylight time

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ The env block changes the environment of the job's processes after it starts. It does not change when the scheduler fires, which stays in UTC.
- **b** ❌ This works only until the next daylight saving change. Without a timezone key the cron is UTC, so the local start time shifts by an hour twice a year.
- **c** ✅ Correct. A schedule entry accepts an optional IANA timezone string next to cron, and GitHub then follows that zone's daylight saving changes.
- **d** ❌ Both entries would fire all year, so the workflow would run twice a day, and the team would still need to manage the UTC offsets by hand.

**The concept.** Scheduled workflows run in UTC unless a timezone is set on the schedule entry.

**Why this is correct.** By default the cron expression in a schedule trigger is evaluated in UTC. GitHub lets you add a timezone key with an IANA zone name beside each cron entry, and the schedule then tracks that zone, including daylight saving changes. A fixed UTC conversion drifts by an hour twice a year, an env variable affects only the job's processes, and two cron entries both fire every day.

**How to reason it out**

1. Identify the requirement: a local wall-clock time that survives daylight saving changes.
2. Recall that cron in a schedule trigger is UTC by default.
3. Recall that each schedule entry can carry an IANA timezone.
4. Reject options that only change the job environment or duplicate the schedule.

> **Exam tip:** Cron is UTC by default; add timezone: <IANA zone> to follow local time and daylight saving.

</details>

📖 Learn this topic: [GitHub Actions Triggers, Events, Caching, Artifacts and Outputs (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-triggers-events-outputs/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 3

*Author and manage workflows*

**A workflow has three jobs: build, test and deploy. Today all three start at the same time. The team wants deploy to start only after both build and test have finished successfully. Which setting should be added to the deploy job?** Choose one.

- **a.** if: ${{ success() }}
- **b.** concurrency: build test
- **c.** environment: production
- **d.** needs: [build, test]

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ success() checks the jobs that deploy depends on, but without needs deploy has no dependencies and still starts at once.
- **b** ❌ A concurrency group limits how many runs or jobs in that group execute at a time and cancels pending ones; it does not make deploy wait for build and test.
- **c** ❌ An environment can add protection rules such as reviewers, but it does not make deploy wait for build and test.
- **d** ✅ needs makes deploy wait for both jobs, and by default deploy is skipped if either of them fails.

**The concept.** Jobs in a workflow run in parallel unless jobs.<job_id>.needs declares a dependency.

**Why this is correct.** needs lists the jobs that must complete successfully before this job starts. With needs: [build, test], deploy waits for both and is skipped if either fails. success() only has meaning relative to the jobs listed in needs, concurrency serialises runs rather than jobs, and an environment adds deployment protection, not ordering.

**How to reason it out**

1. Notice that the jobs currently start together: they have no dependencies.
2. Recall that job ordering is declared with needs.
3. List both upstream jobs so deploy waits for each of them.
4. Reject settings that control conditions, run concurrency or deployment protection.

> **Exam tip:** Jobs run in parallel by default; needs is what orders them.

</details>

📖 Learn this topic: [GitHub Actions Workflow Structure: Jobs, Conditions, Matrices and Contexts (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-structure-jobs-matrix-contexts/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 4

*Author and manage workflows*

**A repository has five deployment environments, but a manually started release workflow may only target staging or production. Operators must pick the target from a drop-down list, free text must not be possible, and staging must be preselected. Which input definition meets these requirements?** Choose one.

- **a.** type: string, options: [staging, production], default: staging
- **b.** type: choice, options: [production, staging], required: false
- **c.** type: environment, required: true, default: staging, description: Target
- **d.** type: choice, options: [staging, production], default: staging

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ A string input renders a free-text box; the options list belongs to choice inputs, so operators could still type any value.
- **b** ❌ With no default, the Run workflow form preselects the first listed option, which here is production, so staging is not preselected.
- **c** ❌ An environment input does render a drop-down, but it lists every environment in the repository, so all five targets would be selectable.
- **d** ✅ Correct. A choice input renders a drop-down limited to the listed options, and default preselects staging.

**The concept.** workflow_dispatch input types and properties.

**Why this is correct.** workflow_dispatch inputs can be boolean, choice, number, environment or string. Only a choice input restricts the operator to an explicit options list, and default sets the preselected value. The environment type is a close twin, but it lists all of the repository's environments, not a chosen subset. A string input is free text whatever else you add.

**How to reason it out**

1. List the requirements: drop-down, only two values, staging preselected.
2. Match 'only these values' to the choice type with an options list.
3. Rule out environment, which shows every repository environment.
4. Confirm a default is set so staging is preselected.

> **Exam tip:** Use type: choice with options and a default when operators must pick from a fixed list.

</details>

📖 Learn this topic: [GitHub Actions Triggers, Events, Caching, Artifacts and Outputs (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-triggers-events-outputs/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 5

*Author and manage workflows*

**In the following workflow, the lint job fails.

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: npm run lint
  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - run: npm test
  report:
    needs: test
    if: always()
    runs-on: ubuntu-latest
    steps:
      - run: ./publish-report.sh
```

What happens to the test and report jobs?** Choose one.

- **a.** test runs, and report runs
- **b.** test is skipped, and report runs
- **c.** test is cancelled, and report is skipped
- **d.** test is skipped, and report is skipped

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ A job whose needed job fails is skipped unless it has its own status-function condition; test has none.
- **b** ✅ test needs lint, so it is skipped when lint fails. report needs test, but always() lets it run even though test was skipped.
- **c** ❌ Dependent jobs are skipped, not cancelled, and always() on report means it is not skipped.
- **d** ❌ That would be true without always(). The status function on report overrides the default skip.

**The concept.** When a needed job fails or is skipped, every job that needs it is skipped unless the dependent job's if uses a status check function such as always().

**Why this is correct.** lint fails, so test (which needs lint and has no condition) is skipped. report needs test; normally it would be skipped too, because skipping propagates down the dependency chain. Its if: always() replaces the implied success() check, so report runs after test has been resolved, even though test never ran.

**How to reason it out**

1. Follow the chain: lint, then test, then report.
2. test has no if, so the implied success() fails and test is skipped.
3. report has always(), which ignores the result of the jobs it needs.
4. So report still runs.

> **Exam tip:** Failures skip the whole downstream chain, except jobs whose if uses always() (or another status function).

</details>

📖 Learn this topic: [GitHub Actions Workflow Structure: Jobs, Conditions, Matrices and Contexts (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-structure-jobs-matrix-contexts/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 6

*Consume and troubleshoot workflows*

**A repository has a workflow with this trigger:

```yaml
on:
  push:
    branches: [main]
    paths: ['src/**']
```

Over one afternoon, four pushes happen. Which push starts a run of this workflow?** Choose one.

- **a.** A push of tag v2.0.0 that points at a commit changing src/app.js
- **b.** A push to main whose commit changes src/app.js and docs/guide.md
- **c.** A push to release/2.0 whose commit changes src/app.js and src/db.js
- **d.** A push to main whose commit changes docs/guide.md and README.md

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ The trigger defines branches but no tags filter, so the workflow does not run for events affecting tags.
- **b** ✅ The branch filter matches main and at least one changed file matches src/**, so both filters are satisfied.
- **c** ❌ The paths match, but release/2.0 is not in the branches list, and both filters must be satisfied.
- **d** ❌ The branch matches, but no changed file matches src/**, so the paths filter is not satisfied.

**The concept.** When a push trigger defines both a branch filter and a paths filter, a push must satisfy both; a trigger that defines only branches does not run for tag pushes.

**Why this is correct.** GitHub runs the workflow only when both the branches and the paths filters are satisfied. A paths filter is satisfied when at least one changed file matches a pattern, so a commit touching src/app.js and docs/guide.md qualifies. The docs-only push fails the paths filter, the release/2.0 push fails the branch filter, and the tag push is excluded because the trigger defines branches but no tags filter, so events affecting tags are ignored.

**How to reason it out**

1. Check the ref: only main passes the branch filter, and a tag is a different kind of ref.
2. Check the files: at least one changed file must match src/**.
3. Require both checks to pass for the same push.
4. Pick the push that passes both.

> **Exam tip:** branches + paths means both must match; a branches-only trigger ignores tag pushes.

</details>

📖 Learn this topic: [Troubleshoot GitHub Actions Workflow Runs: Logs, Matrix Re-runs and Artifacts (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-troubleshoot-workflow-runs-logs-artifacts/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 7

*Consume and troubleshoot workflows*

**A platform team maintains the build-and-test pipeline that 60 repositories use. The pipeline has three jobs that run on different runners. When the team fixes a bug in it, every repository must pick up the fix on its next run without anyone editing those repositories. What should the team provide?** Choose one.

- **a.** A workflow template in the organization's .github repository
- **b.** A workflow file committed to each repository's .github/workflows
- **c.** A composite action that each repository runs as one step
- **d.** A reusable workflow that each repository calls from a job

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ A template is copied into each repository when someone creates a workflow from it. Later fixes to the template do not reach the copies.
- **b** ❌ Separate copies in each repository would each need their own commit for every fix.
- **c** ❌ A composite action is central, but it is a bundle of steps that runs inside the caller's job. It cannot contain three jobs on different runners.
- **d** ✅ A reusable workflow is one central, versioned definition. Callers that reference a branch or moving tag get the fix on the next run, and it can contain several jobs on different runners.

**The concept.** Workflow templates are copied scaffolds; reusable workflows are central definitions called at job level; composite actions are central step bundles.

**Why this is correct.** The two constraints are a multi-job pipeline and fixes that reach every repository without edits. A reusable workflow meets both: it is called with jobs.<id>.uses, keeps one definition, and can hold several jobs, each with its own runs-on. A template is only a starting copy, a composite action runs as steps in one job, and per-repository files need a commit each.

**How to reason it out**

1. Note that the shared logic spans three jobs on different runners.
2. Note that fixes must propagate without editing the consuming repositories.
3. Rule out templates and per-repository files, which are independent copies.
4. Rule out composite actions, which cannot contain jobs.
5. Choose a reusable workflow called from a job.

> **Exam tip:** Copy = template; central jobs = reusable workflow; central steps = composite action.

</details>

📖 Learn this topic: [GitHub Actions Workflow Templates, Reusable Workflows and Composite Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-workflow-templates-reusable-workflows/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 8

*Consume and troubleshoot workflows*

**A workflow in a repository has this trigger:

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

A developer pushes a commit to a new branch feature/login, opens a pull request from feature/login into main, pushes one more commit to feature/login, and finally merges the pull request. All work happens in the same repository. Which runs of this workflow appear in the Actions tab?** Choose one.

- **a.** Three runs: pull request opened, pull request synchronize, and pull request closed
- **b.** Three runs: pull request opened, pull request synchronize, and a push to main
- **c.** Four runs: a push to feature/login, pull request opened, synchronize, and a push to main
- **d.** Two runs: pull request opened and a push to main when the pull request is merged

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ closed is not a default pull_request activity type; the merge is recorded here as a push to main instead.
- **b** ✅ The first push is to a branch outside the push filter. Opening and updating the pull request are default pull_request activity types, and the merge pushes a commit to main.
- **c** ❌ The push filter lists only main, so the first push to feature/login does not start a run.
- **d** ❌ Pushing a new commit to the pull request's head branch is a synchronize activity, which is a default type and starts a run.

**The concept.** Reading a trigger means combining branch filters for each event with the default activity types of pull_request, and knowing that a merge is a push to the base branch.

**Why this is correct.** The push event is filtered to main, so pushing to feature/login creates nothing. pull_request with branches: [main] filters on the base branch; with no types listed it runs for opened, synchronize and reopened, so opening the pull request and pushing another commit each start a run. Merging creates a commit on main, which is a push to main and starts the third run. A closed activity is not in the default list, so there is no separate run for closing.

**How to reason it out**

1. Apply the push branch filter to each push: feature/login fails it, the merge into main passes it.
2. Recall the default pull_request activity types: opened, synchronize and reopened.
3. Match each action: open is opened, the extra commit is synchronize.
4. Count the runs: two pull_request runs plus one push run.

> **Exam tip:** pull_request without types runs on opened, synchronize and reopened; a merge shows up as a push to the base branch.

</details>

📖 Learn this topic: [Troubleshoot GitHub Actions Workflow Runs: Logs, Matrix Re-runs and Artifacts (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-troubleshoot-workflow-runs-logs-artifacts/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 9

*Author and maintain actions*

**A team is writing an action in TypeScript that calls the GitHub REST API through the Actions toolkit. Workflows will run it on Ubuntu, Windows and macOS runners. The action must start quickly and must not install anything when it runs. Which kind of action should the team build?** Choose one.

- **a.** A JavaScript action that runs the compiled, bundled code with Node.js
- **b.** A composite action that runs setup-node, npm ci and the TypeScript compiler
- **c.** A Docker container action that pulls a prebuilt image from a public registry
- **d.** A Docker container action built from a Dockerfile in the action's repository

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ JavaScript actions run directly on the runner's Node.js on all three operating systems and start without building or pulling anything.
- **b** ❌ This works on all three systems, but it installs packages and compiles on every run, which the team ruled out.
- **c** ❌ A prebuilt image saves the build, but container actions still run only on Linux runners.
- **d** ❌ Docker container actions run only on Linux runners, and building the image adds start-up time.

**The concept.** GitHub supports three kinds of custom action: JavaScript, Docker container and composite.

**Why this is correct.** JavaScript actions execute directly on the runner, so they run on Linux, Windows and macOS and are the fastest to start. Docker container actions are limited to Linux runners and must build or pull an image first. A composite action that installs tools and compiles code would run everywhere, but it breaks the requirement to install nothing at run time.

**How to reason it out**

1. List the runner operating systems the action must support.
2. Rule out Docker container actions, which run only on Linux.
3. Check which remaining option installs software on every run.
4. Choose the JavaScript action with bundled code.

> **Exam tip:** Cross-platform and fast start-up points to a JavaScript action; Docker container actions are Linux-only.

</details>

📖 Learn this topic: [Create Custom GitHub Actions: JavaScript, Docker and Composite Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-custom-actions-javascript-docker-composite/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 10

*Author and maintain actions*

**A workflow tests a library with a matrix on ubuntu-latest, windows-latest and macos-latest. One step uses the team's own lint action, whose action.yml has runs.using: docker. The Ubuntu jobs pass, but the lint step fails on the Windows and macOS jobs. Linting must keep running on all three systems with the least ongoing effort. What should the team do?** Choose one.

- **a.** Add a step that installs Docker before the lint step on the other two systems
- **b.** Add if: runner.os == 'Linux' to the lint step so the other jobs skip it
- **c.** Rewrite the lint action as a JavaScript action and release a new version
- **d.** Wrap the image in a composite action step that calls docker run on each runner

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Container actions are only supported on Linux runners, so installing Docker on Windows or macOS does not make the step run.
- **b** ❌ This stops the failures but no longer lints on Windows and macOS, which the team requires.
- **c** ✅ JavaScript actions run on Linux, Windows and macOS runners, so the same step works in every matrix leg.
- **d** ❌ macOS runners have no Docker engine and Windows runners run Windows containers, so a Linux image still fails there.

**The concept.** Docker container actions are supported only on Linux runners; JavaScript and composite actions run on all three operating systems.

**Why this is correct.** The failure is a platform limit, not a configuration error: a container action cannot run on Windows or macOS runners. Installing Docker does not lift that limit, and calling docker run from a composite step fails too because macOS hosted runners have no Docker engine and Windows runners use Windows containers. Skipping the step on non-Linux jobs removes coverage the team needs. Rewriting the logic as a JavaScript action is the one change that runs everywhere without extra per-run setup.

**How to reason it out**

1. Note the action type from runs.using: docker.
2. Recall that container actions run only on Linux runners.
3. Check whether Docker is even usable on hosted Windows and macOS runners for a Linux image.
4. Reject the option that drops linting on two systems.
5. Choose a JavaScript rewrite.

> **Exam tip:** Need an action on Windows or macOS? It must be JavaScript or composite, never a container action.

</details>

📖 Learn this topic: [Create Custom GitHub Actions: JavaScript, Docker and Composite Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-custom-actions-javascript-docker-composite/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 11

*Author and maintain actions*

**A platform team keeps a JavaScript action in a private repository in its organization. Workflows in other private repositories of the same organization fail when a step references the action. The code must stay hidden from anyone outside the organization. What should the team change?** Choose one.

- **a.** Store a personal access token as an organization secret for the consuming workflows
- **b.** Publish the action to GitHub Marketplace from a draft release in the repository
- **c.** Set the action repository's Actions Access option to the organization
- **d.** Give each consuming team Read access to the action repository

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ The runner resolves uses: references itself. A token secret in the caller does not grant access to a private action repository.
- **b** ❌ Marketplace listing requires a public repository, which breaks the requirement to keep the code inside the organization.
- **c** ✅ The Access setting under Settings > Actions > General is what lets workflows in other private repositories of the organization use the action. The repository stays private.
- **d** ❌ Repository permissions for people do not let workflows in other repositories load the action. The Actions Access setting controls that.

**The concept.** Private actions are shared through the action repository's Actions Access setting.

**Why this is correct.** A private repository's actions are usable only by its own workflows until its Access option (Settings > Actions > General) is set to allow repositories in the organization. That keeps the code private while workflows in other private repositories of the organization can reference it with owner/repo@ref. People's Read access and tokens in the caller do not change this, and Marketplace needs a public repository.

**How to reason it out**

1. Note the action lives in a private repository.
2. Note the callers are other private repositories in the same organization.
3. Recall that cross-repository use is controlled by the Actions Access setting.
4. Rule out Marketplace because it requires a public repository.

> **Exam tip:** Private action for the org = Settings > Actions > General > Access on the action's repository.

</details>

📖 Learn this topic: [Share, Publish and Version GitHub Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-publish-marketplace-versioning/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 12

*Author and maintain actions*

**An organization moves its jobs to self-hosted runners. Some workflows use Docker container actions. Which TWO conditions must a self-hosted runner meet to run those actions? (Select TWO.)** Choose 2.

- **a.** The runner is registered at the organization level
- **b.** The runner machine runs macOS with Docker Desktop
- **c.** The runner carries a custom label named docker
- **d.** The runner machine runs a Linux operating system
- **e.** Docker is installed on the runner machine

<details>
<summary>Show answer and explanation</summary>

**Answer: d, e**

- **a** ❌ Registration scope controls which repositories can use the runner, not which action types it can run.
- **b** ❌ Container actions are not supported on macOS runners, with or without Docker Desktop.
- **c** ❌ Labels only route jobs to runners; no label turns on container action support.
- **d** ✅ Container actions are supported only on Linux runners.
- **e** ✅ The runner builds or pulls the action's image and starts it with Docker, so Docker must be present.

**The concept.** Self-hosted runners need Linux and Docker to run Docker container actions.

**Why this is correct.** GitHub's documentation states that to run Docker container actions, self-hosted runners must use a Linux operating system and have Docker installed. Registration level and labels decide where jobs go, not what a runner can execute, and macOS is not a supported platform for container actions.

**How to reason it out**

1. Recall the platform limit of container actions.
2. Recall what the runner uses to start the container.
3. Discard options about routing and registration scope.

> **Exam tip:** Container actions on self-hosted runners: Linux plus Docker.

</details>

📖 Learn this topic: [Create Custom GitHub Actions: JavaScript, Docker and Composite Actions (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-custom-actions-javascript-docker-composite/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 13

*Manage GitHub Actions for the enterprise*

**An owner of the contoso organization wants its workflows to run actions and reusable workflows stored in contoso repositories and the actions GitHub publishes, such as actions/checkout. Actions from every other publisher, including verified Marketplace creators, must be refused. Which allowed-actions configuration meets this?** Choose one.

- **a.** Allow contoso actions and reusable workflows, the most restrictive option, with no non-contoso sources selected
- **b.** Allow all actions and reusable workflows, combined with the requirement that actions are pinned to a full commit SHA
- **c.** Allow contoso, and select non-contoso, actions and reusable workflows, with GitHub and verified creators checked
- **d.** Allow contoso, and select non-contoso, actions and reusable workflows, with only "Allow actions created by GitHub" checked

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ The owner-only option also blocks actions authored by GitHub, so every workflow that uses actions/checkout would fail.
- **b** ❌ SHA pinning controls how actions are referenced, not who publishes them; every publisher would still be allowed.
- **c** ❌ Checking "Allow Marketplace actions by verified creators" admits other publishers' actions, which the requirement forbids.
- **d** ✅ This policy always allows the organization's own actions and reusable workflows, and the one checked sub-option adds the actions in the actions and github organizations. Nothing else is allowed.

**The concept.** The allowed-actions policy has three levels: all, owner-only, and owner plus selected non-owner sources.

**Why this is correct.** Only the "select non-owner" policy can combine the organization's own actions with GitHub-authored ones. Its sub-options are independent: "Allow actions created by GitHub" admits the actions and github organizations, while "Allow Marketplace actions by verified creators" would admit other publishers. The owner-only option is a trap because it blocks actions/checkout too, and SHA pinning is about reference format, not source.

**How to reason it out**

1. List the allowed sources: the organization and GitHub only.
2. Rule out "allow all", which admits every publisher regardless of pinning.
3. Rule out owner-only, which blocks GitHub-authored actions.
4. Choose the select policy with only the GitHub sub-option checked.

> **Exam tip:** Owner-only blocks actions/checkout; use "owner and select non-owner" plus "created by GitHub".

</details>

📖 Learn this topic: [Govern GitHub Actions in the Enterprise: Policies, Templates, Secrets and Variables (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-enterprise-governance-secrets-variables/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 14

*Manage GitHub Actions for the enterprise*

**An enterprise owner set the enterprise policy to allow only enterprise actions and reusable workflows. Now every workflow that uses actions/checkout fails at job setup. Teams otherwise use only actions stored in enterprise repositories. The owner must keep third-party actions blocked and must not maintain copies of GitHub's actions. What should the owner do?** Choose one.

- **a.** Fork GitHub's actions/checkout repository into an enterprise organization and point the workflows at that fork
- **b.** Enable the policy that requires GitHub and third-party actions to be pinned to a full-length commit SHA
- **c.** Change the enterprise policy to allow enterprise and select non-enterprise actions, and check "Allow actions created by GitHub"
- **d.** Ask each organization owner to add actions/checkout@* and the other GitHub actions to the organization's allowed list

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ A fork is an enterprise action and would run, but it is a copy of GitHub's action that someone must keep updated, which the owner must avoid.
- **b** ❌ Pinning changes how an action is referenced. It does not change which publishers the allowed-actions policy accepts.
- **c** ✅ This keeps every third-party publisher blocked while admitting the actions in the actions and github organizations, with no copies to maintain.
- **d** ❌ An organization cannot allow what the enterprise policy blocks; the enterprise setting limits every organization under it.

**The concept.** An enterprise-level allowed-actions policy limits every organization; the enterprise-only setting blocks GitHub-authored actions.

**Why this is correct.** GitHub documents that the enterprise-only option also blocks actions such as actions/checkout. The fix is at the enterprise level: switch to the select policy and admit actions created by GitHub. Organizations cannot loosen an enforced enterprise policy, pinning does not change the source rules, and forking works but creates copies to maintain.

**How to reason it out**

1. Recognise that the enterprise-only option blocks GitHub-authored actions.
2. Remember that organizations cannot override an enterprise restriction.
3. Reject forking because of the no-copies constraint.
4. Switch the enterprise to the select policy with GitHub-created actions allowed.

> **Exam tip:** Fix a too-strict enterprise policy at the enterprise; organizations cannot loosen it.

</details>

📖 Learn this topic: [Govern GitHub Actions in the Enterprise: Policies, Templates, Secrets and Variables (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-enterprise-governance-secrets-variables/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 15

*Manage GitHub Actions for the enterprise*

**An organization has three self-hosted Linux x64 runners. Only one of them has a GPU, and an administrator gave that runner the custom label gpu. A model-training job must run on that machine and on no other. Which runs-on value should the job use?** Choose one.

- **a.** runs-on: [self-hosted, linux, x64]
- **b.** runs-on: [self-hosted, linux, gpu]
- **c.** runs-on: [self-hosted, gpu-linux-x64]
- **d.** runs-on: { group: gpu, labels: linux }

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ All three runners carry these default labels, so the job could land on a machine without a GPU.
- **b** ✅ A job goes only to a runner that has every label in the array, and only the GPU machine carries gpu.
- **c** ❌ No runner has a label named gpu-linux-x64, so the job would wait for a runner that never appears.
- **d** ❌ gpu is a label, not a runner group. This targets a group named gpu, which does not exist.

**The concept.** Self-hosted runner labels route jobs; an array in runs-on must match all of its labels.

**Why this is correct.** When runs-on is an array, GitHub sends the job only to a runner that has every listed label. The three machines share the default labels self-hosted, linux and x64, so adding the custom gpu label narrows the match to the one GPU runner. Listing only default labels matches all three machines, a group key looks for a runner group rather than a label, and a label nobody carries leaves the job queued.

**How to reason it out**

1. Note the only difference between the runners is the custom gpu label.
2. Recall that an array in runs-on must match all labels on one runner.
3. Include self-hosted, the OS label and gpu.
4. Reject group: because gpu is a label, not a group.

> **Exam tip:** runs-on array = AND of labels; add the custom label that only the target runner has.

</details>

📖 Learn this topic: [Manage GitHub Actions Runners at Scale: Self-Hosted, Larger Runners and Runner Groups (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-runners-self-hosted-runner-groups/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 16

*Manage GitHub Actions for the enterprise*

**An organization uses the policy that allows its own actions plus a specified list of others. Its workflows must be able to call the reusable workflow deploy.yml stored in the .github/workflows directory of platform-org/pipelines, a public repository in another organization, at any tag or commit. Which entry should be added to the list?** Choose one.

- **a.** platform-org/pipelines/.github/workflows/deploy.yml@main
- **b.** !platform-org/pipelines/.github/workflows/deploy.yml@*
- **c.** platform-org/pipelines/deploy.yml@*
- **d.** platform-org/pipelines/.github/workflows/deploy.yml@*

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ This matches only calls that reference main, so calls pinned to a tag or SHA would still be refused.
- **b** ❌ The ! prefix blocks a match instead of allowing it, which is the opposite of the requirement.
- **c** ❌ The pattern omits the .github/workflows path, so it does not identify the workflow file that callers reference.
- **d** ✅ Reusable workflows are listed as OWNER/REPOSITORY/PATH/FILENAME@ref, and the * wildcard after @ matches every tag, branch and SHA.

**The concept.** Specified-list entries name actions as OWNER/REPOSITORY@ref and reusable workflows as OWNER/REPOSITORY/PATH/FILENAME@ref.

**Why this is correct.** The list accepts the full path to a reusable workflow file, and * is a wildcard, so @* matches any reference. A fixed @main is too narrow, a pattern without the path does not match the workflow file, and a leading ! turns the entry into a block.

**How to reason it out**

1. Identify the item as a reusable workflow, which needs the file path.
2. Write OWNER/REPOSITORY/.github/workflows/FILENAME.
3. Use @* because any tag or commit must be accepted.
4. Make sure the entry has no ! prefix.

> **Exam tip:** Reusable workflow entries include the file path; @* matches every ref; ! blocks.

</details>

📖 Learn this topic: [Govern GitHub Actions in the Enterprise: Policies, Templates, Secrets and Variables (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-enterprise-governance-secrets-variables/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 17

*Manage GitHub Actions for the enterprise*

**A job declares runs-on: [self-hosted, linux, x64, docker]. The organization's four self-hosted runners show the status Idle and carry the labels self-hosted, Linux and X64. They sit in the default runner group, which every private repository can use. The job has shown "Waiting for a runner to pick up this job" for two hours. What is the cause?** Choose one.

- **a.** The runners lack the docker label, so none of them matches the job's label set
- **b.** GitHub finds no running Docker service on the hosts, so the docker label fails
- **c.** Labels are case-sensitive, so Linux and X64 do not match linux and x64
- **d.** The default runner group accepts jobs from public repositories, not private ones

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Every label in the array must match. None of the runners has docker, so the job stays queued until one appears (and fails after 24 hours).
- **b** ❌ Labels are plain names matched against runs-on; GitHub never checks whether Docker or any other software is installed or running.
- **c** ❌ Runner labels are case-insensitive, so Linux matches linux and X64 matches x64.
- **d** ❌ It is the other way round: by default a runner group admits private repositories and excludes public ones.

**The concept.** A job waits in the queue until an online runner matches all of its runs-on labels.

**Why this is correct.** GitHub assigns a job to an online, idle runner whose labels and group match runs-on. Labels are case-insensitive, so Linux and X64 match, but no runner has the docker label. The job therefore stays queued until a matching runner comes online, and it fails if it is still queued after 24 hours. The default group admits private repositories, and GitHub does not inspect installed software when matching labels.

**How to reason it out**

1. List the labels the job asks for.
2. Compare them with the runners' labels, ignoring case.
3. Find the label no runner carries: docker.
4. Recall that unmatched jobs stay queued (up to 24 hours) rather than failing at once.

> **Exam tip:** Queued with idle runners = a label or group mismatch; labels ignore case.

</details>

📖 Learn this topic: [Manage GitHub Actions Runners at Scale: Self-Hosted, Larger Runners and Runner Groups (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-runners-self-hosted-runner-groups/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 18

*Secure and optimize automation*

**A deployment workflow signs in to AWS with an access key ID and secret access key stored as repository secrets. After a key leaked from another system, the security team wants no long-lived cloud credentials stored in GitHub at all, while the workflow keeps deploying to the same account. What should the team do?** Choose one.

- **a.** Grant id-token: write and have the job assume an IAM role that trusts GitHub's OIDC provider
- **b.** Store the access keys as an organization secret with access limited to the deployment repository
- **c.** Add a scheduled workflow that rotates the access keys weekly and updates the repository secrets
- **d.** Move the access keys to a production environment secret that requires a reviewer's approval

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The job exchanges a short-lived OIDC token for temporary role credentials, so no cloud key is stored in GitHub.
- **b** ❌ Narrowing secret access does not remove the stored long-lived credential the team wants gone.
- **c** ❌ Rotation shortens exposure, but a long-lived key is still stored as a secret between rotations.
- **d** ❌ Approval limits who can use the keys, but long-lived keys are still stored in GitHub.

**The concept.** OIDC federation replaces stored cloud keys with short-lived credentials issued per job.

**Why this is correct.** With id-token: write, the job can request a signed OIDC token from GitHub. The cloud provider trusts GitHub's issuer and exchanges the token for short-lived credentials for a specific role. Environment approval, rotation and narrower secret scopes all keep a long-lived key stored in GitHub, which the requirement rules out.

**How to reason it out**

1. Note the requirement: no long-lived cloud credential stored in GitHub.
2. Eliminate options that only restrict, rotate or rescope a stored key.
3. Choose OIDC federation, which issues short-lived credentials per job.

> **Exam tip:** To store no cloud keys in GitHub, federate with OIDC (id-token: write) and assume a role.

</details>

📖 Learn this topic: [Secure and Optimize GitHub Actions: Permissions, OIDC, Pinning, Attestations and Caching (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-security-oidc-permissions-caching/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 19

*Secure and optimize automation*

**A job assumes an Azure identity through OIDC federation. The identity's federated credential is correct, but azure/login fails because it cannot obtain an ID token from GitHub.

```yaml
permissions:
  contents: read
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
```

Which change fixes the job while keeping the token's other access minimal?** Choose one.

- **a.** Add id-token: write to the permissions block next to contents: read
- **b.** Change the permissions block to contents: write so the job can sign in
- **c.** Add id-token: read to the permissions block next to contents: read
- **d.** Remove the permissions block so the job falls back to the token defaults

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Requesting an OIDC token needs id-token: write; it grants no write access to repository resources.
- **b** ❌ Contents access governs the repository's files and has no effect on requesting an OIDC token.
- **c** ❌ id-token accepts only write or none; read is not a valid level and grants no token request.
- **d** ❌ The default GITHUB_TOKEN permissions do not include id-token: write, so the job still cannot get an ID token, and other scopes widen.

**The concept.** Fetching the OIDC ID token requires the id-token: write permission.

**Why this is correct.** When a permissions block is present, every scope it does not list is none, so id-token is none here and the runner cannot request an ID token. Adding id-token: write enables only the token request. id-token has no read level (only write or none), contents has nothing to do with OIDC, and dropping the block falls back to defaults that still lack id-token: write while widening other scopes.

**How to reason it out**

1. Read the permissions block: only contents: read is granted.
2. Recall that unlisted scopes are set to none, so id-token is none.
3. Recall that OIDC token requests need id-token: write specifically.
4. Add that one scope instead of widening the whole token.

> **Exam tip:** OIDC sign-in needs id-token: write; it allows the token request and nothing else.

</details>

📖 Learn this topic: [Secure and Optimize GitHub Actions: Permissions, OIDC, Pinning, Attestations and Caching (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-security-oidc-permissions-caching/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

## Question 20

*Secure and optimize automation*

**An AWS role trusts GitHub's OIDC provider with the condition sub = repo:octo-org/web:ref:refs/heads/main, and pushes to main deployed fine. To add required reviewers, the team added environment: production to the deploy job, which already has id-token: write. Now the role assumption on main fails with an access-denied error. What should the team change?** Choose one.

- **a.** Add deployment branch rules for main to the production environment settings
- **b.** Change the trust condition to sub = repo:octo-org/web:environment:production
- **c.** Move id-token: write from the deploy job to the workflow-level permissions block
- **d.** Change the trust condition to aud = repo:octo-org/web:environment:production

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Branch rules restrict which refs may deploy, but the token's subject still uses the environment form.
- **b** ✅ A job that references an environment gets a subject in the environment form, so the old ref condition no longer matches.
- **c** ❌ The job already has the permission and gets a token; the claim it carries is what fails the condition.
- **d** ❌ The environment string belongs in the subject claim; the audience claim is a separate value the role also checks.

**The concept.** When a job references an environment, its OIDC subject claim uses the environment form instead of the ref form.

**Why this is correct.** GitHub builds the default sub claim from context: repo:ORG/REPO:environment:NAME when the job references an environment, repo:ORG/REPO:pull_request for pull requests, and repo:ORG/REPO:ref:... otherwise. Adding environment: production changed the subject, so a condition written for the ref form stops matching. Branch rules do not change claims, the audience is a different claim, and the permission was never the problem.

**How to reason it out**

1. Notice what changed: the job now references an environment.
2. Recall the order of default subject forms: environment, then pull_request, then ref.
3. Conclude the token's sub is now the environment form.
4. Update the trust condition to match the new subject.

> **Exam tip:** environment: in a job changes the OIDC sub to repo:ORG/REPO:environment:NAME; update the trust policy to match.

</details>

📖 Learn this topic: [Secure and Optimize GitHub Actions: Permissions, OIDC, Pinning, Attestations and Caching (GH-200)](https://www.savemycert.com/revision/github-actions/github-actions-security-oidc-permissions-caching/?utm_source=github&utm_medium=readme&utm_campaign=gh-200-study-guide)

---

[← Back to the GH-200 study guide](README.md)
