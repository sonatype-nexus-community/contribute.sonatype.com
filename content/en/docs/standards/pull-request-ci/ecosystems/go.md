---
title: Go
description: Preparing the Lifecycle scan input for Go modules, including Terraform providers.
weight: 10
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end. Source: Sonatype's [Go Application Analysis](https://help.sonatype.com/en/go-application-analysis.html).
{{% /alert %}}

## What Lifecycle reads

Go is a **manifest** ecosystem. Lifecycle matches Go components by module path and version. Sonatype's documentation says:

- A valid `go.mod` is required for Go module analysis.
- The **recommended** scan target is a `go.list` file generated with `go list`.
- `go.sum` is also supported, but "may include dependencies that are not used by the application", so it can report more than your build actually uses.
- The Go toolchain does not need to be present when Lifecycle scans the file.

## Why `go.list` and not `go.sum`

`go.sum` records every module version your module graph has ever needed, including ones the build does not use. `go.list` is produced from what the compiler would actually link, so it matches your real dependencies. The tradeoff is that producing it needs the Go toolchain and module downloads, which is why it happens in Tier 1.

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

      - name: Set up Go
        uses: actions/setup-go@b7ad1dad31e06c5925ef5d2fc7ad053ef454303e # v7.0.0
        with:
          go-version-file: go.mod

      - name: Generate go.list
        run: |
          mkdir scan-input
          go list -f '{{define "M"}}{{.Path}} {{.Version}}{{end}}{{with .Module}}
          {{if not .Main}}{{if .Replace}}{{template "M" .Replace}}
          {{else}}{{template "M" .}}{{end}}{{end}}{{end}}' -compiled -deps ./... | sort -u > scan-input/go.list
          cp go.mod scan-input/go.mod

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

The `go list` template is Sonatype's, verbatim. Two things are ours: the `./...` package pattern, which the Sonatype page does not show (its command ends at `-deps`), and copying `go.mod` next to the list, because the page says a valid `go.mod` is required.

{{% alert color="info" %}}
**To confirm with Sonatype:** the exact package pattern the `go list` command should use. We use `./...` so the whole module is covered. A Project with several modules (a `go.work` workspace) needs a `go list` per module.
{{% /alert %}}

## Tier 2

No change from the [reference workflow]({{% relref "../reference-workflows" %}}). The default `scan-targets` of `scan-input` finds `go.list` and `go.mod`.

## Trusted-ref evaluation

Scan the real checkout. Run the same `go list` step, and use `scan-targets: 'go.list go.mod'` or `'.'`.

## What you give up

- **Replace directives and local modules.** The template substitutes a `replace` target's path and version, which is what you want for a module replacement. A `replace` pointing at a local directory has no version, so it cannot be matched. Reviewers should treat new `replace` directives in a Pull Request as a signal to look closely.
- **Versions Lifecycle cannot match.** Sonatype's documentation lists these: `+incompatible` versions "may not be identified and can appear as Unknown"; pseudo-versions have "partial support"; arbitrary pre-release versions "are not supported".
- **Build tags.** `-compiled -deps` follows the build configuration the job runs with. Code behind other build tags or other operating systems is not listed. If you ship for several platforms, run `go list` for each (`GOOS`, `GOARCH`) and combine the results with `sort -u`.
- **Advisory only at PR time.** As for every ecosystem, the contributor produces this file, so the Pull Request result is advisory. See [What a PR-time result means]({{% relref ".." %}}#what-a-pr-time-result-means).

## Terraform providers

`terraform-provider-sonatyperepo` and `terraform-provider-sonatypeiq` are Go modules, so this guide applies unchanged. The provider's `docs/` and example Terraform files are not dependencies and are not scanned.
