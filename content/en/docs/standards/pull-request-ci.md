---
title: Pull Request CI & Secrets
description: How Community Projects run automated checks on Pull Requests from forks without exposing secrets.
weight: 510
---

{{% alert color="warning" %}}
**Status: DRAFT - for community and maintainer review.** This page defines proposed standards and boundaries. Nothing here is yet implemented, and no Project is expected to change anything until the standard is agreed.
{{% /alert %}}

{{% alert color="info" %}}
The use of **must** and **should** indicate whether it is a requirement or a strong guideline.
{{% /alert %}}

## Contents

- [The problem](#the-problem)
- [Principles](#principles)
- [Overview of the tiers](#overview-of-the-tiers)
- [Tier 1: Open checks (no secrets)](#tier-1-open-checks-no-secrets)
- [Tier 2: Sonatype Lifecycle evaluation](#tier-2-sonatype-lifecycle-evaluation)
- [Tier 3: Licensed integration tests](#tier-3-licensed-integration-tests)
- [Trusted refs, releases and tags](#trusted-refs-releases-and-tags)
- [Intermediate branches](#intermediate-branches)
- [Repository and organization settings](#repository-and-organization-settings)
- [Workflow hygiene](#workflow-hygiene)
- [Contributor experience](#contributor-experience)
- [What Projects must, should and must not do](#what-projects-must-should-and-must-not-do)
- [Open questions for review](#open-questions-for-review)
- [Glossary](#glossary)

## The problem

Our Projects run two kinds of automation that need credentials:

1. **Sonatype Lifecycle evaluations**, which need credentials for Sonatype Lifecycle.
2. **Integration tests** against live, licensed installations of Sonatype's commercial products, which need license material and credentials.

Contributors outside the Project's maintainers work from **forks**. GitHub deliberately withholds secrets from workflows triggered by fork Pull Requests (see [Secrets Management](../secrets-management/)), so these checks fail, or are skipped, for exactly the people we most want to encourage.

The obvious workarounds are dangerous. A Pull Request is **untrusted code**: its author can change the application, its tests, its build scripts, its dependency manifests and even the workflow files. If untrusted code runs anywhere a secret is available, the secret should be considered leaked. Hiding the value in logs does not help, because code can send a secret elsewhere in any encoding it likes.

**This standard exists to give contributors fast, useful feedback while keeping untrusted code and secrets apart.**

## Principles

These principles drive every decision below. When a Project's situation is not covered, apply the principles.

1. **Untrusted code never runs where secrets exist.** Any job that executes code from a fork has no secrets, and any job that has secrets does not execute code from a fork.
2. **Privilege is earned by review, not assumed.** Access to secrets follows a maintainer's explicit decision about a specific commit, never an author's identity or history.
3. **Approval is bound to a commit.** A review of one commit is not a review of the next one pushed.
4. **Give as much feedback as possible without secrets.** The fewer checks that need a secret, the fewer contributors wait on a maintainer.
5. **Limit the damage if a secret does leak.** Secrets must be narrowly scoped, short lived and easy to rotate.
6. **Use each ecosystem's documented Sonatype integration.** We follow the [published Sonatype Lifecycle guidance](https://help.sonatype.com/en/analysis.html) for each language rather than inventing intermediate formats.
7. **One standard, many release styles.** The standard does not dictate how a Project cuts a release.

## Overview of the tiers

| Tier | Purpose | Runs for | Secrets | Contributor wait |
|---|---|---|---|---|
| **1** | Build, lint, unit and community-edition tests | Every PR, automatically | None | None |
| **2** | Sonatype Lifecycle evaluation | Every PR, automatically | Lifecycle credentials, held only by trusted code | None |
| **3** | Integration tests needing licensed products | PRs a maintainer has approved; trusted refs | Licensed-product access, held only by trusted code | Maintainer review |

A contributor's Pull Request should show a result for Tier 1 and Tier 2 within minutes of opening. Tier 3 shows as *waiting for maintainer approval* until a maintainer has reviewed the change.

```text
 Fork PR opened / updated
          |
          +--> Tier 1  (untrusted code, NO secrets) ---------> build, lint, tests
          |         |
          |         +--> prepares scan input (data only)
          |                     |
          +--> Tier 2  (trusted code, secrets) <-- reads input as DATA, never executes it
          |
          +--> Tier 3  (trusted code, secrets) <-- only after maintainer approves a specific commit
```

## Tier 1: Open checks (no secrets)

### What

Every Project must run the following on every Pull Request, including those from forks, using the `pull_request` trigger and with no access to secrets:

- Build / compile.
- Linting, formatting and static analysis that does not need credentials.
- Unit tests.
- Integration tests that can run against **unlicensed or community software**, such as Nexus Repository Community Edition running in a container, or mocked/recorded responses.

### Why

- It is the safest tier: a hostile Pull Request can only harm a short-lived, credential-free runner.
- It gives contributors the same feedback as maintainers get, immediately.
- It sets the boundary for Tier 3. **Everything that can be tested without a license must be tested here**, which keeps the licensed tier small.

### The licensed / unlicensed test split

Separating tests that require licensed products is a core part of this standard. Projects **must**:

- Identify which tests need a licensed product and which do not.
- Make the split explicit in the test code, using the ecosystem's native mechanism (build tags, test categories or markers, separate test directories, or an environment flag), not only in CI configuration.
- Ensure that a contributor can run the unlicensed set locally with a single documented command and no credentials.
- Ensure the licensed set **skips cleanly with a clear message** when its credentials are absent, instead of failing.

This benefits contributors beyond CI: they can run everything that applies to them on their own machine.

## Tier 2: Sonatype Lifecycle evaluation

### What

Every Project must evaluate its dependencies with Sonatype Lifecycle on every Pull Request and at release (see [Dependency Management](../dependency-management/)). For fork Pull Requests this standard requires the evaluation to be performed by **trusted code** in a **separate workflow** from the one that builds the Pull Request.

### Why this is harder than it looks

Lifecycle needs credentials. The straightforward approach is to run the scan in the same job that checks out the Pull Request, and that places untrusted code next to the credentials. Two things make a naive fix wrong:

- Switching to `pull_request_target` and checking out the PR head is the classic "pwn request" vulnerability.
- Some ecosystems resolve dependencies by running the project's own build tooling (for example, build plugins or install scripts). Pointing a privileged scan at an untrusted checkout can therefore execute untrusted code, even though nobody intended to "run" the PR.

### Approach: prepare unprivileged, evaluate privileged

The work is split in two so that the privileged side never executes anything from the Pull Request:

1. **Prepare (untrusted, no secrets).** In the Tier 1 workflow, run the ecosystem's *native* preparation step as documented for Lifecycle, so the scan target is exactly what Sonatype's documentation says it should be for that language. Typically this means resolved dependency manifests or lockfiles and, where the ecosystem requires it, the resolved dependency set. Upload the result as a workflow artifact.
2. **Evaluate (trusted, has secrets).** A separate workflow, triggered with `workflow_run` and therefore running the **default branch's** version of the workflow, downloads the artifact **as data only** and submits it to Lifecycle using the standard Sonatype tooling. It then reports the policy result back to the Pull Request.

### Native scanning, not an intermediate SBOM

We do not require, or want, the Pull Request to be converted into an intermediate Software Bill of Materials for evaluation. Lifecycle's results are most accurate when it receives the inputs its own ecosystem integrations expect. Per-ecosystem guidance (what to prepare and what to submit) must follow the [Sonatype Lifecycle documentation](https://help.sonatype.com/en/analysis.html). Where an ecosystem can only be scanned accurately by running its build tooling, the preparation happens in the unprivileged phase and only the *output* is transferred.

{{% alert color="info" %}}
The exact artifact contents differ by ecosystem and will be defined in per-ecosystem addenda to this page after review. See [Open questions](#open-questions-for-review).
{{% /alert %}}

### Rules for the privileged phase

The evaluation workflow **must**:

- Never execute, build, install or `source` anything from the artifact. It is data.
- Take the Pull Request number and commit SHA from the triggering event, never from files in the artifact.
- Check the artifact before use: expected name, a size limit, and an expected file layout.
- Have no `actions/checkout` of the Pull Request's head.
- Use credentials for a **dedicated Lifecycle account with the least privilege that can perform an evaluation**, and with no ability to read or change other applications or policies.
- Evaluate against a Lifecycle application identified per the existing standard (the repository name), at the **`build` stage** for Pull Requests.

### Reporting

The result should appear on the Pull Request as a check run, with a link to the Lifecycle report where the contributor can see it. Contributors should not need a Sonatype account to understand what to fix.

## Tier 3: Licensed integration tests

### What

Tests that need a licensed Sonatype product, such as a licensed Nexus Repository or Lifecycle instance, run only when a maintainer has approved the specific commit under test (or the code is already trusted, see [Trusted refs](#trusted-refs-releases-and-tags)).

### Why

These are the most sensitive credentials we hold, and the tests are the code a hostile contributor would most want to modify. A human review is the one control that reliably addresses that.

### Part A: The approval gate

Projects must gate Tier 3 behind an explicit maintainer action, and the gate **must** meet all of these conditions:

- **It is explicit.** A maintainer applies a label (or equivalent, such as an approving deployment review on a protected GitHub Environment) after reading the diff.
- **It is bound to a commit.** The workflow records the approved commit SHA and runs exactly that commit. If the Pull Request is updated, approval is **revoked automatically** and a maintainer must approve again.
- **It does not trust history.** Having a previous merged Pull Request is not grounds for automatic approval. Accounts get compromised.
- **It may auto-approve maintainers.** Pull Requests from members of the Project's maintainer team may skip the gate, as their branches live in the main repository.
- **Reviewers know what to look for.** Changes to CI configuration, build files, dependency manifests (including module replacement or override directives), test helpers and scripts are the most common ways to extract secrets. Projects should list these in `CODEOWNERS` so that a maintainer is always prompted to review them. See [Repository and organization settings](#repository-and-organization-settings).

### Part B: Don't give the license to the test code

Even after review, mistakes happen. The strongest protection is that **the code under test never holds the real credential**. The preferred design (target state):

1. A trusted job, running only default-branch code, provisions an **ephemeral** licensed instance (for example a container, VM or cloud sandbox) and injects the license there.
2. It passes the test job only the instance's address and **credentials generated for that run**.
3. The test job, which runs the contributor's code, has no organizational secrets. If it leaks what it was given, the leak is a throwaway instance that is destroyed when the run ends.

Projects that cannot yet do this may run licensed tests with real secrets **after the approval gate**, as an interim step. When they do, they must:

- Use secrets scoped to the repository or to a protected Environment, never organization-wide secrets available to all repositories.
- Use credentials and licenses that exist only for CI, are time-limited and are rotated on a schedule.
- Keep the licensed job's permissions minimal.

## Trusted refs, releases and tags

Projects release in different ways: by merging to `main`, by pushing a version tag, by publishing a GitHub Release, or through a manual dispatch. This standard does not dictate which. It defines what makes a trigger **trusted**.

A ref is trusted when only maintainers can create or change it. On a trusted ref, the code has been reviewed and merged, so Tier 2 (at `release` stage) and Tier 3 may run directly with full access to their secrets, exactly as described in [Dependency Management](../dependency-management/).

| Release style | Why it is trusted | Required controls |
|---|---|---|
| Merge to `main` | Branch protection requires reviewed PRs | Branch ruleset on `main` |
| Tag pushed manually | Only maintainers may create tags | **Tag ruleset** restricting creation of release tags (for example `v*`) to maintainers; many Projects forget this and any writer can create tags by default |
| GitHub Release / `workflow_dispatch` | Only users with write access can trigger | Environment with required reviewers for publishing |

In all cases:

- Publishing credentials (package registries, signing keys) **must** be Environment secrets with deployment rules restricted to the trusted refs. A feature branch, or an intermediate branch, must not be able to read them.
- Release workflows **must not** be triggered by a Pull Request event.
- Whatever the trigger, the release workflow should perform the `release`-stage Lifecycle evaluation and the full test set, including Tier 3.

## Intermediate branches

An *intermediate branch* is a branch in the **main repository** that holds a copy of a fork Pull Request's commit, so that it can be tested by the same `push`-triggered workflows used for trusted code.

### Why consider them

- Some Projects already have a Lifecycle and licensed-test workflow for `push` events, which would need changes to support the Tier 2 / Tier 3 split.
- They give maintainers a simple "break-glass" way to run Tier 3 for a PR they have reviewed.
- It lets you apply the trusted-ref controls (protected Environments with branch rules) to a PR.

### Why handle them carefully

- A `push` to any branch that matches a workflow trigger will run that workflow **with the secrets that trigger can see**. If anything other than a human maintainer can create these branches, you have rebuilt the vulnerability.
- The branch's content still came from a contributor. The workflow file *on that branch* is contributor-controlled, and `push` workflows run the version on the pushed branch.

### Conditions for use

If a Project uses intermediate branches, it **must**:

- Use a reserved naming scheme, such as `pr/<number>`, and a **branch ruleset** so that only maintainers (or a tightly scoped automation) can create or update `pr/*` branches.
- Publish only a commit a maintainer has reviewed, recording the reviewed SHA. Mirroring is a result of approval, not a substitute for it.
- Ensure secrets are held in an **Environment whose deployment branches** are limited to an explicit pattern, and not exposed to `pr/*` unless the Project has deliberately decided so.
- Prevent the `pr/*` branches from triggering release or publishing workflows.
- Delete the branch when the Pull Request is closed or merged.
- Make changes to `.github/` in a mirrored commit a **blocking** reason to refuse the mirror, or make the workflow definition come from the default branch instead (for example by calling a reusable workflow at a pinned reference).

{{% alert color="info" %}}
Intermediate branches are an allowed *mechanism* for Tier 3, not a requirement. The label-gated approach described in [Part A](#part-a-the-approval-gate) usually needs fewer moving parts. Intermediate branches are most useful where the Project already has trusted `push` workflows and wants to reuse them.
{{% /alert %}}

## Repository and organization settings

### Actions settings

Each Project must set, under `Settings -> Actions -> General`:

- **Fork pull request workflows from outside collaborators:** require approval (at minimum for first-time contributors).
- **Send write tokens / secrets to workflows from fork pull requests:** **disabled**.
- **Workflow permissions:** default to read-only.

### Secrets

- Secrets used by Tier 2 and Tier 3 **should** be repository- or Environment-scoped, not organization-wide. An organization secret is available to every repository allowed to use it, so a mistake in one repository exposes the secret for all of them.
- Environments holding the most sensitive secrets **should** use deployment branch/tag rules and, where useful, required reviewers.
- Credentials **must** be dedicated to CI, narrowly permissioned and rotated on a schedule, and the process for emergency revocation **must** be documented for the Project's maintainers.

### CODEOWNERS

Following [CODEOWNERS](../github-repository/), Projects should list the files that are most useful to an attacker and so deserve close review, including:

- `.github/` (workflows and actions)
- Build and dependency definitions (for example `Makefile`, `pom.xml`, `build.gradle`, `package.json`, `go.mod`)
- Test helpers, fixtures and scripts that Tier 3 runs

## Workflow hygiene

All Projects should follow these practices in every workflow, as they make the tiers above safer:

- Set `permissions: {}` (or `contents: read`) at the top of the workflow and grant extra permissions per job.
- Pin third-party actions to a full commit SHA.
- Never insert untrusted values (PR titles, branch names, commit messages) directly into `run:` scripts. Pass them as environment variables.
- Avoid `pull_request_target` unless the job does not check out or run contributor code.
- Do not use self-hosted runners for Pull Request workflows from forks.
- Run a workflow linter and a security scanner such as [actionlint](https://github.com/rhysd/actionlint) and [zizmor](https://github.com/woodruffw/zizmor) in CI. They detect the specific mistakes this page warns about.

## Contributor experience

The aim is that a contributor is never confused about why a check isn't running, and never blocked by something they could have fixed themselves.

- **Tier 1 and 2 results arrive on their own.** No approval is needed.
- **Tier 3 explains itself.** A bot comment or check summary should say that licensed tests need a maintainer to approve the change, why this is the case, and roughly how long it takes.
- **Contributors can test locally.** The `CONTRIBUTING` guide of each Project should explain how to run the unlicensed tests locally, for example with a Compose file that starts Nexus Repository Community Edition.
- **Contributors can bring their own license.** A contributor with a Sonatype trial or their own license can run the licensed tests in their own fork with their own secrets and link the run to the Pull Request. This is optional and useful evidence for maintainers, but never a requirement.
- **Maintainers have a response expectation.** Projects should aim to review approval requests promptly and say so in their contributor docs.

## What Projects must, should and must not do

### Must

- Run Tier 1 on every Pull Request without secrets.
- Separate licensed tests from unlicensed tests in the test code itself.
- Perform Lifecycle evaluation for fork PRs only through trusted code that treats PR output as data.
- Gate licensed tests behind a commit-bound maintainer approval.
- Restrict who can create release tags and publishing refs.
- Disable sending secrets to fork PR workflows.

### Should

- Scope secrets to repository or Environment rather than the whole organization.
- Use ephemeral, per-run credentials for licensed instances.
- Pin actions, restrict token permissions, lint workflows.
- Provide a local way to run unlicensed tests.

### Must not

- Check out and execute a fork's code in a job that can read secrets.
- Rely on log masking as a protection.
- Auto-approve a contributor because of their earlier contributions.
- Allow Pull Request events to trigger release or publish workflows.

## Open questions for review

We want feedback from maintainers and contributors on these before implementation starts.

1. **Per-ecosystem Lifecycle preparation.** For each supported ecosystem (Go, Java/Maven/Gradle, npm, Python, .NET and others), what exactly is the "prepare" output that Lifecycle needs, and can it be produced without executing anything privileged? Which ecosystems can scan only a manifest or lockfile, and which need resolved dependencies?
2. **Lifecycle evaluation of fork PRs.** Is a single shared "community pull requests" Lifecycle application appropriate, or should each Project's application receive PR evaluations, given the account permissions needed?
3. **Ephemeral licensed instances.** Which products and hosting (container, VM, cloud) are practical for per-run provisioning, and who funds and operates it?
4. **Reusable workflows.** Should the trusted pieces (approval gate, evaluation workflow, instance provisioning) live in a central repository as reusable workflows, so they are hardened and audited once?
5. **Approval mechanism.** Label, comment command or protected Environment: should the standard pick one, or allow all three?
6. **Intermediate branches.** Is there enough value to allow them, or should we only permit the gated, in-place approach?
7. **Migration.** How and when will existing Projects be asked to adopt this, given varied release processes?

## Glossary

| Term | Meaning |
|---|---|
| **Fork PR** | A Pull Request from a contributor's own copy of the repository. |
| **Untrusted code** | Any code a person outside the maintainers can change, including code in a PR. |
| **Trusted code** | Code on a protected default branch or protected tag. |
| **`pull_request`** | Runs the PR's code with a read-only token and no secrets when it comes from a fork. |
| **`pull_request_target`** | Runs the *default branch's* workflow with secrets. Safe only if it never runs the PR's code. |
| **`workflow_run`** | Runs a workflow (from the default branch) after another workflow completes. Used to pass data from the unprivileged to the privileged side. |
| **Environment** | A GitHub setting that holds secrets together with rules for who and what may use them. |
| **Ephemeral instance** | A temporary product installation created for one test run and destroyed afterwards. |
| **Intermediate branch** | A branch in the main repository holding a reviewed copy of a PR commit. |
