---
title: GitHub Repository
# description: What does your user need to know to try your project?
# categories: [Examples, Placeholders]
# tags: [test, docs]
weight: 100
---

This page documents the configuration that must be applied to each Project's GitHub Repository.

{{% alert color="info" %}}
The use of **must** and **should** indicate whether it is a requirement or strong guideline.
{{% /alert %}}

{{% alert color="warning" %}}
Any setting not explicitly documented below is not considered part of the standards at the time of publication.
{{% /alert %}}

### General Settings

These settings are found under `Settings -> General`.

- **Require contributors to sign off on web-based commits** must be enabled ✅
- **Default branch** should be named `main`
- **Features**:
  - **Wikis** should be disabled ❌
  - **Issues** must be enabled ✅
  - **Sponsorhips** must be disabled ❌
  - **Preserve this repository** should be enabled ✅
  - **Discussions** should be enabled ✅
  - **Projects** should be disbaled ❌
- **Pull Requests**:
  - **Allow merge commits** should be enabled ✅
  - **Allow squash merging** should be enabled ✅
  - **Allow rebase merging** should be disabled ❌
  - **Always suggest updating pull request branches** should be enabled ✅
  - **Allow auto-merge** should be disabled ❌
  - **Automatically delete head branches** should be enabled ✅

### Code and automation

#### Branches

These settings are found under `Settings -> Code and automation -> Branches`.

The following Branch protection rules should be applied.

##### main

* **Require a pull request before merging** Yes ✅
  * **Require approvals** Yes - 1 ✅
  * **Dismiss stale pull request approvals when new commits are pushed** <span class="-text-accent-hot-pink">TBC</span>
  * **Require review from Code Owners** Yes ✅
  * **Allow specified actors to bypass required pull requests** No ❌
* **Require status checks to pass before merging** Yes ✅
  * **Require branches to be up to date before merging** Yes ✅
  * *Checks according to [Code Quality]({{< ref "/docs/standards/code-quality" >}}) and [Dependency Management]({{< ref "/docs/standards/dependency-management" >}}) must be covered*
  * *Every tier of [Pull Request CI & Secrets]({{< ref "/docs/standards/pull-request-ci" >}}) the Project has must be required - see [Required status checks](#required-status-checks)*
* **Require signed commits** Yes ✅
* **Allow force pushes** No ❌
* **Allow deletions** No ❌

##### Required status checks

All tiers must run and pass before a change is merged. A tier that is not required is a tier that does not exist, so every check below is added to the `main` ruleset.

| Check | Comes from | Applies to |
|---|---|---|
| The Project's CI result (one job that fails if any other CI job failed) | Tier 1 | Every Project |
| `Lifecycle / develop` | Tier 2 | Every Project |
| `Tier 3 / ...` (the Project names it, for example `Tier 3 / PRO acceptance`) | Tier 3 | Projects with tests that need a protected resource |
| `SonarCloud Code Analysis` | [Code Quality]({{< ref "/docs/standards/code-quality" >}}) | Every Project |

A few practical points:

* **Use one Tier 1 check.** A CI workflow should end in a single job that depends on all the others and fails if any of them did not succeed. Require that job, and not each individual job. Then adding or renaming a job does not mean editing the ruleset.
* **Tier 2 and Tier 3 report as commit statuses.** They run from separate workflows, so they report on the commit by name. Require the status name exactly as the workflow sets it.
* **A commit that has not been approved for Tier 3 is blocked.** A Tier 3 status is attached to a specific commit. A Pull Request with no status for its latest commit cannot merge until a maintainer approves that commit, which is the intent.
* **Branch protection that is configured as a "branch protection rule" works as well as a ruleset.** Prefer rulesets, because they can also protect tags.

{{% alert color="warning" %}}
A Project cannot merge its first Pull Request that adds these workflows if it also requires the checks they produce. Add the workflows first, and require the checks in the ruleset once they have run on `main`.
{{% /alert %}}

#### Actions

* **Fork pull request workflows from outside collaborators** set to `Require approval for first-time contributors`

#### Custom Properties

Set both `Flagship-Project` and `Project-Status` accordingly.

{{% alert color="warning" %}}TO DOCUMENT{{% /alert %}}

### Security

#### Code security and analysis

* **Private vulnerability reporting** No ❌ - See [Reporting Issues]({{< ref "/docs/contributing/reporting-issues" >}})
* **Dependabot** No ❌ - See [Dependency Management]({{< ref "/docs/standards/dependency-management" >}})
* **Code Scanning** No ❌ - See [Code Quality]({{< ref "/docs/standards/code-quality" >}})
* **Secret scanning** <span class="-text-accent-hot-pink">TBC</span>