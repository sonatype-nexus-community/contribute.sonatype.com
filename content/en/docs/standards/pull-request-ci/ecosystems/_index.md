---
title: Ecosystem Guides
description: For each language, what to prepare for the Lifecycle scan, and the Tier 1 job that prepares it.
weight: 20
---

{{% alert color="warning" %}}
**DRAFT.** These guides are drawn from Sonatype's published [analysis documentation](https://help.sonatype.com/en/analysis.html). The snippets have **not** been run end to end, and where Sonatype's documentation is silent the guide says so, rather than guessing.
{{% /alert %}}

## What these guides are

Under the [Pull Request CI & Secrets]({{% relref ".." %}}) standard, a Tier 1 job (no secrets) prepares a **scan input**, and a separate Tier 2 job (with secrets) hands it to Sonatype Lifecycle. Which files Lifecycle can accurately evaluate depends on the language. These guides record, for each one:

- **What Lifecycle reads**, and so what the scan input must contain.
- **A `prepare-scan-input` job** to paste into your `ci.yaml` (see [Reference Workflows]({{% relref "../reference-workflows" %}})).
- **What accuracy you give up** compared with scanning a full build, so that nobody is surprised.

The Tier 2 workflow is the same for every language. Only the preparation differs.

## Two kinds of ecosystem

| Kind | Examples | What the scan input is | Accuracy |
|---|---|---|---|
| **Manifest** | Go, Rust, PHP, Ruby, Python, .NET, R | A lockfile or a dependency list, produced by the language's own tooling | Good, if the lockfile or list pins exact versions. Transitive dependencies depend on what the file contains. |
| **Binary** | Java, npm with `node_modules`, Docker | The built artifact, or the installed dependencies | Best. Lifecycle fingerprints the files themselves, which also finds modified or repackaged dependencies. |

Binary inputs are large. Allow for that in the artifact limits (see [Reference Workflows]({{% relref "../reference-workflows" %}}#repository-configuration)).

## The guides

| Ecosystem | Used by | Guide |
|---|---|---|
| Go | Nancy, Cheque, Terraform providers, API clients | [Go]({{% relref "go" %}}) |
| Java (Maven, Gradle) | Sherlock Trunks, Nexus CasC Plugin, API clients | [Java]({{% relref "java" %}}) |
| JavaScript (npm, Yarn) | AuditJS, API clients | [JavaScript]({{% relref "javascript" %}}) |
| Python | Jake, API clients | [Python]({{% relref "python" %}}) |
| .NET (NuGet) | No current Project | [.NET]({{% relref "dotnet" %}}) |
| Rust (Cargo) | Cargo Pants | [Rust]({{% relref "rust" %}}) |
| Ruby (Bundler) | Chelsea | [Ruby]({{% relref "ruby" %}}) |
| PHP (Composer) | Bach | [PHP]({{% relref "php" %}}) |
| R (CRAN) | OysterR | [R]({{% relref "r" %}}) |
| C/C++ (Conan) | No current Project | [C/C++]({{% relref "cpp" %}}) |
| Docker images | Container image Projects | [Docker]({{% relref "docker" %}}) |

Terraform providers are written in Go, so their Lifecycle scan is the Go guide.

Not yet covered: Dart and Flutter, Swift, Objective-C, Conda, RPM, and Hugging Face. Sonatype supports these. If your Project needs one of them, or uses a tool or ecosystem not listed at all, see [Your tool or ecosystem is not covered]({{% relref ".." %}}#your-tool-or-ecosystem-is-not-covered).

## Adding a guide

Each guide follows the same shape so they can be compared: what Lifecycle reads, the `prepare-scan-input` job, what you give up, and a link to the Sonatype page it came from. Quote Sonatype's documentation, and say "not documented" where it is silent.
