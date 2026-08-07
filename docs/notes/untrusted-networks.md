---
title: "Untrusted Networks"
icon: material/wifi-cancel
draft: true
#date:
#  created: 2025-06-27
#  updated: 2026-08-06
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

# :material-wifi-cancel: Using Untrusted Networks

!!! abstract "Treat Every Network as Hostile"

    At some point everyone, even security professionals, will need to use an untrusted network for one reason or another. It doesn't make sense, for example, to burn gigs of data tethering to a mobile hotspot when you can configure your device to withstand hostile LAN activty on a random Wi-Fi access point.

    Additionally you may have heard that it's safer to connect to the official DEFCON Wi-Fi instead of using cell service during and around the conference. This can lead to a lot of questions if you don't know why and how that's true.

    The truth is, you should *assume any network is hostile*.

    This post aims to explore and document this. My opinion is if controlling or being on a LAN was such an easy win for attackers ***in the last few years***, it would be trivial to hack anything at any time like it was 15+ years ago. Equally some things are easy to miss that can leave machines exposed. It's taken a long time to get here, and the internet was not built with security in mind, but things have changed in the last 5 or so years.

    Let's look at this from all angles to obtain a general set of "things" to do when preparing to use untrusted (potentially hostile) networks.

!!! warning "Under Construction"

    This section is being built and will be completed over time.

<!-- more -->

## Proposed Sections

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
            - Rouge routes
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
    - Your hosts's networking behavior and how you're accepting DHCP settings (or not)
    - Enforcing data integrity (via TLS, or similar measures)

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