---
title: "Meshtastic"
icon: material/radio-tower
draft: false
#date:
#  created: 2026-01-29
#  updated: 2026-02-01
categories:
  - how-to
  - meshtastic
  - wireless
  - rf
  - node
  - mobile
---

# :material-radio-tower: Meshtastic

!!! abstract "Meshtastic"

    Meshtastic is a project that lets you use inexpensive LoRa radios as a long range off-grid communicator for areas without reliable cellular service. These radios are great for hiking, skiing, paragliding - essentially any hobby where you don't have reliable internet access. Each member of the mesh can send and view text messages and enable optional GPS based location features. The radios automatically create a mesh to forward packets as needed, so everyone in the group can receive messages from even the furthest member. The radios will optionally work with your phone, but no phone is required.

- <https://meshtastic.org/>
- <https://github.com/meshtastic>
- [FAQs](https://meshtastic.org/docs/faq/)

These devices are becoming more and more popular, [firmware was even custom built for DEFCON](https://github.com/meshtastic/defcontastic). There's an excellent official app for mobile devices as well as a web client to interact with the mesh (send and receive messages).

<!-- more -->

## Device Support & Details

For devices running Meshtastic (devices are called "nodes"):

- [Supported Hardware Overview](https://meshtastic.org/docs/hardware/devices/)
- [Meshtastic on Raspberry Pi](https://meshtastic.org/docs/hardware/devices/raspberrypi/)
- [Meshtastic on OpenWrt Routers](https://meshtastic.org/docs/hardware/devices/openwrt/)

There are three main ways to interact with nodes, all effectively offering the same capabilities:

- [Web Client](https://meshtastic.org/docs/software/web-client/)
- [iOS Client](https://meshtastic.org/docs/category/apple-apps/)
- [Android Client](https://meshtastic.org/docs/category/android-app/)
- [Meshtastic CLI](https://meshtastic.org/docs/software/python/cli/)

Install the command line tool with:

```bash
# Using a venv
python3 -m pip install --upgrade "meshtastic[cli]"

# Using pipx
pipx install "meshtastic[cli]"

# Add yourself to the dialout group, likely required
sudo usermod -aG dialout $USER
```

From here you're ready to setup any device.

!!! info "USB Class Codes"

    Common USB class codes include:

    - `{ 02:??:?? 0a:??:?? }` for [CDC communication](https://www.usb.org/defined-class-codes#anchor_BaseClass02h) + [CDC data](https://www.usb.org/defined-class-codes#anchor_BaseClass0Ah)
    - `{ 08:??:?? }` for [USB mass storage](https://www.usb.org/defined-class-codes#anchor_BaseClass08h) when in DFU mode.


## Install & Upgrade

There are really two install / upgrade types: [**ESP32**](https://meshtastic.org/docs/getting-started/flashing-firmware/esp32/) and [**nRF52 & RP2040**](https://meshtastic.org/docs/getting-started/flashing-firmware/nrf52/)

!!! tip "Factory Reset Before Use"

    It's a good idea to [factory erase](https://meshtastic.org/docs/getting-started/flashing-firmware/nrf52/nrf52-erase/) and [flash](https://meshtastic.org/docs/getting-started/flashing-firmware/) devices before first use


### ESP32

You can flash ESP32 firmware and review all supported devices via the web tool: <https://flasher.meshtastic.org/>


### nRF52 & RP2040

!!! quote "Drag & Drop"

    nRF52 and RP2040 based devices have the easiest firmware upgrade process ([drag & drop](https://meshtastic.org/docs/getting-started/flashing-firmware/nrf52/drag-n-drop/)). No driver or software install is required on any platform. It also appears to support OTA (over the air) updates through bluetooth

What this means is once you put the device into DFU (device firmware update) USB mode, a mountable storage space appears on the device. Simply drop the firmware binary into the root of that mount. Once the file is copied over, the device will automatically kick off the update.


## Initial Setup

- Read the [configuration tips](https://meshtastic.org/docs/configuration/tips/) first
- You can enumerate a device's info with [`meshtastic --port /dev/ttyACM0 --info`](https://meshtastic.org/docs/software/python/cli/#--port-port)
- Simiarly [`meshtastic --get all` will list all preferences](https://meshtastic.org/docs/software/python/cli/usage/#getting-a-list-of-user-preferences)
- You can watch a stream of device activity over serial with [`meshtastic --noproto`](https://meshtastic.org/docs/software/python/cli/#--noproto)
- A device can only be connected to one client application at a time (you cannot stream serial activity while also using the mobile app)


### First Boot

- [Factory erase](https://meshtastic.org/docs/getting-started/flashing-firmware/nrf52/nrf52-erase/) and [flash](https://meshtastic.org/docs/getting-started/flashing-firmware/) devices before first use
- Devices appear to have [`region: UNSET`](https://meshtastic.org/docs/configuration/radio/lora/#region) by default until configured, so they aren't sending data until you complete setup
- [Change the default BLE static pairing PIN on any new / flashed device: `meshtastic --set bluetooth.fixed_pin <6_digit_pin>`](https://meshtastic.org/docs/configuration/radio/bluetooth/#fixed-pin)
    - The default PIN is often always `123456`
    - This is ideal for devices without a small display to print rolling codes on demand

### Device

Configure your [device role](https://meshtastic.org/docs/configuration/radio/device/#roles) and [rebroadcast mode](https://meshtastic.org/docs/configuration/radio/device/#rebroadcast-mode) based on usage and opsec requirements.


- [Client roles + comparison](https://meshtastic.org/docs/configuration/radio/device/), and what the [rebroadcast mode](https://meshtastic.org/docs/configuration/radio/device/#rebroadcast-mode) means
- [Tips on rebroadcasting traffic](https://meshtastic.org/docs/configuration/tips/#rebroadcast-public-traffic)

!!! warning "The Default Channel"

    [You **must** be on the Default channel to both, participate in the mesh network, and to be able to even send messages to other channels via nodes discoverable in the Default channel.](https://meshtastic.org/docs/overview/encryption/#explanation)

    Connecting to any channel, or even sending a DM, reveals your node ID, username, device name, and other information in the headers of packets that traverse your device and the devices forwarding them. This will result in receiving numerous spam DM's which will flood your contact list. **There is no way around this if you want to benefit from the larger mesh network, it's part of using Meshtastic**.

    One alternative, that does not prevent sniffing, is setting up a number of nodes running a private primary channel. For example, you and other park rangers install numerous nodes around a park using a custom primary channel, and not the Default public (`AQ==` base64 equivalent of hex `0x01`) channel key. This is effectively creating your own private mesh network, while avoiding the spam DM's.

!!! warning "Rebroadcasting and Sniffing"

    Even without participating in a public or known channel, devices can be configured to simply rebroadcast packets on the LoRa band. [The default rebroadcast setting is `ALL`](https://meshtastic.org/docs/configuration/radio/device/#rebroadcast-mode). This appears to allow nodes to forward packets, even those they can't decrypt because they lack the key.

    This will need tested and confirmed, but even having your own private mesh does not prevent mesh activity from being observed (in other words, node headers).

!!! warning "MUTE vs HIDDEN"

    Practically, `CLIENT_MUTE` is the passive "sniffing" setting. However, by being in any channel, all users of that channel (this applies particularly to the public defualt channel) will be able to see and DM your device.

    Unintuitively `CLIENT_HIDDEN` will reveal your device details eventually by forwarding packets to participate in the network, though at a lower rate than the default `CLIENT` setting.


### LoRa

The [LoRa](https://meshtastic.org/docs/configuration/radio/lora/) section of the documentation covers this.

!!! note "Modem Preset"

    `LONG_FAST` is ideal for general use.

    > Default is unset which equates to `LONG_FAST`. Presets are pre-defined modem settings (Bandwidth, Spread Factor, and Coding Rate) which influence both message speed and range. The default will provide a strong mixture of speed and range, for most users.

    `SHORT_TURBO` is best for super-high density situations (like conferences).

    > `SHORT_TURBO` (Fastest, highest bandwidth, lowest airtime, shortest range. It is not legal to use in all regions due to it's 500kHz bandwidth.)


### Channels & DMs

!!! tip "Messaging"

    The [encryption](https://meshtastic.org/docs/overview/encryption/) overview provides the best walkthrough to how messaging works from a usability and security perspective.

- [PKI exists per-device (user identity) and per-channel (group identity)](https://meshtastic.org/docs/development/reference/encryption-technical/#psk-for-channels-pkc-for-direct-messages-and-admin-messages)
- You can wipe channel and direct messages via "Reset NodeDB" in the mobile application
- [Configuration tips on node roles, (not) location sharing, rebroadcasting traffic, channels, and best practices](https://meshtastic.org/docs/configuration/tips/)
    - [You can completely disable location sharing on public channels, and enable it for private channels](https://meshtastic.org/docs/configuration/tips/#sharing-location-on-a-private-secondary-channel)
- [MQTT](https://meshtastic.org/docs/software/integrations/mqtt/) is off by default, it's a bridge to the internet for meshtastic
    - [Connecting a node to the public MQTT server may publish the locations of all nodes in your mesh to the internet.](https://meshtastic.org/docs/configuration/tips/#best-practices)


## Vulnerabilities

- [DEFCON 33 Blog Post with Findings and Fixes](https://meshtastic.org/blog/that-one-time-at-defcon/)
- [GitHub: Meshtastic Firmware SECURITY.md](https://github.com/meshtastic/firmware/security)


## Privacy & Security

!!! note "Meshtastic vs Other Platforms"

    While Meshtastic states this is not as secure as something like Signal, in other ways it's something that **could** be life saving in the right situation where you have no traditional network or cell service, and you're able to coordinate with your group or get a message to another node during an emergency. There are obviously no guarantees, as **this is not designed to be an emergency service**, but it's a use case one can imagine while hiking or camping in remote areas. Examples in the [Wikipedia entry for Meshtastic](https://en.wikipedia.org/wiki/Meshtastic#Use_cases_and_applications) suggest it's being explored as a backup communications system during natural disasters.

!!! warning "Device Triangulation"

    The [FAQS page](https://meshtastic.org/docs/faq/#mesh) references a project to do just this: <https://github.com/a-f-G-U-C/Meshtastic-ZPS>. The accuracy of this tool has not been confirmed, but like any RF siganl, triangulation should be considered possible.

- [Comparison to WPA3, TLS1.3, and Signal](https://meshtastic.org/docs/overview/encryption/#is-it-as-secure-as-wi-fi-wpa3-https-tls13-or-signal)
    - If you have physical access to a node via a client application, you can dump all the private keys and PINs
    - Change your keys from time to time
    - [Rotating your device's private key **does not** rotate the user or device identity in the network](https://meshtastic.org/docs/configuration/radio/security/#private-key)
- Meshtastic does not implement network auth, [so it is trivial to impersonate anyone else if you have access to the channel key](https://meshtastic.org/docs/overview/encryption/#authentication)
- [DM's use PKC to encrypt and validate messages](https://meshtastic.org/docs/overview/encryption/#direct-messages), which makes them harder to impersonate or modify
- [Update your firmware regularly, follow any notices on GitHub and Discord](https://meshtastic.org/docs/faq/#firmware)
- You do not need to enable [admin mode](https://meshtastic.org/docs/configuration/radio/security/#admin-key), or configure any [remote administration](https://meshtastic.org/docs/configuration/remote-admin/) to secure a personal (non-router) node
- [Backup and restore your keys & config using any of the client applications](https://meshtastic.org/docs/configuration/radio/security/#security-keys---backup-and-restore)
