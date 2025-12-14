---
title: "Ansible Molecule"
icon: simple/ansible
draft: false
#date:
#  created: 2025-07-25
#  updated: 2025-12-14
categories:
  - ansible
  - molecule
  - containers
  - virtualization
  - orchestration
  - how-to
  - devops
  - cicd
  - linting
---

# :simple-ansible: Ansible Molecule

Molecule is part of the Ansible project. It's effectively a way to orchestrate containers or VM's to automate the testing of roles and collections in various environments. For example, using Docker with custom Dockerfiles or QEMU with Vagrant templates, to build the environments to run roles on. Combining this with CI workflows in GitHub or GitLab allows you to automate the process.

!!! tip "Use Cases"

	In theory molecule could be used to automate the testing of really anything within a container or VM as long as the playbook is configured to handle it.

This post tries to fill in any blanks between what the documentation doesn't need to say, and what you'll see when reviewing public examples of projects using molecule. There are a few areas where it's not immediately clear "what" or "how", so this post will be a point of reference from that perspective. Once you see it working and are interacting with it, it all makes sense.

<!-- more -->

## References

The links below are in the order to follow if you're just getting started with Molecule.

- [github.com/ansible/molecule](https://github.com/ansible/molecule)
- [Molecule: Customizing a Docker Image in Molecule](https://ansible.readthedocs.io/projects/molecule/guides/custom-image.html)
- [Molecule: molecule.yml Configuration](https://ansible.readthedocs.io/projects/molecule/configuration.html)
- [Ansible for DevOps: converge.yml Example](https://github.com/geerlingguy/ansible-for-devops/blob/master/molecule/molecule/default/converge.yml)
- [github.com/geerlingguy/docker-fedora42-ansible Dockerfile](https://github.com/geerlingguy/docker-fedora42-ansible/blob/master/Dockerfile)
- [github.com/geerlingguy/docker-rockylinux10-ansible Dockerfile](https://github.com/geerlingguy/docker-rockylinux10-ansible/blob/master/Dockerfile)
- [github.com/geerlingguy/docker-ubuntu2404-ansible Dockerfile](https://github.com/geerlingguy/docker-ubuntu2404-ansible/blob/master/Dockerfile)
- [github.com/geerlingguy/docker-debian12-ansible Dockerfile](https://github.com/geerlingguy/docker-debian12-ansible/blob/master/Dockerfile)
- [Docker: Writing a Dockerfile](https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile/)
- [Docker: Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)
- [Docker Hub: Fedora](https://hub.docker.com/_/fedora/tags)
- [Docker Hub: Rocky](https://hub.docker.com/r/rockylinux/rockylinux)


## Installing Molecule

[Molecule: Install Documentation](https://ansible.readthedocs.io/projects/molecule/installation/)

!!! tip "pipx vs venvs"

    You'll likely want to use a `venv` or `--user` to install molecule in a dev VM (or similar environment).

    Ideally, install `ansible-dev-tools` to have everything available to work with.

    Alternatively, you can use `pipx` with `inject` to ensure the virtual environment pipx creates for molecule has access to `docker` and `molecule-plugins[docker]`

Install Ansible's dev tools, Molecule, and the Docker Python SDK:

=== "pip"

    ```bash
    # Create a virtual environment
    mkdir ~/venv
    python3 -m venv ~/venv
    source ~/venv/bin/activate

    # Install the dev tools if possible
    python3 -m pip install --user ansible-dev-tools

    # Minimally, install molecule, ansible, and any necessary container plugins
    python3 -m pip install --user molecule ansible ansible-lint "molecule-plugins[docker]"

    # Install the docker python SDK
    # https://github.com/docker/docker-py
    python3 -m pip install --user docker
    ```

=== "pipx"

    ```bash
    # Install the latest version of pipx
    python3 -m pip install --user pipx
    python3 -m pipx ensurepath

    # Install each tool into an isolated environment with pipx
    pipx install molecule ansible-lint
    pipx install --include-deps ansible

    # Inject the docker libraries into molecule's pipx environment
    pipx inject molecule "molecule-plugins[docker]" docker
    ```

Finally, install Docker itself if you already haven't.

- [Docker Install Instructions](https://docs.docker.com/engine/install/)
- [Ansible Role to Install Docker](https://github.com/geerlingguy/ansible-role-docker)


## Creating a Scenario

!!! note "A scenario?"

	A [scenario](https://ansible.readthedocs.io/projects/molecule/getting-started/#molecule-scenarios) is the term for running tests through molecule.

The Molecule documentation has examples for Ansible Collections. To create a molecule scenario for a single role, you can do so from the root of the role's project folder. See [geerlingguy](https://github.com/geerlingguy?tab=repositories&q=ansible-role-&type=&language=&sort=)'s `ansible-role-*` repos for examples of this.

```bash
# Move to the root of your role's project folder
cd ~/src/ansible-role-my_role

# Initialize the scenario
molecule init scenario
```

This creates a `molecule/default/` folder with all of the [default scenario files](https://ansible.readthedocs.io/projects/molecule/getting-started/#the-scenario-layout) necessary to run tests.

```txt
$ molecule init scenario
INFO     Initializing new scenario default...

PLAY [Create a new molecule scenario] ******************************************

TASK [Check if destination folder exists] **************************************
changed: [localhost]

TASK [Check if destination folder is empty] ************************************
ok: [localhost]

TASK [Fail if destination folder is not empty] *********************************
skipping: [localhost]

TASK [Expand templates] ********************************************************
changed: [localhost] => (item=molecule/default/molecule.yml)
changed: [localhost] => (item=molecule/default/create.yml)
changed: [localhost] => (item=molecule/default/converge.yml)
changed: [localhost] => (item=molecule/default/destroy.yml)

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Initialized scenario in /home/user/src/ansible-role-my_role/molecule/default successfully.
```

The most important files here are these two (from [Scenario Layout](https://ansible.readthedocs.io/projects/molecule/getting-started/#the-scenario-layout)):

> - `molecule.yml` is the central configuration entry point for Molecule per scenario. With this file, you can configure each tool that Molecule will employ when testing your role.
> - `converge.yml` is the playbook file that contains the call for your role. Molecule will invoke this playbook with ansible-playbook and run it against an instance created by the driver.

!!! note ""

	In other words [`molecule.yml`](https://ansible.readthedocs.io/projects/molecule/getting-started/#inspecting-the-moleculeyml) configures Molecule itself; what driver it will use, how it will run, and more. Think of `converge.yml` as your playbook file that Molecule will execute against each instance it creates. Use `converge.yml` to configure and include the roles or collections you want to assess.


## Running Molecule

To simply run a scenario, from your role's project folder:

```bash
# Specify a driver with -d|--driver-name
molecule test [-d docker]
```

For additional debugging output from molecule:

```bash
molecule -vvv test [-d docker]
```

!!! info "What's happening here?"

    In the most default setup, there are two things happening:

    1. Converge tests are executing your role(s) / collection / playbook(s)
    2. Idempotence tests are running everything again in the same instances, ensuring nothing changes or fails

    As you'll see (below) this can create issues when some tasks aren't meant to be "idempotent" from Ansible's perspective.


### ERROR: The role 'your_role' was not found

If you encounter this error, there are some lines you may need to ensure you have specified in `meta/main.yml` of a role:

- `namespace`
- `author`
- `role_name`

If Molecule can't find your role locally (because let's say it doesn't exist in Ansible-Galaxy or GitHub yet), you may see the following error:

```
ERROR! the role 'my_role' was not found in /home/user/src/ansible-role-my_role/molecule/default/roles:/home/user/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/home/user/src/ansible-role-my_role/molecule/default

The error appears to be in '/home/user/src/ansible-role-my_role/molecule/default/converge.yml': line 17, column 7, but may be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

  roles:
    - role: my_role
      ^ here
```

In this case there a few ways to resolve this.

**Option 1: Symlink**

If you're not using a namespace or plan to publish to Ansible-Galaxy, symlink your role's project path with:

```bash
# The symlink points to the role's project folder
ln -s ~/src/ansible-role-my_role ~/.ansible/roles/my_role
```

**Option 2: Namespace + Role Name (Recommended)**

Ensure you have those 3 fields (`author`, `namespace`, `role_name`) filled out in `meta/main.yml`.

Then specify the namespace + role name in your `converge.yml` file:

```yml
- name: Converge
  hosts: all
  gather_facts: true
  tasks:
    - name: Enumerate Ansible Facts
      ansible.builtin.debug:
        # msg: "This is the effective test"
        var: ansible_facts['system']
  roles:
    - role: my_namespace.my_role

```


### Tags and Molecule

You can skip specific tasks in Molecule testing that either don't work well, or are hard to test without a lot of configuration changes in a containerized environment, by using the `molecule-notest` or `notest` tags.

- [Molecule Docs: Anisble Configuration](https://ansible.readthedocs.io/projects/molecule/configuration/#ansible)
- [GitHub: Support a standard "skip_in_molecule" tag issue:1742](https://github.com/ansible/molecule/issues/1742)

```yaml
- name: "Ensure temporary working folder exists"
  ansible.builtin.file:
    path: "{{ uac_outfolder }}"
    state: directory
    mode: '0700'
  tags:
    - molecule-notest
```


### Idempotence Tests

There are really 2 main tests molecule runs if you're just going with the most default settings. `converge.yml` tests execute your tasks on the target containers, and *idempotence* tests run the converge test once more to check if tasks return as anything other than OK. A "changed" or "failed" state indicates the playbook was not idempotent.

These resources go into more detail around this topic:

- [Disable idempotence check on certain tasks issue:816](https://github.com/ansible/molecule/issues/816)
- [Add `--molecule-idempotence-notest` tag to skip-tags during idempotence... issue:1663](https://github.com/ansible/molecule/pull/1663)
- [Molecule: Skip Idempotence with Tags](https://ansible.readthedocs.io/projects/molecule/configuration/#ansible)
- [Ansible: Adding Tags to Tasks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html#adding-tags-to-individual-tasks)

Ultimately, if you need to exclude a task from idempotence tests, there's now a tag that supports this: `molecule-idempotence-notest`. Simply add this tag to any tasks that will always return as "changed", so that they can be excluded without any work arounds or skipping idempotence testing entirely.

```yaml
- name: "Ensure temporary working folder exists"
  ansible.builtin.file:
    path: "{{ uac_outfolder }}"
    state: directory
    mode: '0700'
  tags:
    - molecule-idempotence-notest
```


### systemd in Docker

By default, Docker containers are minimal environments that do not have systemd out of the box in many cases.

To achieve this, there are a few resources I've found that demonstrate this in an understandable way:

- [Molecule Docs: systemd-container Guide](https://ansible.readthedocs.io/projects/molecule/guides/systemd-container.html)
- [rockylinux's Docker image systemd Integration Notes](https://hub.docker.com/r/rockylinux/rockylinux#systemd-integration)
- [geerlingguy's Docker role `converge.yml` file](https://github.com/geerlingguy/ansible-role-docker/blob/94b787389dd028d6397abf27dd7e996912e059de/molecule/default/converge.yml#L6)

SELinux may add complexity in some case, but generally you're:

- Sharing the host's `/sys/fs/cgroup` with the container
- Run the container with `SYS_ADMIN` capabilities or `privileged` mode.
- Installing systemd using the container OS's package manager
- Running / starting systemd

!!! note "Docker Notes"

    This section was taken directly from the [docker-configs README](https://github.com/straysheep-dev/docker-configs?tab=readme-ov-file#usage).

**Building Images with Dockerfiles**

See [build, tag, and publish an image](https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/).

```bash
# Tag the file with any name you want, and point to the dockerfile with -f
# This also assumes you're in the cwd of the dockerfile with the "." on the end
docker build -t local/my-image-name -f ./some.Dockerfile .
```

**Interactively Running Images**

If you just want to jump into a standard image pulled from Docker Hub, or one you've built:

!!! tip ""

    See [Using Kali Linux Docker Images](https://www.kali.org/docs/containers/using-kali-docker-images/).

```bash
# Example
docker run --tty --interactive <tag/image-name>

# Download and run Kali
docker run --tty --interactive kalilinux/kali-rolling

# Download and run Fedora
docker run --tty --interactive fedora:latest

```

If the image you built uses systemd, you need to start it with `systemd` executed in the background first. The arguments required are the [same that you'd use for running molecule containers with systemd support](https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/#systemd-container). You can see an example of this in [geerlingguy's build.yml using GitHub actions to build and test Docker containers](https://github.com/geerlingguy/docker-rockylinux9-ansible/blob/4c366b8059f5bf993e30b7d38da37b9900b6f17f/.github/workflows/build.yml#L26).

- See the [`docker run` command reference](https://docs.docker.com/reference/cli/docker/container/run/#description)
- `-d` is most important here, it runs as a daemon in the background so `systemd` can start within the container as PID 1
- `--name` can be anything you want to name that instance of the running container
- `--hostname` is also independant of the container image name
- `local/kali-molecule` is the same arg as `-t <tag/name>` when you either pulled or built the image

```bash
docker run -d \
  --name kali-molecule \
  --hostname kali-molecule \
  --privileged \
  --cgroupns=host \
  --tmpfs /run \
  --tmpfs /tmp \
  -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
  -e container=docker \
  local/kali-molecule /sbin/init

```

*Then* interactively execute a shell in the running container once it starts:

```bash
docker exec -it kali-molecule /bin/bash

```


## CI / CD Use

Molecule has [examples](https://ansible.readthedocs.io/projects/molecule/ci/) for various CI platforms.

!!! tip "Change working-directory"

	Something worth noting for GitHub Actions specifically is using `working-directory: 'your_role'` to ensure your shell is running from within the root of your project folder when executing `molecule test`.

Here's the CI file I put together for [deploy_uac](https://github.com/straysheep-dev/ansible-role-deploy_uac), using the references noted in the comments as a starting point:

```yml
# .github/workflows/molecule.yml
# SPDX-License-Identifier: MIT

# Taken from the following examples:
# - https://ansible.readthedocs.io/projects/molecule/ci/#github-actions
# - https://github.com/geerlingguy/ansible-role-docker/blob/master/.github/workflows/ci.yml
# - https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/set-default-values-for-jobs

# Additional Notes:
# - Docker already exists on GitHub's runner images and does not need to be installed
# - https://github.com/actions/runner-images?tab=readme-ov-file#available-images
# - actions/setup-python@v5 can be used to install a specific version of python if needed
# - https://github.com/actions/setup-python

name: molecule
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
defaults:
  run:
    working-directory: 'deploy_uac'
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          path: deploy_uac
      - name: Install dependencies
        run: |
          python3 -m pip install --upgrade pip ansible molecule molecule-plugins[docker] docker
      - name: Test with molecule
        run: |
          molecule test -d docker
        env:
          PY_COLORS: '1'
          ANSIBLE_FORCE_COLOR: '1'

```
