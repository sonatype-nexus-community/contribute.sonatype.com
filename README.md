# Sonatype Open Source Community Handbook?

<!-- Badges Section -->
[![shield_gh-workflow-test]][link_gh-workflow-test]
[![shield_license]][license_file]
<!-- Add other badges or shields as appropriate -->

---

This repository contains the source for our Sonatype Open Source Community Handbook - the published version can be viewed [here](https://sonatype-nexus-community.github.io/community-handbook.sonatype.com/).

This project is based from [Docsy for Hugo](https://www.docsy.dev/docs/get-started/docsy-as-module/installation-prerequisites/#install-hugo).

- [Usage](#usage)
  - [Prerequisite Tools](#prerequisite-tools)
  - [Running the website locally](#running-the-website-locally)
- [Development](#development)
- [The Fine Print](#the-fine-print)

## Usage

### Prerequisite Tools

You'll need the following tools installed locally for development:
- [Hugo Extended](https://gohugo.io) (>=0.128.0)
- NPM

You can find out more about how to install Hugo for your environment in our
[Getting started](https://www.docsy.dev/docs/getting-started/#prerequisites-and-installation) guide.

### Running the website locally

Ensure `npm install` has been run - this ensures `PostCSS` is installed.

Once you've made your working copy of the site repo, from the repo root folder, run:

```bash
hugo server
```

###Running a container locally

You can run docsy-example inside a [Docker](https://docs.docker.com/)
container, the container runs with a volume bound to the `docsy-example`
folder. This approach doesn't require you to install any dependencies other
than [Docker Desktop](https://www.docker.com/products/docker-desktop) on
Windows and Mac, and [Docker Compose](https://docs.docker.com/compose/install/)
on Linux.

1. Build the docker image

   ```bash
   docker-compose build
   ```

1. Run the built image

   ```bash
   docker-compose up
   ```

   > NOTE: You can run both commands at once with `docker-compose up --build`.

1. Verify that the service is working.

   Open your web browser and type `http://localhost:1313` in your navigation bar,
   This opens a local instance of the docsy-example homepage. You can now make
   changes to the docsy example and those changes will immediately show up in your
   browser after you save.

## Development

See [CONTRIBUTING.md](./CONTRIBUTING.md) for details.

## The Fine Print

Remember:

This project is part of the [Sonatype Nexus Community](https://github.com/sonatype-nexus-community) organization, which is not officially supported by Sonatype. Please review the latest pull requests, issues, and commits to understand this project's readiness for contribution and use.

* File suggestions and requests on this repo through GitHub Issues, so that the community can pitch in
* Use or contribute to this project according to your organization's policies and your own risk tolerance
* Don't file Sonatype support tickets related to this project— it won't reach the right people that way

Last but not least of all - have fun!

<!-- Links Section -->
[shield_gh-workflow-test]: https://img.shields.io/github/actions/workflow/status/sonatype-nexus-community/community-handbook.sonatype.com/ci.yaml?branch=main&logo=GitHub&logoColor=white "build"
[shield_license]: https://img.shields.io/github/license/sonatype-nexus-community/community-handbook.sonatype.com?logo=open%20source%20initiative&logoColor=white "license"

[link_gh-workflow-test]: https://github.com/sonatype-nexus-community/community-handbook.sonatype.com/actions/workflows/ci.yaml?query=branch%3Amain
[license_file]: https://github.com/sonatype-nexus-community/community-handbook.sonatype.com/blob/main/LICENSE
