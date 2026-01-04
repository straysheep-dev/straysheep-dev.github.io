---
title: "Untrusted Networks"
icon: material/wifi-cancel
draft: true
#date:
#  created: 2025-06-27
#  updated: 2026-01-04
categories:
  - trust
  - networking
  - netsec
  - dns
  - dhcp
  - vpn
  - mitm
  - proxy
  - poisoning
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
