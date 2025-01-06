---
title: Activating a Project
weight: 70
---

Projects within our Community may occasionally transition between states, such as moving from an unset or archived status to active. This process ensures projects are thoughtfully evaluated and meet the standards required for active maintenance.

## Activating a Project

To activate a project, follow these steps:

### 1. Review the Project

Assess the project's relevance, dependencies, and community interest. This includes:

- Consider recent issues, pull requests, or community engagement
- Review alignment with current Community goals, as applicable
- Confirm the project’s compliance with Community [Project Standards](https://contribute.sonatype.com/docs/standards/)

### 2. Assign Maintainer(s)

Ensure at least one [maintainer](https://contribute.sonatype.com/docs/community_roles/maintainer/) is identified to oversee the project and handle ongoing contributions. This step is crucial for maintaining quality and responsiveness.

Maintainers MUST have previously contributed to another Sonatype Open Source Project. This ensures they are familiar with our Community workflows and have signed our CLA.

Assigning the initial maintainers for a project requires unanimous approval of the [Community Admins](../community_roles/admin.md). 

{{< alert title="Bring your friends" >}}
Additional maintainers may be added at this stage OR a later date with unanimous approval from the existing maintainers and support from at least one Community Admin.
{{< /alert >}}

### 3. Update the Custom Property

Modify the **Project-Status** property using [GitHub Custom Properties](https://docs.github.com/en/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization):
- **Set to `Active`** to indicate the project is actively maintained.

For projects previously archived:
- First, unarchive the repository via the GitHub interface.

### 4. Communicate the Change

- Announce the project’s activation in relevant Community channels.
- Update any associated documentation to reflect its active status.

## Qualifying Projects

For a list of projects currently archived or unset, use the following links:

- [Unset Projects](https://github.com/orgs/sonatype-nexus-community/repositories?q=visibility%3Apublic+archived%3Afalse+no%3Aprops.Project-Status)
- [Archived Projects](https://github.com/orgs/sonatype-nexus-community/repositories?q=visibility%3Apublic+archived%3Atrue)

Activation is a deliberate process aimed at fostering robust and meaningful contributions. By ensuring thoughtful evaluation, we maintain the integrity and quality of our Community projects.
