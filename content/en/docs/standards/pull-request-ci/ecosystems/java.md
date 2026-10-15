---
title: Java
description: Preparing the Lifecycle scan input for Maven and Gradle projects.
weight: 50
---

{{% alert color="warning" %}}
**DRAFT.** Not yet run end to end. Source: Sonatype's [Java Application Analysis](https://help.sonatype.com/en/java-application-analysis.html). Gradle guidance is thinner than Maven, and this guide says where.
{{% /alert %}}

## What Lifecycle reads

Java is a **binary** ecosystem. Sonatype's documentation says Lifecycle scans packaged archives (`.ear`, `.war`, `.jar`, `.zip`, `.tar.gz`) and post-build artifacts, using Advanced Binary Fingerprinting, and that "for the best results" the post-build artifact you plan to deploy should be the scan target.

It can also read `pom.xml` and `build.gradle`, but that is a **manifest** scan with a real limit: "transitive dependencies are not automatically resolved and, therefore not included". It needs the group, artifact and exact version of each direct dependency.

So for an accurate Pull Request scan, the scan input should be the built output. That means the build runs in Tier 1, where running a build is safe.

## Tier 1 job: Maven

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

      - name: Set up Java
        uses: actions/setup-java@de7274f081f381c8f8158605e0321c36c376e2e6 # v6.0.1
        with:
          distribution: temurin
          java-version-file: .java-version # or set java-version: explicitly
          cache: maven

      - name: Build
        run: ./mvnw --batch-mode -DskipTests package

      - name: Gather scan input
        run: |
          mkdir scan-input
          cp -r target/*.jar scan-input/ 2>/dev/null || true
          cp -r target/*.war scan-input/ 2>/dev/null || true
          # A thin jar does not contain its dependencies. Copy them alongside.
          ./mvnw --batch-mode dependency:copy-dependencies -DoutputDirectory=scan-input/dependencies

      - name: Upload scan input
        uses: actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.0.2
        with:
          name: lifecycle-scan-input
          path: scan-input/
          if-no-files-found: error
          retention-days: 3
```

Notes:

- `mvn dependency:copy-dependencies` is the command Sonatype's documentation gives for thin jars.
- An **uber or shaded jar** already contains its dependencies, so the `dependency:copy-dependencies` step is not needed. Sonatype's documentation says to "call the index goal of the Maven plugin at the time of building the uber/shaded jar". Add that to the build step if you produce one.
- `-DskipTests` is for speed here. Your tests run in the `build-and-test` job.
- Use the project's own wrapper (`./mvnw`), so the Maven version is the one the Project uses. A Maven wrapper downloads Maven, which is acceptable in Tier 1 and is why this must not run in Tier 2.

## Tier 1 job: Gradle

```yaml
      - name: Build
        run: ./gradlew --no-daemon assemble

      - name: Gather scan input
        run: |
          mkdir scan-input
          cp -r build/libs/. scan-input/
```

{{% alert color="info" %}}
**Not documented:** Sonatype's Java page names `build/libs` only indirectly and gives no Gradle equivalent of Maven's `dependency:copy-dependencies`. For a Gradle project that produces a thin jar, the dependencies are not in `build/libs`. Until Sonatype documents a recommended approach, a Gradle Project has two choices, and should say which it uses:

1. Build a fat jar (for example with the Shadow plugin) and scan that.
2. Scan `build.gradle` as a manifest, accepting the loss of transitive dependencies. Sonatype's documentation lists the supported scopes (`api`, `apiElements`, `compileClasspath`, `implementation`, `compileOnly`, `compileOnlyApi`, `compile`) and says only Groovy `build.gradle` files are supported, so a Kotlin DSL build is not covered.

The Gradle plugin Project in this community (`scan-gradle-plugin`) is itself a Gradle plugin and its Pull Requests are a good test case for deciding this.
{{% /alert %}}

## Tier 2

Mostly unchanged from the [reference workflow]({{% relref "../reference-workflows" %}}). Two adjustments:

- Binary inputs are larger than lockfiles. Raise `LIFECYCLE_SCAN_MAX_BYTES` if the default of 512 MB is too small, and keep it as low as is workable.

## Trusted-ref evaluation

Build the project, then scan the same output (`scan-targets: 'target'` or `'build/libs'`) at `stage: build` on `main` and `stage: release` for a release. Do not scan `.`, because the source tree is a weaker target than the deployed artifact.

## What you give up

- **Test and provided scope.** `dependency:copy-dependencies` copies test-scope dependencies by default. To scan only what ships, add `-DincludeScope=runtime`. Whether to include test dependencies is a policy decision for the Project.
- **A build that fails.** If the Pull Request does not compile, there is no artifact, so the Pull Request gets no Lifecycle result until it builds. This is consistent with Tier 2 only running after Tier 1 succeeds.
- **Advisory only at PR time.** A Pull Request could build an artifact that omits a dependency. Maintainers reviewing `pom.xml` and `build.gradle` changes is the control for that, and the trusted-ref evaluation is the backstop.

## Nexus plugin Projects

Nexus Repository plugins (for example the CasC plugin) are built as bundles that run inside Nexus Repository. The bundle is the deployed artifact, so scan it. The Nexus platform itself is provided by the host at runtime and is not in the bundle. Whether it should appear in the scan is a policy decision for the Project.
