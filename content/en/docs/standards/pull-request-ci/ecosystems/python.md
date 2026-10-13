---
title: Python
description: Preparing the Lifecycle scan input for Python projects using pip or Poetry.
weight: 80
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end. Source: Sonatype's [Python Application Analysis](https://help.sonatype.com/en/python-application-analysis.html).
{{% /alert %}}

## What Lifecycle reads

Python can be scanned from manifests or from binaries:

| Scan input | Notes |
|---|---|
| `requirements.txt` | Must use `==` with a version and no wildcards. Variable ranges and environment ranges are ignored. |
| `poetry.lock` | Parsed automatically. For Poetry 1.5.1 and later, development dependencies are excluded when `pyproject.toml` is present and discoverable. |
| `Pipfile.lock` | Needs IQ CLI 2.0 or later. Only dependencies under the `default` key are parsed. |
| `.whl` and `.tar.gz` files | **Binary** scan of the actual packages. |

Sonatype's recommendation for the best results is to first create a virtual environment, resolve dependencies with `pip install -r requirements.txt`, and use `pip download` or `pip wheel` to gather the packages as the scan target.

## Tier 1 job: Poetry lockfile (manifest)

This is the simplest option and fits a Project that already commits `poetry.lock`, such as Jake.

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
          cp poetry.lock pyproject.toml scan-input/

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

Include `pyproject.toml` so that Lifecycle can exclude development dependencies. This job runs nothing from the Pull Request.

## Tier 1 job: pip (binary)

```yaml
      - name: Set up Python
        uses: actions/setup-python@5fda3b95a4ea91299a34e894583c3862153e4b97 # v7.0.0
        with:
          python-version-file: .python-version # or set python-version: explicitly

      - name: Download packages
        run: |
          python -m venv "$RUNNER_TEMP/venv"
          . "$RUNNER_TEMP/venv/bin/activate"
          mkdir scan-input
          pip download -r requirements.txt -d scan-input
```

This follows Sonatype's guidance. Use `pip wheel -r requirements.txt -w scan-input` instead if you want wheels built from source distributions.

{{% alert color="warning" %}}
**Building runs setup code.** `pip download` of a source distribution, and `pip wheel`, can execute a package's build scripts. That is acceptable in Tier 1, which has no secrets, and is the reason this must not happen in Tier 2.
{{% /alert %}}

## Tier 2

Unchanged from the [reference workflow]({{% relref "../reference-workflows" %}}). Binary inputs are larger, so check the limits.

## Trusted-ref evaluation

Scan the real `poetry.lock` and `pyproject.toml`, or the downloaded packages, at `stage: release`.

## What you give up

- **Unpinned requirements.** A `requirements.txt` with ranges is skipped for those lines. Pin versions (for example with `pip freeze`, or a tool such as `pip-compile`) before scanning.
- **A single requirements file.** Sonatype's documentation says other variants of requirements files are ignored. If you split them (`requirements-dev.txt`), only the one named exactly is read.
- **Advisory only at PR time**, as for every ecosystem. A Pull Request that edits `poetry.lock` is changing the thing being scanned, and a reviewer should look at it.

## Not documented

Sonatype's page does not state what `site-packages` scans look like, and does not describe a `poetry export` flow. If a Project needs either, ask Sonatype before relying on it.
