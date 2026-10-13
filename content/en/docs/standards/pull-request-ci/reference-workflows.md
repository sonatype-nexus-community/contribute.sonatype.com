---
title: Reference Workflows
description: The complete Tier 1 and Tier 2 workflow pair, and the trusted-ref evaluation, that the ecosystem guides build on.
weight: 10
---

{{% alert color="warning" %}}
**DRAFT.** These workflows implement the [Pull Request CI & Secrets]({{% relref "_index.md" %}}) standard. Every snippet has been checked to parse as YAML. They have **not** been linted with actionlint or zizmor, and **have not been run end to end in GitHub Actions**. Treat them as a design to review, not as tested code.
{{% /alert %}}

## Contents

- [How the pieces fit](#how-the-pieces-fit)
- [Tier 1: the CI workflow](#tier-1-the-ci-workflow)
- [Tier 2: the Lifecycle evaluation workflow](#tier-2-the-lifecycle-evaluation-workflow)
- [Trusted refs: the authoritative evaluation](#trusted-refs-the-authoritative-evaluation)
- [Repository configuration](#repository-configuration)
- [Why each part is the way it is](#why-each-part-is-the-way-it-is)

## How the pieces fit

| File | Trigger | Has secrets? | Runs PR code? | Job |
|---|---|---|---|---|
| `.github/workflows/ci.yaml` | `pull_request`, `push` to `main` | No | Yes | Build, test, and produce the scan input |
| `.github/workflows/lifecycle-evaluate.yaml` | `workflow_run` of `CI` | Yes | **No** | Evaluate the scan input and report |
| `.github/workflows/lifecycle-release.yaml` | `push` to `main` or release tags | Yes | Only reviewed, merged code | Authoritative evaluation |

The name of the first workflow (`CI`) is referenced by the second, so the two must agree.

Pick your language from the [ecosystem guides]({{% relref "ecosystems" %}}) and put its `prepare-scan-input` job in `ci.yaml`. Nothing else in these workflows changes between languages, except where a guide says to set a repository variable.

## Tier 1: the CI workflow

This is the skeleton. The `build-and-test` job stands for your Project's own build and tests. The `prepare-scan-input` job comes from the [ecosystem guide]({{% relref "ecosystems" %}}); the Go version is shown as a worked example.

```yaml
# .github/workflows/ci.yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

# No permissions by default. Each job asks only for what it needs.
permissions: {}

jobs:
  build-and-test:
    name: Build and test
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          persist-credentials: false
      # ... your build, lint and unprotected tests here ...

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

      # --- Ecosystem-specific: replace this block with your guide's steps ---
      - name: Set up Go
        uses: actions/setup-go@b7ad1dad31e06c5925ef5d2fc7ad053ef454303e # v7.0.0
        with:
          go-version-file: go.mod
      - name: Prepare scan input
        run: |
          mkdir scan-input
          go list -f '{{define "M"}}{{.Path}} {{.Version}}{{end}}{{with .Module}}
          {{if not .Main}}{{if .Replace}}{{template "M" .Replace}}
          {{else}}{{template "M" .}}{{end}}{{end}}{{end}}' -compiled -deps ./... | sort -u > scan-input/go.list
          cp go.mod scan-input/
      # --- End ecosystem-specific ---

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

Requirements for the `prepare-scan-input` job, whatever the language:

- It **must** have no secrets and no write permissions.
- It **must** put everything it wants evaluated in a directory named `scan-input`, and upload it as an artifact named `lifecycle-scan-input`.
- It **must not** put anything in the artifact that is not a file Lifecycle reads for that ecosystem.
- It **must** fail if there is nothing to upload (`if-no-files-found: error`), so a broken preparation step is visible.

## Tier 2: the Lifecycle evaluation workflow

This workflow runs with secrets, so it is the sensitive part of the design. It never checks out the Pull Request, and it never runs anything it downloads.

```yaml
# .github/workflows/lifecycle-evaluate.yaml
name: Lifecycle Evaluate

on:
  workflow_run:
    workflows: [CI]
    types: [completed]

permissions: {}

jobs:
  evaluate:
    name: Evaluate scan input
    # Only for pull requests, and only when Tier 1 succeeded. Trusted refs are
    # handled by lifecycle-release.yaml instead.
    if: >-
      github.event.workflow_run.event == 'pull_request' &&
      github.event.workflow_run.conclusion == 'success'
    runs-on: ubuntu-latest
    timeout-minutes: 20
    permissions:
      actions: read # download the artifact from another run
      checks: write # report the result on the pull request
    env:
      # Limits on the downloaded data. Override with repository variables.
      MAX_BYTES: ${{ vars.LIFECYCLE_SCAN_MAX_BYTES || 536870912 }}
      MAX_FILES: ${{ vars.LIFECYCLE_SCAN_MAX_FILES || 50000 }}
    steps:
      - name: Download scan input (as data)
        uses: actions/download-artifact@9000827ccba6bdab643e8b6fd33ac0654aef8333 # v8.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input
          run-id: ${{ github.event.workflow_run.id }}
          github-token: ${{ github.token }}

      - name: Validate scan input
        run: |
          set -euo pipefail
          bytes=$(du -sb scan-input | cut -f1)
          files=$(find scan-input -type f | wc -l)
          if [ "$bytes" -gt "$MAX_BYTES" ]; then
            echo "::error::Scan input is $bytes bytes; the limit is $MAX_BYTES"
            exit 1
          fi
          if [ "$files" -gt "$MAX_FILES" ]; then
            echo "::error::Scan input has $files files; the limit is $MAX_FILES"
            exit 1
          fi
          if [ -n "$(find scan-input ! -type f ! -type d -print -quit)" ]; then
            echo "::error::Scan input contains something other than regular files and directories"
            exit 1
          fi

      - name: Sonatype Lifecycle Evaluation
        id: evaluate
        # Do not fail here, so that the result is always reported below.
        continue-on-error: true
        uses: sonatype/actions/evaluate@b1c823a70e98d3c67478cd72c0128712623455b9 # v1.14.1
        with:
          iq-server-url: ${{ vars.SONATYPE_PLATFORM_URL }}
          username: ${{ secrets.SONATYPE_LIFECYCLE_USERNAME }}
          password: ${{ secrets.SONATYPE_LIFECYCLE_PASSWORD }}
          # Chosen here, from trusted values. Never read from the artifact.
          application-id: ${{ github.event.repository.name }}
          scan-targets: ${{ vars.LIFECYCLE_SCAN_TARGETS || 'scan-input' }}
          stage: develop # Pull Requests use the develop stage, see the standard

      - name: Report result on the pull request
        if: ${{ !cancelled() }}
        uses: actions/github-script@3a2844b7e9c422d3c10d287c895573f7108da1b3 # v9.0.0
        env:
          HEAD_SHA: ${{ github.event.workflow_run.head_sha }}
          EVALUATE_OUTCOME: ${{ steps.evaluate.outcome }}
          REPORT_URL: ${{ steps.evaluate.outputs.report-url }}
        with:
          script: |
            const outcome = process.env.EVALUATE_OUTCOME;
            const passed = outcome === 'success';
            const summaries = {
              success: 'The evaluation passed.',
              skipped: 'The scan input was missing or was rejected by validation, so nothing was evaluated.',
            };
            await github.rest.checks.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              name: 'Sonatype Lifecycle (pull request, advisory)',
              head_sha: process.env.HEAD_SHA,
              status: 'completed',
              conclusion: passed ? 'success' : 'failure',
              details_url: process.env.REPORT_URL || undefined,
              output: {
                title: passed ? 'Passed' : 'Needs attention',
                summary: summaries[outcome] ??
                  'The evaluation reported policy violations or did not complete. Open the report for details.',
              },
            });
```

### What Tier 2 deliberately does not do

- It has **no `actions/checkout`**. The only thing from the Pull Request on the runner is the downloaded artifact.
- It does not read the Pull Request number, branch name, title or any file contents to decide what to do. It uses `head_sha`, which GitHub sets, and the repository name, which is the Project's own.
- Tier 1 must have **succeeded** before Tier 2 runs. A contributor whose tests are failing fixes them first. A Project that wants a Lifecycle result even when tests fail can run the `prepare-scan-input` job in a separate workflow, and point `workflows:` at that one.

## Trusted refs: the authoritative evaluation

On a trusted ref (see [Trusted refs, releases and tags]({{% relref "_index.md" %}}#trusted-refs-releases-and-tags)) the code is reviewed and merged, so the Project can scan the real checkout directly, with secrets, as described in [Dependency Management]({{% relref "/docs/standards/dependency-management" %}}). This evaluation, and not the advisory Pull Request one, is what enforces policy.

```yaml
# .github/workflows/lifecycle-release.yaml
name: Lifecycle Release Evaluation

on:
  push:
    branches: [main]
    tags: ['v*'] # adjust to your release convention

permissions: {}

jobs:
  evaluate:
    name: Evaluate (build on main, release on a tag)
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          persist-credentials: false

      # ... build the project here if your ecosystem is scanned from build output ...

      - name: Sonatype Lifecycle Evaluation
        uses: sonatype/actions/evaluate@b1c823a70e98d3c67478cd72c0128712623455b9 # v1.14.1
        with:
          iq-server-url: ${{ vars.SONATYPE_PLATFORM_URL }}
          username: ${{ secrets.SONATYPE_LIFECYCLE_USERNAME }}
          password: ${{ secrets.SONATYPE_LIFECYCLE_PASSWORD }}
          application-id: ${{ github.event.repository.name }}
          scan-targets: '.' # or the build output, per the ecosystem guide
          # Tags are releases; a push to main is CI. See "Which Lifecycle stage to use".
          stage: ${{ startsWith(github.ref, 'refs/tags/') && 'release' || 'build' }}
```

This workflow is only safe if the refs that trigger it are protected. A branch ruleset on `main` and a **tag ruleset** on the release tags are required, as described in the standard. If a Project releases some other way (a manual dispatch, a published GitHub Release), change the `on:` block to match and keep the protection requirement.

## Repository configuration

For these workflows to work as designed, the repository needs:

| Setting | Value | Why |
|---|---|---|
| Variable `SONATYPE_PLATFORM_URL` | The Lifecycle URL | Not secret, but only Tier 2 and the release workflow read it |
| Secrets `SONATYPE_LIFECYCLE_USERNAME` and `SONATYPE_LIFECYCLE_PASSWORD` | A dedicated, evaluate-only account | Least privilege; see the standard |
| Where those secrets live | Repository or Environment scope, not organization | A mistake then affects one repository |
| Actions setting: send secrets to fork PR workflows | Disabled | Tier 1 must not receive them |
| Actions setting: approval for fork PR workflows | Required for first-time contributors | Limits abuse of runner time |
| Optional variable `LIFECYCLE_SCAN_TARGETS` | A path under `scan-input` | For ecosystems whose target is a single file (for example Docker) |
| Optional variables `LIFECYCLE_SCAN_MAX_BYTES`, `LIFECYCLE_SCAN_MAX_FILES` | Larger limits | For ecosystems with big inputs (for example Docker, `node_modules`) |

The `lifecycle-evaluate.yaml` workflow must be on the default branch before it takes effect, and changes to it, to `ci.yaml`'s `prepare-scan-input` job, and to anything under `.github/` should be listed in `CODEOWNERS`.

## Why each part is the way it is

| Choice | Reason |
|---|---|
| Two workflows, not one | The first runs contributor code and has no secrets; the second has secrets and runs none. This is the whole design. |
| `workflow_run`, not `pull_request_target` | `workflow_run` runs the default branch's copy of the workflow and never checks out the Pull Request by default, so there is no code from the PR to run. |
| `permissions: {}` at the top | Nothing gets a permission it did not ask for. |
| Actions pinned to a commit SHA | A tag can be moved; a commit cannot. The version is kept in a trailing comment so updates are easy to review. |
| `persist-credentials: false` | Checkout does not leave the token in `.git` where later steps could read it. |
| Limits and the file-type check in Tier 2 | The downloaded data is untrusted. A very large or oddly-shaped input is rejected before anything parses it. |
| `continue-on-error` then a check run | A failed evaluation is still reported on the PR. Without the check run, a `workflow_run` result is attached to the default branch and the contributor never sees it. |
| `application-id` from the repository name | It comes from a value the contributor cannot change, and it matches the existing [Dependency Management]({{% relref "/docs/standards/dependency-management" %}}) convention. |
| Trusted-ref workflow separate from Tier 2 | The advisory and authoritative results are different things and should not share a name, a trigger or a failure mode. |
