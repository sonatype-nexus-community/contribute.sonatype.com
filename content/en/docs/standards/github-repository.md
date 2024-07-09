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
  * *Checks according to [Code Quality](/docs/standards/code-quality/) and [Dependency Management](/docs/standards/dependency-management/) must be covered*
* **Require signed commits** Yes ✅
* **Allow force pushes** No ❌
* **Allow deletions** No ❌

#### Actions

* **Fork pull request workflows from outside collaborators** set to `Require approval for first-time contributors`

#### Custom Properties

Set both `Flagship-Project` and `Project-Status` accordingly.

{{% alert color="warning" %}}TO DOCUMENT{{% /alert %}}

### Security

#### Code security and analysis

* **Private vulnerability reporting** No ❌ - See [Reporting Issues](/docs/contributing/reporting-issues/)
* **Dependabot** No ❌ - See [Dependency Management](docs/standards/dependency-management/)
* **Code Scanning** No ❌ - See [Code Quality](docs/standards/code-quality/)
* **Secret scanning** <span class="-text-accent-hot-pink">TBC</span>