---
title: "Untrusted Environments"
icon: material/wifi-cancel
draft: true
#date:
#  created: 2025-06-27
#  updated: 2026-09-01
categories:
  - zerotrust
  - networking
  - netsec
  - dns
  - ntp
  - dhcp
  - ssl
  - tls
  - vpn
  - mitm
  - proxy
  - poisoning
  - tunnelvision
  - lan
  - wifi
  - firewall
---

# :material-wifi-cancel: Operating in Untrusted Environments

!!! abstract "Treat Every Network as Hostile"

    At some point everyone, even security professionals, will need to use an untrusted network for one reason or another. It doesn't make sense, for example, to burn gigs of data tethering to a mobile hotspot when you can configure your device to withstand hostile LAN activity on a random Wi-Fi access point.

    Additionally you may have heard that it's safer to connect to the official DEFCON Wi-Fi instead of using cell service during and around the conference. This can lead to a lot of questions if you don't know why and how that's true.

    The truth is, you should *assume any network is hostile*.

    This post aims to explore and document this. My opinion is if controlling or being on a LAN was such an easy win for attackers ***in the last few years***, it would be trivial to hack anything at any time like it was 15+ years ago. Equally some things are easy to miss that can leave machines exposed. It's taken a long time to get here, and the internet was not built with security in mind, but things have changed in the last 5 or so years.

    Let's look at this from all angles to obtain a general set of "things" to do when preparing to use untrusted (potentially hostile) networks.

<!-- more -->


## Proposed Sections

??? warning "Under Construction"

    This section is being built and will be completed over time.

    - Overview + FAQ
        - How DNS Works
            - DNS over TLS/HTTPS
        - How TLS Works
            - SSLStrip
            - HSTS
            - Certificate Stores
        - Public Key Fingerprint Checking (GPG, SSH, etc)
        - Why the DEFCON Wi-Fi *is* Safer
        - History of Exploits Targeting Network Stacks
            - LAN
            - Wi-Fi
            - Bluetooth
            - Others
        - Cellular vs LAN Threats
        - Firewall vs VPN for Untrusted Networks
    - History of Network Attacks
        - Captive / Hostile Portal
        - Evil Twin
        - Relaying
        - LAN Attacks
            - Poisoning, MITM
            - DHCP resource exhaustion
            - DNS over TLS / HTTPS endpoint blocking via rogue DHCP server
            - Fallback to plaintext conditions
                - Windows can force no fallback
                - Linux can force no fallback
                - iOS **may** fallback and let the user know via a dialogue encrypted DNS is blocked
            - TunnelVision and DHCP Option 121
                - Rogue routes
                - TunnelVision vs DNS via Tailscale
                - TunnelVision vs Application Layer
        - Additional Resources
    - Threat modeling
        - Patch Your Things
        - The Defaults are Often OK
        - Turning Off Services or Devices
        - What Network is Completely Safe?
        - Visibility and Alerting is Key
    - Settings for Untrusted LANs
        - Linux Network Settings
            - Disabling Services
            - `nmcli`, `netplan`, `networkd`
            - Detection and Monitoring
        - Windows Network Settings
            - Disabling Services
            - Detection and Monitoring
        - Android Network Settings
            - Detection and Monitoring
        - macOS Network Settings
            - Objective-See Tools
            - Detection and Monitoring
        - iOS Network Settings
            - .mobileconfig / Tailscale
            - VPN Application Configurations vs DNS .mobileconfig Configurations (Tailscale vs NextDNS / Cloudflare)
            - Detecting Malicious Network (LAN) Behavior on iOS

## Overview

You'll see **untrusted sources** a lot in this article. This is anything we do not control, and cannot guarantee the integrity of.

!!! example "Example"

    We can control our local machine's DNS resolver, but cannot always control the LAN's. Similarly we do not control Google or Cloudflare's time servers, but they are considered trustworthy sources.

!!! tip "Essentials"

    The essential items when dealing with untrusted LANs, in order of most important are:

    - Your host's firewall, and local services (**this is RCE territory**)
    - Your host's networking behavior and how you're accepting DHCP settings (or not)
    - Enforcing data integrity (via TLS, or similar measures)

