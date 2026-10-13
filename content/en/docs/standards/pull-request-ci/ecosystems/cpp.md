---
title: C/C++
description: Preparing the Lifecycle scan input for Conan-based C and C++ projects.
weight: 20
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end. No current community Project uses Conan, so this guide is the least exercised. Source: Sonatype's [C/C++ Application Analysis](https://help.sonatype.com/en/c-c---application-analysis.html).
{{% /alert %}}

## What Lifecycle reads

Sonatype's documentation says it analyses files named `conanfile.txt`, `conanfile.py` or `conaninfo.txt`, taking dependencies "with an exact version" from the `requires` and `build_requires` sections. It describes a scan of "a directory or subdirectories containing" those files. It does not describe a lockfile or a binary scan, and does not say that `conan install` is needed first.

Note that the Conan support is for the dependencies declared in those files. C and C++ code that is vendored into a repository, or fetched some other way, is not covered.

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
          for f in conanfile.txt conanfile.py conaninfo.txt; do
            [ -f "$f" ] && cp "$f" scan-input/
          done
          test -n "$(ls -A scan-input)"

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

This job runs nothing from the Pull Request. The final `test` makes the job fail if none of the files exist, because an empty directory would upload nothing.

## Tier 2

Unchanged from the [reference workflow]({{% relref "../reference-workflows" %}}).

## Trusted-ref evaluation

Scan the real Conan files at `stage: release`.

## What you give up

- **`conanfile.py` is code.** Lifecycle reads it as text and does not run it, but be aware that it is a Python file and that reviewers should treat changes to it as code changes.
- **Exact versions only.** Version ranges are not evaluated.
- **No binary scan** is documented.
- **Advisory only at PR time**, as for every ecosystem.
