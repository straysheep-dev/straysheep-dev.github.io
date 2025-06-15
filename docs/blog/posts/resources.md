---
draft: false
date:
  created: 2019-07-15
  updated: 2025-06-15
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


## :material-toolbox: Utilities

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

??? info "Terramaid"

	> A utility for generating Mermaid diagrams from Terraform configurations.

	- <https://github.com/RoseSecurity/Terramaid>


??? abstract "FileCloud Community Edition"

	The Community Edition is the free tier of the enterprise version, without certain enterprise features. It's geared towards home labs and personal use, while giving you expierence with the FileCloud platform.

	Uniquely, FileCloud has an MSI installer for deployment on Windows servers / desktops, and allows you to mount folders as network drives.

	Discovered on [NetworkChuck's video on Building your Own Cloud](https://www.youtube.com/watch?v=xBIowQ0WaR8).

	- <https://ce.filecloud.com/>

??? abstract "NextCloud"

	A community driven, free and open source, self-hosted cloud solution.

	Discovered on [Network Chuck's video on Building your Own Cloud](https://www.youtube.com/watch?v=xBIowQ0WaR8). The steps demonstrated for the docker install seem to be the easiest way to get setup in a lab environment that can move to "production" use for a home user.

	- <https://github.com/nextcloud>

??? info "opkssh"

	Discovered on the [SANS ISC Daily Stormcast, Monday March 31st episode in 2025](https://isc.sans.edu/podcastdetail/9386).

	> opkssh is a tool which enables ssh to be used with OpenID Connect allowing SSH access management via identities like alice@example.com instead of long-lived SSH keys. It does not replace ssh, but rather generates ssh public keys that contain PK Tokens and configures sshd to verify the PK Token in the ssh public key. These PK Tokens contain standard OpenID Connect ID Tokens. This protocol builds on the OpenPubkey which adds user public keys to OpenID Connect without breaking compatibility with existing OpenID Provider.
	>
	> Currently opkssh is compatible with Google, Microsoft/Azure and Gitlab OpenID Providers (OP). If you have a gmail, microsoft or a gitlab account you can ssh with that account.

	- <https://github.com/openpubkey/opkssh>

??? info "Etcher"

	> Etcher is a powerful OS image flasher built with web technologies to ensure flashing an SDCard or USB drive is a pleasant and safe experience. It protects you from accidentally writing to your hard-drives, ensures every byte of data was written correctly, and much more. It can also directly flash Raspberry Pi devices that support USB device boot mode.

	Etcher is one of the two go-to GUI-based imaging utilities for creating bootable media (the other being Rufus), with Etcher's main difference from Rufus being that it works across Windows macOS and Linux.

	- <https://github.com/balena-io/etcher>

	Supported OS's:

    - Linux; most distros; Intel 64-bit.
    - Windows 10 and later; Intel 64-bit.
    - macOS 10.13 (High Sierra) and later; both Intel and Apple Silicon.


??? info "Rufus"

	> Rufus is a utility that helps format and create bootable USB flash drives.

	Rufus is possibly the best imaging utility if you're on Windows, allowing you to not only flash external media with any image file you have available, but with the options to customize Windows images during the process. All features are [listed in the README](https://github.com/pbatard/rufus#features).

	- <https://github.com/pbatard/rufus>

	Rufus also has [extensive documentation on its security model](https://github.com/pbatard/rufus/wiki/Security), from how it does what it does, to verifying signatures and how this works in Windows world, in a way that is useful and meaningful to a user. The concepts are not exclusive to Rufus.

??? example "etl2pcapng"

	> This tool enables you to view ndiscap and pktmon packet captures with Wireshark. Due to performance problems with the other popular packet capture method (WinPcap, which was included with older versions of Wireshark), these inbox tools should be preferred.

	- <https://github.com/microsoft/etl2pcapng>

	Here's how you can use this, taken from the README:

	> "pktmon" is implemented as an integral part of the Windows operating system. It's capable of capturing packets in many components of the operating system, giving full visibility into the life of the packet as it traverses the system. A capture can be collected with:
	>
	> ```cmd
	> pktmon start --capture
	> <repro>
	> pktmon stop
	> ```
	>
	> "ndiscap" which is implemented as an ETW trace provider. A capture can be collected with:
	>
	> ```cmd
	> netsh trace start capture=yes report=disabled
	> <repro>
	> netsh trace stop
	> ```


## :material-note-text: Note Taking

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


## :material-desktop-tower: Operating Systems

??? info ":fontawesome-brands-windows: Windows"

	Each of the ISOs and disk images are openly available to download, both for evaluation and to enter a product key during install for produciton use.

	- [Direct Download Links to ISO's / VHD's](https://techcommunity.microsoft.com/t5/windows-11/accessing-trials-and-kits-for-windows/m-p/3361125)
	- [Windows ISOs (Production Use)](https://www.microsoft.com/en-us/download/windows)
	- [Windows ISOs (Evaluation Use)](https://www.microsoft.com/en-us/evalcenter)
	- [Windows Developer Evaluation VM](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)
	- [Windows 7/8/10 Legacy VM](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)

??? example ":simple-reactos: ReactOS"

	> ReactOS is an Open Source effort to develop a quality operating system that is compatible with applications and drivers written for the Microsoft Windows NT family of operating systems (NT4, 2000, XP, 2003, Vista, 7).
	>
	> The ReactOS project, although currently focused on Windows Server 2003 compatibility, is always keeping an eye toward compatibility with Windows Vista and future Windows NT releases.
	>
	> The code of ReactOS is licensed under GNU GPL 2.0.

	- <https://github.com/reactos/reactos>

??? danger ":simple-kalilinux: Kali Linux"

	The most robust pentesting Linux distribution. Includes tools for offense, purple teaming, defense, and forensics.

	- <https://cdimage.kali.org/>
	- <https://www.kali.org/downloads/>
	- <https://www.kali.org/docs/introduction/download-images-securely/>
    - For older images of kali: <https://old.kali.org/>
	- [A New Kali Linux Archive Signing Key](https://www.kali.org/blog/new-kali-archive-signing-key/)

	Obtaining the Kali Linux GPG public key and verifying signatures:

	```bash
	wget -q -O - https://archive.kali.org/archive-key.asc | gpg --import
	gpg --keyserver hkps://keyserver.ubuntu.com --recv-key 827C8569F2518CC677FECA1AED65462EC8D5E4C5
	wget -q https://cdimage.kali.org/current/SHA256SUMS{.gpg,}
	gpg --verify SHA256SUMS.gpg SHA256SUMS
	```

	Current GPG Key File: [0xED65462EC8D5E4C5](https://keyserver.ubuntu.com/pks/lookup?search=827C8569F2518CC677FECA1AED65462EC8D5E4C5&fingerprint=on&op=index)

	```bash
	pub   rsa4096/ED65462EC8D5E4C5 2025-04-17 [SC] [expires: 2028-04-17]
		Key fingerprint = 827C 8569 F251 8CC6 77FE  CA1A ED65 462E C8D5 E4C5
	uid                 [ unknown] Kali Linux Archive Automatic Signing Key (2025) <devel@kali.org>
	```

    (Previous) GPG Key File: [0xED444FF07D8D0BF6](https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x44c6513a8e4fb3d30875f758ed444ff07d8d0bf6)

    ```bash
    pub   rsa4096/0xED444FF07D8D0BF6 2012-03-05 [SC] [expires: 2027-02-04]
          Key fingerprint = 44C6 513A 8E4F B3D3 0875  F758 ED44 4FF0 7D8D 0BF6
    uid                             Kali Linux Repository <devel@kali.org>
    sub   rsa4096/0xA8373E18FC0D0DCB 2012-03-05 [E] [expires: 2027-02-04]
    ```

??? bug ":simple-ubuntu: REMnux"

	A Linux toolkit for malware analysts. [Lenny Zeltser](https://zeltser.com/) is [one of the maintainers](https://remnux.org/#people).

	- <https://remnux.org/>


??? abstract ":simple-debian: Debian"

	Debian is one of the "main" operating system families of Linux.

	> The Debian Project is an association of individuals, sharing a common goal: We want to create a free operating system, freely available for everyone. Now, when we use the word "free", we're not talking about money, instead, we are referring to software freedom.

	- <https://www.debian.org>
	- <https://www.debian.org/CD/verify>
	- <https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/>
    - For older images of Debian: <https://cdimage.debian.org/mirror/cdimage/archive/>
	- [Package Search](https://www.debian.org/distrib/packages)

	GPG Key:

	```bash
	gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys 'DF9B 9C49 EAA9 2984 3258  9D76 DA87 E80D 6294 BE9B'
	```

	To automate installs, Debian uses `preseed.cfg` files.

	- [debian.org/DebianInstaller/Preseed](https://wiki.debian.org/DebianInstaller/Preseed)
	- [debian.org example-preseed.txt Template](https://www.debian.org/releases/stable/example-preseed.txt)
	- [debian.org/tasksel](https://wiki.debian.org/tasksel)

??? tip ":simple-ubuntu: Ubuntu"

	Ubuntu is a Debian-base Linux distribution developed by Canonical.

	- <https://releases.ubuntu.com> (main images)
    - `https://old-releases.ubuntu.com/releases/$VERSION/` (is how you can version pin URLs to ISOs and signatures, this contians all released images in one page per version)
	- <https://cdimage.ubuntu.com/releases/> (rpi + alternate flavors)
    - <https://cloud-images.ubuntu.com/> (vagrant and cloud provider images)
	- <https://ubuntu.com/download/raspberry-pi>
	- <https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi>
	- [Package Search](https://packages.ubuntu.com/)

    GPG Key File: [0xD94AA3F0EFE21092](https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x843938df228d22f7b3742bc0d94aa3f0efe21092)

	```bash
    pub   rsa4096/0xD94AA3F0EFE21092 2012-05-11 [SC]
          Key fingerprint = 8439 38DF 228D 22F7 B374  2BC0 D94A A3F0 EFE2 1092
    uid   Ubuntu CD Image Automatic Signing Key (2012) <cdimage@ubuntu.com>
    ```

??? tip ":simple-raspberrypi: Raspberry Pi OS"

	> Raspberry Pi needs an operating system to work. This is it. Raspberry Pi OS (previously called Raspbian) is our official supported operating system.

	Many Linux distributions have a version available to run on Raspberry Pi, however there's also Raspberry Pi OS which is built and maintained by the Raspberry Pi Foundation.

	- Verify the .sig file against the img.xz compressed file, not the SHA signatures.
	- <https://www.raspberrypi.org/about/> links to raspberrypi.com
    - <https://www.raspberrypi.com/>
	- <https://www.raspberrypi.com/software/operating-systems/> (links to main images, use the archive link to obtain the .sig)
    - <https://downloads.raspberrypi.com/raspios_arm64/images/> (folder for standard desktop download)
    - <https://github.com/raspberrypi>
    - <https://www.raspberrypi.org/raspberrypi_downloads.gpg.key> GPG key, indexed by search engines

	```txt
	pub   rsa2048/0x8738CD6B956F460C 2017-04-10 [SC] [expires: 2031-04-07]
		Key fingerprint = 54C3 DD61 0D9D 1B4A F82A  3775 8738 CD6B 956F 460C
	uid                   [ unknown] Raspberry Pi Downloads Signing Key
	sub   rsa2048/0x287C051D4228C4CE 2017-04-10 [E] [expires: 2031-04-07]
	```

	**rpi-imager**

	The Rapsberry Pi Imager is similar to [etcher](https://github.com/balena-io/etcher) or [rufus](https://github.com/pbatard/rufus), in that it allows you to both, write the Raspberry Pi OS to an external device while simultaneously configuring it.

	- <https://downloads.raspberrypi.org/imager/>
	- <https://github.com/raspberrypi/rpi-imager>

	On Linux there's an AppImage binary available if you prefer installing something under a `local/` path instead of using `dpkg`. Regardless, each binary comes with a signature file. You can use the Raspberry Pi public key (above) to verify the file integrity.

	```bash
	# Obtain the signing key
	gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys '54C3 DD61 0D9D 1B4A F82A 3775 8738 CD6B 956F 460C'

	# Set the version information using GitHub's API to obtain the latest release tag version
	gh_release_info="$(curl -Lf 'https://api.github.com/repos/raspberrypi/rpi-imager/releases/latest')"
	latest_version="$(echo "$gh_release_info" | jq -r '.tag_name' | sed 's/^v//')"

	# Download from https://downloads.raspberrypi.org/imager/
	cd ~/Downloads
	curl -LfO "https://downloads.raspberrypi.org/imager/imager_${latest_version}_amd64.AppImage"
	curl -LfO "https://downloads.raspberrypi.org/imager/imager_${latest_version}_amd64.AppImage.sig"

	# Verify the signature
	gpg --verify imager_${latest_version}_amd64.AppImage.sig imager_${latest_version}_amd64.AppImage

	# Install the AppImage locally without apt or dpkg
	sudo cp ./imager_${latest_version}_amd64.AppImage /usr/local/bin/rpi-imager
	sudo chmod +x /usr/local/bin/rpi-imager
	rpi-imager
	```

??? abstract ":simple-redhat: RHEL (Red Hat Enterprise Linux)"

	> Red Hat Enterprise Linux is an enterprise Linux operating system. It is oriented toward enterprise and commercial users, is certified for many hardware and cloud platforms, and is supported by Red Hat via various subscription options. Compared to Fedora, Red Hat Enterprise Linux emphasizes stability and enterprise-readiness over the latest technologies or rapid releases. More information about Red Hat offerings can be found at [Red Hat's web site](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux).
	>
	> Individual software developers can access a free-of-charge subscription as part of the [Red Hat Developer Program](https://developers.redhat.com/about). Developers can use Red Hat Enterprise Linux on up to 16 physical or virtual systems for development, quality assurance, demos, or small production uses. See the Frequently Asked Questions for the [No-cost Red Hat Enterprise Linux Individual Developer Subscription](https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux).

	- <https://www.redhat.com>
	- <https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux>

??? tip ":simple-fedora: Fedora"

	> Fedora is developed by the Fedora Project and sponsored by Red Hat. It follows its own release schedule, with a new version approximately every six months. Fedora provides a modern Linux operating system utilizing many of the latest technologies. It is free for all users and supported via the Fedora community.
	>
	> To create Red Hat Enterprise Linux, some version of Fedora is forked and enters an extensive development, testing and certification process to become a new version of Red Hat Enterprise Linux.

	- <https://getfedora.org>
    - <https://fedoraproject.org/security> (GPG Keys)
	- <https://docs.fedoraproject.org>
    - <https://hub.docker.com/_/fedora>

	To automate installs, Fedora uses [Kickstart files](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/advanced/Kickstart_Installations/).

??? info ":simple-rockylinux: Rocky"

	> Rocky Linux is an open-source enterprise operating system designed to be 100% bug-for-bug compatible with Red Hat Enterprise Linux. Rocky Linux is under intensive development by the community.

	- <https://rockylinux.org/>
	- <https://docs.rockylinux.org/> (The documentation is very straight forward and easy to understand, if you're learning Fedora or RHEL start here)

	Brief history about Rocky Linux:

	> On December 8, 2020, Red Hat announced that they would discontinue development of CentOS, which has been a production-ready downstream version of Red Hat Enterprise Linux, in favor of a newer upstream development variant of that operating system known as "CentOS Stream". In response, the original founder of CentOS, Gregory Kurtzer, announced via a comment on the CentOS website that he would again start a project to achieve the original goals of CentOS. Its name was chosen as a tribute to early CentOS co-founder Rocky McGaugh. By December 12, the code repository of Rocky Linux had become the top-trending repository on GitHub.

	**Verifying Signatures**

	Interestingly the way to verify the ISO hashes is through this git repo, where the commits of the hashes are signed. This is because Keykeeper (part of the peridot build system) does not sign arbitrary files or artifacts outside of the build system. So rather than having a CHECKSUMS and a detached CHECKSUMS.sig, hashes are commited here by the maintainers.

	- <https://rockylinux.org/resources/gpg-key-info>
	- <https://github.com/rocky-linux/checksums>
	- [GPG Key Fingerprints](https://github.com/rocky-linux/checksums/blob/main/keys/fingerprints)

??? danger ":simple-gentoo: Pentoo"

	- <https://pentoo.ch/>

??? danger ":simple-parrotsecurity: Parrot"

	> Parrot Security (ParrotOS, Parrot) is a Free and Open source GNU/Linux distribution based on Debian Stable designed for security experts, developers and privacy aware people.
	>
	> It includes a full portable arsenal for IT security and digital forensics operations. It also includes everything you need to develop your own programs or protect your privacy while surfing the net.
	>
	> Parrot is available in three main editions, Security, Home and Architect Edition, even as Virtual Machine (Virtual Box, Parallels and VMware), on Raspberry Pi and also on Docker.
	>
	> The operating system ships by default with MATE Desktop Environment, but it is possible to install others DEs.

	- <https://www.parrotsec.org/>

??? tip ":simple-archlinux: Arch Linux"

	- <https://archlinux.org/>
	- <https://archlinux.org/download/>

??? info ":simple-openbsd: OpenBSD"

	- <https://www.openbsd.org/>
    - <https://www.openbsd.org/faq/faq4.html#Download>

??? info ":simple-freebsd: FreeBSD"

	- <https://www.freebsd.org/where/>
    - <https://docs.freebsd.org/en/articles/pgpkeys/>
	- <https://docs.freebsd.org/pgpkeys/pgpkeys.txt>

??? tip ":simple-pfsense: pfSense"

	pfSense is an excellent choice for both, a home *and* lab router-firewall to begin learning with and protect your real network.

	> The pfSense project is a free network firewall distribution, based on the FreeBSD operating system with a custom kernel and including third party free software packages for additional functionality. pfSense software, with the help of the package system, is able to provide the same functionality or more of common commercial firewalls, without any of the artificial limitations. It has successfully replaced every big name commercial firewall you can imagine in numerous installations around the world, including Check Point, Cisco PIX, Cisco ASA, Juniper, Sonicwall, Netgear, Watchguard, Astaro, and more.

	- <https://www.pfsense.org/download/>
	- <https://github.com/pfsense/>

??? example ":simple-openwrt: OpenWRT"

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

??? tip ":simple-ubiquiti: Ubiquiti"

	- <https://www.ui.com/download/>
	- <https://dl.ui.com/unifi/firmware/U7PG2/3.7.58.6385/BZ.qca956x.v3.7.58.6385.170508.0957.bin> (UniFi AP AC Lite firmware v3.7.58)

??? tip ":simple-debian: VyOS"

	> VyOS is a fully open-source Linux-based OS for network devices. It focuses on enterprise, service provider, and network geek audiences.

	It's free to build and use, and nightly prerelease ISO's are available, however they operate on a "pay for prebuilt binaries", plus technical support and custom development services if you'd like to support the project.

	- <https://github.com/vyos/>
	- <https://vyos.net/get/nightly-builds/> (this details verifying the signatures with minisign)
		- <https://github.com/vyos/vyos-nightly-build/blob/main/minisign.pub>
	- <https://github.com/vyos/vyos-nightly-build/releases>
	- [VyOS History](https://docs.vyos.io/en/sagitta/introducing/history.html)

??? info ":simple-truenas: TrueNAS SCALE"

	- <https://github.com/truenas>
	- <https://www.truenas.com/docs/>
	- <https://www.truenas.com/download-truenas-scale/>
    - PGP Key: `C8D6 2DEF 767C 1DB0 DFF4 E6EC 358E AA91 12CF 7946`

??? info ":simple-openmediavault: openmediavault"

	> openmediavault is the next generation network attached storage (NAS) solution based on Debian Linux. It contains services like SSH, (S)FTP, SMB/CIFS, RSync and many more. Thanks to the modular design of the framework it can be enhanced via plugins. openmediavault is primarily designed to be used in home environments or small home offices, but is not limited to those scenarios. It is a simple and easy to use out-of-the-box solution that will allow everyone to install and administrate a Network Attached Storage without deeper knowledge.
	>
	> Note: openmediavault (like other NAS solutions) expects to have full, exclusive control over OS configuration and cannot be used within a container. Also, no graphical desktop user interface can be installed in parallel.

	- <https://github.com/openmediavault/openmediavault>
	- <https://docs.openmediavault.org/en/stable/installation/index.html>


## :material-server-network: Hypervisors

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


## :octicons-beaker-16: Labs & Simulations

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

??? example "Nuclei-Templates-Labs"

	> A comprehensive collection of vulnerable environments paired with ready-to-use Nuclei templates for practical security testing and learning!

	- <https://github.com/projectdiscovery/nuclei-templates-labs>

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


## :fontawesome-solid-gear: Hardware

??? example "geerlingguy/mini-rack"

	> Miniature rack builds, for portable or compact Homelabs.

	- <https://github.com/geerlingguy/mini-rack>


## :fontawesome-solid-code: DevOps

!!! abstract "Open Source Licensing"

	If you need to choose a license, or learn more about one, these are the best resources to start with:

	- [choosealicense.com](https://choosealicense.com/appendix/) (created by GitHub)
	- [GitHub Docs: Licensing a Repo](https://docs.github.com/articles/licensing-a-repository)
	- [Open Source Guide: Choosing a License](https://opensource.guide/legal/#which-open-source-license-is-appropriate-for-my-project) (recommended by GitHub)
	- [SPDX License ID List](https://spdx.org/licenses/) (found via [licensee](https://github.com/licensee/licensee))
	- [Open Community and Resource Hub for Open Source Managers and OSPO Practitioners](https://github.com/todogroup) (found via [licensee](https://github.com/licensee/licensee))

	**licensee**

	GitHub uses [licensee](https://github.com/licensee/licensee) to detect a repo license. [This page](https://github.com/licensee/licensee/blob/master/docs/what-we-look-at.md) details what licensee looks at to determine the repo license.

	**SPDX License Identifiers**

	The [SPDX license guidance](https://spdx.dev/learn/handling-license-info/) is the easiest way to note what license(s) each file falls under in your project.

	> In each file in your project, just add a single line in the following format, tailored to your license(s) and the comment style for that file's language:
	>
	> - `// SPDX-License-Identifier: MIT`
	> - `/* SPDX-License-Identifier: MIT OR Apache-2.0 */`
	> - `# SPDX-License-Identifier: GPL-2.0-or-later`

??? example "debos"

	> debos is a tool to make the creation of various Debian-based OS images simpler. While most other tools focus on specific use-cases, debos is designed to be a toolchain making common actions trivial while providing enough rope to do whatever tweaking which might be required behind the scenes.
	>
	> debos expects a YAML file as input and runs the actions listed in the file sequentially. These actions should be self-contained and independent of each other.

	Discovered in the [Kali Linux build-scripts repo for kali-vm](https://gitlab.com/kalilinux/build-scripts/kali-vm/-/tree/main?ref_type=heads#kali-vm-image-builder).

	- <https://github.com/go-debos/debos>

??? example "pulumi"

	> Pulumi Infrastructure as Code is the easiest way to build and deploy infrastructure, of any architecture and on any cloud, using programming languages that you already know and love. Code and ship infrastructure faster with your favorite languages and tools, and embed IaC anywhere with Automation API.
	>
	> Simply write code in your favorite language and Pulumi automatically provisions and manages your resources on AWS, Azure, Google Cloud Platform, Kubernetes, and 120+ providers using an infrastructure-as-code approach. Skip the YAML, and use standard language features like loops, functions, classes, and package management that you already know and love.

	Suggested by [yroc92](https://github.com/yroc92) for infrastructure-as-code workflows.

	- <https://github.com/pulumi/pulumi>
	- <https://www.pulumi.com/docs/>

??? example "Answer Files (Unattend.xml)"

	> Answer files (or Unattend files) can be used to modify Windows settings in your images during Setup. You can also create settings that trigger scripts in your images that run after the first user creates their account and picks their default language.
	>
	> Windows Setup will automatically search for answer files in certain locations, or you can specify an unattend file to use by using the `/unattend:<file.xml>` option when running Windows Setup (setup.exe).

	- [Microsoft: Automate Windows Setup](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/automate-windows-setup?view=windows-11)

	This is a series of useful resources for automated or unattended Windows provisioning.

	- <https://github.com/StefanScherer/packer-windows>
	- <https://github.com/rgl/windows-vagrant>
	- <https://schneegans.de/windows/unattend-generator/>
	- <https://github.com/cschneegans/unattend-generator/>
	- [Microsoft: Create your own Answer file](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/update-windows-settings-and-scripts-create-your-own-answer-file-sxs?view=windows-11)
	- [Microsoft: Unattended Windows Setup Reference](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/)
	- [Microsoft: Answer File Search Order](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-automation-overview?view=windows-11#implicit-answer-file-search-order)
	- [Microsoft: setup.exe /unattend](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-command-line-options?view=windows-11#unatten)
	- [Microsoft: Disk Configuration XML](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-setup-diskconfiguration-disk-modifypartitions-modifypartition-typeid#xml-example)
		- [Microsoft: How to Configure UEFI/GPT-based Disks (Manually)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-8.1-and-8/hh824839(v=win.10))
		- [Microsoft: How to Configure UEFI/GPT-based Disks (Autounattend.xml)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-8.1-and-8/hh825702(v=win.10))
		- [Microsoft: How to Configure BIOS/MBR-based Disks](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-8.1-and-8/hh825146(v=win.10))

??? example "Windows unattend.xml Generator"

	This is both a .NET library and website that can be used to generate customized `unattend.xml` files to provision Windows machines. Playing around with the settings is a great way to see how these files can be built, especially if you compare them to the files found in [StefanScherer/packer-windows](https://github.com/StefanScherer/packer-windows).

	Suggested by [rpinz](https://github.com/rpinz) as a way to pick up Windows automation faster.

	- <https://github.com/cschneegans/unattend-generator>
	- <https://schneegans.de/windows/unattend-generator/>


### :simple-git: Git

??? example "Git Submodules"

	Submodules allow you to integrate a separate project into another using git. The project included as a submodule becomes version pinned to the commit at the point-in-time it was integrated. It's effectively a nested git repo within what becomes your "superproject", and can be pulled + updated to stay in sync with the submodule's repo. This is easy to do and helps you avoid duplicating work by reusing existing code.

	- <https://git-scm.com/book/en/v2/Git-Tools-Submodules>

	To add a submodule to a repo:

	```bash
	# <submodule-name> can be specified as an option, if you want to change it
	# Otherwise the repo name is used by default
	git submodule add <repo-url> [<submodule-name>]
	```

	This creates the following `.gitmodules` file in the root of your repo:

	```yaml
	[submodule "<submodule-name>"]
        path = <submodule-name>
        url = <repo-url>
	```

	You can manually edit and revise these files.

	> There is a `foreach` submodule command to run some arbitrary command in each submodule. This can be really helpful if you have a number of submodules in the same project.

	```bash
	# Update all submodules in a repo
	git submodule foreach 'git pull'
	```


### :simple-github: GitHub

??? example ":octicons-terminal-16: gh cli"

	> `gh` is GitHub on the command line. It brings pull requests, issues, and other GitHub concepts to the terminal next to where you are already working with git and your code.

	- <https://github.com/cli/cli>
	- GPG Key File:
		- [0x23F3D4EA75716059](https://keyserver.ubuntu.com/pks/lookup?search=2C61+0620+1985+B60E+6C7A++C873+23F3+D4EA+7571+6059&fingerprint=on&op=index)
		- <https://cli.github.com/packages/githubcli-archive-keyring.gpg>

	```txt
	pub   rsa4096/0x23F3D4EA75716059 2022-09-06 [SC] [expires: 2026-09-05]
		Key fingerprint = 2C61 0620 1985 B60E 6C7A  C873 23F3 D4EA 7571 6059
	uid                             GitHub CLI <opensource+cli@github.com>
	sub   rsa4096/0xE5FAF19590714157 2022-09-06 [E] [expires: 2026-09-05]
	```


### :material-ansible: Ansible

!!! abstract "Ansible"

	Most of my Ansible notes are a part of the README over at my [straysheep-dev/ansible-configs](https://github.com/straysheep-dev/ansible-configs) repo.

	With that said, the most general and useful notes or links can be found here.

	- [Ansible GitHub](https://github.com/ansible/ansible)
	- [Installing `ansible`](https://docs.ansible.com/ansible/latest/installation_guide/index.html)
	- [Installing `anisble-lint`](https://ansible.readthedocs.io/projects/lint/installing/)
	- [`ansible-lint` Rules](https://ansible.readthedocs.io/projects/lint/rules/)

??? example "Ansible-Galaxy"

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

??? example "Molecule"

	> Molecule aids in the development and testing of Ansible content: collections, playbooks and roles.

	Discovered by reviewing [geerlingguy's ansible-role-docker ci.yml file](https://github.com/geerlingguy/ansible-role-docker/blob/master/.github/workflows/ci.yml).

	- <https://github.com/ansible/molecule>
	- <https://ansible.readthedocs.io/projects/molecule/>

??? example "Ansible for DevOps"

	> This repository contains Ansible examples developed to support different sections of Ansible for DevOps, a book on Ansible by Jeff Geerling.
	>
	> Many examples use Vagrant, VirtualBox, and Ansible to boot and configure VMs on your local workstation.
	>
	> Not all playbooks follow all of Ansible's best practices, as they illustrate particular Ansible features in an instructive manner.

	- <https://github.com/geerlingguy/ansible-for-devops>

??? example "pfsensible"

	Ansible modules for managing a pfSense firewall.

	- <https://github.com/pfsensible/core>


#### :simple-docker: Docker

??? example "setup-docker-action"

	> GitHub Action to set up (download and install) Docker CE. Works on Linux, macOS and Windows.
	>
	> This action is useful if you want to pin against a specific Docker version or set up a custom daemon configuration or if Docker is not available on your runner. If you're using GitHub-hosted runners on Linux or Windows, Docker is already up and running, so it might not be necessary to use this action.

	- <https://github.com/docker/setup-docker-action>

??? example "Docker SDK for Python"

	> A Python library for the Docker Engine API. It lets you do anything the docker command does, but from within Python apps - run containers, manage containers, manage Swarms, etc.

	- <https://github.com/docker/docker-py>
	- [Ansible for Devops: Molecule Examples](https://github.com/geerlingguy/ansible-for-devops/tree/master/molecule)


### :simple-hashicorp: HashiCorp

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


## :material-information-variant-circle: Information Technology

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


## :material-security: Information Security


### :material-dns: DNS

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

??? info "DNS4EU"

	> Supported by the European Union Agency for Cybersecurity (ENISA), the European Union's DNS4EU secure-infrastructure project provides a protective, privacy-compliant, and resilient DNS service to strengthen digital sovereignty and security for EU citizens, governments, and critical infrastructure.

	Discovered on the [SANS ISC Stormcast from Tuesday, June 10th, 2025](https://isc.sans.edu/podcastdetail/9486).

	- <https://www.joindns4.eu/>
	- [Resolver Configuration Information](https://www.joindns4.eu/for-public#resolver-options)

	Currently there's no browser-based diagnostic endpoint dedicated to verifying you're using their resolvers like Cloudflare or Quad9 have.


## :material-sword-cross: Offense

!!! abstract "PTES (Penetration Testing Execution Standard)"

	The PTES is a standard first drafted in 2009. It's designed to provide both businesses and security service providers with a common language and scope for performing penetration testing. It's been referenced in a number of training courses and by those who helped create it over the years. [The FAQ provides additonal overview](https://github.com/OpenSBK/ptes/blob/main/faq.md).

	Kevin Johnson (SecureIdeas, OpenSBK) was on [Paul's Security Weekly #785](https://securityweekly.com/psw-785) to talk about this updated version of the PTES, now on GitHub for others to contribute to. It was mentioned again in [SWN-453](https://securityweekly.com/swn-453), where one of OpenSBK's goals in addition to updating the PTES will be defining the language used in InfoSec.

	- OpenSBK's PTES: <https://github.com/OpenSBK/ptes>
	- Original Site: <http://www.pentest-standard.org/index.php>
	- Original Site (Wayback Machine): <https://web.archive.org/web/20211220050516/http://www.pentest-standard.org/index.php/Main_Page>

!!! quote "Unified Entity Context"

	[Unified Entity Context: The AI Stack Everyone is Building Without Realizing It](https://www.youtube.com/watch?v=IHUqk90ch7I)

	This was discussed as one piece of another episode of Unsupervised Learning by Daniel Miessler. The gist of the episode is AI with a complete context of information, which includes accuracy, understanding of stale or historical information, having access to current information, organizational goals, and more, leads to a world where decision making and understanding isn't splintered and siloed by front ends, platforms, or databases. It's effectively central to an organization with RBAC handled through identity. While all data "lives" in a central location, this removal of silos and handling of access maintains security while improving decision making and understanding by those interacting with the data.

	The highlight though is **the approach to a security assessment** that was discussed.

	Ask these questions to each "layer" of an organization, starting with the C-suite:

	- What is the core function of the business?
	- What are the company's goals?
	- Where is input and output data flowing?

	Use the answers to build a visual map, to illustrate what the company really looks like:

	- Data flow and storage
	- Vendors
	- Who is accessing what
	- What should and should not be in this map

	This creates the context or foundation for you to begin any assessment by revealing areas to focus on you would otherwise not know about without this context.

??? abstract "HackTricks"

	HackTricks is one of the largest resources of hacking knowledge. The original author also created [PEASS](https://github.com/peass-ng/PEASS-ng).

	- <https://book.hacktricks.wiki/>
	- <https://github.com/HackTricks-wiki/hacktricks>

### :material-network: Network

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


### :material-application-brackets: Web Application

!!! abstract ":simple-owasp: OWASP Top 10"

	> The OWASP Top 10 is a standard awareness document for developers and web application security. It represents a broad consensus about the most critical security risks to web applications.
	>
	> Globally recognized by developers as the first step towards more secure coding.
	>
	> Companies should adopt this document and start the process of ensuring that their web applications minimize these risks. Using the OWASP Top 10 is perhaps the most effective first step towards changing the software development culture within your organization into one that produces more secure code.

	- <https://owasp.org/www-project-top-ten/>
	- <https://github.com/OWASP/www-project-top-ten/blob/master/index.md>

!!! abstract ":simple-owasp: OWASP Web Security Testing Guides"

	>  The Web Security Testing Guide is a comprehensive Open Source guide to testing the security of web applications and web services.

	- <https://github.com/OWASP/wstg>
	- [WSTG Table of Contents](https://github.com/OWASP/wstg/tree/master/document)
	- [WSTG Checklists](https://github.com/OWASP/wstg/tree/master/checklists)
	- [OWASP Cheat Sheets](https://github.com/OWASP/CheatSheetSeries/tree/master/cheatsheets)

??? abstract "PayloadsAllTheThings"

	A collection of payloads and techniques for application security / web application pentesting.

	- <https://swisskyrepo.github.io/PayloadsAllTheThings/>
	- <https://github.com/swisskyrepo/PayloadsAllTheThings>

??? example ":simple-burpsuite: Burpsuite"

	Web application security testing tool, has free and paid licenses.

	- <https://portswigger.net/burp>

??? danger ":simple-zap: ZAProxy"

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


### :simple-linux: Linux & Unix-like

??? danger "GTFOBins"

	> GTFOBins is a curated list of Unix binaries that can be used to bypass local security restrictions in misconfigured systems.

	- <https://gtfobins.github.io/>

??? danger "LOLESXi"

	> LOLESXi features a comprehensive list of binaries/scripts natively available in VMware ESXi that adversaries have utilised in their operations. The information on this site is compiled from open-source threat research.

	- <https://github.com/LOLESXi-Project/LOLESXi>

??? bug "Curing"

	> Curing is a POC of a rootkit that uses io_uring to perform different tasks without using any syscalls.

	Discovered on [Schneier's blog](https://www.schneier.com/blog/archives/2025/04/new-linux-rootkit.html).

	- <https://github.com/armosec/curing>


### :material-microsoft-windows: Windows

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

??? danger "Bypassing Windows Defender Application Control (WDAC) with Loki C2"

	> ...because I replaced the Teams /resources/app/ directory with Loki C2 Agent's code, the Electron-based Teams application now executes Loki C2 Agent's JavaScript inside the trusted Teams process.

	- <https://securityintelligence.com/x-force/bypassing-windows-defender-application-control-loki-c2/>

	Additional resources from the article:

	- [Microsoft: Applications that can Bypass WDAC and How to Block Them](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/applications-that-can-bypass-appcontrol)
	- [T1218.015: System Binary Proxy Execution: Electron Applications](https://attack.mitre.org/techniques/T1218/015/)
	- <https://github.com/mttaggart/quasar>
	- <https://taggart-tech.com/quasar-electron/>

	To try and summarize this:

	- WDAC is a security boundary to prevent executing untrusted software
	- [LOLBAS](https://lolbas-project.github.io/) details these kinds of bypasses
	- Electron applications are interesting targets
		- They're the reverse of EXE / DLL's in that the Electron EXE exposes API's to .js files and .node modules
		- Goal is to execute arbitrary JS and leverage existing Node modules through a trusted Electron application
		- Electron apps can enforce integrity checking on JS scripts they execute which can help mitigate this vector
	- Node modules have limitations, but can be stealthier
		- Largely undocumented internal structures
		- If you use a function of a signed Node module instead of PowerShell, you avoid spawning processes and being caught
		- Execute everything in JS to remain stealthier
	- The post uses the legacy version of Teams to achieve code execution and C2 deployment via JS

??? danger "Bolthole"

	> A proof-of-concept ClickOnce payload for Red Teams to establish initial access in authorized penetration tests.

	- <https://github.com/rvrsh3ll/Bolthole>
	- [Exploring ClickOnce and .NET Hijacking for SSH Initial Access](https://www.youtube.com/watch?v=Zid7tB0Iyss)

	This is another BHIS tool presentation, this time by Steve Borosh. To try and summarize:

	- Use Azure WebApps to deliver the payload through a ClickOnce dialogue, on a "trusted" domain
	- This is the same style of prompt you get when opening application links in the browser
	- Effectiveness depends on the ruse / social engineering / circumstance
	- The app is a signed ssh / sshd package in .NET
	- It's designed and embedded with the key material to connect back to your own server
	- The connection does not allow command execution
	- The connecting user has no shell `sudo useradd -m -s /usr/sbin/nologin clientnameuser`
	- The attacker gains a full SOCKS proxy through the endpoint
	- This may not look entirely malicious to some tools

	How to detect this:

	- Look at networked processes on endpoints
	- Inspecting protocols on ALL ports
	- Inspecting code signatures on binaries
	- Preventing execution of unauthorized applications

??? danger "CarbonCopy"

	> A tool which creates a spoofed certificate of any online website and signs an Executable for AV Evasion. Works for both Windows and Linux.

	- <https://github.com/paranoidninja/CarbonCopy>

	*Mentioned in Discord at [42:57 of Exploring ClickOnce and .NET Hijacking for SSH Initial Access](https://www.youtube.com/watch?v=Zid7tB0Iyss).*

??? danger "SigThief"

	> Stealing Signatures and Making One Invalid Signature at a Time.

	- <https://github.com/secretsquirrel/SigThief>

	*Mentioned in Discord at [42:57 of Exploring ClickOnce and .NET Hijacking for SSH Initial Access](https://www.youtube.com/watch?v=Zid7tB0Iyss).*

??? danger "LazySign"

	> Create fake certs for binaries using windows binaries and the power of bat files.

	- <https://github.com/jfmaes/LazySign>

	*Mentioned in Discord at [42:57 of Exploring ClickOnce and .NET Hijacking for SSH Initial Access](https://www.youtube.com/watch?v=Zid7tB0Iyss).*

??? danger "Limelighter"

	> A tool which creates a spoof code signing certificates and sign binaries and DLL files to help evade EDR products and avoid MSS and sock scruitney. LimeLighter can also use valid code signing certificates to sign files. Limelighter can use a fully qualified domain name such as `acme.com`.

	- <https://github.com/Tylous/Limelighter>

	*Mentioned in Discord at [42:57 of Exploring ClickOnce and .NET Hijacking for SSH Initial Access](https://www.youtube.com/watch?v=Zid7tB0Iyss).*


### :simple-apple: macOS

	⚠️ TO DO ⚠️


### :material-microsoft-windows: Active Directory

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

### :material-wifi: Wireless

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

??? tip "Chrome's Built-in Bluetooth Scanner"

	Discoverd on [PSW #867](https://securityweekly.com/psw-867).

	Access the interface via: `chrome://bluetooth-internals` to scan for nearby bluetooth devices.

	- This works on most desktop versions of Chrome
	- Does not appear to work on iOS


### :material-cloud-cog-outline: Cloud

??? danger "MSFTRecon"

	> MSFTRecon is a reconnaissance tool designed for red teamers and security professionals to map Microsoft 365 and Azure tenant infrastructure. It performs comprehensive enumeration without requiring authentication, helping identify potential security misconfigurations and attack vectors.

	- <https://github.com/Arcanum-Sec/msftrecon>

??? danger "GraphRunner"

	> GraphRunner is a post-exploitation toolset for interacting with the Microsoft Graph API. It provides various tools for performing reconnaissance, persistence, and pillaging of data from a Microsoft Entra ID (Azure AD) account.
	>
	> It consists of three separate parts:
	>
    > - A PowerShell script where the majority of modules are located
    > - An HTML GUI that can leverage an access token to navigate and pillage a user's account
    > - A simple PHP redirector for harvesting authentication codes during an OAuth flow

	- <https://github.com/dafthack/GraphRunner>

??? example "Cloud S.L.A.W"

	Cloud Security Lab a Week is a free and high-quality cloud security training platform.

	Discovered on [Unsupervised Learning NO. 471 ](https://newsletter.danielmiessler.com/p/ul-471).

	- <https://slaw.securosis.com/>


### :material-power-plug-battery-outline: ICS & OT

??? abstract "HardwareAllTheThings"

	> Hardware/IOT Pentesting Wiki

	- <https://swisskyrepo.github.io/HardwareAllTheThings/>
	- <https://github.com/swisskyrepo/HardwareAllTheThings>


### :material-lan-connect: C2

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

??? danger "TrevorC2"

	> TrevorC2 is a legitimate website (browsable) that tunnels client/server communications for covert command execution. Written by: Dave Kennedy (@HackingDave) Website: <https://www.trustedsec.com>.
	>
	> TrevorC2 is a client/server model for masking command and control through a normally browsable website. Detection becomes much harder as time intervals are different and does not use POST requests for data exfil.

	- <https://github.com/trustedsec/trevorc2>


## :material-security: Defense

### :material-server-security: Security Platforms

!!! abstract "Why Platforms?"

	This sections covers what I would call "Security Platforms". This is intentionally generic, and meant to be a catch-all for the security "things" out there that are not a single-purpose tool, framework, or a solution specific to any system.

	This section may be broken down later, but for now it includes everything from XDR, SIEM, and AppSec solutions to vulnerability scanners and monitoring platforms.

??? example "wazuh"

	wazuh is a fully open source XDR and SIEM. It's very easy to setup and use, supporting deployment on a number of endpoints. Anything it can't deploy an agent on can ship logs to a central log server that can be ingested later by wazuh.

	- <https://wazuh.com/>
	- <https://github.com/wazuh>

	Overview:

	- SIEM / Log analysis
	- EDR
	- File integrity monitoring
	- Vulnerability detection
	- Configuration assessment

	One drawback to wazuh is that the agent is a regular system process and not a kernel-level process. Ideally you would be catching something before it reaches the kernel and evades detection, but this is what some may consider its weakness compared to other XDR solutions.

??? example "Secuity Onion"

	- <https://github.com/Security-Onion-Solutions>

??? question "runZero"

	runZero is an asset discovery and attack surface management platform designed to work without agents or authentication. There's a [community edition](https://www.runzero.com/platform/community-edition/) with full features available for up to 100 IPs, which is free for personal / lab use.

	Founded by HD Moore (creator of Metasploit), this has come up on the PSW family of shows on various occassions for its ability to accurately fingerprint devices.

	- <https://www.runzero.com/>
	- <https://github.com/runZeroInc>

??? question "Nessus"

	Nessus is possibly the most well known vulnerability scanner. Any pentesting course is likely to introduce Nessus as a method of assessing targets via a network scan.

	See my [notes on Nessus](../posts/nessus.md).

	- <https://www.tenable.com/products/nessus>

??? question "OpenVAS"

	OpenVAS is the [only remaining open-source fork](https://greenbone.github.io/docs/latest/history.html) of the original Nessus vulnerability scanner before it went closed-source back in 2005.

	> All [release files](https://github.com/greenbone/openvas/releases) are signed with the [Greenbone Community Feed integrity key](https://community.greenbone.net/t/gcf-managing-the-digital-signatures/101). This gpg key can be downloaded at https://www.greenbone.net/GBCommunitySigningKey.asc and the fingerprint is `8AE4 BE42 9B60 A59B 311C  2E73 9823 FAA6 0ED1 E580`.

	- <https://github.com/greenbone/openvas-scanner>
	- <https://greenbone.github.io/docs/latest/>

	OpenVAS is available in Kali as the `gvm` package.

	- <https://greenbone.github.io/docs/latest/22.4/kali/index.html>
	- <https://www.kali.org/blog/configuring-and-tuning-openvas-in-kali-linux/>
	- <https://www.kali.org/tools/gvm/>

??? question "Sandfly Security"

	Sandfly is an agentless Linux EDR and IR platform, doing everything via SSH. Their blog has a number of interesting examples and resources on Linux threat detection.

	- <https://sandflysecurity.com/>
	- <https://sandflysecurity.com/blog>

??? example "ntop"

	- <https://github.com/ntop/ntopng>

??? example "BLUESPAWN"

	- <https://github.com/ION28/BLUESPAWN>

??? example "OpenEDR"

	- <https://github.com/ComodoSecurity/openedr>

??? example "LimaCharlie"

	- <https://limacharlie.io/>


### :material-microsoft-windows: Windows

??? question "audit-inspector (Windows)"

	> Audit Inspector is a binary tool written in Rust for Windows audit configuration and auditing.

	- <https://github.com/blackhillsinfosec/audit-inspector>


### :simple-linux: Linux

??? tip "Kernel Lockdown Mode"

	You can set the Linux kernel to lockdown mode by enabling Secure Boot, or by setting the mode manually in GRUB.

	- <https://github.com/torvalds/linux/blob/master/security/lockdown/Kconfig>
	- [manpages: kernel_lockdown](https://manpages.ubuntu.com/manpages/hirsute/man7/kernel_lockdown.7.html)

	What this does:

	- Modifications to the kernel at runtime will require a reboot
	- Makes an attacker's job more difficult at gaining kernel level persistence or escaping the hypervisor. (Stops a number of known rootkits)
	- Disabling Secure Boot at the UEFI level requires physical or serial console access for MOK in the firmware during the boot process

	How this can be defeated:

	- If signing scripts and keys are present on the system at the time of compromise
	- Novel attack against the Linux kernel

	How this can be detected:

	- [Monitor and log root command execution with auditd](https://github.com/Neo23x0/auditd) or [Sysmon](https://github.com/Sysinternals/SysmonForLinux)
	- Watch invokation of `/usr/src/linux-headers-$(uname -r)/scripts/sign-file` and `mokutil`
	- If you did not schedule a shutdown or reboot that occured, you may have been compromised.


	Kernel Command Line Parameters (full list)

	- [kernel.org/admin-guide/kernel-parameters (txt)](https://www.kernel.org/doc/html/latest/_sources/admin-guide/kernel-parameters.rst.txt)
	- [kernel.org/admin-guide/kernel-parameters (html)](https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html)

	>         lockdown=       [SECURITY]
	>                         { none | integrity | confidentiality }
	>                         Enable the kernel lockdown feature. If set to
	>                         integrity, kernel features that allow userland to
	>                         modify the running kernel are disabled. If set to
	>                         confidentiality, kernel features that allow userland
	>                         to extract confidential information from the kernel
	>                         are also disabled.


	Configuring Lockdown Mode:

	- `lockdown=integrity` is enabled by default on systems with UEFI Secure Boot
	- **The recommended way of making persistent kernel-command-line modifications is via GRUB/GRUB2**
	- This is done with `grub` (Debian/Ubuntu) or `grubby` (RHEL/Fedora) via the boot parameters.

	Get the current lockdown setting:

	```bash
	cat /sys/kernel/security/lockdown
	```

	**Setting Lockdown Mode on Debian/Ubuntu or Fedora/RHEL**

	```bash
	sudo nano /etc/default/grub
	```

	Change `GRUB_CMDLINE_LINUX=""` to have `GRUB_CMDLINE_LINUX="lockdown=<mode>"` *appended* (if there are already other CMDLINE parameters).

	Mode can be one of `none`, `integrity`, or `confidentiality`

	EXAMPLE: `GRUB_CMDLINE_LINUX="lockdown=integrity"`

	After making any changes:

	```bash
	# Debian / Ubuntu
	sudo update-grub

	# Fedora / RHEL
	sudo grub2-mkconfig > /dev/null
	```

	**Setting Lockdown Mode on Raspberry Pi**

	This file points to all of the boot parameters, typically in the same directory:

	```txt
	/boot/firmware/config.txt
	```

	To modify a kernel command line parameter as you would in /etc/default/grub:

	```bash
	sudo echo 'lockdown=confidentiality' >> /boot/firmware/nobtcmd.txt
	```

	**All Distros**

	No matter which Linux distribution you're making these changes on, to complete the process you'll need to reboot.

	```bash
	# Reboot after making changes
	sudo systemctl reboot
	```

	To verify the changes on reboot:

	```bash
	cat /sys/kernel/security/lockdown
	none [integrity] confidentiality
	```

??? "CPU Vulnerabilities & Mitigations"

	This was originally discovered on the Ubuntu Security Podcast / Ubuntu blog. The original source will need to be linked and added if it can be tracked down.

	Running this command will enumerate all existing mitigiations, issues, or otherwise, affecting the current CPU. This can be run from both a guest VM or the host machine.

	```bash
	head -n-0 /sys/devices/system/cpu/vulnerabilities/*
	```


### :material-magnify-scan: Threat Hunting

??? abstract "Pivot Atlas"

	A visualization of paths in threat intelligence.

	- <https://gopivot.ing/>
	- <https://github.com/korniko98/pivot-atlas>

??? abstract "LOLRMM"

	> Welcome to LOLRMM (Living Off the Land Remote Monitoring and Management), a community-driven project that provides a curated list of Remote Monitoring and Management (RMM) tools that could potentially be abused by threat actors. Our mission is to assist security professionals in staying informed about these tools and their potential for misuse, providing the community a catalog of these tools which can be used for threat hunting, detection and prevention policy creations.

	- <https://lolrmm.io/>
	- <https://github.com/magicsword-io/LOLRMM>

??? question "pspy"

	`pspy` is potentially the most useful application for examining system activity in real time. Because of this, it's as much a threat hunting tool as it is a threat.

	Static and linked binaries are available to download from the GitHub release page.

	- <https://github.com/DominicBreuker/pspy>

	```bash
	## pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855

	curl -LfO 'https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy32'
	7cd8fd2386a30ebd1992cc595cc1513632eea4e7f92cdcaee8bcf29a3cff6258  pspy32

	curl -LfO 'https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy32s'
	0265a9d906801366210d62bef00aec389d872f4051308f47e42035062d972859  pspy32s

	curl -LfO 'https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64'
	f7f14aa19295598717e4f3186a4002f94c54e28ec43994bd8de42caf79894bdb  pspy64

	curl -LfO 'https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64s'
	c769c23f8b225a2750768be9030b0d0f35778b7dff4359fa805f8be9acc6047f  pspy64s
	```

??? bug "YARA"

	Malware rule, pattern, and classification Language. Yara can take a pattern or rule file, and look at other files or processes either on a live system, or offline. This includes disk images or memory dumps, text dumps of strings, and more.

	- <https://github.com/VirusTotal/yara>
	- <https://yara.readthedocs.io/en/stable/>

??? example "yarGen"

	Programmatic YARA rule generation.

	- <https://github.com/Neo23x0/yarGen>

??? example "YARA-Rules Repository"

	- <https://github.com/Yara-Rules/rules>

??? bug "RITA"

	> Real Intelligence Threat Analytics (RITA) is a framework for detecting command and control communication through network traffic analysis.

	- <https://github.com/activecm/rita>

	RITA is possibly the ultimate open-source network threat hunting tool. Using Zeek logs, it can determine:

    > - Beaconing Detection: Search for signs of beaconing behavior in and out of your network
    > - Long Connection Detection: Easily see connections that have communicated for long periods of time
    > - DNS Tunneling Detection: Search for signs of DNS based covert channels
    > - Threat Intel Feed Checking: Query threat intel feeds to search for suspicious domains and hosts

	The deployment method has changed since they have moved to using Ansible. It can be dropped onto the local system with:

	```bash
	./install_rita.sh "localhost"
	```

	Or one-or-more remote systems by passing a comma-separated list of endpoints to the installer like this:

	```bash
	./install_rita.sh "root@<ip_1>,<ip_2>,myhost.internal"
	```

	A RITA "server" can ingest Zeek logs into multiple data sets, either on a recurring / rolling basis as part of a crontask or as needed into unique data sets. This means you can continue to add to specific data sets and perform analysis of specific networks over time.


??? danger "Zeek"

	> Zeek is a powerful network analysis framework that is much different from the typical IDS you may know.

	- <https://zeek.org/>
	- <https://github.com/zeek/zeek>

	Zeek is the tool you'll want to use to create the necessary logs to ingest with RITA for network threat hunting. Zeek itself doesn't need to be running on every endpoint, you can simply write and log packet captures of network activity using something like `netsh`, `pktmon`, `tshark` or `tcpdump` to be sent to a central logging server where Zeek can transform those captures into Zeek logs.

	If you're using the built-in Windows `netsh` and `pktmon` tools, you'll likely need to convert the capture files into pcaps with [etl2pcapng](https://github.com/microsoft/etl2pcapng).

??? danger "Suricata"

	- <https://suricata.io/download/>
	- <https://github.com/OISF/suricata>

??? danger "Velociraptor"

	Agent based incident response tool.

	- <https://docs.velociraptor.app/>
	- <https://github.com/Velocidex/velociraptor>

??? danger "Chainsaw"

	> Chainsaw provides a powerful first-response capability to quickly identify threats within Windows forensic artefacts such as Event Logs and the MFT file. Chainsaw offers a generic and fast method of searching through event logs for keywords, and by identifying threats using built-in support for Sigma detection rules, and via custom Chainsaw detection rules.

	- <https://github.com/countercept/chainsaw>

??? danger "Hayabusa"

	> Hayabusa is a sigma-based threat hunting and fast forensics timeline generator for Windows event logs.

	- <https://github.com/Yamato-Security/hayabusa>

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

??? example "rkchk"

	> Rust Linux Kernel Module designed for LKM rootkit detection.

	Discovered on [PSW #867](https://securityweekly.com/psw-867), this is a proof-of-concept kernel module to detect other LKM-based rootkits. The README has a list of LKM rootkits it has been tested against.

	- <https://github.com/thalium/rkchk>
	- <https://blog.thalium.re/posts/linux-kernel-rust-module-for-rootkit-detection/>


## :fontawesome-regular-rectangle-list: GRC

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

??? abstract "Software Bill of Materials (SBOM)"

	⚠️ TO DO ⚠️


## :material-debug-step-over: Reverse Engineering

??? bug "Ghidra"

	> Ghidra is a software reverse engineering (SRE) framework.

	- <https://github.com/NationalSecurityAgency/ghidra>

??? bug "Ghidriff"

	> Python Command-Line Ghidra Binary Diffing Engine.

	- <https://github.com/clearbluejar/ghidriff>

??? bug "GhidraMCP"

	> ghidraMCP is an Model Context Protocol server for allowing LLMs to autonomously reverse engineer applications. It exposes numerous tools from core Ghidra functionality to MCP clients.

	See the videos in the README for a demo and install steps.

	- <https://github.com/LaurieWired/GhidraMCP>

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


## :material-shield-bug-outline: Malware Analysis

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


### :material-bug-check: Known Samples

??? bug "xz-utils Backdoor"

	> Malicious code was discovered in the upstream tarballs of xz, starting with version 5.6.0. Through a series of complex obfuscations, the liblzma build process extracts a prebuilt object file from a disguised test file existing in the source code, which is then used to modify specific functions in the liblzma code. This results in a modified liblzma library that can be used by any software linked against this library, intercepting and modifying the data interaction with this library.

	- <https://nvd.nist.gov/vuln/detail/cve-2024-3094>
	- <https://isc.sans.edu/diary/30802>
	- <https://gist.github.com/smx-smx/a6112d54777845d389bd7126d6e9f504>

??? bug "Unicode Steganography & Google Calendar C2"

	[NPM Attack Leveraging Unicode Steganography and Google Calendar C2](https://www.veracode.com/resources/sophisticated-npm-attack-leveraging-unicode-steganography-and-google-calendar-c2/)

	Check the screenshot of the hexdump, the many dots following the `|` character are the obfuscated glyphs carrying the base64 data in their low byte. This is only a small piece of this research, but the invisible unicode characters are likely the most interesting part.

	The research does the best job of unwinding this, but essentially:

	- A .js file had a decode()-to-eval() happening on (what seems like) a single `|` character, which makes no sense at first glance
	- The [Variation Selectors Supplement](https://en.wikipedia.org/wiki/Variation_Selectors_Supplement) range (U+E0100 to U+E01EF) was used to hide base64 data
	- This provides 240 possible characters, base64 only requires 64 characters `[A-Za-z0-9+/]` with `=` for padding
	- These characters have no "glyph", and are therefore *invisible*
	- The key here is `xx` in `U+E01xx`, which remember the available range is U+E0100 to U+E01EF, offered enough space and variation to store valid base64 data
	- This is only "visible" by hexdumping the .js file

	**Demo**

	The easiest way to visualize this is using the [`chr()`](https://docs.python.org/3/library/functions.html#chr) function in python:

	```python3
	# Prints the copyright symbol
	print(chr(0x00A9))

	# Prints one of these invisible glyphs
	print(chr(0xE0100))
	```

	**Detection**

	- Line length
	- Hexdump
	- Changing `eval()` to `console.log()`

	Despite how awesome this technique is, it eventually has to decode itself to plaintext to execute. This will likely appear in some type of log or network capture.


## :octicons-cpu-16: Firmware


### :octicons-info-16: Information

??? abstract "bootloaders.io"

	> bootloaders.io is a curated list of known malicious bootloaders for various operating systems. The project aims to assist security professionals in staying informed and mitigating potential threats associated with bootloaders.

	- <https://github.com/magicsword-io/bootloaders>

??? abstract "DBX Update Process"

    - <https://eclypsium.com/2022/07/26/firmware-security-realizations-part-1-secure-boot-and-dbx/>


### :octicons-gear-24: Projects

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


### :material-bug-check: Vulnerabilities

??? bug "BootHole"

	This includes the vulnerability check for both bash and PowerShell.

    - <https://github.com/eclypsium/BootHole>

??? danger "baton drop (CVE-2022-21894)"

	> Windows Boot Applications allow the truncatememory setting to remove blocks of memory containing "persistent" ranges of serialised data from the memory map, leading to Secure Boot bypass.

	- <https://github.com/Wack0/CVE-2022-21894>

??? bug "LogoFAIL"

	> The Binarly REsearch team investigates vulnerable image parsing components across the entire UEFI firmware ecosystem and finds all major device manufacturers are impacted on both x86 and ARM-based devices.
	>
	> This attack vector can give an attacker an advantage in bypassing most endpoint security solutions and delivering a stealth firmware bootkit that will persist in an ESP partition or firmware capsule with a modified logo image.

	- <https://www.binarly.io/blog/the-far-reaching-consequences-of-logofail>

??? bug "GRUB Unpatched for 15 Months in Major Distros"

	Discovered on [PSW #867](https://securityweekly.com/psw-867), GRUB has been vulnerable to UEFI / boot-level attacks for 15+ months, and none of the major distros have integrated the patches as of March in 2025.

	- [Phoronix Post by Michael Larabel](https://www.phoronix.com/news/GRUB2-73-Patches-Security-2025)
	- [GRUB2 Mailing List Thread Detailing the 73 Patches](https://lists.gnu.org/archive/html/grub-devel/2025-02/msg00024.html)

	A few points based on the show notes and the article:

	- If Microsoft doesn't update the DBX or SBAT to prevent loading these versions of GRUB, you can use a vulnerable version of GRUB via USB boot to bypass Secure Boot
	- This potentially allows USB-based boot attacks to bypass GPG signed boot, GRUB passwords, and possibly full disk encryption
	- Also includes a vulnerability similar to [LogoFAIL](https://www.binarly.io/blog/the-far-reaching-consequences-of-logofail), exploiting image parsers


## :material-magnify: Forensics

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


### :material-memory-arrow-down: Memory Acquisition

??? danger "avml (Acquire Volatile Memory for Linux)"

	Do this remotely with ssh + avml (acquire volatile memory for linux).

	- <https://github.com/microsoft/avml>


??? danger "lmg (Linux Memory Grabber)"

	Do this remotely with ssh + lmg (linux memory grabber).

	- <https://github.com/halpomeranz/lmg>


## :material-search-web: OSINT

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

??? question "ipinfo.io"

	IP address data, this site was originally discovered in TCM's ethical hacking course videos on YouTube as a way to get your current IP address, and lookup other IP addresses. Their GitHub has a CLI utility to use with the API. See their [pricing](https://ipinfo.io/pricing) for usage details and example data, from the free tier up to enterprise.

	- <https://ipinfo.io/>
	- <https://ipinfo.io/what-is-my-ip>
	- <https://github.com/ipinfo>

	The easiest way to interact with the service is via their website with `curl` to check your IP. This does not require auth or any subscription to check.

	```bash
	curl https://ipinfo.io/json
	```


### :octicons-file-code-24: Vulnerability Research

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
	- <https://github.com/CVEProject/cvelistV5> (CVE cache of the official CVE List in CVE JSON 5 format)

??? question "CVEMap"

	> Navigate the Common Vulnerabilities and Exposures (CVE) jungle with ease using CVEMAP, a command-line interface (CLI) tool designed to provide a structured and easily navigable interface to various vulnerability databases.
	>
	> CVEMap CLI is built on top of the CVEMap API that requires API Token from ProjectDiscovery Cloud Platform that can be configured using environment variable named PDCP_API_KEY or using interactive -auth option.

	- <https://github.com/projectdiscovery/cvemap>

	References within the README:

	- **[National Vulnerability Database (NVD)](https://nvd.nist.gov/developers)**: Comprehensive CVE vulnerability data.
	- **[Known Exploited Vulnerabilities Catalog (KEV)](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)**: Exploited vulnerabilities catalog.
	- **[Exploit Prediction Scoring System (EPSS)](https://www.first.org/epss/data_stats)**: Exploit prediction scores.
	- **[HackerOne](https://hackerone.com/hacktivity/cve_discovery)**: CVE discoveries disclosure.
	- **[Nuclei Templates](https://github.com/projectdiscovery/nuclei-templates)**: Vulnerability validation templates.
	- **[Trickest CVE](https://github.com/trickest/cve) / [PoC-in-GitHub](https://github.com/nomi-sec/PoC-in-GitHub/)** GitHub Repository: Vulnerability PoCs references.

??? bug "Google OSV (Open Source Vulnerabilities)"

	> An open, precise, and distributed approach to producing and consuming vulnerability information for open source.

	OSV is a project and service developed by Google. All advisories in this database use the OpenSSF OSV format, which was developed in collaboration with open source communities.

	- <https://osv.dev/>
	- <https://google.github.io/osv.dev/>

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


### :material-bug: Malware Research

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


## :material-wallet: Cryptocurrency

??? info "Trezor Wallet"

	Trezor is a physical hardware wallet for cryptocurrency. These devices also have other functionality, such as U2F, FIDO, and WebAuth.

	- <https://trezor.io/>
	- <https://docs.trezor.io/>

	The easiest way to use a Trezor hardware wallet is through the Trezor Suite. Download the Trezor suite AppImage (recommended) and check the signatures:

	- <https://github.com/trezor/trezor-suite/releases>
	- Trezor [signing key via trezor.io](https://trezor.io/security/satoshilabs-2021-signing-key.asc)
	- Trezor [signing key on the Ubuntu Keyserver](https://keyserver.ubuntu.com/pks/lookup?search=EB483B26B078A4AA1B6F425EE21B6950A2ECB65C&fingerprint=on&op=index)

	```
	pub   rsa4096/0xE21B6950A2ECB65C 2021-01-04 [SC]
		Key fingerprint = EB48 3B26 B078 A4AA 1B6F  425E E21B 6950 A2EC B65C
	uid                             SatoshiLabs 2021 Signing Key
	```

	Or access the web version here:

	- <https://suite.trezor.io/web>
	- Chrome (web-USB or `trezord`) or Firefox only
	- `trezord` is a local daemon that was required when only the web version existed

	Install udev rules for the hardware wallet's usb connection: [51-trezor.rules](https://github.com/trezor/trezor-suite/blob/develop/packages/suite-data/files/bin/udev/51-trezor.rules)

	```bash
	sudo cp 51-trezor.rules /etc/udev/rules.d/
	sudo udevadm control --reload
	```

	For reference, Yubico's udev rules with Trezor are here: [70-u2f.rules](https://github.com/Yubico/libu2f-host/blob/master/70-u2f.rules)

	Next, you'll likely need to install the previous FUSE binaries.

	- <https://github.com/appimage/appimagekit/wiki/fuse>

	```bash
	sudo add-apt-repository universe

	# 24.04 and later
	sudo apt install libfuse2t64

	# 22.04 and earlier
	sudo apt install fuse libfuse2

	sudo modprobe fuse
	sudo groupadd fuse

	user="$(whoami)"
	sudo usermod -a -G fuse $user

	sudo systemctl reboot
	```

	If you have usbguard running, when updating the Trezor wallet's firmware, you'll need to "add" it again in the policy list. When the device reboots into bootloader mode for updates, its characteristics change. To usbguard this is a different device.

	```bash
	usbguard list-devices
	sudo usbguard allow-device <id> -p
	```


## :material-robot: Artificial Intelligence

!!! abstract "OWASP AI Exchange"

	The OWASP AI Exchange is a collection of projects related to AI privacy and security. The testing resources are similar to the OWASP Top 10, but for AI.

	- <https://owaspai.org/>
	- <https://github.com/OWASP/www-project-ai-security-and-privacy-guide>
	- [AI Security Testing](https://owaspai.org/docs/5_testing/)


### :octicons-tools-16: Tools

??? example "ollama"

	> Get up and running with Llama 3.3, DeepSeek-R1, Phi-4, Gemma 3, Mistral Small 3.1 and other large language models.

	- <https://github.com/ollama/ollama>
	- <https://ollama.com/>
	- [Documentation](https://github.com/ollama/ollama/tree/main/docs)

	ollama was showcased in [Keeping Things Local - Making Your Own Private LLM w/ Bronwen Aker](https://www.youtube.com/watch?v=DbzkJRl_xnk). It's incredibly easy to get started, and have a local LLM running without a ton of resources.

	Think of ollama as a CLI frontend with access to a repo of vetted LLM's. This is different from Hugging Face, which is not a frontend, but is more like GitHub (the wild west).

	Essentially, you can download and run models, customize them fairly quickly and easily using files similar to [Daniel Miessler's fabric patterns](https://github.com/danielmiessler/fabric/tree/main/patterns), and through RAG (retrieval-augmented generation) contextualize them around a local set of files or data.

	All of this was learned from Bronwen's presentation.

??? quote "Keeping Things Local - Making Your Own Private LLM w/ Bronwen Aker"

	- <https://www.youtube.com/watch?v=DbzkJRl_xnk>
	- [Slides](https://www.blackhillsinfosec.com/wp-content/uploads/2025/04/SLIDES_Keeping-Things-Local-BHIS-webcast-2025.04.03.pdf)

	This is a full walkthrough of how to do this yourself, put together by Bronwen over at [BHIS](https://www.blackhillsinfosec.com/blog/). It's straight forward and easy to understand if you're looking to get started with local LLM's.

	- ollama is essentially a frontend to interface with and retrieve vetted LLM's
	- Hugging Face is more like GitHub, and not every model is vetted, however official models from trusted sources are available there as well
	- The resoure requirements aren't high, WSL can run these too since Windows passes the GPU through
	- It's easy to customize a model
	- RAG (retrieval-augmented generation) is the process of referencing documents and data without re-training a model (which is expensive and hard)

??? example "fabric"

	> fabric is an open-source framework for augmenting humans using AI. It provides a modular framework for solving specific problems using a crowdsourced set of AI prompts that can be used anywhere.

	- <https://github.com/danielmiessler/fabric>
	- [Patterns](https://github.com/danielmiessler/fabric/tree/main/patterns)

	The pattern files are raw markdown that provide context to each query you make.

	The [Keeping Things Local - Making Your Own Private LLM w/ Bronwen Aker](https://www.youtube.com/watch?v=DbzkJRl_xnk) demo was a reminder of this. It works not only with external API-based services, but also local LLM's.


??? example "Google Gemini"

	Google's AI model (previously known as Bard). As of March 2025 it ranks the highest of any model for coding.

	- <https://gemini.google.com>
	- [Gemini Apps Privacy Hub](https://support.google.com/gemini/answer/13594961)

	If you don't want your input and output to be used for training or human review, you can turn off Gemini Apps Activity. This does not delete past data. As with any AI assume any inputs or outputs may become public or used for training.

	Gemini is usable within Google Workspace. According to their policy it will not share or train on input or output of emails, documents, or otherwise unless you decide to allow that.

	Other interesting features include:

	- NotebookLM: Process text, PDFs, Google Docs, websites, and more into summaries that include a podcast-like audio conversation about the content
	- [Sec-Gemini](https://security.googleblog.com/2025/04/google-launches-sec-gemini-v1-new.html) is built to augment SOC workflows with Mandiant and OSV data

??? example "OpenAI"

	Possibly the most popular general-use AI. It's capable of text and audio conversation, coding, image and video generation and editing. There's also web search, deep research, and scheduled tasks.

	- <https://openai.com/>
	- <https://chatgpt.com/>
	- <https://sora.com/>

	If you don't want your input and output to be used for training or shared to the explore feed in Sora, you can turn this off in the settings of both ChatGPT and Sora, where they're on by default. As with any AI assume any inputs or outputs may become public or used for training.

??? example "Cloudflare: AI Labyrinth"

	> ...we decided to use a new offensive tool in the bot creator's toolset that we haven't really seen used defensively: AI-generated content. When we detect unauthorized crawling, rather than blocking the request, we will link to a series of AI-generated pages that are convincing enough to entice a crawler to traverse them. But while real looking, this content is not actually the content of the site we are protecting, so the crawler wastes time and resources.
	>
	> As an added benefit, AI Labyrinth also acts as a next-generation honeypot. No real human would go four links deep into a maze of AI-generated nonsense. Any visitor that does is very likely to be a bot, so this gives us a brand-new tool to identify and fingerprint bad bots, which we add to our list of known bad actors. Here's how we do it...

	- <https://blog.cloudflare.com/ai-labyrinth/>


### :material-lightning-bolt-circle: Attacks

??? danger "The Policy Puppetry Attack"

	> The attacks in this blog leverage the Policy Puppetry Attack, a novel prompt attack technique created by HiddenLayer researchers. By reformulating prompts to look like one of a few types of policy files, such as XML, INI, or JSON, an LLM can be tricked into subverting alignments or instructions. As a result, attackers can easily bypass system prompts and any safety alignments trained into the models. Instructions do not need to be in any particular policy language. However, the prompt must be written in a way that the target LLM can interpret as policy. To further improve the attack's strength, extra sections that control output format and/or override specific instructions given to the LLM in its system prompt can be added.

	- <https://hiddenlayer.com/innovation-hub/novel-universal-bypass-for-all-major-llms/>

	Discovered on <https://securityweekly.com/swn-471>.


## :fontawesome-solid-book-atlas: Blogs & Authors

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

??? quote "AI for Humans"

	Randomly discovered while looking for shows to cover AI news. This is a comedy show that packs a lot of the weekly news into an hour with links to tools and examples.

	- <https://www.aiforhumans.show/>
