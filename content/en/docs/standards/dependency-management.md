---
title: Dependency Management
# description: What does your user need to know to try your project?
# categories: [Examples, Placeholders]
# tags: [test, docs]
weight: 300
---

We use [Sonatype Lifecycle](https://www.sonatype.com/products/open-source-security-dependency-management) to ensure our Community Projects use only the best open-source dependencies.

Each project should include Sonatype Lifecycle analysis scans during each Pull Request and upon each Release.

You can check out the real world implementation for this handbook - here [for Continuous Integration](https://github.com/sonatype-nexus-community/the-cla/blob/main/.github/workflows/ci.yml) and here [for Release](https://github.com/sonatype-nexus-community/the-cla/blob/main/.github/workflows/release.yml).

When implementing your scans, do reference the [official Sonatype Lifecycle documentation](https://help.sonatype.com/en/analysis.html) that relates to the langugages and ecosystems in the project.

## Example GitHub Action for Continuous Integration

```yaml
env:
    LC_APPLICATION_ID: community-handbook.sonatype.com # <-- Our standard is to use the GitHub Repository Name

jobs:
    release:
        ...
        steps:
        ...
            - name: Sonatype Lifecycle Evaluation
              id: evaluate
              uses: sonatype/actions/evaluate@v1.0.1
              with:
                  iq-server-url: ${{ vars.SONATYPE_PLATFORM_URL }}
                  username: ${{ secrets.SONATYPE_LIFECYCLE_USERNAME }}
                  password: ${{ secrets.SONATYPE_LIFECYCLE_PASSWORD }}
                  application-id: ${{ env.LC_APPLICATION_ID }}
                  scan-targets: '.'
                  stage: build # <!-- Set to 'build' for the Continuous Integration
    ...
```


## Example GitHub Action for Release

```yaml
env:
    LC_APPLICATION_ID: community-handbook.sonatype.com # <-- Our standard is to use the GitHub Repository Name

jobs:
    release:
        ...
        steps:
        ...
            - name: Sonatype Lifecycle Evaluation
              id: evaluate
              uses: sonatype/actions/evaluate@v1.0.1
              with:
                  iq-server-url: ${{ vars.SONATYPE_PLATFORM_URL }}
                  username: ${{ secrets.SONATYPE_LIFECYCLE_USERNAME }}
                  password: ${{ secrets.SONATYPE_LIFECYCLE_PASSWORD }}
                  application-id: ${{ env.LC_APPLICATION_ID }}
                  scan-targets: '.'
                  stage: release # <!-- Set to 'release' for the Release Workflow
    ...
```