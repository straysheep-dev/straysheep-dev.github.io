---
title: "Ansible"
icon: simple/ansible
draft: false
#date:
#  created: 2023-08-20
#  updated: 2026-01-04
categories:
  - ansible
  - how-to
  - automation
  - devops
  - cicd
---

# :simple-ansible: Ansible

Getting started with Ansible. If you don't know what Ansible is or what it's used for:

!!! quote "What is Ansible?"

	Ansible is a radically simple IT automation platform that makes your applications and systems easier to deploy and maintain. Automate everything from code deployment to network configuration to cloud management, in a language that approaches plain English, using SSH, with no agents to install on remote systems. <https://docs.ansible.com>.

!!! note "About this Post"

	This post is a mirror of [straysheep-dev/ansible-configs](https://github.com/straysheep-dev/ansible-configs)'s README. It's being added here as both, a searchable reference within this mkdocs site, and because the recent post on [Molecule](../posts/ansible-molecule.md) needed to be split off into it's own post due to the amount of examples and details. This may end up being the primary source to maintain these notes going forward, with the ansible-configs project README linking back to this post.

**Resources**

- <https://github.com/ansible/ansible>
- <https://docs.ansible.com/ansible/latest/index.html>

<!-- more -->

## ansible-configs Project

The [ansible-conifgs](https://github.com/straysheep-dev/ansible-configs) project is a monorepo of every Ansible role I've created or forked for my own use. Since this post was originally just the README for that project, you can use it to follow along below and get started in a test VM to learn about Ansible. The only reason this might be useful to anyone getting started is because when the monorepo was created, I was also just getting started. A lot of the notes here and roles were created from that perspective.

!!! example "[ansible-conifgs](https://github.com/straysheep-dev/ansible-configs)"

	A collection of ansible roles.

	This repo is both a set of various roles to mirror my own bash scripts, and my notes on using Ansible for easy reference.

	It's structured to be easy to clone, modify, and run only the roles you need. In most cases all you need to change is:

	- `playbook.yml`'s roles
	- Your preferred `inventory/` file's hosts + users

	Use whichever inventory format works best for you. The `.ini` files allow specifying users as inline variables per host, which is useful if each host in a group has different users you'll be connecting as.

	Each inventory example connects to the ansible controller (the localhost of the machine you're running ansible from) by default. Modify these files to add your own remote connections.

	When you're done, run the playbook with one of the following:

	```bash
	# For using sudo on the remote host, where you aren't using the root account
	ansible-playbook -i inventory/inventory.ini [-b] --ask-become-pass -v playbook.yml

	# For using the root account on the remote host, typically to deploy and provision cloud resources
	ansible-playbook -i inventory/inventory.ini -v playbook.yml
	```

	- `-b` will automatically elevate all tasks so you don't need to specify "become sudo" across every task (don't do this unless you need to)
	- `--ask-become-pass` takes the sudo password for the remote user
	- `-v` will show a useful amount of information without being too verbose

	To do: how to specify each remote user's password (if there are multiple remote users listed, each with a unique password).


## Install

Install Ansible:

- [Install Ansible via pipx](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pipx)
- [pipx: Install](https://pipx.pypa.io/stable/#install-pipx)
- [Install Ansible via pip](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#ensuring-pip-is-available)
- [Ensure pip, setuptools, wheel are up to date](https://packaging.python.org/en/latest/tutorials/installing-packages/#ensure-pip-setuptools-and-wheel-are-up-to-date)
- [Install pip on Debian/Ubuntu](https://packaging.python.org/en/latest/guides/installing-using-linux-tools/#debian-ubuntu-and-derivatives)


### Ubuntu 23.10+, Fedora:

It's recommended to use `pipx`.

```bash
# Ubuntu
sudo apt update
sudo apt install pipx

# Fedora
sudo dnf install pipx

pipx ensurepath
sudo pipx ensurepath --global # optional to allow pipx actions with --global argument
```

Install Ansible using pipx:

```bash
pipx install --include-deps ansible
```

[Inject additional libraries with pipx](https://github.com/pypa/pipx?tab=readme-ov-file#inject-a-package), like `passlib` to generate Unix password hashes:

```bash
pipx inject ansible passlib
```

Upgrade Ansible:

```bash
pipx upgrade --include-injected ansible
```


### Ubuntu 22.04 and Older

If you're missing `pip`, the recommended way to install it on Ubuntu, or other Debian derivitives is through apt (Ansible's documentation also mentions the `python3-pip` package):

```bash
sudo apt update
sudo apt install python3-pip
```

Install Ansible using pip:

```bash
python3 -m pip install --user ansible
```

Upgrade Ansible:

```bash
python3 -m pip install --upgrade --user ansible
```


### Install Multiple Versions

See: [Ansible Community Changelogs](https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-community-changelogs)

Using [pipx you can install multiple versions of packages side by side](https://github.com/pypa/pipx/pull/445). This is useful when you want the latest version of a package, and also a specific version of a package on the same system for testing.

To install all of the `ansible-` tools, [you need to specify `ansible-core`](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#pipx-install).

```bash
version_number="1.2.3"
package_name='ansible-lint' # or ansible-core
pipx install --suffix=_"$version_number" "$package_name"=="$version_number"
```

Call the version-pinned installation using the suffix:

```bash
ansible-lint_1.2.3 --version # If installing ansible-lint==1.2.3
ansible-playbook_1.2.3 --version  # If installing ansible-core==1.2.3
```

Remove any versions you may have installed through `pipx` by following the steps above for testing:

```bash
version_number="2.12.10"
package_name='ansible-core' # or ansible-core
pipx uninstall "$package_name"_"$version_number"
```


#### Older Systems

The latest versions of Ansible will not execute on systems with older versions of python3 installed. You will see an error similar to this, where it isn't even able to print the required version information:

```bash
fatal: [10.10.10.55]: FAILED! => {"ansible_facts": {}, "changed": false, "failed_modules": {"ansible.legacy.setup": {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python3"}, "exception": "Traceback (most recent call last):

<SNIP>

  File \"/tmp/ansible_ansible.legacy.setup_payload_zlebszdb/ansible_ansible.legacy.setup_payload.zip/ansible/module_utils/basic.py\", line 17
    msg=f\"ansible-core requires a minimum of Python version {'.'.join(map(str, _PY_MIN))}. Current version: {''.join(sys.version.splitlines())}\",

```

!!! tip "Ansible 2.13.0"

	The versions found to be the most successful for these use cases are Ansible 2.13.0 or above.


## How Ansible Works

!!! quote "[Getting Started with Ansible](https://docs.ansible.com/projects/ansible/latest/getting_started/index.html)"

    Ansible automates the management of remote systems and controls their desired state.

The diagrams in the documentation linked in the quote block above summarize it best, Ansible consists of three components:

- Control Node: The "brain" machine, executes Ansible code to run on the inventory being provisioned or managed
- Inventory: A definition of assets and relevant variables (all text)
- Managed Node: An endpoint being provisioned or managed through Ansible by a Control Node

The [Intro to Ansible](https://docs.ansible.com/projects/ansible/latest/getting_started/introduction.html) goes into this further:

- Scale and predictability is what Ansible does, when bash + SSH reach their limits
- No agent, just a remote administration mechanism is required (SSH, WinRM, etc.)
- Configures desired "states"

With the basics established it's worth also considering:

- Ansible excels in CI/CD workflows with deploying and building infrastructure-as-code
- There are various secrets management integrations (Anisble Vault, SOPS) that make guarding secrets easy
- Molecule testing makes maintaining and developing (porting your bash code to Ansible) easy

There are more unique pieces to consider, such as how playbooks can built as consolidated single files, or split into numerous files, roles, collections, and more Ansible-specific things. This section tries to summarize those points since it's easy to forget all of the ways how everything can tie together.

!!! tip "**Ansible Safety Measures**"

    ⚠️ To be added.

!!! tip "**Analyzing Playbooks and Anisble Code**"

    ⚠️ To be added.

!!! tip "**Setting Sources**"

    It's important to remember, for example, the `ansible.builtin.copy` module copies files *from* **the control node** *to* **managed nodes**, unless [`remote_src: yes`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html#parameter-remote_src) is set.

    If `remote_src: yes` is set, `ansible.builtin.copy` will only use source paths on the remote host and not the control node.

    Basically, *all tasks are typically executed on remote targets*. This means using `ansible.builtin.find` + registering a variable + `ansible.builtin.copy`, to copy arbitrary files *from* the control node won't work.

    In that case, `ansible.builtin.find` will execute on the remote host, and not find the files. `ansible.builtin.copy` will attempt to use source paths on the remote host that don't exist instead of paths on the control node, causing this operation to fail.


## Configure

Ansible uses a file called `ansible.cfg` for configuration and modification. [Ansible Configuration Settings](https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html#ansible-configuration-settings) describes this file and how it works.

This file can exist in a few places, and there's a search order used to find these files and load their definitions when running Ansible.

!!! tip "[ansible.cfg file search order priority](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)"

	- `ANSIBLE_CONFIG` (environment variable if set)
	- `ansible.cfg` (in the current directory)
	- `~/.ansible.cfg` (in the home directory)
	- `/etc/ansible/ansible.cfg`

	You can set just one option in your environment, and Ansible will [still use the defaults or whatever is in your `ansible.cfg` file, for everything else](https://docs.ansible.com/ansible/latest/reference_appendices/general_precedence.html#general-precedence-rules).

!!! quote "[Avoiding security risks with `ansible.cfg` in the current directory](https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html#avoiding-security-risks-with-ansible-cfg-in-the-current-directory)"

    If Ansible were to load `ansible.cfg` from a world-writable current working directory, it would create a serious security risk. Another user could place their own config file there, designed to make Ansible run malicious code both locally and remotely, possibly with elevated privileges. For this reason, Ansible will not automatically load a config file from the current working directory if the directory is world-writable.


### Better CLI Output

By default, Ansible will write output from playbooks and tasks without newlines interpretted. This makes reading playbooks executed with `-v` difficult.

- [jeffgeerling.com: Ansible YAML Callback Plugin for a Better CLI Experience](https://www.jeffgeerling.com/blog/2018/use-ansibles-yaml-callback-plugin-better-cli-experience)
- [Ansible: Setting a Callback Plugin](https://docs.ansible.com/ansible/latest/plugins/callback.html#setting-a-callback-plugin-for-ansible-playbook)
- [`ansible-doc -t callback -l`](https://docs.ansible.com/ansible/latest/plugins/callback.html#plugin-list)
- [Ansible: ansible.builtin.default Now Supports YAML](https://docs.ansible.com/ansible/latest/collections/community/general/yaml_callback.html#deprecated)
- [Ansible: ansible.builtin.default `result_format` Options](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/default_callback.html#parameter-result_format)

You can list all of the built-in (or available) callbacks with `ansible-doc -t callback -l`, then print similar examples to what's available on Anisble's online documentation with something like `ansible-doc -t callback ansible.builtin.default`.

Not all of these callbacks are specifically for affecting stderr and stdout. You'll want to review callbacks that seem interesting and check for options or environment variables that can be applied.

To simply have more human-readable CLI output, use the YAML `result_format`:

```bash
export ANSIBLE_CALLBACK_RESULT_FORMAT='yaml'
# Now execute a playbook in the same shell environment
```

To make this a persistent change, add it to your `ansible.cfg`:

```yaml
# https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/default_callback.html#parameter-result_format
[defaults]
callback_result_format = yaml
```


## Variables and Inventories

*Currently, most roles in this repo have variables defined in `vars/main.yml`. This file takes precedence in most cases. Using `defaults/main.yml` for variables instead allows you to define the default there, and override those defaults in your inventory file(s) on a per-host or per-group level. This note will be removed and changed when all current roles are revised to reflect this.*

Example default value for a variable in `defaults/main.yml`:

```yml
some_var: "false"
```

Ansible has modular ways of approaching and maintaining both, variables and an inventory at the same time.

- [Assign variables per-machine](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#assigning-a-variable-to-one-machine-host-variables)
- [Assign variables to machine groups](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#assigning-a-variable-to-many-machines-group-variables)

Change `some_var` to `"true"` for just one host in your inventory:

```ini
10.0.0.40:22 ansible_user=user some_var="true"
```

Change `some_var` to `"true"` for all hosts in a specific inventory group:

```ini
[remotegroup]
10.0.0.41:22 ansible_user=user
10.0.0.42:22 ansible_user=user
10.0.0.43:22 ansible_user=user

[remotegroup:vars]
some_var="true"
```

Finally, be sure your `playbook.yml` file allows for either `all` groups, or the groups defined in your inventory file(s). If using `all`, you must ensure each inventory file has unique definitions to avoid collisions.

```yml
- name: "Default Playbook"
  hosts:
    # List groups from your inventory here
    # You could also use the built in "all" or "ungrouped"
    # "all" is necessary when Vagrant is auto-generating the inventory
    all
    #localgroup
    #remotegroup
    #tester_nodes
    #target_nodes
  roles:
  <SNIP>
```

See the following reference:

- [Playbook Variables: Tips on where to set variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#tips-on-where-to-set-variables)


## Windows Provisioning

You effectively have two options for opening Windows endpoints to Ansible provisioning:

- WinRM (Domain-Joined, ideally with Kerberos auth, [otherwise there's a less secure work around](https://github.com/ansible/ansible-documentation/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1))
- [SSH (Best for non-domain-joined endpoints)](https://github.com/straysheep-dev/windows-configs/blob/main/Manage-OpenSSHServer.ps1)

*[There's also PSRemoting over SSH, available to Windows, Linux, and macOS. The PowerShell version installed must be 7.X or later](https://learn.microsoft.com/en-us/powershell/scripting/security/remoting/ssh-remoting-in-powershell?view=powershell-7.4).*

Update your `inventory.ini` file by appending the following options to your Windows endpoints:

- `cmd` is the default shell for SSH on Windows
- Change this to `powershell` if you've defined PowerShell as the default SSH login shell
- `ansible_become_user` is better to be specified per host in the inventory file
- `ansible_become_password` may be necessary (with LAPS), use an ansible-vault to store these values
- `ansible_become_method: runas` can be specified per task just like `sudo`

```ini
[remotehosts]
# "Minimum" possible settings, if tasks specify `become_method: runas`
10.55.55.30:22 ansible_user=User ansible_become_user=User ansible_connection=ssh ansible_shell_type=cmd

# Additional settings for password, and become_method
10.55.55.31:22 ansible_user=User ansible_become_user=User ansible_become_password='{{ User_runas_pass }}' ansible_connection=ssh ansible_shell_type=cmd ansible_become_method=runas
```

Your tasks will have to reflect these kinds of settings as well, using `runas` instead of `sudo` when Windows is detected.

See the following references:

- [Ansible Privilege Escalation: `become` Connection Variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_privilege_escalation.html#become-connection-variables)
- [Ansible Playbook Fails on Windows](https://devops.stackexchange.com/questions/16532/ansible-playbook-fails-on-windows-server)
- [Ansible Playbook Become Error](https://stackoverflow.com/questions/66671945/ansible-playbook-error-the-powershell-shell-family-is-incompatible-with-the-sud)

Execute with:

```bash
~/.local/bin/ansible-playbook -i inventory.ini -v ./playbook.yml
```


## Ansible-Vault

- [ansible-vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html)
- [become-connection-variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_privilege_escalation.html#become-connection-variables)
- [playbooks-vault](https://docs.ansible.com/ansible/latest/vault_guide/vault_using_encrypted_content.html#playbooks-vault)

This utility is included with `ansible`, and allows you to create encyrpted ansible `.yml` files. It's primarily used for encrypting secrets used for plays, but can even be used to encrypt an entire role.

Create an encrypted file (called a vault):
```bash
ansible-vault create vault.yml
# Enter a password, then edit / write the file in the default text editor (vim)
```


**Vault Password File + Environment Variables**

In cases where you're running multiple playbooks, it can be tedious to repeatedly enter the vault password. Ansible has a `--vault-pass-file` option that can read the password from a file. *Unfortunately, Ansible doesn't have a built in environment variable you can pass to it for this purpose.* Writing this secret in a plaintext file isn't the best idea, and interestingly enough **you can specify commands or scripts as the vault-pass-file**. This means you can use a similar trick to [configuring terraform environment variables](https://github.com/straysheep-dev/terraform-configs#quick-start-environment-variables) and read the vault password from an environment variable.

See these references for a full breakdown, they're summarized below:

- [Enter Vault Password Once for Multiple Playbooks](https://stackoverflow.com/questions/77622261/how-to-pass-a-password-to-the-vault-id-in-a-bash-script)
- [How to Pass an Ansible Vault a Password](https://stackoverflow.com/questions/62690097/how-to-pass-ansible-vault-password-as-an-extra-var/76236662)
- [Get Password from the Environment with `curl`](https://stackoverflow.com/questions/33794842/forcing-curl-to-get-a-password-from-the-environment/33818945#33818945)
- [Get Password from Shell Script without Echoing](https://stackoverflow.com/questions/3980668/how-to-get-a-password-from-a-shell-script-without-echoing#3980904)

First, enter the vault password with `read`:

```bash
echo "Enter Vault Password"; read -r -s vault_pass; export ANSIBLE_VAULT_PASSWORD=$vault_pass
```

- `-s` hides the text as you type
- `-r` interprets any backslashes correctly, see [SC2162](https://www.shellcheck.net/wiki/SC2162)
- The environment variable only appears in the `env` of that shell session
- It does not appear in the history of that shell
- Another shell running under the same user context cannot see that environment variable without a process dump

Execute with:

```bash
ansible-playbook -i <inventory> -e "@~/secrets/auth.yml" --vault-pass-file <(cat <<<$ANSIBLE_VAULT_PASSWORD) -v ./playbook.yml
```

- Uses `<(cat <<<$VARIABLE)` process substitution and creates a here-string
- The raw value will not appear in your process list
- Using [pspy](https://github.com/DominicBreuker/pspy) you can verify this
- Be sure `kernel.yama.ptrace_scope` is set to `1` or higher, as `0` will allow process dumping without root


**Use Case: Manage Remote Hosts with Unique Sudo Passwords**

This covers the following scenario:

- You have two or more remote hosts with a normal user using `sudo` instead of root
- You need to update all of them weekly
- You do not want plain text passwords in yaml files
- Each remote host's `sudo` user has your ssh public key, and only accepts public key authentication

A clear way to manage this and illustrate how this works is by creating a new vault file (we'll call it `auth.yml`) containing all of the remote user passwords.

```bash
ansible-vault create auth.yml
# Specify a vault password, generate and save this to a password manager
```

The content of auth.yml could look like this:

```conf
admin_sudo_pass: 53Zbr3DPpfzGKSbWxNgWareBgNptKt5s
sql_admin_sudo_pass: 3KYRoAndmF53XDu33No7jfsNv2jrrpLi
```

Then the contents of `inventory/inventory.ini` could look like this:

```ini
10.20.30.40:2222 ansible_user=admin ansible_become_password='{{ admin_sudo_pass }}'
10.20.30.41:2222 ansible_user=sql_admin ansible_become_password='{{ sql_admin_sudo_pass }}'
```

To execute the playbook, specifying the `auth.yml` file with `-e "@auth.yml"`, and instead of `--ask-become-pass`, use `--ask-vault-pass`. Ansible will check the vaulted `auth.yml` file for the sudo passwords now instead of expecting them to be passed right after executing this command where typically it will only accept one input string for `become_pass`, which is the problem this solves.

```bash
ansible-playbook -i inventory/inventory.ini --ask-vault-pass --extra-vars "@auth.yml" -v playbook.yml
```

This can be taken further by also encrypting the usernames as variables in `auth.yml`.


## SOPS

!!! quote "What is SOPS?"

    [SOPS](https://getsops.io/) allows to encrypt and decrypt files using various key sources (GPG, AWS KMS, GCP KMS, ...). For structured data, such as YAML, JSON, INI and ENV files, it will encrypt values, but not mapping keys. For YAML files, it also encrypts comments. This makes it a great tool for encrypting credentials with Ansible: you can easily see which files contain which variable, but the variables themselves are encrypted.

    The ability to utilize various keysources makes it easier to use in complex environments than Ansible Vault.

SOPS is effectively the more robust replacement to Ansible Vaults. The benefit is being able to read secrets files to see the variable names without exposing the secrets, and the flexibility in use with encrypted data. The preferred encryption tool is [age](https://github.com/FiloSottile/age), but you can use GPG or a number of other methods described [here](https://github.com/getsops/sops?tab=readme-ov-file#usage).

[Protecting Ansible secrets with SOPS](https://docs.ansible.com/ansible/latest/collections/community/sops/docsite/guide.html) is the quick-start guide with examples for doing complex lookups, among other things. It also details using SOPS to protect (inventory) secrets, but does not showcase this as well as it could, and if you're used to working with Ansible Vaults, this is where your mind is at if you're looking at migrating those secrets to SOPS. While an Ansible Vault can be as simple as a dedicated YAML file with unique variable names that the inventory file(s) reference (providing your vault password when running the playbook for runtime decryption), the SOPS guide details how this works via SOPS but does not show step-by-step examples of doing it. If you're unfamiliar with `group_vars` and `host_vars`, since maybe you didn't need to structure your inventory like that, this becomes confusing.

This section shows you step-by-step, how to use SOPS to encrypt an inventory's sudo passwords for headless use. Before that, there's one key point that needs mentioned. Expand the note here to read more, otherwise jump below to start with SOPS.

??? warning "Headless Workflows Come Down to One Secret"

    Whether it's an Ansible Vault, SOPS, or your local machine's keyring, everything in DevOps (primarily headless operations) appears to come down to one secret, somewhere, that if compromised, theoretically, so is everything it has access to.

    If you're using SOPS + age like this guide suggests, you quickly realize the age key is a plaintext file, with the secret exposed. GPG and SSH have endless documentation around password protecting your keys, and the point is if you don't password protect your keys, it's simply much more dangerous. The problem is, if you want to run headless operations, either scheduled or on demand, you'd need to load those passwords somehow regularly and safely at runtime. Now you run into more conundrums:

    - System keystores are decrypted for the session, even if there's a way to do that temporarily it still exposes the key... to your key
    - Similarly, keystores often require GUI interaction to approve operations
    - Do you use a plaintext environment variable or environment file that's read-protected? If so, we're back to square one here
    - How about setting up a Hashicorp Vault? You still have the same problem *but*, there's a difference here in terms of scope and access
    - What about tying the secret to the TPM? Same issues still apply

    Each of those points above reveal the weird difference between vaults and password managers. Even though they are more or less the same thing, the level of manual interaction password manager require is part of their strength. Tools still exist to scrape a system's memory for password manager data, but it's not easy. The programmatic nature of vaults means they're more reliable for this. No matter what you decide to do, there's either going to be narrow scope or manual action required to protect the blast radius of a compromise.

    This guide recommends starting with the plaintext age key for testing, then replacing it with a call to your preferred vault solution where the secret itself is retrieved to decrypt your SOPS secrets. Vaults allow you to scope who / what / when / why / how, and other things. For example, depending on the vault, you could limit what hosts what key has access to, and when / from where it can be retrieved. This accepts that a system is handling a secret, but it's giving you a signal to look for if something seems unusual with key usage or logins. Similarly, the bottom line is the system running this type of workload should be very narrowly scoped and tightly controlled. This is a good way to think of an Ansible Controller. It should only perform Ansible tasks, not run any other workloads, and be heavily guarded + audited.

Install age using your system's package manager.

> A simple, modern and secure encryption tool (and Go library) with small explicit keys, no config options, and UNIX-style composability.

- <https://github.com/FiloSottile/age>

See the GitHub README for an overview and quick-start on usage. This is the recommended encryption tool to use with [SOPS](https://github.com/getsops/sops) instead of GPG.

```bash
# Ubuntu 22.04+
sudo apt install age

# Fedora 33+
sudo dnf install age
```

You can easily install SOPS using the `community.sops.install` role that comes with Ansible.

```bash
# Install SOPS on Ansible controller
ansible-playbook --ask-become-pass community.sops.install_localhost

# Install SOPS on remote servers
echo "Enter Vault Password"; read -r -s vault_pass; export ANSIBLE_VAULT_PASSWORD=$vault_pass
ansible-playbook community.sops.install --inventory ~/.inventory.yml -e "@~/.vault.yml" --vault-pass-file <(cat <<<$ANSIBLE_VAULT_PASSWORD)
```

- [https://github.com/getsops/sops](https://github.com/getsops/sops)
- [community.sops.install role - Install SOPS](https://docs.ansible.com/ansible/latest/collections/community/sops/install_role.html)

```bash
# SOPS uses predictable config folder paths, create one on Linux
mkdir -p $HOME/.config/sops/age/

# Generate an age public / private key pair
age-keygen -o $HOME/.config/sops/age/keys.txt
```

Now you need to create the inventory structure SOPS requires. This is what isn't highlighted in the official quick-start guide, but is covered in the section on [Working with Encrypted Variables](https://docs.ansible.com/projects/ansible/latest/collections/community/sops/docsite/guide.html#working-with-encrypted-variables).

```bash
# This assumes the system is a general purpose Ansible Controller
mkdir -p ~/ansible/inventory
```

Create an inventory file, `nano ~/ansible/inventory/inventory.yaml`, we'll target the local machine in this example in two ways, via localhost, and it's IP, `127.0.0.1`. It makese sense to think of this file as the "meta" inventory file, describing only machines, and the groups they belong to. You don't have to do it this way, but it's more organized and easier to follow.

```yaml
---
# inventory/inventory.yml

# https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html
# https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html
# https://docs.ansible.com/ansible/latest/getting_started/index.html
# ansible all --list-hosts -i inventory/inventory.yml

# A sample yaml inventory file with one group, called "localgroup"
localgroup:
  hosts:
    localhost:
    127.0.0.1:
```

Now we can create either, `inventory/host_vars/` and or `inventory/group_vars/` paths within the inventory folder. The [Organizing Host and Group Variables](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html#organizing-host-and-group-variables) documentation walks you through this.

```bash
# Create the vars paths
mkdir -p ~/ansible/inventory/host_vars
mkdir -p ~/ansible/inventory/group_vars
```

!!! note "How `host_vars` and `group_vars` Work"

    It can get complex with variable load priority but for simplicity, `host_vars` basically assigns variables to hosts based on the path / file name matching the hostname.

Now create the variable file for `localhost` using SOPS:

```bash
# Obtain public key
age_pubkey="$(grep -i public $HOME/.config/sops/age/keys.txt | awk '{print $4}')"

# Create the YAML file using SOPS, encrypted with the age key
sops --age "$age_pubkey" ~/ansible/inventory/host_vars/localhost.sops.yaml
```

!!! tip "[Valid Extensions](https://docs.ansible.com/projects/ansible/latest/collections/community/sops/docsite/guide.html#working-with-encrypted-variables)"

    > Put files with the following extensions into the `group_vars` and `host_vars` directories:

    - `.sops.yaml`
    - `.sops.yml`
    - `.sops.json`

    > For the `host_group_vars` plugin, your host and group variable files ***must*** use YAML syntax. Valid file extensions are `.yml`, `.yaml`, `.json`, or no file extension.

Set the content to include any and all variables you'd like to protect for localhost.

```yaml
---
ansible_connection: local
ansible_become_password: "my_passw0rd!"

```

Now do the same for 127.0.0.1. It gets its own file under `host_vars/127.0.0.1.sops.yaml`.

```bash
# Obtain public key
age_pubkey="$(grep -i public $HOME/.config/sops/age/keys.txt | awk '{print $4}')"

# Create the YAML file using SOPS, encrypted with the age key
sops --age "$age_pubkey" ~/ansible/inventory/host_vars/127.0.0.1.sops.yaml
```

```yaml
---
ansible_user: user1
ansible_port: 22
ansible_become_password: "an0ther_passw0rd!"
```

!!! tip "Edit SOPS Encrypted Files"

    To edit a SOPS file it's just `sops /path/to/file.sops.yaml`.

    SOPS is automatically aware of your keys under `~/.config/sops/age/keys.txt` so if the key used to encrypt the file is there, you don't need to do anything more to decrypt the file.

Enable both, `host_group_vars` and `community.sops.sops`, in your `ansible.cfg`. See the [Configure](#configure) section for a quick overview on how this file works.

```yaml
[defaults]
vars_plugins_enabled = host_group_vars,community.sops.sops
```

Now, we can create and execute the following example `playbook.yml` file:

```yaml
---
# This example playbook invokes sudo, proving we can decrypt SOPS encrypted sudo passwords
# from the relevant host_vars files.
- name: "Debug"
  hosts:
    localgroup
  tasks:
    - name: Show facts available on the system
      ansible.builtin.command: /usr/bin/id
      become: true
      become_method: ansible.builtin.sudo

```

Run to confirm sudo execution succeeds, `id` will print `stdout: uid=0(root) gid=0(root) groups=0(root)` in the results.

```bash
ansible-playbook -i inventory/ -v playbook.yml
```


## Ansible-Lint

For guidance on writing Ansible code, reference the [Ansible Lint Documentation](https://ansible.readthedocs.io/projects/lint/).

`ansible-lint` can be used on your playbooks, roles, or collections to check for common mistakes when writing ansible code.

- [Installing `ansible-lint`](https://ansible.readthedocs.io/projects/lint/installing/#installing-the-latest-version)
- [GitHub Action: run-ansible-lint](https://github.com/marketplace/actions/run-ansible-lint)
- [GitHub Action: ansible-lint.yml Example](https://github.com/ansible/ansible-lint?tab=readme-ov-file#using-ansible-lint-as-a-github-action)

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

- [Configuring ansible-lint](https://ansible.readthedocs.io/projects/lint/configuring/)

The "new" way to do this, if you also intend to leverage the [latest GitHub action](https://github.com/ansible/ansible-lint) in your CI/CD pipeline, is to use a configuration file to specify what `ansible-lint` should be checking. `ansible-lint` will look in the current directory, and then ascend directories, until getting to the git project root, [looking for one of the following filenames](https://ansible.readthedocs.io/projects/lint/configuring/#using-local-configuration-files):

- `.ansible-lint`, this file lives in the project root
- `.config/ansible-lint.yml`, this file exists within a `.config` folder
- `.config/ansible-lint.yaml`, same as the previous file

!!! note ""

	*When using the `.config/` path, any paths specified in the `ansible-lint.yml` config file must have `../` prepended so ansible-lint can find them correctly.*

The easiest way to start, is with a [profile](https://ansible.readthedocs.io/projects/lint/profiles/), and excluding the `meta/` and `tests/` paths in roles. This is a less verbose version of the `.ansible-lint` file used in this repo.

```yml
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

**Errors**

Older versions of ansible-lint may produces errors that are difficult to diagnose. When this happens, use a very simple main.yml file, and start slowly adding tasks or vars to this file. Once you identify a task that creates an error, you can begin narrowing down which line(s) in the task or vars are producing the error.

One example of this is new versions of ansible lint will want you to use `become_method: ansible.builtin.sudo`, while older versions require `become_method: sudo` and will generate a `schema[tasks]` error in this case.


## Ansible-Galaxy

This is the suggested way to share and run roles or collections. Though you ***can*** run them using local paths, you're meant to reference them using a fully-qualified-name (FQN in Ansible terms).

- [Ansible-Galaxy](https://galaxy.ansible.com/ui/)
- [Ansible-Galaxy User Guide (`ansible-galaxy`)](https://docs.ansible.com/ansible/devel/galaxy/user_guide.html)
- [Ansible-Galaxy User Guide (Platform and API)](https://ansible.readthedocs.io/projects/galaxy-ng/en/latest/community/userguide.html#servers)
- [Create Ansible Roles or Collections](https://docs.ansible.com/ansible/latest/cli/ansible-galaxy.html#role-init)
- [Role Naming Conventions](https://docs.ansible.com/ansible/devel/dev_guide/developing_collections_structure.html#roles-directory)

When creating roles that will be published, you'll need to fill out the `meta.yml` details:

- [Role Meta: Supported Platforms (GH Issue: 52)](https://github.com/ansible/galaxy/issues/52)
- [Role Meta: Supported Platforms (`ansible-lint` schema)](https://github.com/ansible/ansible-lint/blob/main/src/ansiblelint/schemas/meta.json)

The documentation (at the time of writing) suggested making an `~/.ansible.cfg` to target the beta galaxy_ng server, however this is no longer working (in other words, the galaxy_ng server is no longer in beta and is now the default). It seems the best way to do this is by providing your API key on the command line, through an environment variable:

```bash
echo "Enter Galaxy API Token"; read -r -s ansible_galaxy_token; export ANSIBLE_GALAXY_TOKEN=$ansible_galaxy_token
# [type or paste your key, it won't echo or show up in your bash history]
ansible-galaxy role import -vvv --token $ANSIBLE_GALAXY_TOKEN <github-username> <ansible-role-my_repo>
```

The above command will add (import) a role from your GitHub, to your own Ansible-Galaxy namespace, so that others can download and install it directly from the `ansible-galaxy` command.


## References

This repo was inspired by, and created after learning from [IppSec's parrot-build](https://github.com/IppSec/parrot-build) repo and video.
