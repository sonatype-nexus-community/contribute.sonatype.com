---
title: Project Lifecycle
# description: What does your user need to know to try your project?
# categories: [Examples, Placeholders]
# tags: [test, docs]
weight: 100
---

We use [GitHub Custom Properties](https://docs.github.com/en/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization) to indentify a Project's Classification within our Community.

## Custom Properties

### Flagship-Project

This is a boolean. Where set to `true`, this indicates the Project has Flagship Status - i.e. it's popular or something we want to highlight.

### Project-Status

There are four options to choose from:

1. **Active** - the project is being actively maintained
2. **Archived** - the project is not being maintained
3. **Graduated** - the project has been superseded by official Sonatype Project

The fourth option is to leave this field *unset*. This means the project's status has not yet been assessed and confirmed.

You can see a list of all Projects that have not yet been assessed for status by visiting [here](https://github.com/orgs/sonatype-nexus-community/repositories?q=visibility%3Apublic+archived%3Afalse+no%3Aprops.Project-Status).