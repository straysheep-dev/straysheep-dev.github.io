---
title: "Linting Code"
icon: material/code-tags-check
draft: false
#date:
#  created: 2024-12-28
#  updated: 2025-12-14
categories:
  - how-to
  - automation
  - git
  - cicd
  - code
  - lint
---

# :material-code-tags-check: Linting Code

Linting is running documented checks to statically analyze code for common mistakes and errors.

It can also be a great way to learn a new programming language as you'll be pointed to coding conventions, often in the form of the problematic code snippet, suggestions on how to refactor it, and the reasoning behind why.

Not every language has a standard linter, and some languages have multiple linters that are popular to use.

This guide is meant to get you started with linting, from "how to install" to "how to use" linters. It contains examples for both interactive CLI and automated CI/CD-focused workflows in Python, bash, PowerShell, Ansible, Packer, Terraform, with more to be added over time.

!!! abstract "Additional Resources"

    The following resources will be useful if you're getting started with linting or CI/CD.

    - [Wikipedia: Linting](https://en.wikipedia.org/wiki/Lint_(software))
    - [OWASP: Linting](https://owasp.org/www-project-devsecops-guideline/latest/01b-Linting-Code)
    - [CI/CD](https://www.redhat.com/en/topics/devops/what-is-ci-cd)
    - [VSCode](https://devblogs.microsoft.com/python/python-linting-video/)
    - [GitHub Actions](https://github.com/features/actions)

<!-- more -->

---

## actions/checkout

!!! abstract "The First Step"

    This action is core to basically every GitHub workflow, by checking out the necessary repositories using any specific arguments, all defined as a GitHub action.

    > Checkout a Git repository at a particular version

    - <https://github.com/actions/checkout>
    - <https://github.com/marketplace/actions/checkout>

**GitHub Actions**

```yaml
# https://github.com/actions/checkout?tab=readme-ov-file#checkout-multiple-repos-side-by-side
- name: Checkout
  uses: actions/checkout@v6
  with:
    path: main

- name: Checkout another repo
  uses: actions/checkout@v6
  with:
    repository: my-org/my-tools
    path: my-tools
```


## Python

!!! success "actions/setup-python"

    > Set up a specific version of Python and add the command-line tools to the PATH

    This action is used to test, build, and publish numerous official python packages. See the [Python packaging documentaiton for examples](https://packaging.python.org/en/latest/guides/publishing-package-distribution-releases-using-github-actions-ci-cd-workflows/). It has also been verified by GitHub.

    - [github.com/actions/setup-python](https://github.com/actions/setup-python)
    - [GitHub Marketplace](https://github.com/marketplace/actions/setup-python)

    [Basic usage](https://github.com/marketplace/actions/setup-python#basic-usage):

    ```yaml
    steps:
    - uses: actions/checkout@v5
    - uses: actions/setup-python@v6
      with:
        python-version: '3.13'
    - run: python my_script.py
    ```

!!! tip "Virtual Environments"

    In most cases you'll want to use a [`venv`](https://docs.python.org/3/library/venv.html) (virtual environment) or [`pipx`](https://github.com/pypa/pipx?tab=readme-ov-file#install-pipx), which achieves the same.

    - Use virtual environments for programming projects, or installing libraries
    - Use `pipx` when you're installing a python package that will run as a program would from the CLI
    - You can use either to version-pin python packages, or have multiple versions of a package installed

    See [ansible-lint](#ansible-lint) below for an example.

!!! note "`pip install --user`"

    In cases where a virtual environment or pipx isn't practical, such as an ephemeral developer VM or a system dedicated to running a specific workload, you can avoid polluting your system libraries by using the `--user` option when installing pip packages. In the worst case scenario, these only break other packages installed locally for that user, and can be removed / reinstalled with a higher chance of success than if these replaced system-level packages.


### pylint

!!! abstract "Overview"

    > Pylint is a static code analyser for Python 2 or 3. The latest version supports Python 3.10.0 and above.
    >
    > Pylint analyses your code without actually running it. It checks for errors, enforces a coding standard, looks for code smells, and can make suggestions about how the code could be refactored.

    - [Website / Documentation](https://pylint.readthedocs.io/en/stable/)
    - [GitHub](https://github.com/pylint-dev/pylint)
    - [Configuration](https://pylint.readthedocs.io/en/stable/user_guide/configuration/index.html)
    - [VS Marketplace Extension](https://marketplace.visualstudio.com/items?itemName=ms-python.pylint)

!!! tip "pylint vs flake8"

    `pylint` is generally much more strict than `flake8`.

**Install**

Install via pip, in a venv, or possibly with pipx:

```bash
python3 -m pip install pylint
```

**Usage**

This will run recursively in the current directory:

```bash
pylint .
```

**Configure**

To configure, you can auto-generate an INI file, and save it in the project root as `.pylintrc`:

```bash
pylint --disable=bare-except,invalid-name --class-rgx='[A-Z][a-z]+' --generate-rcfile | tee .pylintrc >/dev/null
```

**GitHub Actions**

```yaml
# .github/workflows/pylint.yml

# Built from the following examples:
# - https://www.shellcheck.net/wiki/GitHub-Actions
# - https://github.com/geerlingguy/ansible-role-docker/blob/master/.github/workflows/ci.yml

name: pylint
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
jobs:
  pylint:
    name: pylint
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v5
    - uses: actions/setup-python@v6
    #  with:
    #    python-version: '3.13'
    - name: Run pylint
      run: |
        python3 -m pip install pylint
        pylint .
```


### flake8

!!! abstract "Overview"

    > flake8 is a python tool that glues together pycodestyle, pyflakes, mccabe, and third-party plugins to check the style and quality of some python code.

    - [Website / Documentation](https://flake8.pycqa.org/en/latest/)
    - [GitHub](https://github.com/PyCQA/flake8)
    - [Configuration](https://flake8.pycqa.org/en/latest/user/configuration.html)
    - [VS Marketplace Extension](https://marketplace.visualstudio.com/items?itemName=ms-python.flake8)

!!! tip "pylint vs flake8"

    `pylint` is generally much more strict than `flake8`.

**Install**

Install via pip, in a venv, or possibly with pipx:

```bash
python3 -m pip install flake8
```

**Usage**

This will run recursively in the current directory:

```bash
flake8 .
```

**Configure**

To configure, create an INI file in the project root titled `.flake8`:

```plain
[flake8]
extend-ignore = E203
exclude =
    .git,
    __pycache__,
    docs/source/conf.py,
    old,
    build,
    dist
max-complexity = 10
```

**GitHub Actions**

```yaml
# .github/workflows/flake8.yml

# Built from the following examples:
# - https://www.shellcheck.net/wiki/GitHub-Actions
# - https://github.com/geerlingguy/ansible-role-docker/blob/master/.github/workflows/ci.yml

name: flake8
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
jobs:
  flake8:
    name: flake8
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v5
    - uses: actions/setup-python@v6
    #  with:
    #    python-version: '3.13'
    - name: Run flake8
      run: |
        python3 -m pip install flake8
        flake8 .
```


### mypy

!!! abstract "Overview"

    > Mypy is a static type checker for Python.
    >
    > Type checkers help ensure that you're using variables and functions in your code correctly. With mypy, add type hints (PEP 484) to your Python programs, and mypy will warn you when you use those types incorrectly.
    >
    > Python is a dynamic language, so usually you'll only see errors in your code when you attempt to run it. Mypy is a static checker, so it finds bugs in your programs without even running them!

    - [Website](https://www.mypy-lang.org/)
    - [Documentation](https://mypy.readthedocs.io/en/stable/)
    - [GitHub](https://github.com/python/mypy)
    - [VS Marketplace Extension](https://marketplace.visualstudio.com/items?itemName=ms-python.mypy-type-checker)

**Install**

Install via pip, in a venv, or possibly with pipx:

```bash
python3 -m pip install mypy
```

**Usage**

```bash
mypy program.py
```

---


## Bash

### shellcheck

!!! abstract "Overview"

    > ShellCheck is a GPLv3 tool that gives warnings and suggestions for bash/sh shell scripts.
    >
    > The goals of ShellCheck are:
    >
    > - To point out and clarify typical beginner's syntax issues that cause a shell to give cryptic error messages.
    > - To point out and clarify typical intermediate level semantic problems that cause a shell to behave strangely and counter-intuitively.
    > - To point out subtle caveats, corner cases and pitfalls that may cause an advanced user's otherwise working script to fail under future circumstances.

    - [GitHub](https://github.com/koalaman/shellcheck)
    - [Website](https://www.shellcheck.net/)
    - [Azure Pipelines](https://www.shellcheck.net/wiki/Azure-Pipelines)
    - [GitHub Actions](https://www.shellcheck.net/wiki/GitHub-Actions)
    - [GitLab CI](https://www.shellcheck.net/wiki/GitLab-CI)
    - [Ignoring Errors](https://www.shellcheck.net/wiki/Ignore)

**Install**

```bash
sudo apt update
sudo apt install -y shellcheck

```

**Usage**

```bash
shellcheck ./some-script.sh

```

**Configure**

The most common way to tune `shellcheck` output is with exclusions directly within each script.

```bash
#!/bin/bash

# shellcheck disable=SC2059
some_function() {
    <SNIP>
```

You could also create a `~/.shellcheckrc` with the same directives.

```txt
# ~/.shellcheckrc
disable=SC2059,SC2034 # Disable individual error codes
disable=SC1090-SC1100 # Disable a range of error codes
```

**GitHub Actions**

```yaml
# .github/worflows/shellcheck.yml

# Built from the following examples:
# - https://github.com/koalaman/shellcheck/wiki/GitHub-Actions

name: "ShellCheck"
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  shellcheck:
    name: ShellCheck
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Run ShellCheck
      run: find . -type f -name "*.sh" -exec shellcheck {} +
```

---


## PowerShell

### PSScriptAnalyzer

!!! abstract "Overview"

    > PSScriptAnalyzer is a static code checker for PowerShell modules and scripts. PSScriptAnalyzer checks the quality of PowerShell code by running a set of rules. The rules are based on PowerShell best practices identified by PowerShell Team and the community. It generates DiagnosticResults (errors and warnings) to inform users about potential code defects and suggests possible solutions for improvements.

    - [Installation](https://learn.microsoft.com/en-us/powershell/utility-modules/psscriptanalyzer/overview?view=ps-modules)
    - [Usage](https://learn.microsoft.com/en-us/powershell/utility-modules/psscriptanalyzer/using-scriptanalyzer?view=ps-modules)
    - [GitHub](https://github.com/PowerShell/PSScriptAnalyzer)
    - [GitHub Actions](https://github.com/microsoft/psscriptanalyzer-action) (this may no longer be maintained, the example below does not use it)

!!! quote "Install & Usage"

    Supported PowerShell Versions and Platforms

    - Windows PowerShell 5.1 or greater

    - PowerShell 7.2.11 or greater on Windows/Linux/macOS

    Install using PowerShellGet 2.x:

    ```powershell
    Install-Module -Name PSScriptAnalyzer -Force
    ```

    Install using PSResourceGet 1.x:

    ```powershell
    Install-PSResource -Name PSScriptAnalyzer -Reinstall
    ```

    The **Force** or **Reinstall** parameters are only necessary when you have an older version of

    `PSScriptAnalyzer` installed. These parameters also work even when you don't have a previous version

    installed.

    To lint PowerShell code, you can use various built-in presets via the `-Settings <Preset>` option:

    ```powershell
    Invoke-ScriptAnalyzer -Path /path/to/module/ -Settings PSGallery -Recurse
    ```

**GitHub Actions**

```yaml
# .github/worflows/psscriptanalyzer.yml

# Built from the following examples:
# - https://github.com/koalaman/shellcheck/wiki/GitHub-Actions
# - https://learn.microsoft.com/en-us/powershell/utility-modules/psscriptanalyzer/overview?view=ps-modules

name: "PSScriptAnalyzer"
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  psscriptanalyzer:
    name: PSScriptAnalyzer
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v5
    - name: Run PSScriptAnalyzer
    run: |
      Install-Module -Name PSScriptAnalyzer -Force
      Invoke-ScriptAnalyzer -Path .

```

---


## YAML

### yamllint

!!! abstract "Overview"

    > A linter for YAML files.
    >
    > yamllint does not only check for syntax validity, but for weirdnesses like key repetition and cosmetic problems such as lines length, trailing spaces, indentation, etc.

    - [yamllint Docs: Quick Start](https://yamllint.readthedocs.io/en/stable/quickstart.html)
    - [Configuring yamllint](https://yamllint.readthedocs.io/en/stable/configuration.html)
    - [github.com/adrienverge/yamllint](https://github.com/adrienverge/yamllint/)

!!! tip "What uses yamllint?"

    [yamllint](https://yamllint.readthedocs.io) is [used by ansible-lint](https://docs.ansible.com/projects/lint/rules/yaml/) to validate Ansible code.

**Install**

[Install via your package manager, or pip / pipx](https://yamllint.readthedocs.io/en/stable/quickstart.html#installing-yamllint):

```bash
# apt, Debian 8+ / Ubuntu 16.04+
sudo apt-get install yamllint

# dnf
sudo dnf install yamllint

# pip / pipx
pip install --user yamllint
```

**Usage**

[Running yamllint](https://yamllint.readthedocs.io/en/stable/quickstart.html#running-yamllint):

```bash
# Target specific files
yamllint file.yml other-file.yaml

# Run on project folder
yamllint .
```

**GitHub Actions**

```yaml
# .github/workflows/yamllint.yml

# Built from the following examples:
# - https://www.shellcheck.net/wiki/GitHub-Actions
# - https://github.com/geerlingguy/ansible-role-docker/blob/master/.github/workflows/ci.yml

name: YAML
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
jobs:
  lint:
    name: Lint YAML files
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v5
    - uses: actions/setup-python@v6
    #  with:
    #    python-version: '3.13'
    - name: Run yamllint
      run: |
        python3 -m pip install yamllint
        yamllint .
```

---


## JSON

### jq

!!! abstract "Overview"

    [jq](https://github.com/jqlang/jq) is the most common way to parse JSON data.

    > `jq` is a lightweight and flexible command-line JSON processor akin to `sed`,`awk`,`grep`, and friends for JSON data. It's written in portable C and has zero runtime dependencies, allowing you to easily slice, filter, map, and transform structured data.

    - [github.com/jqlang/jq](https://github.com/jqlang/jq) (previously https://github.com/stedolan/jq)
    - [jq Wiki](https://github.com/jqlang/jq/wiki)
    - [Homepage](https://jqlang.org/) (redirect from http://jqlang.github.io/jq)

!!! note "Not jQuerry"

    This is a CLI parser like `sed` or `awk` but for JSON. It's not related to the [jQuerry](https://en.wikipedia.org/wiki/JQuery) JavaScript library.

**Install**

It's common to use package managers to install it, such as `apt`, `brew`, or even Docker.

```bash
sudo apt update; sudo apt install -y jq
```

**Usage**

The [tutorial](https://jqlang.org/tutorial/) is the best way to visualize this. `jq` requires a filter, and the input data, at a minimum.

```bash
# jq [options...] filter [files...]
jq '.' file.json
```

**GitHub Actions**

```yaml
# .github/workflows/json.yml

# Built from the following examples:
# - https://www.shellcheck.net/wiki/GitHub-Actions
# - https://www.shellcheck.net/wiki/SC2013
# - https://github.com/geerlingguy/ansible-role-docker/blob/master/.github/workflows/ci.yml

name: JSON
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
jobs:
  lint:
    name: Lint JSON files
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v5
    - name: Install jq
      run: apt update; apt install -y jq
    - name: Process files
      run: |
        find . -type f -name "*.json" | while IFS= read -r file
        do
            echo "[*]Parsing $file"
            if ! jq '.' "$file" >/dev/null
            then
                echo "[-]Error processing $file"
                break
            fi
        done

```

---


## Ansible

### ansible-lint

!!! abstract "Overview"

    For guidance on writing Ansible code, reference the [Ansible Lint Documentation](https://ansible.readthedocs.io/projects/lint/).

    `ansible-lint` can be used on your playbooks, roles, or collections to check for common mistakes when writing Ansible code.

    - [Installing ansible-lint](https://ansible.readthedocs.io/projects/lint/installing/#installing-the-latest-version)
    - [GitHub Action: run-ansible-lint](https://github.com/marketplace/actions/run-ansible-lint)
    - [GitHub Action: ansible-lint.yml Example](https://github.com/ansible/ansible-lint?tab=readme-ov-file#using-ansible-lint-as-a-github-action)

**Install**

There are a number of ways to do this, but you can install `ansible-lint` just like `ansible`.

With `pipx`:

```bash
pipx install ansible-lint
```

With `pipx`, using a specific version:

```bash
version_number="1.2.3"
package_name='ansible-lint'
pipx install --suffix=_"$version_number" "$package_name"=="$version_number"
```

With `pip`:

```bash
python3 -m pip install --user ansible-lint
```

**Usage**

This will operate recursively on the current directory:

```bash
ansible-lint .
```

**Configure**

- [Configuring ansible-lint](https://ansible.readthedocs.io/projects/lint/configuring/)

The "new" way to do this, if you also intend to leverage the latest [GitHub action](https://github.com/ansible/ansible-lint) in your CI/CD pipeline, is to use a configuration file to specify what `ansible-lint` should be checking. `ansible-lint` will look in the current directory, and then ascend directories, until getting to the git project root, [looking for one of the following filenames](https://ansible.readthedocs.io/projects/lint/configuring/#using-local-configuration-files):

- `.ansible-lint`, this file lives in the project root

- `.config/ansible-lint.yml`, this file exists within a `.config` folder

- `.config/ansible-lint.yaml`, same as the previous file

***NOTE**: When using the `.config/` path, any paths specified in the `ansible-lint.yml` config file must have `../` prepended so ansible-lint can find them correctly.*

The easiest way to start, is with a [profile](https://ansible.readthedocs.io/projects/lint/profiles/), and excluding the `meta/` and `tests/` paths in roles. This is a less verbose version of the `.ansible-lint` file used in this repo.

```yaml
# .ansible-lint

# Full list of configuration options:
# https://ansible.readthedocs.io/projects/lint/configuring/

# Profiles: null, min, basic, moderate, safety, shared, production
# From left to right, the requirements to pass the profile checks become more strict.
# Safety is a good starting point.
profile: safety

# Shell globs are supported for exclude_paths:
# - https://github.com/ansible/ansible-lint/pull/1425
# - https://github.com/ansible/ansible-lint/discussions/1424
exclude_paths:
  - .cache/      # implicit unless exclude_paths is defined in config
  - .git/        # always ignore
  - .github/     # always ignore
  - "*/tests/"   # ignore tests/ folder for all roles
  - "*/meta/"    # ignore meta/ folder for all roles

# These are checks that may often cause errors / failures.
# If you need to make exceptions for any check, add it here.
warn_list:
  - yaml[line-length]

# Offline mode disables installation of requirements.yml and schema refreshing
offline: true

```

Over time you may want to shift the profile to `shared` or `production`, and also tell `ansible-lint` to check the `tests/` and `meta/` paths for each role if you intend to publish them to ansible-galaxy.

**GitHub Actions**

This is an example file that runs `ansible-lint` on your GitHub repo when a new commit or PR is made to the "main" branch.

!!! warning "GitHub Actions Version vs Pip"

    Sometimes the version of `ansible-lint` installed by the GitHub Actions workflow is aheads of the latest stable release you'd get from pip. This may cause failures that you may not see while running the version installed via `pip`/`pipx` locally.

```yaml
# .github/workflows/ansible-lint.yml

# Taken from the following example:
# https://github.com/ansible/ansible-lint?tab=readme-ov-file#using-ansible-lint-as-a-github-action

name: ansible-lint
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
jobs:
  build:
    name: Ansible Lint # Naming the build is important to use it as a status check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run ansible-lint
        uses: ansible/ansible-lint@main # or version tag instead of 'main'
```

**Errors**

Older versions of ansible-lint may produces errors that are difficult to diagnose. When this happens, use a very simple main.yml file, and start slowly adding tasks or vars to this file. Once you identify a task that creates an error, you can begin narrowing down which line(s) in the task or vars are producing the error.

One example of this is new versions of Ansible lint will want you to use `become_method: ansible.builtin.sudo`, while older versions require `become_method: sudo` and will generate a `schema[tasks]` error in this case.

---


## Packer

!!! abstract "Overview"

    > Packer is a tool for building identical machine images for multiple platforms from a single source configuration.
    >
    > Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer supports various platforms through external plugin integrations, the full list of which can be found at <https://developer.hashicorp.com/packer/integrations>.
    >
    > The images that Packer creates can easily be turned into Vagrant boxes.

    - [Packer Install](https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli)
    - [Packer Commands](https://developer.hashicorp.com/packer/docs/commands)
    - [GitHub Actions: setup-packer](https://github.com/hashicorp/setup-packer)

**Install**

This downloads the HashiCorp GPG key, adds the repo information, and installs `packer` on Debian/Ubuntu.

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install packer
```

For reference, this is the current HashiCorp GPG key data:

```txt
pub   rsa4096/0xAA16FCBCA621E701 2023-01-10 [SC] [expires: 2028-01-09]
      Key fingerprint = 798A EC65 4E5C 1542 8C8E  42EE AA16 FCBC A621 E701
uid                             HashiCorp Security (HashiCorp Package Signing) <security+packaging@hashicorp.com>
sub   rsa4096/0x706E668369C085E9 2023-01-10 [S] [expires: 2028-01-09]
```

**Usage**

Initialize a packer template. This downloads any missing plugins described in the `packer {}` block to your local machine.

!!! warning "Malicious Plugins"

    **Review plugins described in the `packer {}` block of any template before executing this**.

```bash
cd /path/to/my-packer-project
packer init .
```

Format the packer templates. Rewrites HCL2 config files to canonical format. *Run this after making edits to your templates or as part of CI/CD*.

```bash
packer fmt .
```

Validate the Packer templates. *Run this after making edits to your templates or as part of CI/CD*.

```bash
packer validate .
```

**GitHub Actions**

This is an example file that runs packer code validation on your GitHub repo when a new commit or PR is made to the "main" branch.

```yaml
# .github/workflows/packer.yml

# Taken from:
# https://github.com/hashicorp/setup-packer

name: packer

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

env:
  PRODUCT_VERSION: "latest"

jobs:
  packer:
    runs-on: ubuntu-latest
    name: Run Packer
    steps:
      - name: Checkout
        uses: actions/checkout@v5

      - name: Setup `packer`
        uses: hashicorp/setup-packer@main
        id: setup
        with:
          version: ${{ env.PRODUCT_VERSION }}

      - name: Run `packer init`
        id: init
        run: |
          packer init .

      - name: Run `packer validate`
        id: validate
        run: |
          packer validate .
```

---


## Terraform

!!! abstract "Overview"

    > Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.

    - [Terraform Install](https://developer.hashicorp.com/terraform/install)
    - [Terraform CLI](https://developer.hashicorp.com/terraform/cli/commands)
    - [GitHub Actions: setup-terraform](https://github.com/hashicorp/setup-terraform)

**Install**

This downloads the HashiCorp GPG key, adds the repo information, and installs `terraform` on Debian/Ubuntu.

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

For reference, this is the current HashiCorp GPG key data:

```txt
pub   rsa4096/0xAA16FCBCA621E701 2023-01-10 [SC] [expires: 2028-01-09]
      Key fingerprint = 798A EC65 4E5C 1542 8C8E  42EE AA16 FCBC A621 E701
uid                             HashiCorp Security (HashiCorp Package Signing) <security+packaging@hashicorp.com>
sub   rsa4096/0x706E668369C085E9 2023-01-10 [S] [expires: 2028-01-09]
```

**Usage**

> Initialize a working directory for validation without accessing any configured backend, use:

```bash
terraform init -backend=false
```

Format the Terraform templates. Rewrites Terraform config files to canonical format. *Run this after making edits to your templates or as part of CI/CD*.

```bash
terraform fmt
```

Validate the Terraform templates. *Run this after making edits to your templates or as part of CI/CD*.

```bash
terraform validate [options]
```

**GitHub Actions**

```yaml
# .github/workflows/packer.yml

# Taken from:
# https://github.com/hashicorp/setup-packer

name: terraform

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  terraform:
    runs-on: ubuntu-latest
    name: Run Terraform
    steps:
      - name: Checkout
        uses: actions/checkout@v5

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Terraform fmt
        id: fmt
        run: terraform fmt -check
        continue-on-error: true

      - name: Terraform Init
        id: init
        run: terraform init -input=false

      - name: Terraform Validate
        id: validate
        run: terraform validate -no-color
```
