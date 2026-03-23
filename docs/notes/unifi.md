---
title: "UniFi"
icon: simple/ubiquiti
draft: true
#date:
#  created: 2026-01-04
#  updated: 2026-03-13
categories:
  - networking
  - unifi
  - firewall
  - switch
  - routing
---

# :simple-ubiquiti: UniFi

A guide to choosing, deploying, and integrating UniFi into infrastructure.

!!! abstract "Overview"

    UniFi specializes in network and security equipment. This includes switches, wireless access points, cameras, door access, and more. Their selling point is enterprise level gear and features without licensing or subscription fees.

    **Pros**

    Their strength is the ease of use once everything is set up. If you need a switch or access point, their devices are the best option for individual users or professionals with home labs looking to get started or upgrade their network.

    **Cons**

    The downside to how UniFi's system works is it becomes more "hands off" than most enterprise gear. Another way to look at this is the philosophy is different when compared to things like Linux or BSD distros designed for networking. You become locked-in to their ecosystem and lose the ability to automate or customize deployments with agnostic devops tools like Terraform or Ansible. Similarly your ability to debug and get granular insights into each device is not always there, leaving you stuck troubleshooting from a secondary set of devices to pinpoint issues that aren't being logged or raising alarms. As of 2025 this is slowly getting better, with more access to the internals of certain devices, but it still has a ways to go.

<!-- more -->

## Network Stack

The UniFi network stack is the management OS that handles all network devices. Gateways, switches, APs, and more. UniFi has a cloud-hosted option as well as a physical device (called a CloudKey) that handles this for you.

