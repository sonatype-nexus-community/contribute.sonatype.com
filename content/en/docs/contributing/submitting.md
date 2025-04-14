---
title: Submitting a Contribution
# description: What does your user need to know to try your project?
# categories: [Examples, Placeholders]
# tags: [test, docs]
weight: 10
---

We welcome contributions via Pull Requests to our Community Projects.

Please ensure they meet these guidelines:

1. Has a clear and singular purpose
2. It is backed by one or more GitHub Issues in the project
3. Has appropriate test coverage for the Pull Requests purpose (if you add or modify functionality, make sure you add tests for this!)
4. Meet the Projects Code Style Convention and Contribution Guidelines (see `CONTRIBUTING.md` in the specific Project)

If you haven't yet, please review the requirements for [contributors](../community_roles/contributor.md).

{{< alert color="warning" >}}Contributions that don't meet the above requirements unfortunately cannot be accepted.{{< /alert >}}

## Contribution Requirements

### Signed Commits

In order to help verify the authenticity of contributed code, we ask that your [commits be signed](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits). All commits must be signed off to show that you agree to publish your changes under the current terms and licenses of the project.
  
Here are some notes we found helpful in configuring a local environment to automatically sign git commits:
  - [GPG commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification#gpg-commit-signature-verification)
  - [Telling Git about your GPG key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key#telling-git-about-your-gpg-key)

If you forgot to sign your commits before pushing them, you can retroactively sign all commits on your current branch by running:
```
git rebase --root -S --committer-date-is-author-date
```

You can then re-push your branch and all commits should be signed.