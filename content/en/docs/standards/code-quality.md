---
title: Code Quality
# description: What does your user need to know to try your project?
# categories: [Examples, Placeholders]
# tags: [test, docs]
weight: 200
---

We require all Sonatype Community Projects to undertake scans by SonarCloud (first party code analysis) and Sonatype Lifecycle (third party dependency analysis) as a minimum.

## First Party Code Analysis

We utilise [SonarCloud's Automatic Analysis](https://docs.sonarsource.com/sonarcloud/advanced-setup/automatic-analysis/).

Request your project is added to the Sonatype Nexus Community Sonar Cloud instance by reaching out to the Community Maintainers.

Once configured in SonarCloud, analysis will be automatic. You should configure SonarCloud Analysis as a required check for PRs into your `main` branch.

Additional configuration can be controlled by through the use of a `.sonarcloud.properties` file - read more [here](https://docs.sonarsource.com/sonarcloud/advanced-setup/automatic-analysis/#additional-analysis-configuration).

### Status Badge

We encourage projects to include a SonarCloud status badge in their readme. An example to add to your README might be as follows:

```markdown
[![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=sonatype-nexus-community_community-handbook.sonatype.com&metric=security_rating)](https://sonarcloud.io/summary/new_code?id=sonatype-nexus-community_community-handbook.sonatype.com)
```

Replace `community-handbook.sonatype.com` with your repository name.

## Dependency Analysis

We utilise a dedicated Cloud instance of [Sonatype Lifecycle](https://www.sonatype.com/products/open-source-security-dependency-management) for Sonatype Community Projects.

To add analysis, you should include something similar to the below GitHub Workflow example below.

```yaml
name: Continue Integration Checks

on:
  pull_request:

  push:
    branches:
      - main

  workflow_dispatch:

# Env Vars
env:
  LC_APPLICATION_ID: $(echo "${{ github.repository }}" | cut -d '/' -f2)

jobs:
 
  # You might have other jobs to run in parallel here  

  code_quality:
        name: Code Quality
        runs-on: ubuntu-latest
        timeout-minutes: 5
        steps:
            - name: Checkout Code
              uses: actions/checkout@v4
              with:
                  # Disabling shallow clone is recommended for improving relevancy of reporting
                  fetch-depth: 0

            # Run any preparation steps here - such as `npm install`

            - name: Sonatype Lifecycle Evaluation
              uses: sonatype-nexus-community/iq-github-action@master
              with:
                  serverUrl: ${{ secrets.SONATYPE_LIFECYCLE_URL }}
                  username: ${{ secrets.SONATYPE_LIFECYCLE_USERNAME }}
                  password: ${{ secrets.SONATYPE_LIFECYCLE_PASSWORD }}
                  applicationId: ${{ env.LC_APPLICATION_ID }}
                  stage: Build
                  target: .
```

The referenced secrets are provided at a GitHub Organization level.