## Attacks

!!! danger "Evil Twins are Trivial on Public / Guest Networks (or why the DEFCON WiFi is safer)"

    **Why This Works**: Without TLS certificate validation, the pre-shared key is doing *everything* here. (auth + verification)

    Most public networks freely publish the ESSID and pre-shared key (if there even is one). In the best case scenario, the venue assigns each user a unique pre-shared key, runs a fully patched AP with WPA3, and fully isolates all client traffic. This is completely doable, but it's not something you should rely on.

    In most cases, the ESSID *is* the pre-shared key. Try this on your home WiFi, without any hacking tools, you can run another AP (Raspberry Pi, Ubuntu + hostapd, etc.) that has the same:

    * ESSID
    * BSSID (MAC address)
    * Pre-shared Key
    * Authentication Type (WPA2/3)

    At this point, how does your client device know which one is the real one? **There's actually no way to verify which one is the real AP**.

    The DEFCON WiFi by contrast provisions each user with a unique TLS certificate. So long as you obtained the certificate from the real DEFCON website, and enforce client-side certificate validation, evil twin attacks aren't possible without a novel exploit.

    Home and small office WPA2/3 networks are entirely protected by the pre-shared key. If that's ever discovered, authentication, verification, and integrity are lost. WPA3 at least [prevents offline brute-force of the key exchange frames](https://datatracker.ietf.org/doc/html/rfc7664) ***and*** retroactively decrypting previously captured traffic if the key is ever discovered.

