---
title: "UniFi"
icon: simple/ubiquiti
draft: true
#date:
#  created: 2026-01-04
#  updated: 2026-02-01
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


## Protect Stack

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