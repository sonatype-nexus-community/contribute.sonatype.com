---
title: Standard Files
# description: What does your user need to know to try your project?
# categories: [Examples, Placeholders]
# tags: [test, docs]
weight: 150
---

This page outlines a set of **required** standard files and their contentst that
each Project **must** adhere to.

If you created your GitHub Repository from our
[Community Template](https://github.com/sonatype-nexus-community/community-project-template),
these files will be created in your repository - you'll just need to amend
content as directed below.

{{% alert color="info" %}} There are a number of files you might think need
creating that are not in the Template - see
[Organization Files](#organization-defined-files). {{% /alert %}}

## Files **YOU** should create

### Code Owners

{{% alert color="info" %}} This file must be named **CODEOWNERS** and be in the
`.github` folder at the root of your project. {{% /alert %}}

Every repository **must** define who owns it.

The owners **must** be the Project's own maintainers team, `@sonatype-nexus-community/[PROJECT]` (for example `@sonatype-nexus-community/the-cla`), so that the people who review changes are named by team and not by individual. See [Project Maintainer]({{< ref "/docs/community_roles/maintainer" >}}).

{{% alert color="warning" %}}
**The maintainers team is also the approval authority for Tier 3.** [Pull Request CI & Secrets](/docs/standards/pull-request-ci/#who-can-approve) lets only a person with the **Maintain** role approve licensed or otherwise protected tests, and the cleanest way to give that role is through this team. So the team must exist, must be granted **Maintain** on the repository, and must be the entry in `CODEOWNERS`. Do not list individuals in `CODEOWNERS` in place of the team, and do not rely on `@sonatype-nexus-community/community-leaders` alone: it does hold Maintain on repositories today, but it is the whole community's leadership, not the people who maintain this Project.
{{% /alert %}}

To set this up for a Project that does not have a team yet:

1. Create the team `@sonatype-nexus-community/[PROJECT]` and add the Project's maintainers.
2. Grant the team the **Maintain** role on the repository.
3. Replace the entries in `.github/CODEOWNERS` with the team.

Here are some groups you can reference if appropriate:

| Group                                         | Purpose                                       |
| --------------------------------------------- | --------------------------------------------- |
| `@sonatype-nexus-community/community-leaders` | Leaders of the Sonatype Open Source Community |

## Template files from the Community Project Template

### Contributing

{{% alert color="info" %}} This file must be named **CONTRIBUTING.md** and be in
the root of your project. {{% /alert %}}

This file contains project-specific information aiding others in providing
contributions to the project. We recommend using
[the template](https://github.com/sonatype-nexus-community/community-project-template/blob/main/CONTRIBUTING.md)
and expanding to include your project specific information such as:

- Development Guidelines
- Coding Conventions
- How to test and testing expectations

### License

{{% alert color="info" %}} This file must be named **LICENSE** and be in the
root of your project. {{% /alert %}}

This file contains the Open Source license applicable to this project. You can
just copy
[the license](https://github.com/sonatype-nexus-community/community-project-template/blob/main/LICENSE)
from the template repository.

{{% alert color="warning" %}} The approved license for Sonatype Community
Projects is **Apache-2.0**. {{% /alert %}}

### Readme

{{% alert color="info" %}} This file must be named **README.md** and be in the
root of your project. {{% /alert %}}

This is your projects "shop-window". We've provided a
[boilerplate template](https://github.com/sonatype-nexus-community/community-project-template/blob/main/README.md)
for you. Use this to introduce the project, define it's purpose, explain how to
use it etc...

You should not repeat information contained in other documentation files (such
as Contributing), but you should link to other documentation files.

{{% alert color="warning" %}} There must be a section **The Fine Print** at the
end of the file as per the template. {{% /alert %}}

## Organization Defined Files

The following files are defined at the GitHub Organization level (in
[this repository](https://github.com/sonatype-nexus-community/.github)) and you
DO NOT need to copy or reproduce them in your project.

### Code of Conduct

This file defines our conduct to define community standards, signal a welcoming
and inclusive project, and outline procedures for handling abuse.

### Security Policy

Provides instructions for how to report a security vulnerability in your project
by adding a security policy to your repository.
