---
title: Rust
description: Preparing the Lifecycle scan input for Cargo projects.
weight: 110
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end. Source: Sonatype's [Rust Application Analysis](https://help.sonatype.com/en/rust-application-analysis.html).
{{% /alert %}}

## What Lifecycle reads

Rust is a **manifest** ecosystem, and the only file Sonatype's documentation describes is `Cargo.lock`: "the Cargo coordinate-based matching feature provides the ability to scan and evaluate Rust dependencies found in the Cargo.lock file". The `name` and `version` of each `package` entry are evaluated. The documentation does not mention `Cargo.toml` or built binaries.

Because the lockfile is the input, the Project must **commit `Cargo.lock`**. Library crates often do not. A Project that does not commit it has nothing to scan, and should either start committing it or generate it in Tier 1 with `cargo generate-lockfile` (which resolves to the newest versions, not the ones the Project tested against, and is therefore a weaker signal).

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

      - name: Gather scan input
        run: |
          mkdir scan-input
          cp Cargo.lock scan-input/

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

This job runs nothing from the Pull Request. For a workspace with several `Cargo.lock` files, copy each into its own subdirectory.

## Tier 2

Unchanged from the [reference workflow]({{% relref "../reference-workflows" %}}).

## Trusted-ref evaluation

Scan the real `Cargo.lock` at `stage: release`.

## What you give up

- **No binary scan is documented.** The result is only as honest as `Cargo.lock`. A Pull Request that edits the lockfile is changing the thing being scanned, and a reviewer should look at it.
- **Path and git dependencies** may not match published crates. Sonatype's page does not say how they are handled.
- **Advisory only at PR time**, as for every ecosystem.
