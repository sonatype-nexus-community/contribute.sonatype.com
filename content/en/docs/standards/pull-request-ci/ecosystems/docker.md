---
title: Docker
description: Preparing the Lifecycle scan input for container images.
weight: 30
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end. Container images are the largest scan input, and the one with the most open questions. Source: Sonatype's [Docker Image Analysis](https://help.sonatype.com/en/docker-image-analysis.html).
{{% /alert %}}

## What Lifecycle reads

Docker is a **binary** ecosystem. Sonatype's documentation describes the scan target as an image saved to a tar archive with `docker save`, which the CLI then accepts directly: for example `docker save webgoat/webgoat-7.1:latest > webgoat-7.1.tar`. It says the Sonatype CLI "relies on the Docker daemon to package the image as an archive file".

That is helpful for this standard. If the Tier 1 job builds the image and saves the tar, the Tier 2 job has **no need for a Docker daemon**, because the CLI is given a file. It also means the Tier 2 job never pulls an image from a registry, so it never needs registry credentials.

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

      - name: Build image
        run: docker build --tag scan-candidate:pr .

      - name: Save image
        run: |
          mkdir scan-input
          docker save scan-candidate:pr --output scan-input/image.tar

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

A `docker build` runs the Dockerfile's instructions, which is code from the Pull Request. That is acceptable in Tier 1.

## Tier 2

Two changes from the [reference workflow]({{% relref "../reference-workflows" %}}):

- Set the repository variable `LIFECYCLE_SCAN_TARGETS` to `scan-input/image.tar`, so that the scan target is the single archive.
- Raise `LIFECYCLE_SCAN_MAX_BYTES`. Images are routinely hundreds of megabytes to gigabytes, and the default of 512 MB will reject many.

An image archive is large and contains whole filesystems, so also see [the remaining exposure]({{% relref ".." %}}#the-remaining-exposure).

## Trusted-ref evaluation

Build the image from the real checkout, save it, and scan the archive at `stage: build` on `main` and `stage: release` for a release. A Project that publishes the image should scan the same image it publishes.

## What you give up

- **Time and storage.** An image artifact is expensive to upload and download. Keep `retention-days` short, and consider building a smaller image for the scan if your Dockerfile has a multi-stage build with a test stage.
- **Base image drift.** The result reflects the base image as pulled at build time, which can change between runs.
- **Platforms.** `docker save` of a multi-platform build saves one platform. Choose the one you ship, or scan each.
- **Advisory only at PR time**, as for every ecosystem. In particular, a Dockerfile change that adds a layer is exactly what a hostile Pull Request might try, and the image contents are what get scanned, which is a mitigating factor.
