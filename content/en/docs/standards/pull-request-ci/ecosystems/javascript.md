---
title: JavaScript
description: Preparing the Lifecycle scan input for npm and Yarn projects.
weight: 30
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end. Sources: Sonatype's [npm Application Analysis](https://help.sonatype.com/en/npm-application-analysis.html) and [Preparing to Run npm Application Analysis](https://help.sonatype.com/en/preparing-to-run-npm-application-analysis.html).
{{% /alert %}}

## What Lifecycle reads

JavaScript can be scanned two ways, and the choice sets the accuracy:

| Scan input | Mode | Accuracy |
|---|---|---|
| `package.json` plus a lockfile (`package-lock.json`, `npm-shrinkwrap.json`, `yarn.lock`, `pnpm-lock.yaml`) | **Manifest** matching | Good. Evaluates dependencies from metadata only. |
| `package.json`, the lockfile **and** `node_modules` | **Binary** (ABF) | Best. Sonatype recommends this for "the most precise results". |

Sonatype's documentation says that when JavaScript files are included, ABF is used and the JavaScript files are the primary identification source, and that "manifest entries without corresponding JavaScript files are ignored". `package.json` must come with a `package-lock.json`.

The binary mode needs `node_modules` from "a clean production flag build".

## Which should a Project choose

- **Manifest** is small, fast and cheap to move between jobs. Choose it as the default for Pull Requests.
- **Binary** matters when you ship or bundle `node_modules`, or need to detect modified copies of packages. It is large. Choose it for the trusted-ref evaluation and, if your Project wants it, for Pull Requests too.

## Tier 1 job: manifest

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
          cp package.json scan-input/
          # Use whichever lockfile your Project has.
          cp package-lock.json scan-input/

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

This job does not run `npm`, so it runs nothing from the Pull Request at all. It is the simplest and safest way to prepare a scan input.

For a monorepo or workspace, copy each package's `package.json` into a matching subdirectory of `scan-input`, so that each lockfile stays next to its manifest.

## Tier 1 job: binary

```yaml
      - name: Set up Node
        uses: actions/setup-node@949feb2413d6458794dcd2491c4babbbce0c15c1 # v7.1.0
        with:
          node-version-file: .nvmrc

      - name: Install production dependencies
        run: npm install --production
        # For Yarn: yarn install --prod

      - name: Gather scan input
        run: |
          mkdir scan-input
          cp package.json package-lock.json scan-input/
          cp -r node_modules scan-input/node_modules
```

`npm install --production` is the command Sonatype's preparation page gives, as is `yarn install --prod`. Use these exactly and not `npm ci`, until Sonatype says the others are equivalent.

{{% alert color="warning" %}}
**Install scripts run here.** `npm install` runs package install scripts. In Tier 1 that is acceptable, because Tier 1 has no secrets. It is the reason this step must never move into Tier 2.
{{% /alert %}}

## Tier 2

Unchanged from the [reference workflow]({{% relref "../reference-workflows" %}}) for the manifest mode. For the binary mode, `node_modules` is large and has many files. Raise `LIFECYCLE_SCAN_MAX_BYTES` and `LIFECYCLE_SCAN_MAX_FILES` for this Project, and note that the artifact upload and download will be slow.

## Trusted-ref evaluation

Run `npm install --production` on the real checkout, then scan `package.json`, the lockfile and `node_modules` together, at `stage: release`.

## What you give up

- **Manifest mode** cannot detect a package whose files were altered, and relies on the lockfile being honest. A lockfile that a Pull Request changed deserves a reviewer's attention.
- **Development dependencies.** The production flag means they are excluded in binary mode. If your policy covers build-time dependencies, scan them separately.
- **Name matching sunset.** Sonatype's documentation notes that matching unmatched JavaScript files by name ended on 2025-09-10. Only files that match known hashes are counted.
- **Advisory only at PR time**, as for every ecosystem.