!!! tip "Self-host or Gateway"

	[The network OS can be self-hosted](https://help.ui.com/hc/en-us/articles/34210126298775-Self-Hosting-UniFi). All(?) UniFi gateway devices have the network controller OS built-in. If you're not adding a gateway immediately, it's highly recommended to simply self-host the network controller on a Linux VM in something like Proxmox. ARM does not yet appear to be supported, so Raspberry Pi's aren't an option until an arm64 release happens.

	A CloudKey only makes sense if you:

	- Need access to the other stacks outside of Network (Protect, Talk, Connect, and Access) and do not have the hardware for those
	- Do not have the infrastructure to self-host
	- Do not want to use the cloud-hosted option which has a subscription cost

!!! tip "Management Hidden in Menus"

    It often feels as though the "advanced" management features like SSH are buried in menus that aren't intuitive to find.

    | Device | Menu Path |
    | --- | --- |
    | Gateways | Settings > Control Plane > Console > SSH |
    | Switches | UniFi Devices > [Your Switch] > Settings (Gear) > Debug |
    | APs | UniFi Devices > [Your AP] > Settings (Gear) > Debug |

!!! tip "Update Cadence"

    UniFi's updates are very stable. That's why enabling aggressive auto-updating is relatively safe with little downside.

    - Some devices have multiple software stacks (e.g. the OS is separate from the Network or Protect stacks which have their own updates)
    - Most devices are configured to check once a week for new updates
    - You can safely change this to daily checks


### Gateways

Gateways can vary in capabilities containing some or all of the following features:

- Firewall (always?)
- Routing (always?)
- Network Controller stack (always?)
- IDS
- Integrated WiFi
- Switching (does each port have full 1/2.5/10GbE capability or is it a shared backplane?)

The key is, until this is tested with specific gateways and proven otherwise, it appears **you cannot adopt a gateway appliance into an existing self-hosted network controller**. In other words, if you're already running another stack (like pfSense or OPNSense and OpenWRT as the "heart" of your network) but adding UniFi devices to build and extend functionaility such as switching or WiFi, you'll have two options:

- Allow the new gateway to take the place of your self-hosted controller server, use the controller as a fallback if you ever remove the gateway
- Have "two sites", where the self-hosted controller and gateway are managed separately


### Switches

UniFi Switches are one of the best ways to extend and grow a network as well as reduce power adapters via PoE++ support.

These resources are key if you're appending UniFi switches or APs onto an existing network:

- [Understand and Mitigate Network Loops](https://help.ui.com/hc/en-us/articles/24292724428311-Understand-and-Mitigate-Network-Loops-STP)
    - [Best Practices for IOT Devices](https://help.ui.com/hc/en-us/articles/18930473041047-Best-Practices-for-Sonos-Devices)
    - [Best Practices for Chromecast and AirPlay](https://help.ui.com/hc/en-us/articles/4409866388887-Best-Practices-for-Chromecast-and-AirPlay)

!!! tip "STP Should Always be Enabled"

	Echoing what is emphasized in the best practice links above, there is almost never a reason to disable STP, even with just one switch, and specifically if there is more than one device doing switching in any network.

	Credit to [t3cht0n1c](https://github.com/t3cht0n1c) for review and discussion of this topic.


### Access Points

One of the most common setups is deploying a UniFi AP behind something like pfSense. In this case you'll need a Network Controller, either a self-hosted container/VM, a local CloudKey device, or a cloud-hosted option with UniFi.

!!! warning "Management"

	The AP's themselves cannot be managed directly like normal AP's without the controller stack running from another point on the network, ***or***, use of the UniFi Network mobile application, though management here is stand-alone and limited. This also creates a tricky situation if you have strict subnet or VLAN isolation via firewall rules. You will need to ensure the AP itself can talk to your controller.

**Debug Console**

Some APs allow you to acces the debug console. UniFi OS is based on OpenWrt, so many of the commands you'd expect are there to do basic troubleshooting and review of the system.

- UniFi OS UI > UniFi Devices > [Your AP] > Settings (Gear) > Debug
- This will open a small web interface dropping you into a shell
- There's a useful copy button `[+]` at the top right of this window, so you can dump the text into a text editor for review

!!! tip "Obtaining VAP MAC Addresses for Nzyme"

    One use case is tying the algorithmically changing MAC addresses back to the VAP and SSID they belong to, so you can configure Nzyme correctly to stop throwing alerts on valid access points. Frustratingly, this information is not available in log files or the UniFi OS UI.

    ```bash
    # This will dump all wireless interface information
    iw dev
    ```


### Travel Router

The [UniFi Travel Router](https://store.ui.com/us/en/products/utr) is an interesting and fun device. Assessing how this device performs and works will be documented here.

!!! abstract "Summary"

    Effectively, the Travel Router abstracts your local networking away from potentially hostile LANs but doesn't entirely remove the risks. If using it as a VPN tunnel + killswitch, it's maybe the best option out there for what it does. The only downsides are in its limited configurability (which could change with OS updates) and lack of any internal battery, even for moving to a new location (it will consume your phone's battery very fast if you're power it from your phone).


<div class="grid cards" markdown>

-   :material-web-plus:{ .lg .middle } __Pros__

    ---

    :material-plus-box: Tunnel to UniFi Gateways via Teleport

    :material-plus-box: Tunnel to any Wireguard VPN

    :material-plus-box: VPN killswitch option

    :material-plus-box: Great for a *less* hostile LAN, you control

    :material-plus-box: Very portable

    :material-plus-box: Great performance

    :material-plus-box: OK price of $79.00

    :material-plus-box: Can function as a WiFi extension

    :material-plus-box: WPA3

-   :material-web-minus:{ .lg .middle } __Cons__

    ---

    :material-minus-box: No DNS over TLS / HTTPS options (as of March, 2026)

    :material-minus-box: Limited SSH or advanced management features

    :material-minus-box: Cannot save known network keys

    :material-minus-box: Limited WiFi config (bands, client isolation)

    :material-minus-box: Coverage area can be small (environmental factors)

    :material-minus-box: No built-in minimal DNS logging, or system auditing

    :material-minus-box: No built-in cellular option

    :material-minus-box: Requires external power at all times (no battery)

</div>

As a final note, while it's an unreasonable ask, the option to continually scan for and attempt to connect to OPN / OWE networks would be interesting for on-the-go use.

!!! dannger "Hostile LAN Risks"

    To provide more context, some of the previously mentioned risks this will protect you from include:

    - Protects machines that talk *locally* over insecure protocols
    - Protects all LAN traffic if you can use the Teleport VPN
    - Protects LAN machines without strict deny-all inbound firewall rules
    - Protects vulnerable IOT devices

    What needs reviewed further, is if you can machine-in-the-middle updates or other outbound conversations the Travel Router makes on its own. It's very talkative on the "WAN" side, making frequent connectivity checks.


:simple-statuspal: **Setup**

The device setup is easy. The default WiFi network it spins up cannot be auto-joined, and does not appear to have a known hard-coded credential. Once you take ownership of the device through one of the methods below, you can configure it and it's "yours". The easiest and best way to do this is via the UniFi mobile app over Bluetooth.

- UniFi Network App + Bluetooth
- LAN / USB + management interface

!!! tip "OSINT"

    You can identify other nearby UTR's in the UniFi app and see what AP's they're connected to. However, if you try to "adopt" one that's already in use, it prompts you for the UniFi account's password.

:material-router-wireless: **Portability**

It's ***portable***. It may not be clear from the marketing material but it's tiny, and lightweight. The specifications are available on the store page, but for anecdotal comparison:

- 2/3 as long as the latest model iPhones, similar width and height
- Same length as a FlipperZero, half the height, and 1.5x the width
- 1/2 as tall as the Pineapple Pager, similar width and length
- Feels about as heavy as a FlipperZero

!!! tip "Heat"

    Those three "pocket" sized devices were compared intentionally, for this exact point: it doesn't get hot, but it's produces the most heat of all three devices by idling.

    For reference, the Pineapple Pager consistently sits around 57c while idling.

:material-router:**Performance**

!!! quote "[**Technical Details**](https://store.ui.com/us/en/products/utr)"

    Up to 6.5 Mbps to 866.7 Mbps (MCS0 - MCS9 NSS1/2, VHT 20/40/80) on an 802.11ac WiFi 5 network.

In my testing, the Travel Router was on a different floor, connecting to a WiFi 7 capable AP, over 5 GHz. It's less than 100' or 30m away if you were to draw a direct line between the two devices in space.

The results show it did generally get within range of what connecting directly to WiFi would provide. The low upload over Teleport may be due to the network configuration there, but this table will be updated as it's tested across more environments.

| Teleport | Download Avg | Upload Avg | Latency Avg | Band | Connection |
| --- | --- | --- | --- | --- | --- |
| :octicons-x-circle-fill-12: | ~55 mbps | ~43.5 mbps | 12ms | 2.4 GHz | :octicons-arrow-right-16: UTR :octicons-arrow-right-16: WiFi |
| :material-check-bold: | ~55 mbps | ~10 mbps | 9ms | 2.4 GHz | :octicons-arrow-right-16: UTR :octicons-arrow-right-16: WiFi :octicons-arrow-right-16: Control Plane :octicons-arrow-right-16: UniFi Gateway |
| :octicons-x-circle-fill-12: | ~62 mbps | ~95 mbps | 15ms | 5 GHz | :octicons-arrow-right-16: WiFi |

:material-pencil-box: *Averages are rounded for simplicity.*

One thing that stands out as missing, is you can't set the Travel Router's WiFi band to either 2.4 or 5 GHz specifically, it seems to default to 2.4 for some reason. I connected to an SSID that only advertised on the 5 and 6 GHz bands, and the router still set its own AP to 2.4 GHz.

It's also worth mentioning, without an internal battery, it always needs connected to a power source. If you use your phone as a power source, it will consume the battery very quickly, usually within 4-6 hours on most modern phone models as of early 2026.

:material-wifi-lock-open: **Security**

!!! note "Summary Expanded"

    This section goes into more detail on the points covered in the summary above.

    The [FAQ on the store page](https://store.ui.com/us/en/products/utr) also speaks to some of the physical security questions. Once you've adopted and configured the device with a reasonably strong credential, it makes taking over the device difficult.

    For physical security of the device, assume WiFi will always be open to attack. Therefore, securing the physical ports, especially if this device tunnels back to your main network, is critical should it be stolen.

    | Connection Type | Secure Setting |
    | --- | --- |
    | Ethernet | Disabled when away |
    | WiFi | Enabled, WPA3 |
    | USB | Disabled when away |

    You can revoke Teleport access from your Network manager, and realistically someone with physical access is likely stealing the device and factory resetting it to sell or use.

DNS being one of the critical pieces to securing your internet usage, it's disappointing to see the Travel Router has no dedicated options to enforce DNS over TLS or HTTPs. What you can do is enforce a specific DNS server via the DHCP options, such as 1.1.1.1 and 9.9.9.9, but this is all clear text. Below is a query made from a client directly to the Travel Router's WiFi DNS listener, and that request captured on the upstream gateway and network it was forwarded across.

!!! tip ""

    `192.168.10.49` is the IP the UTR receives when connecting to another WiFi network in the examples below.

=== "DNS Querry"

    ```bash
    $ dig @10.10.10.1 kali.org +short
    104.18.5.159
    104.18.4.159
    ```

=== "DNS Capture"

    ```bash
    $ sudo tcpdump -n -vv -i vlan0.10 -Z nobody 'host 192.168.10.49 and (port 53 or port 853)' | grep -iE "kali.org"
    dropped privs to nobody
    tcpdump: listening on vlan0.10, link-type EN10MB (Ethernet), snapshot length 262144 bytes
        192.168.10.49.36545 > 9.9.9.9.53: [udp sum ok] 12345+ [abc] A? kali.org. ar: . OPT UDPsize=1234 [COOKIE <redacted>] (56)
        192.168.10.49.36545 > 1.1.1.1.53: [udp sum ok] 12345+ [abc] A? kali.org. ar: . OPT UDPsize=1234 [COOKIE <redacted>] (56)
        1.1.1.1.53 > 192.168.10.49.36545: [udp sum ok] 12345 q: A? kali.org. 1/2/3 kali.org. A 104.18.4.159, kali.org. A 104.18.5.159 ar: . OPT UDPsize=1234 (56)
        9.9.9.9.53 > 192.168.10.49.36545: [udp sum ok] 12345 q: A? kali.org. 1/2/3 kali.org. A 104.18.4.159, kali.org. A 104.18.5.159 ar: . OPT UDPsize=123 (45)
    ```

!!! note "SSH-?"

    As of March 2026, SSH seems to be unavaialble in the latest version of the UniFi mobile app and Travel Router firmware after you finish updating it. During initial setup, and right after any firmware updates, it's optional to configure an SSH user/password, however some users are reporting rolling back app versions and factory resetting restores SSH access. Either way, seems it's not officially supported for now as a consistent option, which is unfortunate.

What else is listening on the device? The network interfaces themselves are reasonably secure, only exposing port 53 on both the LAN and "WAN" side.

=== "LAN-side"

    ```bash
    nmap -n -v -Pn -sT -p- --reason --packet-trace -e wlan0 --open -T4 10.10.10.1
    # SNIP
    CONN (9.2310s) TCP localhost > 10.10.10.1:31081 => Connection refused
    CONN (9.2310s) TCP localhost > 10.10.10.1:48539 => Connection refused
    CONN (9.2310s) TCP localhost > 10.10.10.1:27851 => Connection refused
    CONN (9.2322s) TCP localhost > 10.10.10.1:15352 => Connection refused
    CONN (9.2322s) TCP localhost > 10.10.10.1:39363 => Connection refused
    CONN (9.2322s) TCP localhost > 10.10.10.1:33604 => Connection refused
    Completed Connect Scan at 01:23, 9.22s elapsed (65535 total ports)
    Nmap scan report for 10.10.10.1
    Host is up, received user-set (0.046s latency).
    Not shown: 65534 closed tcp ports (conn-refused)
    PORT   STATE SERVICE REASON
    53/tcp open  domain  syn-ack

    Read data files from: /usr/bin/../share/nmap
    Nmap done: 1 IP address (1 host up) scanned in 9.24 seconds

    ```

=== "WAN-side"

    ```bash
    nmap -n -v -Pn -sT -p- --reason --packet-trace -e wlan0 --open -T4 192.168.10.49
    # SNIP
    CONN (59.6083s) TCP localhost > 192.168.10.49:20737 => Connection refused
    CONN (59.6087s) TCP localhost > 192.168.10.49:20147 => Operation now in progress
    CONN (59.6095s) TCP localhost > 192.168.10.49:5387 => Connection refused
    CONN (59.6097s) TCP localhost > 192.168.10.49:8122 => Connection refused
    CONN (59.6135s) TCP localhost > 192.168.10.49:20147 => Connection refused
    Completed Connect Scan at 01:23, 59.41s elapsed (65535 total ports)
    Nmap scan report for 192.168.10.49
    Host is up, received user-set (0.0035s latency).
    Not shown: 65528 closed tcp ports (conn-refused), 6 filtered tcp ports (no-response)
    Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
    PORT   STATE SERVICE REASON
    53/tcp open  domain  syn-ack

    Read data files from: /usr/local/share/nmap
    Nmap done: 1 IP address (1 host up) scanned in 59.83 seconds

    ```

It's odd that port 53 is even listening on the "WAN" side, but it does not respond to DNS queries.

```bash
$ dig @192.168.10.49 microsoft.com +short
;; communications error to 192.168.10.49#53: timed out
;; communications error to 192.168.10.49#53: timed out

$ nc -nv 192.168.10.49 53
Connection to 192.168.10.49 53 port [tcp/*] succeeded!

$ nc -nuv 192.168.10.49 53
Connection to 192.168.10.49 53 port [udp/*] succeeded!

```

!!! note "runZero Findings"

    I also reviewed [runZero](https://console.runzero.com/) logs after a scheduled scan across a network completed, where the UTR was connected as a client. Not only did it correctly identify the device as a UniFi Travel Router, ***it was able to talk to 10001/udp*** and:

    - Determine the exact firmware version
    - Pull all interface IP addresses and MAC addresses

    This is interesting because it hints at what subnet is behind the UTR, meaning the subnet that's providing WiFi to your devices connected to the UTR as clients. This will also need reviewed further.

That covers an initial review and assessment of the UniFi Travel Router. More notes will be added over time.


## Protect Stack

!!! tip "Alarm Profiles"

    It's not well detailed in the official UniFi docs, but you ***can*** effectively create "Armed Stay" and "Armed Away"-style profiles if you intend to replace a legacy security service in your office or home.

    You'll get *notifications* for configured "alarms", even those not tied to a profile. But if you have a UniFi Siren, you'll note it doesn't sound despite the condition of "motion" or "open" being met. Here's how this works.

    Under Alarm Manager:

    - Create Alarm Profiles, for example "Armed Stay" and "Armed Away"
    - Create a new alarm
    - Tie the trigger (event) to a scope (of devices)
    - ***Choose the profile this alarm / alert belongs to***
    - The alarm sounding (and more) must be set as ***actions***
    - Save

    Create and assign as many alarms to as many profiles as you want. Now you'll see at the top of the Protect application, there's an option to arm that location based on a profile. Now all of the ***actions*** (such as the Siren sounding) should work when you test them.


### NVRs

Unless you are a medium to large scale operation requiring numerous cameras and 24/7 video storage for compliance reasons, the entry-level UNVR-Instant is the best way to get started with the UniFi Protect stack.

- Each port is PoE to support the connected devices (cameras or superlink)
- One internal HDD is completely fine if you configure recordings to be based on events rather than 24/7
- Automatically gives you full Protect application access via the mobile app and UniFi's backplane for end-to-end encrypted remote access

AI processing can be done on-device depending on the camera, so you do not need anything other than the UNVR-Instant and any additonal cameras or sensors you might need for your business or home.

All NVR's past the UNVR-Instant jump sharply into medium / large scale territory with numerous storage bays and the need for separating all of the features and capabilities out amongst other devices in your network infrastructure, namely ports and PoE to the protect devices need to be handled by other gear. These NVR's are purely for storage, and will require additional investment to tie everything together.


### SuperLink

!!! tip "Update First"

	Ensure you login to the Web UI, or force a manual firmware update from the Protect mobile application. When these were first released, door and other sensors would not adopt until the SuperLink was on the latest firmware.

This can be PoE powered via a UniFi switch or the UNVR Instant, and is managed through the Protect application. All SuperLink connected devices use the same IP as the SuperLink, so if you're locking down clients in the NVR / IOT subnet by MAC address, you only need to scope the SuperLink's MAC into your DHCP server.

The SuperLink SubGHz protocol will go up the 2km with full line-of-sight, and through numerous walls, floors, and other paths within a normal building or floor.

More review of this protocol will need to be done.


### USL-Sensors

This includes door, window, motion, and other environmental sensors. Adopt them via the Protect application (mobile or web) and configure them from there. Naming them based on their location and function makes them easy to review in the web and mobile applications.


## Integrating & Deploying

TODO


## Managing


### Web Interface

The UniFi Network and UniFi Protect (camera / NVR) stacks have separate interfaces and mobile applications. All hardware and self-hosted Network servers can be accessed and managed locally via an IP on one of their interfaces, or more securely via the <https://unifi.ui.com/> backplane.


### Mobile Applications

The mobile applications provide visibility to all of your UniFi ecosystem natively, and the convenience is a huge selling point. The main difference with these apps of course is the level of detail you can get into. It's fairly deep, but you don't have absolutely every configuration feature available that you might in the web interface.


## Troubleshooting

TODO
