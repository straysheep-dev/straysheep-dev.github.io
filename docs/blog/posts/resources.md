---
draft: false
date:
  created: 2019-07-15
categories:
  - links
  - resources
---

# ⭐ Resources

Information, compiled for easy reference.

!!! abstract "CI/CD Your Notes"

    This set of links and notes has been my longest running note file, originally started back in cherrytree before making its way to this page.

    The idea has been to create a searchable location pointing to each of these things, sorted by category, and with notes around them. This is currently a work in progess as entries need to be reformatted and updated as they move from my notes onto this page.

<!-- more -->


## Utilities

??? info "rustdesk"

	An open source, robust remote desktop alternative, has desktop and mobile clients and is designed for self-hosting.

	This was [covered by Network Chuck](https://www.youtube.com/watch?v=EXL8mMUXs88) as an alternative to every other remote desktop option out there.

	- <https://github.com/rustdesk/rustdesk>


??? info "ghostty"

	Fast, feature rich, native terminal emulator.

	This tool was mentioned on Daniel Miessler's [UL NO. 463](https://newsletter.danielmiessler.com/p/ul-463).

	- <https://ghostty.org/>
	- <https://github.com/ghostty-org/ghostty>

??? question "unfurl"

	> Unfurl takes a URL and expands ("unfurls") it into a directed graph, extracting every bit of information from the URL and exposing the obscured. It does this by breaking up a URL into components, extracting as much information as it can from each piece, and presenting it all visually.

	Discovered in [this SANS diary](https://isc.sans.edu/forums/diary/Unfurl%20v2025.02%20released/31716/). This tool has both a local web interface and a command line option for viewing the data returned.

	- <https://github.com/obsidianforensics/unfurl>

??? example "bubbletea"

	> The fun, functional and stateful way to build terminal apps. A Go framework based on The Elm Architecture. Bubble Tea is well-suited for simple and complex terminal applications, either inline, full-window, or a mix of both.

	- <https://github.com/charmbracelet/bubbletea>

??? bug "Dangerzone"

	> Take potentially dangerous PDFs, office documents, or images and convert them to a safe PDF.
	>
	> Dangerzone works like this: You give it a document that you don't know if you can trust (for example, an email attachment). Inside of a sandbox, Dangerzone converts the document to a PDF (if it isn't already one), and then converts the PDF into raw pixel data: a huge list of RGB color values for each page. Then, outside of the sandbox, Dangerzone takes this pixel data and converts it back into a PDF.

	- <https://github.com/freedomofpress/dangerzone>

	List of features (from the README):

	- Sandboxes don't have network access, so if a malicious document can compromise one, it can't phone home
	- Sandboxes use gVisor, an application kernel written in Go, that implements a substantial portion of the Linux system call interface.
	- Dangerzone can optionally OCR the safe PDFs it creates, so it will have a text layer again
	- Dangerzone compresses the safe PDF to reduce file size
	- After converting, Dangerzone lets you open the safe PDF in the PDF viewer of your choice, which allows you to open PDFs and office docs in Dangerzone by default so you never accidentally open a dangerous document

??? question "magika"

	> Magika is a novel AI powered file type detection tool that relies on the recent advance of deep learning to provide accurate detection. Under the hood, Magika employs a custom, highly optimized Keras model that only weighs about a few MBs, and enables precise file identification within milliseconds, even when running on a single CPU.

	- <https://github.com/google/magika>

??? info "Ladybird"

	> Ladybird is a truly independent web browser, using a novel engine based on web standards.

	Discovered by following [p0dalirius](https://github.com/p0dalirius?tab=stars), this appears to be focused on *nix and macOS desktop environments for it's first release in 2026 / 2027.

	- <https://github.com/LadybirdBrowser/ladybird>

??? info "topgrade"

	Discovered on [Paul's Security Weekly #864](https://securityweekly.com/psw-864). topgrade is a way to "upgrade all the things" on a Linux system. This includes apt/dnf, snap, flatpak, LVFS / firmware, and so on.

	- <https://github.com/topgrade-rs/topgrade>


## Note Taking

The best advice I've heard about note taking is 1) it should work for you, and 2) it should export to a common format like Markdown so you can move to another notes platform easily. This can be multiple apps, or just one. It's whatever works best for you and the goals you have.

??? note "Standard-Notes"

	> Standard Notes is an end-to-end encrypted note-taking app for digitalists and professionals. Capture your notes, files, and life's work all in one secure place.

	It works across all major platforms, desktop and web, using end-to-end encrypted syncing through their servers to all of your devices. Robust markdown formatting, a secrets vault, and even limited spreadsheet capability for paid subscriptions. Standard Notes is one of the most security and privacy focused of the various note taking applications, making efforts to function similar to a password manager with how it handles memory and data on-device.

	You could also self-host the client and server components, using tailscale to safely access it from your endpoints.

	- <https://standardnotes.com/>
	- <https://github.com/standardnotes/app>
	- License: AGPL-3.0

??? note "Obsidian"

	Perhaps the most popular notes application for infosec professionals and developers (as of the time of writing this). Obsidian features robust note taking and visualization capabilities through linking notes and creating a graph of how they relate. It works using markdown files saved to your device. Paid plans include end-to-end encrypted note sync.

	- <https://obsidian.md/>
	- <https://github.com/obsidianmd>

??? note "Joplin"

	> Joplin is a free, open source note taking and to-do application, which can handle a large number of notes organised into notebooks. The notes are searchable, can be copied, tagged and modified either from the applications directly or from your own text editor. The notes are in Markdown format. The notes can be securely synchronised using end-to-end encryption with various cloud services including Nextcloud, Dropbox, OneDrive and Joplin Cloud.

	This application is often referenced in relation to note taking for pentesting courses and certifications. It's fairly easy to install and functions entirely offline by default.

	- <https://github.com/laurent22/joplin>
	- License: AGPL-3.0-or-later

??? note "Notion"

	Notion is a note taking platform. Desktop apps are available but you can use it entirely through the web application. It is possibly the most feature-rich note taking platform available. If your subscription includes the AI component, it's practical and immediately usable out of the box.

	These notes are not end-to-end encrypted, and may not be the best option without an enterprise subscription if your notes include customer data.

	- <https://www.notion.com/>

??? note "Cherrytree"

	> A hierarchical note taking application, featuring rich text and syntax highlighting, storing data in either a single file (xml or sqlite) or multiple files and directories.

	This is often available by default in Kali Linux and frequently referenced in relation to note taking for pentesting courses and certifications.

	Primarily supports Windows and Linux (snap and flatpaks are available too).

	- <https://github.com/giuspen/cherrytree>
	- License: GPL-3.0


## Operating Systems

??? info "Windows"

	Each of the ISOs and disk images are openly available to download, both for evaluation and to enter a product key during install for produciton use.

	- [Direct Download Links to ISO's / VHD's](https://techcommunity.microsoft.com/t5/windows-11/accessing-trials-and-kits-for-windows/m-p/3361125)
	- [Windows ISOs (Production Use)](https://www.microsoft.com/en-us/download/windows)
	- [Windows ISOs (Evaluation Use)](https://www.microsoft.com/en-us/evalcenter)
	- [Windows Developer Evaluation VM](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)
	- [Windows 7/8/10 Legacy VM](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)

??? danger "Kali Linux"

	The most robust pentesting Linux distribution. Includes tools for offense, purple teaming, defense, and forensics.

	- <https://cdimage.kali.org/>
	- <https://www.kali.org/downloads/>
	- <https://www.kali.org/docs/introduction/download-images-securely/>
    - For older images of kali: <https://old.kali.org/>

	Obtaining the Kali Linux GPG public key and verifying signatures:

	```bash
	wget -q -O - https://archive.kali.org/archive-key.asc | gpg --import
	gpg --keyserver hkps://keyserver.ubuntu.com --recv-key 44C6513A8E4FB3D30875F758ED444FF07D8D0BF6
	wget -q https://cdimage.kali.org/current/SHA256SUMS{.gpg,}
	gpg --verify SHA256SUMS.gpg SHA256SUMS
	```

    GPG Key File: [0xED444FF07D8D0BF6](https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x44c6513a8e4fb3d30875f758ed444ff07d8d0bf6)

    ```bash
    pub   rsa4096/0xED444FF07D8D0BF6 2012-03-05 [SC] [expires: 2027-02-04]
          Key fingerprint = 44C6 513A 8E4F B3D3 0875  F758 ED44 4FF0 7D8D 0BF6
    uid                             Kali Linux Repository <devel@kali.org>
    sub   rsa4096/0xA8373E18FC0D0DCB 2012-03-05 [E] [expires: 2027-02-04]
    ```

??? bug "REMnux"

	A Linux toolkit for malware analysts. [Lenny Zeltser](https://zeltser.com/) is [one of the maintainers](https://remnux.org/#people).

	- <https://remnux.org/>


??? abstract "Debian"

	Debian is one of the "main" operating system families of Linux.

	> The Debian Project is an association of individuals, sharing a common goal: We want to create a free operating system, freely available for everyone. Now, when we use the word "free", we're not talking about money, instead, we are referring to software freedom.

	- <https://www.debian.org>
	- <https://www.debian.org/CD/verify>
	- <https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/>
    - For older images of Debian: <https://cdimage.debian.org/mirror/cdimage/archive/>

	GPG Key:

	```bash
	gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys 'DF9B 9C49 EAA9 2984 3258  9D76 DA87 E80D 6294 BE9B'
	```

	To automate installs, Debian uses `preseed.cfg` files.

	- [debian.org/DebianInstaller/Preseed](https://wiki.debian.org/DebianInstaller/Preseed)
	- [debian.org example-preseed.txt Template](https://www.debian.org/releases/stable/example-preseed.txt)
	- [debian.org/tasksel](https://wiki.debian.org/tasksel)

??? tip "Ubuntu"

	Ubuntu is a Debian-base Linux distribution developed by Canonical.

	- <https://releases.ubuntu.com> (main images)
    - `https://old-releases.ubuntu.com/releases/$VERSION/` (is how you can version pin URLs to ISOs and signatures, this contians all released images in one page per version)
	- <https://cdimage.ubuntu.com/releases/> (rpi + alternate flavors)
    - <https://cloud-images.ubuntu.com/> (vagrant and cloud provider images)
	- <https://ubuntu.com/download/raspberry-pi>
	- <https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi>

    GPG Key File: [0xD94AA3F0EFE21092](https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x843938df228d22f7b3742bc0d94aa3f0efe21092)

	```bash
    pub   rsa4096/0xD94AA3F0EFE21092 2012-05-11 [SC]
          Key fingerprint = 8439 38DF 228D 22F7 B374  2BC0 D94A A3F0 EFE2 1092
    uid   Ubuntu CD Image Automatic Signing Key (2012) <cdimage@ubuntu.com>
    ```

??? tip "Raspberry Pi OS"

	> Raspberry Pi needs an operating system to work. This is it. Raspberry Pi OS (previously called Raspbian) is our official supported operating system.

	Many Linux distributions have a version available to run on Raspberry Pi, however there's also Raspberry Pi OS which is built and maintained by the Raspberry Pi Foundation.

	- Verify the .sig file against the img.xz compressed file, not the SHA signatures.
	- <https://www.raspberrypi.org/about/> links to raspberrypi.com
    - <https://www.raspberrypi.com/>
	- <https://www.raspberrypi.com/software/operating-systems/> (links to main images, use the archive link to obtain the .sig)
    - <https://downloads.raspberrypi.com/raspios_arm64/images/> (folder for standard desktop download)
    - <https://github.com/raspberrypi>
    - <https://www.raspberrypi.org/raspberrypi_downloads.gpg.key> GPG key, indexed by search engines

??? abstract "RHEL (Red Hat Enterprise Linux)"

	> Red Hat Enterprise Linux is an enterprise Linux operating system. It is oriented toward enterprise and commercial users, is certified for many hardware and cloud platforms, and is supported by Red Hat via various subscription options. Compared to Fedora, Red Hat Enterprise Linux emphasizes stability and enterprise-readiness over the latest technologies or rapid releases. More information about Red Hat offerings can be found at [Red Hat's web site](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux).
	>
	> Individual software developers can access a free-of-charge subscription as part of the [Red Hat Developer Program](https://developers.redhat.com/about). Developers can use Red Hat Enterprise Linux on up to 16 physical or virtual systems for development, quality assurance, demos, or small production uses. See the Frequently Asked Questions for the [No-cost Red Hat Enterprise Linux Individual Developer Subscription](https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux).

	- <https://www.redhat.com>
	- <https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux>

??? tip "Fedora"

	> Fedora is developed by the Fedora Project and sponsored by Red Hat. It follows its own release schedule, with a new version approximately every six months. Fedora provides a modern Linux operating system utilizing many of the latest technologies. It is free for all users and supported via the Fedora community.
	>
	> To create Red Hat Enterprise Linux, some version of Fedora is forked and enters an extensive development, testing and certification process to become a new version of Red Hat Enterprise Linux.

	- <https://getfedora.org>
    - <https://fedoraproject.org/security> (GPG Keys)
	- <https://docs.fedoraproject.org>
    - <https://hub.docker.com/_/fedora>

	To automate installs, Fedora uses [Kickstart files](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/advanced/Kickstart_Installations/).

??? danger "Pentoo"

	- <https://pentoo.ch/>


??? danger "Parrot"

	> Parrot Security (ParrotOS, Parrot) is a Free and Open source GNU/Linux distribution based on Debian Stable designed for security experts, developers and privacy aware people.
	>
	> It includes a full portable arsenal for IT security and digital forensics operations. It also includes everything you need to develop your own programs or protect your privacy while surfing the net.
	>
	> Parrot is available in three main editions, Security, Home and Architect Edition, even as Virtual Machine (Virtual Box, Parallels and VMware), on Raspberry Pi and also on Docker.
	>
	> The operating system ships by default with MATE Desktop Environment, but it is possible to install others DEs.

	- <https://www.parrotsec.org/>

??? tip "Arch Linux"

	- <https://archlinux.org/>
	- <https://archlinux.org/download/>

??? info "OpenBSD"

	- <https://www.openbsd.org/>
    - <https://www.openbsd.org/faq/faq4.html#Download>

??? info "FreeBSD"

	- <https://www.freebsd.org/where/>
    - <https://docs.freebsd.org/en/articles/pgpkeys/>
	- <https://docs.freebsd.org/pgpkeys/pgpkeys.txt>

??? tip "pfSense"

	pfSense is an excellent choice for both, a home *and* lab router-firewall to begin learning with and protect your real network.

	> The pfSense project is a free network firewall distribution, based on the FreeBSD operating system with a custom kernel and including third party free software packages for additional functionality. pfSense software, with the help of the package system, is able to provide the same functionality or more of common commercial firewalls, without any of the artificial limitations. It has successfully replaced every big name commercial firewall you can imagine in numerous installations around the world, including Check Point, Cisco PIX, Cisco ASA, Juniper, Sonicwall, Netgear, Watchguard, Astaro, and more.

	- <https://www.pfsense.org/download/>
	- <https://github.com/pfsense/>

??? example "OpenWRT"

	Tracking latest stable release notes

	- <https://openwrt.org/releases/start>
	- `https://openwrt.org/releases/<version>/start`
	- `https://openwrt.org/releases/<version>/notes-<version>`

	Downloading latest stable releases

	- <https://downloads.openwrt.org/releases/>
	- <https://openwrt.org/toh/views/toh_fwdownload> (Ctrl+F search device name)
	- <https://openwrt.org/docs/guide-user/security/signatures>

	For UniFi AP AC Lite:

	- `https://downloads.openwrt.org/releases/<version>/targets/<target>/<type>`
	- <https://downloads.openwrt.org/releases/22.03.0/targets/ath79/generic/> (ath79 is the latest target)
		- ubnt_unifiac-lite-squashfs-sysupgrade.bin
		- sha256sums
		- sha256sums.asc

??? tip "Ubiquiti"

	- <https://www.ui.com/download/>
	- <https://dl.ui.com/unifi/firmware/U7PG2/3.7.58.6385/BZ.qca956x.v3.7.58.6385.170508.0957.bin> (UniFi AP AC Lite firmware v3.7.58)

??? tip "VyOS"

	> VyOS is a fully open-source Linux-based OS for network devices. It focuses on enterprise, service provider, and network geek audiences.

	It's free to build and use, and nightly prerelease ISO's are available, however they operate on a "pay for prebuilt binaries", plus technical support and custom development services if you'd like to support the project.

	- <https://github.com/vyos/>
	- <https://vyos.net/get/nightly-builds/> (this details verifying the signatures with minisign)
		- <https://github.com/vyos/vyos-nightly-build/blob/main/minisign.pub>
	- <https://github.com/vyos/vyos-nightly-build/releases>

??? info "TrueNAS SCALE"

	- <https://github.com/truenas>
	- <https://www.truenas.com/docs/>
	- <https://www.truenas.com/download-truenas-scale/>
    - PGP Key: `C8D6 2DEF 767C 1DB0 DFF4 E6EC 358E AA91 12CF 7946`

??? info "openmediavault"

	> openmediavault is the next generation network attached storage (NAS) solution based on Debian Linux. It contains services like SSH, (S)FTP, SMB/CIFS, RSync and many more. Thanks to the modular design of the framework it can be enhanced via plugins. openmediavault is primarily designed to be used in home environments or small home offices, but is not limited to those scenarios. It is a simple and easy to use out-of-the-box solution that will allow everyone to install and administrate a Network Attached Storage without deeper knowledge.
	>
	> Note: openmediavault (like other NAS solutions) expects to have full, exclusive control over OS configuration and cannot be used within a container. Also, no graphical desktop user interface can be installed in parallel.

	- <https://github.com/openmediavault/openmediavault>
	- <https://docs.openmediavault.org/en/stable/installation/index.html>


## Hypervisors

??? example "Proxmox"

	Proxmox is a complete open-source platform for virtualization. Built on Debian, it uses a web frontend to manage VM's and containers. You can install and use GUI and headless VM's, as well as manage and work with the underlying OS from a shell. For example it's entirely possible to install an EDR agent onto Proxmox, since it's Debian under the hood.

	- <https://www.proxmox.com/en/>
	- <https://pve.proxmox.com/pve-docs/>

	The ISO URL points to an /iso/ folder on the Proxmox webiste. Browsing this manually reveals the following files:

	<https://enterprise.proxmox.com/iso/SHA256SUMS.txt>
	<https://enterprise.proxmox.com/iso/SHA256SUMS.asc>

	You can use these along with the following public key to verify the ISO's integrity.

	```bash
	gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys 'F4E136C67CDCE41AE6DE6FC81140AF8F639E0C39'

	# list keys
	pub   rsa4096/0x1140AF8F639E0C39 2022-11-27 [SC] [expires: 2032-11-24]
		Key fingerprint = F4E1 36C6 7CDC E41A E6DE  6FC8 1140 AF8F 639E 0C39
	uid                   [ unknown] Proxmox Bookworm Release Key <proxmox-release@proxmox.com>
	```

??? example "VMware"

	⚠️ TO DO ⚠️

??? example "VirtualBox"

	VirtualBox is often available in the default Linux repositories, but the **latest supported and patched** version is available directly from VirtualBox's offical repo.

	> VirtualBox is a general-purpose full virtualization software for x86_64 hardware (with version 7.1 additionally for macOS/Arm), targeted at laptop, desktop, server and embedded use.

	It works on Windows, macOS, Linux, and more. You'll also want the VirtualBox Extension Pack if you require USB pass-through or any other advanced features. It's free for personal use and is under a separate license agreement.

	- <https://www.virtualbox.org/>
	- <https://www.virtualbox.org/download/oracle_vbox_2016.asc>

	```txt
	B9F8 D658 297A F3EF C18D  5CDF A2F6 83C5 2980 AECF
	Oracle Corporation (VirtualBox archive signing key) <info@virtualbox.org>
	```

??? example "QEMU"

	QEMU is available through your Linux distro's default package repos, and macOS's Homebrew / MacPorts. This is the recommended way to install it.

	QEMU could be thought of as the Hyper-V of Linux. It benefits from the KVM acceleration on Linux making the performance incredible, and in some sense is the most ubiquitously supported hypervisor across the distros, likely because of the KVM integration. You'll see this on Ubuntu where `apt` now checks to see if QEMU VM's are running.

	This is just my opinion and experience after moving to Hyper-V and QEMU from VMWare and VirtualBox. QEMU is also being used by Proxmox, which is another benefit to building and understanding QEMU images if you're using QEMU for desktop use cases and Proxmox as a virtualization server.

	- <https://www.qemu.org/download/>
	- <https://www.qemu.org/documentation/>
	- <https://gitlab.com/qemu-project/qemu>


## Labs & Simulations

??? danger "Atomic Red Team"

	[Invoke-AtomicReadTeam](https://github.com/redcanaryco/invoke-atomicredteam) is an execution framework in PowerShell to run [Atomic Tests](https://github.com/redcanaryco/atomic-red-team/wiki/Getting-Started#choose-a-test), which are adversary emulation tests mapped to MITRE ATT&CK.

	- <https://github.com/redcanaryco/atomic-red-team>

??? danger "PANIX (Persistence Against *NIX)"

	> Customizable Linux Persistence Tool for Security Research and Detection Engineering.

	- <https://github.com/Aegrah/PANIX>
	- [Linux Detection Engineering - A Primer on Persistence Mechanisms](https://www.elastic.co/security-labs/primer-on-persistence-mechanisms)
	- [Linux Detection Engineering - A Sequel on Persistence Mechanisms](https://www.elastic.co/security-labs/sequel-on-persistence-mechanisms)
	- [Linux Detection Engineering - A Continuation on Persistence Mechanisms](https://www.elastic.co/security-labs/continuation-on-persistence-mechanisms)
	- [Linux Detection Engineering - Approaching the Summit on Persistence Mechanisms](https://www.elastic.co/security-labs/approaching-the-summit-on-persistence)
	- [Linux Detection Engineering - The Grand Finale on Linux Persistence](https://www.elastic.co/security-labs/the-grand-finale-on-linux-persistence)

??? example "GOAD (Game of Active Directory)"

	> The purpose of this tool is to give pentesters a vulnerable Active directory environment ready to use to practice usual attack techniques. The idea behind this project is to give you an environment where you can try and train your pentest skills without having the pain to build all by yourself. This repository was build for pentest practice.

	Effectively, this can spin up really fast (as quick as a couple hours) on a Kali Linux host running VirtualBox as the Hypervisor. There's also a Proxmox option, but for this you may want to look at Ludus below.

	It goes without saying you need a fair amount of resources to run this, and it will be extremely expensive in the cloud.

	- <https://github.com/Orange-Cyberdefense/GOAD>
	- <https://orange-cyberdefense.github.io/GOAD/>

??? example "Ludus"

	This was initially discovered on [Paul's Security Weekly #861](https://securityweekly.com/psw-861). It appears to be built on top of Proxmox *and* Game of AD, with a number of other options and features for a completely automated home lab experience.

	> Ludus is a system to build easy-to-use cyber environments, or "ranges" for testing and development.
	>
	> Built on Proxmox, Ludus enables advanced automation while still allowing easy manual modifications or setup of virtual machines and networks.

	- <https://gitlab.com/badsectorlabs/ludus>

??? example "SamuraiWTF (Web Training Framework)"

	This version of the project is expirimental only for now.

	- <https://github.com/secureideas/samuraiwtf>


??? example "VulnServer"

	An intentionally vulnerable listening process, meant to demonstrate buffer overflows.

	This project is referenced in many buffer overflow practice courses. The source code is short enough to read over in a few minutes and can be compiled fairly easily, meaning you can make changes and attempt to patch the code to remove the vulnerability too.

	- <https://github.com/stephenbradshaw/vulnserver>

??? example "OWASP Juice Shop"

	> OWASP Juice Shop is probably the most modern and sophisticated insecure web application! It can be used in security trainings, awareness demos, CTFs and as a guinea pig for security tools! Juice Shop encompasses vulnerabilities from the entire OWASP Top Ten along with many other security flaws found in real-world applications!

	- <https://github.com/juice-shop>
	- <https://github.com/bkimminich/juice-shop#docker-container>
	- <https://github.com/juice-shop/pwning-juice-shop>

??? example "VulHub"

	Dockerized Vulnerable Software. These should be run in an isolated environment, such as a VM you would use for malware analysis.

	- <https://github.com/vulhub/vulhub>

??? example "Snyk Goof"

	> A vulnerable Node.js demo application.

    - <https://github.com/snyk-labs/nodejs-goof>


## DevOps


### Ansible

??? example "pfsensible"

	Ansible modules for managing a pfSense firewall.

	- <https://github.com/pfsensible/core>


### HashiCorp

!!! abstract "HashiCorp Utilities"

	> Use infrastructure as code to build, deploy, and manage the infrastructure that underpins cloud applications.

	Validate your installs with one of these public keys:

	- [HashiCorp public keys](https://www.hashicorp.com/trust/security) `C874 011F 0AB4 0511 0D02 1055 3436 5D94 72D7 468F`
	- [HashiCorp public keys on keybase.io](https://keybase.io/hashicorp) `C874 011F 0AB4 0511 0D02 1055 3436 5D94 72D7 468F`
	- [HashiCorp apt gpg key](https://apt.releases.hashicorp.com/gpg) `798A EC65 4E5C 1542 8C8E 42EE AA16 FCBC A621 E701`
	- [How to Verify a Hashicorp Binary](https://developer.hashicorp.com/well-architected-framework/operational-excellence/verify-hashicorp-binary)

??? example "Packer"

	> Packer is a tool that lets you create identical machine images for multiple platforms from a single source template. Packer can create golden images to use in image pipelines.

	- <https://developer.hashicorp.com/packer>

	Packer literally builds virtual machines images. Think of installing Kali or Windows from the ISO manually in VMware, Hyper-V, VirtualBox, QEMU, or anywhere else (supported by packer). Packer actually automates those steps, even down to the boot key presses so you can build and configure a VM with zero interaction. You can see this happen in real time if you aren't running packer in headless mode. The VM GUI window will open as if you were doing the install yourself, and you can watch it run.

	To really utilize packer, you'll need to learn and implemenet (depending on the OS and technology available) cloud-init, autoinstall, preseed, or similar auto-provisioning mechanisms so you don't have to manually intervene.

	You can also post-process the install with additional shell commands, scripts, Ansible playbooks, and more.

	The resulting images can be imported into a hypervisor, sent to a cloud provider as the disk image for a cloud VM, or converted into a Vagrant box image.

??? example "Vagrant"

	> Vagrant is the command line utility for managing the lifecycle of virtual machines. Isolate dependencies and their configuration within a single disposable and consistent environment.

	- <https://developer.hashicorp.com/vagrant>

??? example "Terraform"

	> Terraform is an infrastructure as code tool that lets you build, change, and version infrastructure safely and efficiently. This includes low-level components like compute instances, storage, and networking; and high-level components like DNS entries and SaaS features.

	- <https://developer.hashicorp.com/terraform>



## Information Technology

??? tip "RFC"

	Request for Comments.

	Get the full story on the [about page](https://www.rfc-editor.org/about/), but effectively RFC's are published and reviewed documentation that can include information about various internet standards.

	- <https://www.rfc-editor.org/retrieve/>

??? info "IETF"

	Internet Engineering Task Force.

	> The Internet Engineering Task Force (IETF), founded in 1986, is the premier standards development organization (SDO) for the Internet. The IETF makes voluntary standards that are often adopted by Internet users, network operators, and equipment vendors, and it thus helps shape the trajectory of the development of the Internet. But in no way does the IETF control, or even patrol, the Internet.

	- <https://www.ietf.org/>

??? info "InterNIC"

	Public Information Regarding Internet Domain Name Registration Services.

	> InterNIC is a registered service mark of the U.S. Department of Commerce. It is licensed to the Internet Corporation for Assigned Names and Numbers, which operates this web site.

	- <https://www.internic.net>
	- [Named root-hints file, use curl or wget](https://www.internic.net/domain/named.root)

??? info "ICANN"

	The Internet Corporation for Assigned Names and Numbers.

	> ICANN's mission is to help ensure a stable, secure, and unified global Internet. To reach another person on the Internet, you need to type an address - a name or a number - into your computer or other device. That address must be unique so computers know where to find each other.
	>
	> ICANN helps coordinate and support these unique identifiers across the world. ICANN was formed in 1998 as a nonprofit public benefit corporation with a community of participants from all over the world.

	- <https://www.icann.org/>
	- <https://lookup.icann.org/>

??? info "IANA"

	Internet Assigned Numbers Authority

	> The global coordination of the DNS Root, IP addressing, and other Internet protocol resources is performed as the Internet Assigned Numbers Authority (IANA) functions.

	- <https://www.iana.org/>
	- [DNS response codes](https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml)

??? info "ARIN"

	American Registry for Internet Numbers.

	> Established in December 1997, the American Registry for Internet Numbers (ARIN) is a nonprofit, member-based organization that supports the operation and growth of the Internet.
	>
	> ARIN accomplishes this by carrying out its core service, which is the management and distribution of Internet number resources such as Internet Protocol (IP) addresses and Autonomous System Numbers (ASNs). ARIN manages these resources within its service region, which is comprised of Canada, the United States, and many Caribbean and North Atlantic islands. ARIN also coordinates policy development by the community and advances the Internet through informational outreach.

	You can perform WHOIS lookups and obtain ownership information on network ranges and addresses through ARIN.

	- <https://www.arin.net/>

??? info "MDN (Mozilla Developer Network) Documentation"

	This is an essential resource for anything web standards (or web code) related.

	> MDN Web Docs is an open-source, collaborative project that documents web platform technologies, including CSS, HTML, JavaScript, and Web APIs. We also provide extensive 🧑‍🎓 learning resources for beginning developers and students.

	- <https://github.com/mdn>
	- [Web Specification Docs](https://github.com/mdn/content/)

??? info "W3C (World Wide Web Consortium)"

	> An international community that develops open standards to ensure the long-term growth of the Web.

	- <https://github.com/w3c>
	- <https://w3c.github.io/html-reference/>

??? info "Usb Specifications & Documentation"

	usb.org details everything about the USB specification. This is useful for example if you're [observing USB communications with Wireshark](https://wiki.wireshark.org/CaptureSetup/USB) or [building rules for USBGuard](https://usbguard.github.io/documentation/rule-language.html).

	- <https://usb.org/>
	- <https://www.usb.org/defined-class-codes>


??? info "FCCID"

	FCC documentation and searchable databases.

	- <https://www.fcc.gov/oet/ea/fccid>
	- <https://fccid.io>


## Information Security


### DNS

??? info "Cloudflare 1.1.1.1"

	> 1.1.1.1 offers an encrypted service through DNS over HTTPS (DoH) or DNS over TLS (DoT) for increased security and privacy.

	- <https://developers.cloudflare.com/dns/>
	- <https://developers.cloudflare.com/1.1.1.1/>

	To see if you're on 1.1.1.1, go to <https://one.one.one.one/help/>

??? info "Quad9 9.9.9.9"

	> Quad9 is a free service that replaces your default ISP or enterprise Domain Name Server (DNS) configuration.

	Features include DNS over TLS, HTTPS, DNSSEC, threat blocking, and more.

	- <https://quad9.net/>

	To see if you're on Quad9, to to <https://on.quad9.net/>.

??? info "NextDNS"

	NextDNS is essentially a web-based solution to give you full visibility and control over your DNS usage. There are both free and paid plans.

	- <https://nextdns.io/>

	To see if you're on NextDNS, go to <https://test.nextdns.io/>.

## Offense

!!! abstract "PTES (Penetration Testing Execution Standard)"

	The PTES is a standard first drafted in 2009. It's designed to provide both businesses and security service providers with a common language and scope for performing penetration testing. It's been referenced in a number of training courses and by those who helped create it over the years. [The FAQ provides additonal overview](https://github.com/OpenSBK/ptes/blob/main/faq.md).

	Kevin Johnson (SecureIdeas, OpenSBK) was on [Paul's Security Weekly #785](https://securityweekly.com/psw-785) to talk about this updated version of the PTES, now on GitHub for others to contribute to. It was mentioned again in [SWN-453](https://securityweekly.com/swn-453), where one of OpenSBK's goals in addition to updating the PTES will be defining the language used in InfoSec.

	- OpenSBK's PTES: <https://github.com/OpenSBK/ptes>
	- Original Site: <http://www.pentest-standard.org/index.php>
	- Original Site (Wayback Machine): <https://web.archive.org/web/20211220050516/http://www.pentest-standard.org/index.php/Main_Page>


??? abstract "HackTricks"

	HackTricks is one of the largest resources of hacking knowledge. The original author also created [PEASS](https://github.com/peass-ng/PEASS-ng).

	- <https://book.hacktricks.wiki/>
	- <https://github.com/HackTricks-wiki/hacktricks>

### Network

This includes general network information as well as network-focused tools.

??? question "nmap"

	Perhaps the most well known network scanner available.

	- <https://nmap.org/>

??? question "naabu"

	> A fast port scanner written in go with a focus on reliability and simplicity. Designed to be used in combination with other tools for attack surface discovery in bug bounties and pentests

	naabu's release binaries are statically compiled. This is incredibly useful if you're struggling to get a statically compiled `nmap` to run on a machine.

	- <https://github.com/projectdiscovery/naabu>

??? question "masscan"

	> TCP port scanner, spews SYN packets asynchronously, scanning entire Internet in under 5 minutes.

	- <https://github.com/robertdavidgraham/masscan>

??? question "SSH-Snake"

	> SSH-Snake is a self-propagating, self-replicating, file-less script that automates the post-exploitation task of SSH private key and host discovery.

	- <https://github.com/MegaManSec/SSH-Snake>


### Web Application

!!! abstract "OWASP Web Security Testing Guides"

	>  The Web Security Testing Guide is a comprehensive Open Source guide to testing the security of web applications and web services.

	- <https://github.com/OWASP/wstg>
	- [WSTG Table of Contents](https://github.com/OWASP/wstg/tree/master/document)
	- [WSTG Checklists](https://github.com/OWASP/wstg/tree/master/checklists)
	- [OWASP Cheat Sheets](https://github.com/OWASP/CheatSheetSeries/tree/master/cheatsheets)

??? abstract "PayloadsAllTheThings"

	A collection of payloads and techniques for application security / web application pentesting.

	- <https://swisskyrepo.github.io/PayloadsAllTheThings/>
	- <https://github.com/swisskyrepo/PayloadsAllTheThings>

??? example "Burpsuite"

	Web application security testing tool, has free and paid licenses.

	- <https://portswigger.net/burp>

??? danger "ZAProxy"

	Open source and free alternative to Burpsuite maintained by OWASP.

	- <https://github.com/zaproxy/>


	```bash
	# Install in Kali
	sudo apt install -y zaproxy
	```

??? bug "Caido"

	A new web application pentesting tool, similar to Burpsuite, has free and paid plans.

	- <https://caido.io/>
	- <https://github.com/caido>

??? danger "JS-TAP"

	> JavaScript payload and supporting software to be used as XSS payload or post exploitation implant to monitor users as they use the targeted application. Also includes a C2 for executing custom JavaScript payloads in clients, and a "mimic" feature that automatically generates custom payloads.

	Trap Mode uses the [iframe trap technique](https://trustedsec.com/blog/persisting-xss-with-iframe-traps). Implant mode means you've embedded JS-TAP into a javascript file on the server after gaining access to the server.

	- <https://github.com/hoodoer/JS-Tap>
	- <https://trustedsec.com/blog/js-tap-weaponizing-javascript-for-red-teams>

??? quote "SquareX: Polymorphic Extensions"

	Researchers demonstrate malicious browser extensions can effectively mimic and disable other browser extensions without really notifiying the user. The example in the demo video shows a proof-of-concept tool mimicking the user's password manager browser extension.

	Discovered on [Security Weekly News #457](https://securityweekly.com/swn-457).

	- <https://sqrx.com/polymorphic-extensions>


### Linux & Unix-like

??? danger "GTFOBins"

	> GTFOBins is a curated list of Unix binaries that can be used to bypass local security restrictions in misconfigured systems.

	- <https://gtfobins.github.io/>

??? danger "LOLESXi"

	> LOLESXi features a comprehensive list of binaries/scripts natively available in VMware ESXi that adversaries have utilised in their operations. The information on this site is compiled from open-source threat research.

	- <https://github.com/LOLESXi-Project/LOLESXi>


### Windows

??? danger "LOLBAS"

	> Living Off The Land Binaries, Scripts and Libraries

	- <https://lolbas-project.github.io/#>

??? danger "LOLDrivers"

	> An extensive and well-organized collection of vulnerable and malicious Windows drivers.

	- <https://github.com/magicsword-io/LOLDrivers>
	- <https://www.loldrivers.io/>

??? danger "HijackLibs"

	> This project aims to keep a record of publicly disclosed DLL Hijacking opportunities.
	>
	> You can find the web version of this repository on https://www.hijacklibs.net.

	- <https://github.com/wietze/HijackLibs>


### macOS

	⚠️ TO DO ⚠️


### Active Directory

??? info "adsecurity.org"

	One of the most robust resources to Active Directory security, hardening, pentesting, and information in general.

	- <https://adsecurity.org/>

??? abstract "InternalAllTheThings"

	> Active Directory and internal pentest cheatsheets.

	- <https://swisskyrepo.github.io/InternalAllTheThings/>
	- <https://github.com/swisskyrepo/InternalAllTheThings>

??? danger "SharpLAPS"

	> Retrieve LAPS password from LDAP.

	- <https://github.com/swisskyrepo/SharpLAPS>

### Wireless

??? abstract "WiFi Challenge Lab Walkthrough"

	Think of this as "WirelessAllTheThings". This walkthrough will demonstrate a number of WiFi attacks, from recon and basic commands to operate wireless tools on Linux, to attacking OPN, WEP, WPA/2/3 and MGT enterprise networks.

	- <https://r4ulcl.com/posts/walkthrough-wifichallenge-lab-2.0/>

??? question "USB-WiFi Guide"

	> The mission of this site is to provide educational information, reviews of USB WiFi adapters and links to specific adapters that are known to perform well with Linux ( see The Plug and Play List ).

	- <https://github.com/morrownr/USB-WiFi>
	- [Recommended Adapters for Kali Linux](https://github.com/morrownr/USB-WiFi/blob/main/home/Recommended_Adapters_for_Kali_Linux.md)

	The cards using the mediatek/mt76 drivers appear to be the most supported as of early 2025.

	When comparing available cards, note the AXML and AXM cards look the same but are different from the ACH and ACM cards in that they also have Bluetooth functionality built-in (which casues issues on Ubuntu in some cases). All 4 use the mediatek/mt76 drivers.

??? example "rtl8812au"

	These kernel drivers are no longer included in Kali by default, and these repos are where you'll need to obtain the source to compile and load them.

	You'll run into this if you use some of the popular ALFA USB cards like the AWUS036ACH, or AWUS036ACS.

	- <https://github.com/aircrack-ng/rtl8812au> (Deprecated)
	- <https://github.com/lwfinger/rtw88> (Current)

??? example "WiFiChallengeLab-docker"

	This is one of the most useful resources to learning nearly all wireless recon and attack paths in a safe and isolated environment. Deployment options include:

	- Pull the VM from Vagrant or GitHub releases
	- Alternatively install and run the entire lab through docker on a VM or host you have

	My preference is to configure a Kali Linux VM as I would to prepare it for wireless testing, and run the lab via docker on top of it. Your VM will have virtual wireless cards added through the 80211_hwsim kernel module. You can use these to interact with the containerized AP's as if they were "real".

	> Docker version of WiFiChallenge Lab with modifications in the challenges and improved stability. Ubuntu virtual machine with virtualized networks and clients to perform WiFi attacks on OPN, WPA2, WPA3 and Enterprise networks.

	- <https://github.com/r4ulcl/WiFiChallengeLab-docker>


??? example "Wifi-Forge"

	Virtual wireless pentesting lab from BHIS.

	> Wi-Fi Forge provides a safe and legal environment for learning WiFi hacking. Based on the open source Mininet-Wifi, this project automatically sets up the networks and tools needed to run a variety of WiFi exploitation labs, removing the need for the overhead and hardware normally required to perform these attacks.

	- <https://github.com/blackhillsinfosec/Wifi-Forge>

??? question "Rayhunter"

	> Rust tool to detect cell site simulators on an orbic mobile hotspot.

	- <https://github.com/EFForg/rayhunter>


### Cloud

??? danger "MSFTRecon"

	> MSFTRecon is a reconnaissance tool designed for red teamers and security professionals to map Microsoft 365 and Azure tenant infrastructure. It performs comprehensive enumeration without requiring authentication, helping identify potential security misconfigurations and attack vectors.

	- <https://github.com/Arcanum-Sec/msftrecon>


??? example "Cloud S.L.A.W"

	Cloud Security Lab a Week is a free and high-quality cloud security training platform.

	Discovered on [Unsupervised Learning NO. 471 ](https://newsletter.danielmiessler.com/p/ul-471).

	- <https://slaw.securosis.com/>


### ICS & OT

??? abstract "HardwareAllTheThings"

	> Hardware/IOT Pentesting Wiki

	- <https://swisskyrepo.github.io/HardwareAllTheThings/>
	- <https://github.com/swisskyrepo/HardwareAllTheThings>


### C2

??? danger "Sliver"

	> Sliver is an open source cross-platform adversary emulation/red team framework, it can be used by organizations of all sizes to perform security testing. Sliver's implants support C2 over Mutual TLS (mTLS), WireGuard, HTTP(S), and DNS and are dynamically compiled with per-binary asymmetric encryption keys.
	>
	> The server and client support MacOS, Windows, and Linux. Implants are supported on MacOS, Windows, and Linux (and possibly every Golang compiler target but we've not tested them all).

	- <https://github.com/BishopFox/sliver>

??? danger "Merlin"

	>  Merlin is a cross-platform post-exploitation HTTP/2 Command & Control server and agent written in golang.

	- <https://github.com/Ne0nd0g/merlin>

??? danger "Mythic"

	> A collaborative, multi-platform, red teaming framework.

	- <https://github.com/its-a-feature/Mythic>


## Defense

### Threat Hunting

??? abstract "Pivot Atlas"

	A visualization of paths in threat intelligence.

	- <https://gopivot.ing/>
	- <https://github.com/korniko98/pivot-atlas>

??? bug "YARA"

	Malware rule, pattern, and classification Language.

	- <https://github.com/VirusTotal/yara>
	- <https://yara.readthedocs.io/en/stable/>

??? example "yarGen"

	Programmatic YARA rule generation.

	- <https://github.com/Neo23x0/yarGen>

??? example "YARA-Rules Repository"

	- <https://github.com/Yara-Rules/rules>

??? bug "RITA"

	- <https://github.com/activecm/rita>

??? danger "ZEEK"

	- <https://zeek.org/>

??? danger "Suricata"

	- <https://suricata.io/download/>
	- <https://github.com/OISF/suricata>

??? danger "Velociraptor"

	Agent based incident response tool.

	- <https://docs.velociraptor.app/>
	- <https://github.com/Velocidex/velociraptor>

??? danger "OSQuery"

	- <https://github.com/osquery/osquery>

??? danger "BeaKer"

	- <https://github.com/activecm/BeaKer>

??? bug "Raccine"

	- <https://github.com/Neo23x0/Raccine>

??? example "SIGMA"

	General signature format for SIEM systems.

	- <https://github.com/SigmaHQ/sigma>
	- <https://github.com/SigmaHQ/sigma/tree/master/rules/windows/builtin>

??? example "Canary Tokens"

	Active defense alerts using secrets, commands, documents, files and more.

	- <https://canarytokens.org/generate#>
	- <https://blog.thinkst.com/p/canarytokensorg-quick-free-detection.html>
	- <https://notes.huskyhacks.dev/notes/content-creators-i-will-teach-you-cyber-jiu-jitsu>

??? bug "iVerify"

	Discovered on [Schneier's blog post: Detecting Pegasus Infections](https://www.schneier.com/blog/archives/2024/12/detecting-pegasus-infections.html).

	It's able to do this through diagnostic data and system information without needing (or being able) to read any data from applications or files.

	- <https://iverify.io/products/basic>
	- <https://iverify.io/blog/engineering-threat-hunting-for-ios-and-android>
	- <https://iverify.io/frequently-asked-questions>

??? danger "MISP (Malware Information Sharing Platform)"

	> MISP Project - Open Source Threat Intelligence Platform & Open Standards For Threat Information Sharing.

	- <https://github.com/MISP>

??? danger "The Hive Project (IR Platform)"

	An open source IR platform.

	> One Case Management Platform for all SOCs, CERTs & CSIRTs

	- <https://github.com/TheHive-Project>
	- <https://strangebee.com/>
	- <https://thehive-project.org/> (old url)

??? bug "Elastic Detection Rules"

	> Detection Rules is the home for rules used by Elastic Security. This repository is used for the development, maintenance, testing, validation, and release of rules for Elastic Security's Detection Engine.

	- <https://github.com/elastic/detection-rules/tree/main>

??? danger "SquareX (Browser Detection and Response)"

	SquareX has done [a lot of interesting research](https://sqrx.com/research) into browser-based attacks. The browser is one of the most critical points of failure, and is still kind of esoteric when it comes to applying defenses. They plan to release research throughout 2025 that focuses on attack paths in browsers exploiting functionality that cannot be easily patched.

	- <https://sqrx.com/>


## GRC

All things standards, configuration, compliance, and policy related.

??? danger "MITRE ATT&CK"

	> MITRE ATT&CK is a globally-accessible knowledge base of adversary tactics and techniques based on real-world observations.

	- <https://attack.mitre.org/>
	- <https://github.com/mitre/cti>
	- <https://attack.mitre.org/resources/working-with-attack/> (the "learn more" sections have complete download lists of the frameworks as xlsx)
	- <https://attack.mitre.org/docs/enterprise-attack-v9.0/enterprise-attack-v9.0-techniques.xlsx>

??? abstract "MITRE CWE"

	> A community developed list of software and hardware weaknesses that can become vulnerabilities.

	- <https://cwe.mitre.org/index.html>
	- <https://cwe.mitre.org/top25/archive/2021/2021_cwe_top25.html> (great historical reference)

??? abstract "OpenSCAP"

	> Security automation content in SCAP, Bash, Ansible, and other formats.

	These resources primarily focus on Unix-like operating systems. Windows support is no longer maintained as of the time of writing this.

	- <https://github.com/ComplianceAsCode/content> (bash and ansible deployment)
	- <https://www.open-scap.org/security-policies/choosing-policy/> (list of policies)

??? abstract "STIG"

	> Security Technical Implementation Guides (STIGs) by The United States Department of Defense specify how government computers must be configured and managed.

	- <https://public.cyber.mil/stigs/>

	The majority of the STIG policies (for Unix-like machines and browsers) can be viewed online through OpenSCAP's project page, and can be deployed through bash or Ansible using the ComplianceAsCode GitHub release files.

	- <https://www.open-scap.org/>
	- <https://www.open-scap.org/security-policies/choosing-policy/>
	- <https://github.com/ComplianceAsCode/content/releases>

??? abstract "CIS Benchmarks"

	> The CIS Benchmarks are prescriptive configuration recommendations for more than 25+ vendor product families. They represent the consensus-based effort of cybersecurity experts globally to help you protect your systems against threats more confidently.

	- <https://www.cisecurity.org/cis-benchmarks>

	The majority of the CIS policies (for Unix-like machines and browsers) can be viewed online through OpenSCAP's project page, and can be deployed through bash or Ansible using the ComplianceAsCode GitHub release files.

	- <https://www.open-scap.org/>
	- <https://www.open-scap.org/security-policies/choosing-policy/>
	- <https://github.com/ComplianceAsCode/content/releases>

??? abstract "Microsoft Baselines"

	The download center link has all of the baselining tools available, including LGPO.exe and the PolicyAnalyzer.

	The idea with the .PolicyRules files is they are configurations that are pre-made by Microsoft and ready to be installed using LGPO.exe

	You can do all of this manually with PowerShell, and you will ultimately want to familiarize yourself with the descriptions of each setting should you run into any issues, but this will save a ton of time in getting things up and running.

	Use PolicyAnalyzer.exe to view the `*.PolicyRules` files, compare them to other `*.PolicyRules` files, or even your current system settings.

	The remaining files are the raw policies for each "thing", including Windows 10 and 11 endpoints, Windows Server, Microsoft Edge, and M365 apps. When applied they create a hardened environment from a security perspective while maintaining functionality.

	- [windows-security-baselines](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
	- [Microsoft Download Center: Security Baselines](https://www.microsoft.com/download/details.aspx?id=55319)


??? question "audit-inspector (Windows)"

	> Audit Inspector is a binary tool written in Rust for Windows audit configuration and auditing.

	- <https://github.com/blackhillsinfosec/audit-inspector>


## Reverse Engineering

??? bug "Ghidra"

	> Ghidra is a software reverse engineering (SRE) framework.

	- <https://github.com/NationalSecurityAgency/ghidra>

??? bug "Ghidriff"

	> Python Command-Line Ghidra Binary Diffing Engine.

	- <https://github.com/clearbluejar/ghidriff>

??? bug "GEF"

	> GEF (pronounced "Jeff") is a set of commands for x86/64, ARM, MIPS, PowerPC and SPARC to assist exploit developers and reverse-engineers when using old school GDB. It provides additional features to GDB using the Python API to assist during the process of dynamic analysis and exploit development. Application developers will also benefit from it, as GEF lifts a great part of regular GDB obscurity, avoiding repeating traditional commands, or bringing out the relevant information from the debugging runtime.

	- <https://github.com/hugsy/gef>

	In addition to the easy install options in the GitHub README, you can also install GEF directly from GitHub by downloading the latest [gef.py](https://github.com/hugsy/gef/raw/refs/heads/main/gef.py) file to `$HOME/.gef.py`, and sourcing that file from `~/.gdbinit` like this:

	```txt
	# Installs GDB Enhanced Features (GEF)
	source ~/.gef.py
	```

??? bug "Cutter"

	> Cutter is a free and open-source reverse engineering platform powered by rizin. It aims at being an advanced and customizable reverse engineering platform while keeping the user experience in mind. Cutter is created by reverse engineers for reverse engineers.

	Cutter is an excellent macOS, Linux and Windows binary disassembler + decompiler. The GitHub releases page has ready to use AppImages for 64-bit Linux.

	- <https://github.com/rizinorg/cutter>


## Malware Analysis

??? bug "DidierStevensSuite"

	Numerous, essential, forensics and analysis tools.

	- <https://github.com/DidierStevens/DidierStevensSuite>
	- <https://github.com/DidierStevens/Beta>

??? bug "Qu1cksc0pe"

	An all-in-one malware analysis tool, excellent for triage. Originally discovered on [this SANS diary](https://isc.sans.edu/diary/29984). The diary post has an alternate docker file available to use.

	- <https://github.com/CYB3RMX/Qu1cksc0pe>
	- <https://isc.sans.edu/diary/29984>

??? bug "MalAPI"

	Common Windows API calls used by malware.

	- <https://malapi.io/>


### Known Samples

??? bug "xz-utils Backdoor"

	> Malicious code was discovered in the upstream tarballs of xz, starting with version 5.6.0. Through a series of complex obfuscations, the liblzma build process extracts a prebuilt object file from a disguised test file existing in the source code, which is then used to modify specific functions in the liblzma code. This results in a modified liblzma library that can be used by any software linked against this library, intercepting and modifying the data interaction with this library.

	- <https://nvd.nist.gov/vuln/detail/cve-2024-3094>
	- <https://isc.sans.edu/diary/30802>
	- <https://gist.github.com/smx-smx/a6112d54777845d389bd7126d6e9f504>


## Firmware

### Information

??? abstract "DBX Update Process"

    * <https://eclypsium.com/2022/07/26/firmware-security-realizations-part-1-secure-boot-and-dbx/>


### Projects

??? info "Coreboot"

    - <https://doc.coreboot.org/>

??? info "EDK2 / Tianocore"

    - <https://github.com/tianocore/edk2>

??? info "System76 Open Firmware"

	> An open source distribution of firmware utilizing coreboot, EDK2, and System76 firmware applications

    - <https://github.com/system76/firmware-open>
	- [Secure Boot support was added in 2023-04-03](https://github.com/system76/firmware-open/blob/master/docs/uefi.md)

??? info "LVFS"

	Linux Vendor Firmware Service.

	- <https://fwupd.org/>
	- <https://github.com/fwupd/fwupd>

??? abstract "ChipSec"

	> CHIPSEC is a framework for analyzing the security of PC platforms including hardware, system firmware (BIOS/UEFI), and platform components. It includes a security test suite, tools for accessing various low level interfaces, and forensic capabilities. It can be run on Windows, Linux, and UEFI shell.

	- <https://github.com/chipsec/chipsec>

??? bug "EMBA"

	EMBA was discovered through Paul's Security Weekly. It's been covered in too many episodes to pinpoint which one *I* initially heard it from.

	> EMBA is designed as the central firmware analysis and SBOM tool for penetration testers, product security teams, developers and responsible product managers. It supports the complete security analysis process starting with firmware extraction, doing static analysis and dynamic analysis via emulation, building the SBOM and finally generating a web based vulnerability report. EMBA automatically discovers possible weak spots and vulnerabilities in firmware. Examples are insecure binaries, old and outdated software components, potentially vulnerable scripts, or hard-coded passwords. EMBA is a command line tool with the possibility to generate an easy-to-use web report for further analysis.

	EMBA requires *a lot* of compute resources. See the [prerequisites](https://github.com/e-m-b-a/emba/wiki/Installation#prerequisites) for details. For reference, your EMBA VM should have 8vCPU's and 16GB RAM as the *minimum*.

	- <https://github.com/e-m-b-a/emba>

??? bug "OFRAK"

	OFRAK allows you to unpack, modify, and repack binaries.

	> It supports a range of embedded firmware file formats beyond userspace executables, including: compressed filesystems, compressed & checksummed firmware, bootloaders and RTOS/OS kernels.

	- <https://github.com/redballoonsecurity/ofrak>
	- [Red Balloon Security](https://redballoonsecurity.com/)

??? info "UEFITool"

	UEFI firmware image editor and viewer.

	- <https://github.com/LongSoft/UEFITool>

??? info "UEFI Firmware Parser"

	- <https://github.com/theopolis/uefi-firmware-parser>

??? abstract "Flashrom"

	Read, write, edit firmware.

	- <https://github.com/flashrom/flashrom>


### Attacks

??? bug "BootHole"

	This includes the vulnerability check for both bash and PowerShell.

    - <https://github.com/eclypsium/BootHole>

??? danger "baton drop (CVE-2022-21894)"

	> Windows Boot Applications allow the truncatememory setting to remove blocks of memory containing "persistent" ranges of serialised data from the memory map, leading to Secure Boot bypass.

	- <https://github.com/Wack0/CVE-2022-21894>


## Forensics

??? question "uac (Unix-like Artifacts Collector)"

	> UAC is a Live Response collection script for Incident Response that makes use of native binaries and tools to automate the collection of AIX, ESXi, FreeBSD, Linux, macOS, NetBSD, NetScaler, OpenBSD and Solaris systems artifacts. It was created to facilitate and speed up data collection, and depend less on remote support during incident response engagements.

	- <https://github.com/tclahr/uac>

	UAC is primarily an **evidence collection tool**. This isn't something like [linpeas](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS) that will alert you to possible leads as it's gathering evidence. It's a ton of output, and is a haystack in which you'd be trying to find a very small needle if you didn't already have an idea of what you're looking for. With this in mind, there are a few effective ideas on how to leverage this tool:

	- Assume you're compromised, hit each endpoint with UAC and pull the resulting evidence back to your IR machine
	- Investigate supsicious activity with system internals, using what you know while UAC runs
	- Use [linpeas](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS) to assess endpoints, it's not just a hacking tool, it will find IOC's

	If you have a possible lead or IOC, then your UAC evidence becomes invaluable. With the evidence back on your IR machine:

	- You can of course dig into specific evidence based on your potential IOC's
	- You could also use **[YARA](https://github.com/VirusTotal/yara)** to hit the entire UAC evidence folder for IOC's, which is incredibly fast


### Memory Acquisition

??? danger "avml (Acquire Volatile Memory for Linux)"

	Do this remotely with ssh + avml (acquire volatile memory for linux).

	- <https://github.com/microsoft/avml>


??? danger "lmg (Linux Memory Grabber)"

	Do this remotely with ssh + lmg (linux memory grabber).

	- <https://github.com/halpomeranz/lmg>


## OSINT

??? question "Shodan"

	A search engine for devices.

	- <https://www.shodan.io/>

??? info "Hurrican Electric"

	> Hurricane Electric operates its own global IPv4 and IPv6 network and is considered the largest IPv6 backbone in the world as measured by number of networks connected.

	There's also an iOS and Android application that provides a suite of [network tools](https://networktools.he.net/) on the go.

	- [BGP Toolkit](https://bgp.he.net/) shows information about your connection.
	- [Looking Glass](https://lg.he.net/) allows you to make network queries for for BGP, ping, and traceroute.

??? info "crt.sh"

	Certificate search.

	> Enter an Identity (Domain Name, Organization Name, etc), a Certificate Fingerprint (SHA-1 or SHA-256) or a crt.sh ID.

	- <https://crt.sh>
	- <https://github.com/crtsh>


### Vulnerability Research

Sources used when attempting to triage and produce a proof-of-concept exploit or demonstrate risk.

??? example "NIST / NVD"

	National Institute of Standards and Technology.

	- <https://www.nist.gov/>
	- <https://csrc.nist.gov/Topics/technologies/software-firmware/bios>

	The NVD (National Vulnerability Database) is a resource for researching vulnerabilities.

	- <https://nvd.nist.gov/>

??? success "CISA / KEV"

	Cybersecurity & Infrastructure Security Agency. Follow the KEV (known-exploited-vulnerabilities-catalog).

	- <https://www.cisa.gov/known-exploited-vulnerabilities-catalog>
	- <https://www.cisa.gov/publication/cyber-essentials-toolkits>

??? abstract "CVE.ORG"

	> Identify, define, and catalog publicly disclosed cybersecurity vulnerabilities.

	- <https://www.cve.org/>
	- <https://cve.mitre.org/> (old site)

??? bug "Exploit-DB"

	`searchsploit` (<https://www.kali.org/tools/exploitdb/>) might have different results than the online exploit-db database.

	> The Exploit Database is a CVE compliant archive of public exploits and corresponding vulnerable software, developed for use by penetration testers and vulnerability researchers. Our aim is to serve the most comprehensive collection of exploits gathered through direct submissions, mailing lists, as well as other public sources, and present them in a freely-available and easy-to-navigate database. The Exploit Database is a repository for exploits and proof-of-concepts rather than advisories, making it a valuable resource for those who need actionable data right away.

	- <https://www.exploit-db.com/>

??? danger "GitHub Advisory Database"

	> Security vulnerability database inclusive of CVEs and GitHub originated security advisories from the world of open source software.

	- <https://github.com/advisories>

??? info "Ubuntu Security Notices"

	> Developers issue an Ubuntu Security Notice when a security issue is fixed in an official Ubuntu package.
	>
	> To report a security vulnerability in an Ubuntu package, please contact the Security Team.
	>
	> The Security Team also produces OVAL files for each Ubuntu release. These are an industry-standard machine-readable format dataset that contain details of all known security vulnerabilities and fixes relevant to the Ubuntu release, and can be used to determine whether a particular patch is appropriate. OVAL files can also be used to audit a system to check whether the latest security fixes have been applied.

	- <https://ubuntu.com/security/notices>

??? bug "Red Hat Bugzilla"

	Here you can search the entirety of Red Hat's Bugzilla instance for security notices or fixes. This isn't limited to just RHEL, but any project tracked here. For example, set the "Product" to `Fedora` and "Summary contains all of the strings" to `CVE` and sort results by "Last Changed" to see the latest issues being addressed and their current state.

	- <https://bugzilla.redhat.com/query.cgi?format=advanced>

??? danger "Google Project Zero"

	Zero day and exploit research.

	- <https://googleprojectzero.blogspot.com/>

??? danger "snyk.io (Vulnerability Database)"

	- <https://security.snyk.io>

??? danger "wpscan (Vulnerability Database)"

	- <https://wpscan.com/wordpresses/>
	- <https://wpscan.com/themes>
	- <https://wpscan.com/plugins>

??? danger "Vulners (nmap NSE script)"

	>  For each available CPE the script prints out known vulns (links to the correspondent info) and correspondent CVSS scores.

	- <https://nmap.org/nsedoc/scripts/vulners.html>


### Malware Research

??? bug "VirusTotal"

	The ultimate resource for malware information, connections, and behavior.

	The VT API has a 500 requests per day, 4 per minute limit.

	- <https://www.virustotal.com/>

??? bug "abuse.ch"

	> Independent, community-driven cyber threat intelligence.

	- <https://abuse.ch/>
	- <https://bazaar.abuse.ch/> (live malware samples)
	- <https://urlhaus.abuse.ch/>

??? bug "ANY.RUN"

	Cloud-based live malware analysis.

	- <https://app.any.run/>

??? question "URLScan"

	Third-party URL scanning.

	> urlscan.io is a free service to scan and analyse websites. When a URL is submitted to urlscan.io, an automated process will browse to the URL like a regular user and record the activity that this page navigation creates. This includes the domains and IPs contacted, the resources (JavaScript, CSS, etc) requested from those domains, as well as additional information about the page itself. urlscan.io will take a screenshot of the page, record the DOM content, JavaScript global variables, cookies created by the page, and a myriad of other observations. If the site is targeting the users one of the more than 900 brands tracked by urlscan.io, it will be highlighted as potentially malicious in the scan results.
	>
	> urlscan.io itself is a free service, but we also offer commercial products for heavy users and organisations that need additional insight.

	- <https://urlscan.io>

??? danger "GREYNOISE"

	> GreyNoise empowers your security team to work on the most urgent and critical threats without being overwhelmed by noisy, low-priority alerts. We provide real-time, verifiable threat intelligence powered by a global network of proprietary sensors.

	The API has a 50 searches per week limit.

	- <https://viz.greynoise.io/>
    - <https://docs.greynoise.io/docs/using-the-greynoise-community-api>

??? bug "vxunderground"

	A collection of malware samples, code, and research.

    - <https://github.com/vxunderground/MalwareSourceCode>
	- <https://github.com/vxunderground/ThreatIntelligenceDiscordBot>

??? bug "theZoo"

	> theZoo is a project created to make the possibility of malware analysis open and available to the public.

	- <https://github.com/ytisf/theZoo>

??? bug "Joe Security / JoeSandbox"

	- <https://github.com/joesecurity>
    - <https://www.joesecurity.org/>

??? bug "PhishTank"

	> PhishTank is a collaborative clearing house for data and information about phishing on the Internet. Also, PhishTank provides an open API for developers and researchers to integrate anti-phishing data into their applications at no charge.

	- <https://www.phishtank.com>
	- <https://www.phishtank.com/faq.php>


## Blogs & Authors

??? quote "Daniel Miessler"

	> Building AI that upgrades humans.

	- <https://danielmiessler.com/>
	- <https://github.com/danielmiessler/fabric>
	- <https://github.com/danielmiessler/SecLists>

??? quote "tcm-sec (TheCyberMentor)"

	Training, tutorials, and all things infosec.

	- <https://www.youtube.com/c/TheCyberMentor>
	- <https://github.com/TCM-Course-Resources>

??? quote "Husky Hacks"

	- <https://github.com/HuskyHacks>
	- <https://notes.huskyhacks.dev/notes>

??? quote "OA Labs"

	Malware analysis and reverse engineering.

	- <https://www.twitch.tv/oalabslive>
	- <https://github.com/OALabs>
	- <https://www.unpac.me/#/> (automated unpacking service)

??? quote "Malware Unicorn"

	- <https://github.com/malware-unicorn>
	- <https://malwareunicorn.org/#/>

??? quote "maldev-for-dummies"

	A malware development course.

	- <https://github.com/chvancooten/maldev-for-dummies>

??? quote "Josh Stroschein (CyberYeti)"

	Malware analysis and reverse engineering.

	- <https://github.com/jstrosch>
	- <https://www.youtube.com/@jstrosch>

??? quote "S1REN"

	- <https://sirensecurity.io/blog/linux-privilege-escalation-resources/>
	- <https://sirensecurity.io/blog/windows-privilege-escalation-resources/>

??? quote "13Cubed (Richard Davis DFIR)"

	- <https://www.youtube.com/c/13cubed/featured>
	- <https://www.13cubed.com/>
	- <https://github.com/13Cubed>

??? quote "Hal Pomeranz"

	- <https://github.com/halpomeranz>
    - <https://righteousit.com/author/halpomeranz/> (Blog)
    - <https://archive.org/search?query=creator%3A%22Hal+Pomeranz%22> (Training materials released on archive.org)
    - [Hiding Linux Processes with Bind Mounts](https://righteousit.com/2024/07/24/hiding-linux-processes-with-bind-mounts/)
    - [Systemd Timers](https://righteousit.com/2024/05/05/systemd-timers/)

??? quote "IppSec"

	- <https://www.youtube.com/c/ippsec/>
	- <https://ippsec.rocks> (YouTube video topic search)

??? quote "Security Weekly"

	AKA Paul's Security Weekly.

	- <https://securityweekly.com/>

??? quote "SANS"

	The [ISC Daily StormCast](https://isc.sans.edu/podcast.html) is a great way to digest daily news with links to tools and sources.

	- <https://isc.sans.edu/>

??? quote "NSA"

	National Security Agency.

	- <https://www.nsa.gov/news-features/>
	- <https://github.com/nsacyber/>
	- <https://github.com/nsacyber/Hardware-and-Firmware-Security-Guidance>

??? quote "Schneier"

	Bruce Schneier's security blog is a great way to digest daily news with links to research and sources.

	- <https://www.schneier.com/>

??? quote "Reversing Labs"

	- [Malicious ML models discovered on Hugging Face platform](https://www.reversinglabs.com/blog/rl-identifies-malware-ml-model-hosted-on-hugging-face)

??? quote "p0dalirius"

	Security researcher and author of a number of very popular pentesting tools.

	- <https://github.com/p0dalirius>

??? quote "mobile-hacker"

	Discovered on [Paul's Security Weekly #864](https://securityweekly.com/psw-864). mobile-hacker's tutorials have excellent DIY guides for on-the-go projects using Raspberry Pi's, Android, and more.

	- [Portable Kali on Raspberry Pi with Touchscreen](https://www.mobile-hacker.com/2025/02/26/building-a-portable-kali-box-with-raspberry-pi-and-touchscreen/)