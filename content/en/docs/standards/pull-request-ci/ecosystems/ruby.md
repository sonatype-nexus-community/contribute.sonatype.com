---
title: Ruby
description: Preparing the Lifecycle scan input for Bundler projects.
weight: 100
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end. Source: Sonatype's [Ruby Application Analysis](https://help.sonatype.com/en/ruby-application-analysis.html).
{{% /alert %}}

## What Lifecycle reads

Ruby is a **manifest** ecosystem, with a binary option. Sonatype's documentation says it scans `Gemfile.lock`, `.gem` files and `vendor/bundle`. For the lockfile, "components from sections GIT, GEM and PATH and with an exact version will be analyzed". For the best results it recommends scanning "after either installing the packages from a clean environment or storing the gems in a local cache".

The Project must **commit `Gemfile.lock`**. Gem projects (libraries) often do not, and then there is nothing to scan.

## Tier 1 job: lockfile

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
          cp Gemfile.lock scan-input/

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

This job runs nothing from the Pull Request.

## Tier 1 job: installed gems (binary)

Add this to scan the gems themselves. It runs `bundle`, which may execute native extension builds, so it belongs in Tier 1.

```yaml
      - name: Install gems into a local path
        run: |
          bundle config set --local path vendor/bundle
          bundle install
          mkdir scan-input
          cp Gemfile.lock scan-input/
          cp -r vendor/bundle scan-input/vendor-bundle
```

{{% alert color="info" %}}
**Not documented:** Sonatype's Ruby page does not say whether the directory has to be named `vendor/bundle` to be recognised. This snippet renames it to `vendor-bundle` to avoid a path clash, which has not been tested. Keep the original name if the scan does not pick it up.
{{% /alert %}}

## Tier 2

Unchanged from the [reference workflow]({{% relref "../reference-workflows" %}}).

## Trusted-ref evaluation

Scan the real `Gemfile.lock`, or the installed gems, at `stage: build` on `main` and `stage: release` for a release.

## What you give up

- **Only exact versions** are analysed, so a lockfile entry without one is skipped.
- **The Gemfile is not read.** The lockfile is the truth, which means a Pull Request that edits it is changing the thing being scanned, and a reviewer should look at it.
- **Advisory only at PR time**, as for every ecosystem.