!!! bug "Cellular Baseband Exploits"

    2G and 3G are notably [disabled in iOS's lockdown mode](https://support.apple.com/en-us/105120), primarily because the authentication, verification, and / or encryption in these protocols is broken.

    - [CVE-2022-21744](https://nvd.nist.gov/vuln/detail/CVE-2022-21744)
    - [CVE-2025-31214](https://nvd.nist.gov/vuln/detail/CVE-2025-31214)
    - TODO: add more examples

!!! danger "Physical Attacks"

    "Physical access means adversaries win" is a general rule, but you can take steps to make it significantly harder. Ultimately you end up with a base level of protection at best and tamper detection at worst.

    Rubber ducky devices can be connected directly, or between existing USB devices, to inject keystrokes on unlock. Detection is somewhat obvious once it happens, and response speed depends on what the payload typed out and what your local security tools are capable of.

    Similar to rubber ducky attacks, simply attaching a mass storage device that gets auto-mounted can have consequences depending on the configuration of your OS. URL files, autoruns (which shouldn't be a thing on modern Windows) or a file type that a local system service will automatically parse resulting in some sort of unintended action (this is more uncommon but not impossible) are all examples in this category.

    Shoulder surfing is another real threat, making biometrics or MFA on your workstation a prevention layer. In extreme circumstances biometrics can backfire, (e.g. unlock via fingerprint while asleep or similar).

    Chassis intrusion is also on this list. Between a firmware password, secure boot, full disk encryption, EDR, and putting paint or nail polish in the screw heads of the chassis (yes, this is a thing) this can be at least detected.

---

## DHCP

Dynamic host configuration protocol.

!!! tip "TLDR"

    You can harden your system's intake of DHCP data if you understand how your system's networking stack behaves. This is a large part of the battle on untrusted LANs.

    Setting DNS and NTP manually, while ignoring the DHCP options, is critical. Similarly, setting routes manually and ignoring those provided can be easy or tedious depending on the system, but can prevent many of the LAN attack paths, including TunnelVision (below).

    Additionally you can set a randomized MAC address here for your system too.

!!! info "[TunnelVision (CVE-2024-3661)](https://www.leviathansecurity.com/blog/tunnelvision)"

    [https://github.com/leviathansecurity/TunnelVision](https://github.com/leviathansecurity/TunnelVision)

    This research is the best example, of what a hostile DHCP server can do.

    The DHCP-based attack vectors on a LAN depend on what device is being malicious:

    - Malicious neighbor: [ARP cache poisoning](https://en.wikipedia.org/wiki/ARP_spoofing) or [DHCP lease starvation](https://wazuh.com/blog/monitoring-dhcp-starvation-attack-with-suricata-and-wazuh/) + a hostile DHCP server hosted by the attack's machine
    - Malicious router (mostly transparent)

    This is not breaking VPN crypto, but a hostile DHCP server can tell your system to send tunneled traffic to the LAN first. This is transparent to the user. It also is not an immediate win for attackers, but here are some scenerios where it can be a risk. If you're sending any of the following data over a VPN, an attacker's machine may get a copy first and potentially respond:

    - Any unencrypted traffic relying on the tunnel
    - Web requests to self-signed CA's (internal or dev applications)
    - Active directory connections (NTLM hashes, domain information)
    - DNS requests

    What does this mean for VPN protocols & providers?

    - This is a DHCP option (121), it's not breaking any VPN protocol, it's abusing system networking behavior
    - If your host, or the VPN application is configured to account for this, that's one remedy
    - Not all hosts and VPN applications can, and do, guard against this

    [Tailscale's analysis](https://tailscale.com/blog/tunnelvision-analysis) of this for their own service illustrates it very well. Wireguard itself (used by Tailscale) has [a mechanism on Linux](https://www.wireguard.com/netns/#the-new-namespace-solution) that can be used to mitigate this issue. But each OS is different.

??? example "Apple Devices & Tailscale - Extended Review"

    The Apple ecosystem (excluding AppleTV for some reason) is largely still vulnerable to this attack, particularly when using full tunneling (exit-nodes). In this specific case, *not* using full tunneling makes the attack on your Tailnet assets a little trickier, potentially mitigating it (but with no guarantee). With DHCP option 121, the more specific route wins. That's how TunnelVision works. On iOS and macOS, in addition to the more specific route, it's unclear what happens if there's a tie, as in two `/32` routes for the same host are pushed, but on different interfaces. Both Google and Claude suggest the most recently defined route wins in a tie, which would mean the malicious route always wins if it gets defined. [This forum post suggests there's a priority](https://developer.apple.com/forums/thread/724430), however there's nothing that seems to answer this specifically in the [developer docs](https://developer.apple.com/documentation/networkextension/routing-your-vpn-network-traffic), and instead this should be tested.

    For example, you can use the [Hurricane Electric Tools app for iOS](https://networktools.he.net/) to examine your systems routes. On macOS you can simply use the CLI tools, which iOS does not have. As of the time of writing, every Tailnet asset gets a `/32` route. An attacker would need to flood the routing table with `/32` routes for the entire CGNAT range. This becomes a less likely attack scenario, but you're still left without a security guarantee.

    The bottom line seems to be, do not use untrusted networks, while also needing to rely on tunneling, with Apple operating systems. macOS now has a way to detect this and alert the user (in Tailscale's case), but generally, use a hotspot or LAN you control if possible. This does not mean you cannot safely use untrusted networks on macOS, just be aware that TunnelVision is an attack vector if you do.

---

## NTP

NTP can be overlooked, because it doesn't really lead to sensitive data interception or code execution. But it's a critical part of a system.

!!! tip "Why it's Important"

    [Network time protocol](https://en.wikipedia.org/wiki/Network_Time_Protocol) servers can be set by DHCP. Even a non-malicious, bad server can break:

    - Many crypto functions such as OTP code calculation, kerberos, and more
    - Other system services and internals that rely on accurate system clocks
    - Logging and timestamps for forensics

    Since this can be set by DHCP, we should set it ourselves and ignore any NTP settings that come from untrusted sources.

!!! info "Network Time Security"

    This is the DNS over TLS / HTTPS of NTP. It's not encrypted, but packet integrity is guaranteed.

    [It's described in RFC 8915](https://blog.cloudflare.com/nts-is-now-rfc/) and many NTP daemons support it.

!!! info "Good NTP Servers"

    See [resources#ntp](../resources.md#ntp).

---

## DNS

!!! tip "TLDR"

    DNS is another thing that DHCP can set. Therefore we must set it ourselves, and ignore any DNS settings coming from DHCP (untrusted sources).

!!! info "Good DNS Servers"

    See [resources#dns](../resources.md#dns)

!!! abstract "DNS over TLS/HTTPS"

    These overviews by Cloudflare on [what DNS is](https://www.cloudflare.com/learning/dns/what-is-dns/) and [securing DNS](https://www.cloudflare.com/learning/dns/dns-over-tls/) demonstrate how this works in detail.

    This does *not* rely on HSTS (below), but is handled by the built-in system certificate store (OS or browser). It uses TLS to prevent you from using a poisoned DNS server, or a third-party from seeing and tampering with your DNS queries.

    This is a must, in general, for untrusted LANs or not, as interception can happen at any point on the network path.

---

## TLS

Most connections effectively depend on TLS certificate stores (though, they aren't the only crypto / signature system that can be used for confidentiality / integrity / access). These are often shipped as part of the OS, a browser, or sometimes specific apps ship their own certs, pinning the app to them to ensure communications are secure. All of this is to avoid mitm attacks, or interception and modification of network data in transit.

See how SSL and TLS works [here](https://www.cloudflare.com/learning/ssl/how-does-ssl-work/).

The mechanisms below are how TLS works today, and how it has been attacked.

!!! tip "TLDR"

    During mitm attacks, in most cases domains will fail the TLS certificate check and display the error in your browser. There are only some edge cases where this can be side-stepped transparently to you, if the data itself is not tunneled or otherwise protected.

!!! quote "[HTTP Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security)"

    The HTTP Strict-Transport-Security response header (often abbreviated as HSTS) informs browsers that the host should only be accessed using HTTPS, and that any future attempts to access it using HTTP should automatically be upgraded to HTTPS. Additionally, on future connections to the host, the browser will not allow the user to bypass secure connection errors, such as an invalid certificate. HSTS identifies a host by its domain name only.

    ...

    One weakness of HSTS is that it does not take effect until the browser has made at least one secure connection to the host and received the Strict-Transport-Security header. If the browser loads an insecure http URL prior to knowing that the host is an HSTS host, the initial request is vulnerable to network attacks. Preloading mitigates this problem.

!!! quote "Preloading"

    Google [maintains an HSTS preload service](https://hstspreload.org/). By following the guidelines and successfully submitting your domain, you can ensure that browsers will connect to your domain only via secure connections. While the service is hosted by Google, all browsers are using this preload list. However, it is not part of the HSTS specification and should not be treated as official.

    Information regarding the HSTS preload list in Chrome: [https://www.chromium.org/hsts/](https://www.chromium.org/hsts/)

    Consultation of the Firefox HSTS preload list: [nsSTSPreloadList.inc](https://searchfox.org/firefox-main/source/security/manager/ssl/nsSTSPreloadList.inc)

!!! tip "Check HSTS"

    Review HSTS and other settings in your browsers:

    - [Chrome](chrome://net-internals/#hsts)
    - [Firefox](about:networking)

**References**

- [wikipedia: HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)
- [mitmproxy/sslstrip.py](https://github.com/mitmproxy/mitmproxy/blob/d3df6b956732888eb0c6081c31bc194846e9edae/examples/contrib/sslstrip.py#L1)
- [bettercap/hstshijack](https://github.com/bettercap/caplets/tree/master/hstshijack)

---

## Hardening Guidance

Covering the bases of hardening considerations.

### Windows

=== "Firewall"

    "[Shields Up Mode](https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/tools#shields-up-mode-for-active-attacks)", this setting alone will prevent services that may have installed their own firewall allow-rules from lowering the security of your system. This is a generally recommended setting for workstations, even in enterprise environments. Apply it to all network profiles, and you'll be covered as far as firewall rules go no matter what network you're on.

    Localhost, Virtual Machine (WSL, Windows Sandbox, and Hyper-V) all still function just fine with this enabled.

=== "DNS"

    [Enforce DNS over HTTPS](https://learn.microsoft.com/en-us/windows-server/networking/dns/doh-client-support), and prevent falling back to insecure DNS settings.

=== "Other System Settings"

    This can include hardening measures to the OS, the browser, services, and more. These guides stand out.

    - [HotCakeX/Harden-Windows-Security](https://github.com/HotCakeX/Harden-Windows-Security), a Windows hardening guide that only uses methods and features native and officially supported by Microsoft. No third-party dependencies required.
    - [Disassembler0/Win10-Initial-Setup-Script](https://github.com/Disassembler0/Win10-Initial-Setup-Script), now archived, but a great reference for dealing with registry adjustments without needing Active Directory. My version of this is [here](https://github.com/straysheep-dev/windows-configs/blob/main/Set-SecurityBaseline.ps1).
    - [Microsoft Edge Policies](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-policies), complete list of registry keys for Edge, my version automating this is [here](https://github.com/straysheep-dev/windows-configs/blob/main/Set-EdgePolicy.ps1).
    - All of my own resources and scripts are published under [straysheep-dev/windows-configs](https://github.com/straysheep-dev/windows-configs).

    Generally you want to account for:

    - Prevent unauthorized behavior on the lock screen, e.g. changing network settings or shutting down the machine.
    - Enforce all of the ASR policies, few should really prevent any normal use.
    - Use Windows Sandbox for untrusted files or browsing; keep in mind it also needs a configuration file.
    - Ensure auto-mount, auto-run, and auto-play behavior is disabled or limited as much as the GPOs allow.

---

### macOS

The best resource for this is [Dr Duh's macOS security and privacy guide](https://github.com/drduh/macos-security-and-privacy-guide).

---

### Linux

Assuming you're using a Linux desktop, it can quickly feel very complicated with all of the components involved. Fortunately, everything can be tuned without much trouble.

=== "Firewall"

    If on Debian-based Linux, you're using `ufw`:

    ```bash
    sudo ufw reset --force
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw default deny routed
    sudo ufw enable --force
    sudo ufw status verbose
    ```

    You can also comment out the lines in `/etc/ufw/before.rules` and `/etc/ufw/before6.rules` to prevent UPnP and mDNS from being allowed in.

    ```bash
    sudo sed -i -E '/^-A .*--dport (5353|1900) -j ACCEPT/ s/^/#/' /etc/ufw/before.rules
    sudo sed -i -E '/^-A .*--dport (5353|1900) -j ACCEPT/ s/^/#/' /etc/ufw/before6.rules
    sudo ufw reload
    sudo grep -E "5353|1900" /etc/ufw/before.rules /etc/ufw/before6.rules
    ```

    If on Fedora-based Linux, you're using `firewalld`:

    ```bash
    sudo systemctl enable --now firewalld
    sudo firewall-cmd --set-default-zone=drop
    sudo firewall-cmd --reload
    sudo firewall-cmd --get-default-zone
    sudo firewall-cmd --zone=drop --list-all
    ```

=== "Wired & Wireless"

    This can feel more complicated than it is, just know that generally on desktop environments, NetworkManager and by extension, `nmcli`, govern networking, while servers tend to simply use `systemd-networkd`. If you're using cloud-init, keep in mind it can overwrite certain configuration options on boot unless you undo those settings. Check `/etc/cloud/cloud.cfg.d/` if you're finding some settings don't persist after reboot.

    **Keep in mind `netplan` has limitations compared to NetworkManager with configuration writing. Some specific settings require passing through NetworkManager properties outside of the default schema.**

    **NetworkManager**

    See [my references for using NetworkManager](https://straysheep.dev/notes/commands-network/#networkmanager-nmcli).

    If your network renderer is NetworkManager, you should use `nmcli`. I have [templates published for both](https://github.com/straysheep-dev/linux-configs/tree/main/NetworkManager).

    This allows you to:

    - Ignore DHCP-provided DNS
    - Ignore DHCP-provided routes, easily set your own after looking at the network
    - Randomize your MAC
    - Ignore LLMNR and mDNS

    **netplan**

    If your network renderer is `systemd-networkd` (like it would be on Ubuntu Server), you'll want to use [netplan](https://netplan.readthedocs.io/en/stable/netplan-yaml/) (that link is the full configuration reference).

    I have my own configuration reference [published here](https://straysheep.dev/notes/commands-network/#netplan).

=== "DNS"

    In Linux `/etc/nsswitch.conf` generally determines what handles name resolution, and uses `/etc/resolv.conf` in nearly every case. Otherwise, your DNS is either governed by a DNS daemon like `systemd-resolved`, `bind`, `stubby`, or `unbound`, or even your web browser. Check with the following:

    ```bash
    # See what's configured.
    cat /etc/nsswitch.conf

    # See what's actually listening locally, if nsswitch.conf doesn't clarify this.
    sudo ss -tulnp | grep -E ':53\b|:853\b'
    ```

    I have templates for all of these [here](https://github.com/straysheep-dev/linux-configs/tree/main/dns) (except web browsers, those are [configured separately](https://github.com/straysheep-dev/browser-configs)). Each has their own configuration syntax, but the idea is always the same:

    - Ignore any DNS from the local subnet
    - Enforce your own, with DNS over TLS or HTTPS, and no plaintext fallback
    - Optionally log all queries

=== "NTP"

    Enforcing the use of a trusted NTP server prevents crypto from breaking. Things like MFA codes won't generate or be accepted on machines without properly synced time.

    I have templates for `systemd-timesyncd` [here](https://github.com/straysheep-dev/linux-configs/tree/main/ntp).

=== "Lockscreen"

    **USBGuard**

    Two things help here. First, USBGuard will prevent any unapproved USB device from functioning on your system. Be careful installing this; the way it works is at install time on Ubuntu and Kali, it will allow all currently connected devices on the ports they're connected to, but nothing else. You'll need to add devices manually after.

    ```bash
    sudo apt update; sudo apt install -y usbguard
    ```

    **Polkit**

    Next, polkit can modify the lockscreen behavior. I have a policy file template [here](https://github.com/straysheep-dev/linux-configs/tree/main/polkit) with comments that can be modified as needed.

    **MFA**

    You can also use a [Yubikey](https://support.yubico.com/s/article/Ubuntu-Linux-login-guide-U2F) or the `libpam-google-authenticator` package to setup MFA for your login. This reduces the effectiveness of shoulder surfing, as anyone would need MFA to also login to the machine if they have your password.

    ```bash
    sudo apt update; sudo apt install -y libpam-google-authenticator

    # Walk through the prompts and save the QR code / seed.
    google-authenticator
    ```

=== "Kernel"

    **Sysctl**

    Numerous kernel params exist that can be hardened. I maintain an [Ansible role](https://github.com/straysheep-dev/ansible-role-configure_kernel) that's fully commented, and applies everything possible without impacting normal functionality.

    **Lockdown Mode**

    Additionally whether you have secure boot enforced or not, you can manually set the Kernel's lockdown mode to confidentiality. Lockdown prevents unsigned kmods from loading, and confidentiality mode prevents even root from dumping certain process details. See [Kernel Lockdown](https://straysheep.dev/resources/#linux-linux-unix-like_1) for full details.

=== "Disk Encryption"

    Always encrypt the full disk. This does two things; prevents use of your data in the event of a theft (remove hard drive, read it on another machine to gather data) is a signal if someone power cycles the device while you're away, and you left your device locked (and even asleep).

    Failing to encrypt the full disk means determined attackers can still remove the disk, modify system files, and reinstall the disk. Effectively backdooring the system with trivial tools like system services or rootkits that are hidden from the OS or EDR. If you're following all pieces of this guide, the physical security and OPSEC considerations should alert you to this.

=== "Firmware Settings"

    Ideally you use UEFI and not BIOS. Additionally if possible, set a boot password on your machine. This way if someone tries to power cycle or otherwise tamper with the device (e.g. booting from a removeable USB) they cannot proceed after powering on without a password.

    Some firmware also supports chassis intrusion detection and remote wipe. This varies in firmware and actual effectiveness. Test it before you rely on it.

=== "Physical Security and OPSEC"

    This is broad, but two practical points taken from Dr Duh's guide for macOS and the SANS ISC podcast:

    - Use biometrics or MFA for login to mitigate shoulder surfing, or store half your login password on a Yubikey and the other half in your head
    - Use paint or nail polish on the chassis screw heads to detect tampering

---

### EDR

=== "Wazuh"

    [Wazuh](https://wazuh.com/) is a great choice if you want to roll your own EDR.

    I have extensive [notes published on deploying and maintaining Wazuh](https://straysheep.dev/notes/wazuh-tailscale/) for reference.
