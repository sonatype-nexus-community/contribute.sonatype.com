---
title: Running GitHub Actions Locally
# description: What does your user need to know to try your project?
# categories: [Examples, Placeholders]
# tags: [test, docs]
weight: 10
---

You can locally test the GitHub Actions defined in this your project using [nektos act](https://github.com/nektos/act).



## Setup

{{% alert color="info" %}}
The first time you run [act](https://github.com/nektos/act), it can take a long time (with no output) to download
the various Docker goodies. Give it time before deciding it is stuck.
{{% /alert %}}

Simply run:
```bash
act
```

### Apple Silcon

If running on Apple Silicon (ARM), launch `act` with this flag:
```bash
   act --container-architecture linux/amd64
```

Without this flag, I saw this warning:
```bash
WARN  ⚠ You are using Apple M-series chip and you have not specified container architecture, you might encounter issues while running act. If so, try running it with '--container-architecture linux/amd64'. ⚠
```

and this error:
```bash
...   🐳  docker exec cmd=[bash --noprofile --norc -e -o pipefail /var/run/act/workflow/1] user= workdir=
| docker compose build
[+] Building 0.0s (0/0)                                                         
| permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied
...
```

## Usage

To get a list of available jobs, run:
```bash
    $ act -l
```

To run a specific job, use the `-j` flag:
```bash
    $ act -j <job-name>
```

For example, to run the `build` job from the `ci.yml` file, use this command:
```bash
    $ act --workflows .github/workflows/ci.yml -j build
```