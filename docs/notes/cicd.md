---
draft: false
date:
  created: 2025-01-20
  updated: 2025-06-15
categories:
  - how-to
  - automation
  - git
  - cicd
  - code
  - lint
---

# :fontawesome-solid-gears: CI/CD

!!! abstract "Continuous Integration, Continuous Deployment"

    Improve the quality and security of your code using CI/CD workflows. This is best summarized in [GitHub's Quickstart for Securing Repos](https://docs.github.com/en/code-security/getting-started/quickstart-for-securing-your-repository).


<!-- more -->

## Dependabot

The best way to understand this is to walk through the [quickstart-guide](https://docs.github.com/en/code-security/getting-started/dependabot-quickstart-guide). This has you fork the [github.com/dependabot/demo](https://github.com/dependabot/demo) repo, and run Dependabot on it by enabling it in the repo settings.

This creates `.github/dependabot.yml` and targets languages based on those it detects in the repo. When you first enable it you may be asked to manually edit the `dependabot.yml` file.

Dependabot is best suited for code that uses packages, or is built on other libraries and modules. There's less use here for static, self-contained scripting. However if you have a requirements file or version dependancies this is where it shines.


## CodeQL Analysis

!!! quote "[Code Scanning](https://docs.github.com/en/code-security/getting-started/quickstart-for-securing-your-repository#configuring-code-scanning)"

    You can configure code scanning to automatically identify vulnerabilities and errors in the code stored in your repository by using a CodeQL analysis workflow or third-party tool. Depending on the programming languages in your repository, you can configure code scanning with CodeQL using the default setup, in which GitHub automatically determines the languages to scan, query suites to run, and events that will trigger a new scan.

Unlike the Dependabot demo, code scanning does not create any files within the repo itself. Instead it runs on top of GitHub separately, and like Dependabot, posts results and alerts to the Security tab of the repo.
