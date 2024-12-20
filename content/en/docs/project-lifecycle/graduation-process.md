---
title: Graduation Process
weight: 120
---

For some of our Community Projects, there comes a time where Sonatype officially supports the features or functions provided by a Community Project. In these situations we mark the Community Project as *Graduated* by following the below process.

## Graduation Preparation

### Project Graduation Proposal

For the sake of historical context and community insight, a GitHub Issue should be created to explain the reasoning and expectations for graduation. This is benefitial even in cases where community engagement is low.

[Look here](https://github.com/sonatype-nexus-community/nexus-blobstore-google-cloud/issues/132) to see a great real-world example!

### Project Graduation Notice

The Community Project should have it's `README` on it's main branch updated to contain an appropriate *Graduation Notice*. An example might be:

```
> ℹ️ As of 7th November 2024, this community project has [graduated](https://contribute.sonatype.com/docs/project-lifecycle/) and is offered as part of Sonatype's commercial offerings - see [here](https://help.sonatype.com/en/configuring-blob-stores.html#google-cloud-blob-store) for full details.
>
> 🚧 This community project will receive no further updates or maintenance.
```

It's good practice to be specific about **when** the project graduated, and where interested parties can head to find out more.

### CI/CD Closure

Any existing CI/CD configuration should be removed and projects deprovisioned from CI/CD systems.

### Project Properties

The project's [Custom Properties]({{< ref "/docs/project-lifecycle/#custom-properties" >}}) should be updated to reflect the Graduated status.

### Open Issues

Any open issues should be closed with a notice of the graduation.

## Graduating

Once the preparation steps are executed, the project should be marked as archived in GitHub.
