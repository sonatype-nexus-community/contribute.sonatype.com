---
title: .NET
description: Preparing the Lifecycle scan input for NuGet projects.
weight: 10
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end. No current community Project uses .NET, so this guide is the least exercised. Source: Sonatype's [NuGet Application Analysis](https://help.sonatype.com/en/nuget-application-analysis.html).
{{% /alert %}}

## What Lifecycle reads

Sonatype's documentation says Lifecycle scans `.nupkg` files and PE binaries (`.dll`, `.exe` and others), as a **binary** scan, and project files (`.csproj`, `packages.config`) as a **manifest** scan. In manifest mode "only components with an exact version specified are evaluated", floating versions such as `3.6.*` are not supported, and transitive dependencies are not included.

Running `dotnet restore` is not required, but it is recommended for a more complete analysis. Restoring into a directory (`dotnet restore --packages packages`) gives a folder of packages to scan.

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

      - name: Set up .NET
        uses: actions/setup-dotnet@a98b56852c35b8e3190ac28c8c2271da59106c68 # v6.0.0
        with:
          global-json-file: global.json

      - name: Restore packages
        run: |
          mkdir scan-input
          dotnet restore --packages scan-input/packages
          find . -name '*.csproj' -not -path './scan-input/*' -exec cp --parents {} scan-input/ \;

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

`dotnet restore` can run MSBuild targets from the project, which is code from the Pull Request. That is acceptable in Tier 1 and is why it must not run in Tier 2.

## Tier 2

Unchanged from the [reference workflow]({{% relref "../reference-workflows" %}}).

## Trusted-ref evaluation

Restore and scan the same way against the real checkout, at `stage: build` on `main` and `stage: release` for a release. Scanning the published output (`dotnet publish`) is the closest equivalent of the Java "deployed artifact" advice, but Sonatype's page does not state this for .NET.

## What you give up

- **Manifest mode skips transitive dependencies.** Restoring into a package folder avoids this.
- **Floating versions** are not evaluated. Pin exact versions.
- **`.msi` files** are not supported.
- **Advisory only at PR time**, as for every ecosystem.
