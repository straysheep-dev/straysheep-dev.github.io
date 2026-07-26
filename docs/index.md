---

title: "Main"
hide:
  - navigation

---

<!--
You can use HTML in MD pages to modify them.
https://github.com/squidfunk/mkdocs-material/discussions/2893
https://developer.mozilla.org/en-US/docs/Web/CSS/:first-of-type
https://developer.mozilla.org/en-US/docs/Web/CSS/display
-->

<style>
h1:first-of-type {
    display: none;
}
</style>

# Main

<div class="grid cards" markdown>

- :material-transit-connection-variant:{ .lg .middle } __Whoami__

    ---

    Hi, I'm straysheep-dev. :material-hand-wave:{ .middle }

    I'm here learning security from an offensive perspective and documenting things in a useful way as I go.

    I also focus on building defensive (or "visibility") tools, and configuration templates learned from applying offensive techniques to systems.

- :octicons-location-16:{ .lg .middle } __Connect__

    ---

    :material-github: [https://github.com/straysheep-dev](https://github.com/straysheep-dev)

    :material-gitlab: [https://gitlab.com/straysheep-dev](https://gitlab.com/straysheep-dev)

    :simple-ansible: [straysheep_dev](https://galaxy.ansible.com/ui/standalone/namespaces/22091/)

    :fontawesome-brands-discord: straysheep_dev

    :simple-hackthebox: straysheepdev

    :simple-tryhackme: [straysheep.dev](https://tryhackme.com/p/straysheep.dev)

    :material-key: [straysheep-dev/public-keys](https://github.com/straysheep-dev/public-keys)

</div>

---

=== "Pinned Repos"

    <div class="grid cards" markdown>

    - :simple-linux:{ .lg .middle } __Linux Configs__

        ---

        Utilities and configuration files for any Unix-like OS. [ansible-configs](#ansible-configs) generally uses files from here.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/linux-configs)

    - :material-microsoft-windows:{ .lg .middle } __Windows Configs__

        ---

        PowerShell modules and scripts used to configure windows or automate tasks. Targets Windows-desktop-specific items such as Hyper-V, Windows Sandbox, WSL + USB, logging & monitoring, and more.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/windows-configs)

    - :material-ansible:{ .lg .middle } __Ansible Configs__

        ---

        A collection of ansible roles, including example playbook and inventory files, as git submodules.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/ansible-configs)

        [:octicons-arrow-right-24: Ansible notes :octicons-book-16:](https://straysheep.dev/notes/ansible/)

        [:octicons-arrow-right-24: Molecule notes :octicons-book-16:](https://straysheep.dev/notes/ansible-molecule/)

    - :material-terraform:{ .lg .middle } __Terraform Configs__

        ---

        Ready to use terraform templates, and an easy to follow guide to go from installing, to deploying resources.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/terraform-configs)

    - :simple-vagrant:{ .lg .middle } __Vagrant Configs__

        ---

        How vagrant works; includes a sample Vagrantfile for Kali on Hyper-V and VirtualBox showing what provider settings you may want to use. Quirks of Hyper-V and how to resolve them are covered. Lastly, resources to be aware of for building a home lab are linked.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/vagrant-configs)

    - :simple-packer:{ .lg .middle } __Packer Configs__

        ---

        Packer templates ready-to-use, with resources to help you learn, modify, and build upon what's here. [ansible-configs](#ansible-configs) is a submodule of this super project.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/packer-configs)

    - :material-docker:{ .lg .middle } __Docker Configs__

        ---

        Systemd-enabled Docker configuration files for molecule testing. All submodules in [ansible-configs](#ansible-configs) run molecule CI workflows using these.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/docker-configs)

    - :material-bell-ring:{ .lg .middle } __Alert Service__

        ---

        Webhook based alerting (Slack, Discord) using [cyber deception](https://github.com/strandjs/IntroLabs/blob/master/IntroClassFiles/navigation.md) to catch malicious activity early, without noise.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/alert-service)

    - :material-shield-half-full:{ .lg .middle } __Compliance Profile Builder__

        ---

        Uses ComplianceAsCode's content release files and Ansible tags to easily create, test, and apply customized profiles

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/compliance-profile-builder)

    - :octicons-agent-16:{ .lg .middle } __Agent Configs__

        ---

        Agent configuration files to document and bootstrap workflows based on my existing codebase. Narrowly scoped agents build reusable, agent-free workflows, so they're only solving new problems.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/agent-configs)

    - :octicons-browser-16:{ .lg .middle } __Browser Configs__

        ---

        Browser configuration files. [Documentation](https://straysheep.dev/resources/#web-browsers).

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/browser-configs)

    </div>

=== "Certifications"

    <div class="grid cards" markdown>

    - :material-wifi:{ .lg .middle } __CWP__{ title="Certified WiFiChallenge Professional" }

        ---

        Certified WiFiChallenge Professional

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

=== "Guides & Utilities"

    <div class="grid cards" markdown>

    - :material-file-document-check-outline:{ .lg .middle } __OpenSCAP Practical Usage__

        ---

        A complete guide to starting with OpenSCAP content focusing on Ansible.

        - Install OpenSCAP
        - Pull compliance profiles from GitHub/ComplianceAsCode
        - Debug policies with Ansible's `-C` and `-D` options
        - Apply, test, and maintain policies with Ansible tags.

        [:octicons-arrow-right-24: Go to blog post :simple-materialformkdocs:](notes/openscap.md)

    - :simple-linux:{ .lg .middle } __Linux Utils__

        ---

        Visualization tools with built in parsing options **in color**. These tools are in the base of the linux-configs repo.

        `check-auditd.sh` Parse + search auditd

        `check-baseline.sh` Parse aide results

        `check-baseline.sh` rkhunter / chkrootkit in color

        `check-processes.sh` Dump system + network process in color

        `check-strings.sh` bstrings-like recursive string parser

        [:octicons-arrow-right-24: Go to blog post :simple-materialformkdocs:](#) CHECK BACK LATER!

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/linux-configs)

    - :simple-vmware:{ .lg .middle } __VMware Kernel Module Signing__

        ---

        To run VMware on Linux with SecureBoot enforced, the vmmon and vmnet modules require signing to load into the kernel.

        - Automates this process
        - Run after each kernel update

        [:octicons-arrow-right-24: Go to script :material-github:](https://github.com/straysheep-dev/linux-configs/blob/main/hypervisors/vmware/vmware-sign-modules.sh)

    - :simple-pfsense:{ .lg .middle } __pfSense Administration__

        ---

        This guide covers CLI usage and other things like:

        - Home office / lab use
        - pkgs for Zeek, sudo, and more
        - GUI and CLI quirks
        - External storage and ZFS

        [:octicons-arrow-right-24: Go to blog post :simple-materialformkdocs:](notes/pfsense.md)

    - :material-package-variant-plus:{ .lg .middle } __Deploy auditd__

        ---

        Installs and configures auditd to adhear to a specified policy on Debian / RedHat family systems.

        - Use built in rules for PCI, STIG, OSPP
        - Load your own custom rules instead
        - Choose log size, number, and type
        - Locks rules to prevent live modification

        [:octicons-arrow-right-24: Go to ansible role :material-github:](https://github.com/straysheep-dev/ansible-configs/tree/main/install_auditd)

        [:octicons-arrow-right-24: Go to shell script :material-github:](https://github.com/straysheep-dev/setup-auditd)

    - :material-package-variant-plus:{ .lg .middle } __Deploy & Manage Sysinternals__

        ---

        Interactive PowerShell script to load Sysinternals onto a Windows machine.

        - Deploys sysmon
        - Can update sysmon
        - Option to use SwiftOnSecurity config
        - Option to supply your own config instead
        - Option to add essential monitoring tools
        - Option to add entire suite (malware analysis)

        [:octicons-arrow-right-24: Go to ps1 script :material-github:](https://github.com/straysheep-dev/windows-configs/blob/main/Manage-Sysinternals.ps1)

    - :material-database-alert-outline:{ .lg .middle } __Deploy & Manage AIDE__

        ---

        Ansible role to deploy, run, and manage AIDE at scale

        - Install AIDE (advanced intrusion detection environment)
        - Initialize a database if one does not exist
        - Check existing systems for integrity
        - Update a database if one exists (optional)

        [:octicons-arrow-right-24: Go to ansible role :material-github:](https://github.com/straysheep-dev/ansible-configs/tree/main/aide)

    - :simple-wireguard:{ .lg .middle } __Wireguard VPN / IDS Server__

        ---

        Combines and automates a number of components to monitor traffic on a wireguard interface.

        - Provision with terraform
        - Build with ansible
        - [Generates a QR code to onboard first client](https://github.com/angristan/wireguard-install)
        - [Logs minimum pcap data for Zeek](https://www.activecountermeasures.com/filtering-out-high-volume-traffic/)
        - Set retention period for pcaps

        [:octicons-arrow-right-24: Go to ansible role :material-github:](https://github.com/straysheep-dev/ansible-configs/tree/main/build_wireguard_server)

    - :simple-tailscale:{ .lg .middle } __Build Tailscale Node__

        ---

        Automates deployment of a Tailscale node.

        - Provision with terraform
        - Build with ansible
        - Automatically enroll the node into your Tailnet with an Ansible vault
        - [Logs minimum pcap data for Zeek on the tailscale0 interface](https://www.activecountermeasures.com/filtering-out-high-volume-traffic/)

        [:octicons-arrow-right-24: Go to ansible role :material-github:](https://github.com/straysheep-dev/ansible-configs/tree/main/build_tailscale_node)

    - :material-tools:{ .lg .middle } __Manage OpenSSH Server on Windows__

        ---

        OpenSSH Server is not always available by default, and is time consuming to configure each deployment manually.

        - Installs + modifies OpenSSH Server
        - Enforces public key auth
        - Can change the listening port
        - Updates firewall rules
        - Imports public keys

        [:octicons-arrow-right-24: Go to ps1 script :material-github:](https://github.com/straysheep-dev/windows-configs/blob/main/Manage-OpenSSHServer.ps1)

    - :material-reload-alert:{ .lg .middle } __Tail-EventLogs PS Module__

        ---

        Windows has no `tail -f` equivalent to visualize live Event Logs. This is especially useful in tuning and testing sysmon rules locally.

        - Can tail any event log
        - Filter based on Event ID
        - Write to file with `Tee-Object`

        [:octicons-arrow-right-24: Go to ps1 module :material-github:](https://github.com/straysheep-dev/windows-configs/blob/main/Tail-EventLogs.ps1)

    - :material-package-variant-plus:{ .lg .middle } __Windows Sandbox Configs__

        ---

        Detailed examples and premade .wsb files for:

        - Ephemeral environment
        - Development environment
        - Malware analysis

        The .wsb files and [scripts](https://github.com/straysheep-dev/windows-configs/tree/main/Windows-Sandbox) are in the base of the windows-configs repo.

        [:octicons-arrow-right-24: Go to repo :material-github:](https://github.com/straysheep-dev/windows-configs#windows-sandbox)

    - :fontawesome-brands-usb:{ .lg .middle } __Connect-UsbipSSHTunnel PS Module__

        ---

        Convenience script to open a reverse ssh tunnel to the Windows host from WSL, giving WSL access to usbipd devices on localhost tcp/3240 without any inbound firewall rules active on the host.

        Requirements:

        - WSL is up to date
        - [usbipd](https://github.com/dorssel/usbipd-win) is version 4.0.0 or later
        - The Windows host has an ssh key that the target WSL instance will accept
        - The ssh identity is loaded into Windows ssh-agent
        - WSL accepts incoming ssh connections
        - You can execute commands as admin (this script can run as a normal user but you need to know an admin's credentials)
        - You have sudo privileges within WSL

        [:octicons-arrow-right-24: Go to ps1 module :material-github:](https://github.com/straysheep-dev/windows-configs/blob/main/Connect-UsbipSSHTunnel.ps1)

    - :simple-gnubash:{ .lg .middle } __Customizing Shell Profiles__

        ---

        Use your shell prompt to track the following (and more) in real time:

        - Username
        - Hostname
        - TTY
        - Date & time
        - Network interface information
        - Working directory

        [:octicons-arrow-right-24: Go to blog post :simple-materialformkdocs:](blog/posts/custom-shell-profiles.md)

    </div>

<div class="grid cards" markdown>

- :octicons-info-16:{ .middle } __About__

    ---

    This site was created as a better way to document, maintain, and share notes with demonstrations or visual components, cross-platform.

    The [blog](blog/index.md) section (at the top) is where this content lives, and is an easily searchable archive of anything I've found useful to demonstrate. Try using the :octicons-search-16: search function at the top of the page. It autocompletes suggestions from all of my content.

    Using [Zensical](https://zensical.org/) to build this makes it both a searchable "database" with no backend, and an archive with everything organized through git.

</div>
