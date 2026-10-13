---
title: R
description: Preparing the Lifecycle scan input for R (CRAN) packages.
weight: 80
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end, and the thinnest guide: Sonatype's documentation for R is brief. Source: Sonatype's [R (CRAN) Application Analysis](https://help.sonatype.com/en/r--cran--application-analysis.html).
{{% /alert %}}

## What Lifecycle reads

The only input Sonatype's documentation describes is a file named `cran-installed.packages`, "exported via `installed.packages()` command in CRAN console". It does not describe scanning `DESCRIPTION`, `renv.lock` or package tarballs. Matching is by package name and version.

That means the scan input has to be generated from an R installation that has the package's dependencies installed. The Project's own repository does not contain it.

## Tier 1 job

```yaml
  prepare-scan-input:
    name: Prepare Lifecycle scan input
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          persist-credentials: false

      - name: Set up R
        uses: r-lib/actions/setup-r@f9a764fea8d5c63df6ef9a5c7795bf7deb5d7e05 # v2.14.0
        with:
          use-public-rspm: true

      - name: Install dependencies and export package list
        run: |
          mkdir scan-input
          Rscript -e 'install.packages("remotes"); remotes::install_deps(dependencies = TRUE)'
          Rscript -e 'write.table(installed.packages()[, c("Package", "Version")], file = "scan-input/cran-installed.packages", sep = "\t", quote = FALSE, row.names = FALSE)'

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

{{% alert color="info" %}}
**Not documented, and not yet verified:** Sonatype's page names the file and the command that produces it but not the exact column layout the scanner expects. The `write.table` line above is our guess at a sensible export and **must be checked against a real scan** before this guide is relied on.

OysterR is the Project this applies to. It has a `DESCRIPTION` file and no lockfile, so it would need this approach.
{{% /alert %}}

## Tier 2

Unchanged from the [reference workflow]({{% relref "../reference-workflows" %}}).

## Trusted-ref evaluation

Generate the same file from the real checkout and scan it at `stage: release`.

## What you give up

- **Only what is installed.** The scan covers what the Tier 1 job installed, which depends on the repositories and versions available at that moment, so two runs can differ.
- **Name and version only.** There is no binary fingerprinting for R in Sonatype's documentation.
- **Installing packages runs their code** (compilation, `.onLoad` hooks in some cases). That is acceptable in Tier 1 and is why this must not run in Tier 2.
- **Advisory only at PR time**, as for every ecosystem.
