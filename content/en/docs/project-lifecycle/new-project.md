---
title: Starting a New Project
# description: What does your user need to know to try your project?
# categories: [Examples, Placeholders]
# tags: [test, docs]
weight: 20
---

You can share your idea for a new Project by starting a [Discussion here](https://github.com/orgs/sonatype-nexus-community/discussions/new?category=project-proposal).

[Organization Admins](../community_roles/admin.md) can execute the below procedure to create new Project within the Sonatype Open Source Community.

## New Project Proposal

### Prerequisites

- Ensure you have defined a clear purpose for the new project
- [Maintaniers](../community_roles/maintainer.md) of the project must have previously contributed to another Sonatype Open Source Project

{{< alert title="Why?" >}}
Having contributed previously means you'll have signed our CLA and understand our Community workflows.
{{< /alert >}}

### Creating the Project

- Create a new GitHub Repository in the [Sonatype Community GitHub Organization](https://github.com/organizations/sonatype-nexus-community/repositories/new) using the [Community Project Template](https://github.com/sonatype-nexus-community/community-project-template) as the template repository
- See our [GitHub Repository Configuration Standards]({{< ref "/docs/standards/github-repository" >}})


{{< alert title="Heads up!" >}}
This will cause a number of default files to be placed into your new repository.
It will also automatically inherit a number of Organization defined files that you do not need to redefine in your project.
{{< /alert >}}

## External Project Contributions

External contributors may propose an existing repository for migration into the Sonatype Open Source Community. Follow these steps to accept and onboard the repository:

### 1. Proposal Submission

- The external contributor should initiate a [Project Proposal Discussion](https://github.com/orgs/sonatype-nexus-community/discussions/new?category=project-proposal).
- Include the following details in the discussion:
  - Public Repository URL
  - A summary of the project’s purpose and goals
  - Any existing contributors who intend to transition as maintainers.

### 2. Initial Review

- Ensure the repository aligns with the Community’s goals and standards.
- Confirm that the repository has an open-source license compatible with the [Sonatype Open Source Community licensing policy](../standards).
- Assess the project's activity level, community engagement, and quality of documentation.

### 3. Feedback Period

- Allow a period for community members and Organization Admins to review and provide feedback on the proposal.
- A thorough review must be performed by Sonatype’s legal team.
- Address any concerns or required changes before proceeding.

### 4. Repository Transfer

Once the proposal is approved, the repository must be transferred to the [Sonatype Community GitHub Organization](https://github.com/sonatype-nexus-community).

The transfer process includes:

  - Coordinating with the repository owner to initiate the transfer.
  - Updating any necessary repository settings to align with [GitHub Repository Configuration Standards](../standards).

### 5. Assign Roles and Responsibilities

- Ensure that proposed maintainers meet the [Maintainer Requirements](../community_roles/maintainer.md).
- Assign the necessary roles and permissions to maintainers and contributors.
- Update the repository's [Code Owners](/docs/standards/repository-contents/#code-owners) and the respective GitHub Team.

### 6. Announce and Promote

- Publicly announce the inclusion of the project in the Community through appropriate channels.
- Update the [Community Project List](../../projects/active) to reflect the new addition.
