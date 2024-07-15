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
