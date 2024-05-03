---

hide:
  - navigation

---

# Main

---

=== "About"

    ## Hi, I'm straysheep-dev. :material-hand-wave:
    
    I'm here learning security from an offensive perspective and documenting things in a useful way as I go. 
    
    I also focus on building defensive (or "visibility") tools, and configuration templates learned from applying offensive techniques to systems.

    ### About This Site

    This site was created as a better way to document, maintain, and share notes with demonstrations or visual components, cross-platform.

    The [blog](blog/) section (at the top) is where this content lives, and is an easily searchable archive of anything I've found useful to demonstrate. Try using the :octicons-search-16: search function at the top of the page. It autocompletes suggestions from all of my content. Using [mkdocs](https://squidfunk.github.io/mkdocs-material/) to build this makes it both a searchable "database" with no backend, and an archive with everything in chronological order.

=== "Social"

    :material-github: [https://github.com/straysheep-dev](https://github.com/straysheep-dev)

    :material-gitlab: [https://gitlab.com/straysheep-dev](https://gitlab.com/straysheep-dev)

    :fontawesome-brands-discord: straysheep_dev
    
    :material-key: `9906 9EB1 2D40 9EA9 3BD1  E52E B09D 00AE C481 71E0`

=== "Certifications"

    <div class="grid cards" markdown>

    - :simple-dragonframe:{ .lg .middle } __OSCP__{ title="OffSec Certified Professional" }

        ---

        OffSec Certified Professional

    - :material-wifi:{ .lg .middle } __OSWP__{ title="OffSec Wireless Professional" }

        ---

        OffSec Wireless Professional

    - :material-server-network:{ .lg .middle } __PNPT__{ title="Practical Network Penetration Tester" }

        ---

        Practical Network Penetration Tester

    - :fontawesome-solid-bug-slash:{ .lg .middle } __eCMAP__{ title="Certified Malware Analysis Professional" }

        ---

        Certified Malware Analysis Professional

    - :fontawesome-solid-network-wired:{ .lg .middle } __eCPPT__{ title="Certified Professional Penetration Tester" }

        ---

        Certified Professional Penetration Tester

    - :material-server-network-outline:{ .lg .middle } __eJPT__{ title="Junior Penetration Tester" }

        ---

        Junior Penetration Tester

    </div>

---

## Pinned Repos

<div class="grid cards" markdown>

-   :simple-linux:{ .lg .middle } __Linux Configs__

    ---

    Various configuration files for Unix/Linux operating systems

    [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/linux-configs)

-   :material-microsoft-windows:{ .lg .middle } __Windows Configs__

    ---

    Various configuration files for Microsoft Windows operating systems

    [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/windows-configs)

-   :material-ansible:{ .lg .middle } __Ansible Configs__

    ---

    A collection of ansible roles

    [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/ansible-configs)

-   :material-terraform:{ .lg .middle } __Terraform Configs__

    ---

    Various configuration templates for terraform

    [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/terraform-configs)

-   :simple-vagrant:{ .lg .middle } __Vagrant Configs__

    ---

    Various notes and configurations for Vagrant

    [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/vagrant-configs)

-   :material-bell-ring:{ .lg .middle } __Alert Service__

    ---

    Send an alert (to Discord, Slack, or any webhook) based on a condition

    [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/alert-service)

</div>

---

## Useful Guides & Tools

=== "Automation"

    - Automated [terraforming](https://github.com/straysheep-dev/terraform-configs) of a [wireguard VPN server](https://github.com/straysheep-dev/ansible-configs/tree/main/build-wireguard-server) with interface monitoring
    - [Interactive deployment of auditd in Ubuntu / Fedora](https://github.com/straysheep-dev/setup-auditd)
    - [Interactive deployment of Sysinternals](https://github.com/straysheep-dev/windows-configs/blob/main/Manage-Sysinternals.ps1)
    - [Deploy and manage OpenSSH server on Windows](https://github.com/straysheep-dev/windows-configs/blob/main/Manage-OpenSSHServer.ps1)

=== "Home Labs"

    - [pfSense as an office router or in a home lab](https://github.com/straysheep-dev/cheatsheets/blob/main/pfsense/pfsense.md)

=== "Linux"

    - [VMware kernel module signing for SecureBoot on Ubuntu](https://github.com/straysheep-dev/linux-configs/blob/main/hypervisors/vmware/vmware-sign-modules.sh)

=== "Active Defense"

    - [Webhook based alerting (Slack, Discord...) for events, account access, honey files](https://github.com/straysheep-dev/alert-service)
