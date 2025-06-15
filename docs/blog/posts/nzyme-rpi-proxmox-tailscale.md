---
draft: false
date:
  created: 2024-12-01
  updated: 2024-12-01
categories:
  - nzyme
  - raspberry-pi
  - wifi
  - ids
  - home-lab
  - wireless
  - bluetooth
  - proxmox
  - tailscale
---

# nzyme x [Raspberry Pi, Proxmox, Tailscale]

This guide walks through deploying [nzyme](https://github.com/nzymedefense/nzyme) on a Raspberry Pi (4B or 5) leveraging autoconfiguration tools available on each system. For Raspberry Pi OS this includes writing additional files into the bootfs and rootfs partitions. For Ubuntu server images cloud-init or Ansible could be used.

The result is nzyme running as either a stand-alone system, or as a distributed node + tap where the nzyme-node lives on a server like Proxmox with the nzyme-tap sending data back to it from the external Raspberry Pi. Tailscale is also brought into focus for secure remote access.

This guide doesn't offer anything new in terms of nzyme usage that the official documentation doesn't already answer, but more of what you might run into and steps on how to achieve certain deployments if your goal is anything like what's described below.

<!-- more -->

!!! abstract "Prerequisites"

	This a short list of items you may want to use, or mix and match, to follow along. Required items are in green, while optional items are in blue.

	**Software**:

	- 🟢 SSH keys
	- 🟢 Tailscale account
	- 🔵 Hypervisor (Proxmox, VMware, Hyper-V, QEMU)

	**Hardware**:

	- 🟢 Raspberry Pi 4B or 5 (one or more)
	- 🟢 External wireless card (Alfa adapters)
	- 🟢 USB 32GB or more, or an external SSD for the Raspberry Pi


## References

External links to referenced software and services you may need as you follow along.

- [github.com/nzymedefense](https://github.com/nzymedefense/nzyme)
- [nzyme Documentation](https://docs.nzyme.org/)
- [nzyme Wireless Adapter Compatability](https://docs.nzyme.org/getting-started/requirements/#wifi-adapters)
- [GitHub Discussion Forum](https://github.com/nzymedefense/nzyme/discussions)
- [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/)
- [cloud-init Documentation](https://cloudinit.readthedocs.io/en/latest/index.html)
- [Tailscale](https://tailscale.com/)
- [Tailscale installer.sh](https://github.com/tailscale/tailscale/blob/main/scripts/installer.sh)
- [Proxmox-VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview)
- [WiFi Challenge Lab](https://github.com/r4ulcl/WiFiChallengeLab-docker)


## Tailscale ACLs

If you're planning to use Tailscale to securely access the nzyme dashboard from anywhere (as well as allow the tap(s) to securely communicate to the node(s), you'll need to define the following ACL's in your Tailnet. These allow you as the nzyme-admin to access all nzyme endpoints over SSH, as well as the WebUI on all nzyme-nodes. It also allows nzyme-tap endpoints to connect back to nzyme-nodes to send data.

```mermaid
graph LR
	A{💻 nzyme-admin};
	B(🛜 nzyme-tap);
	C(📦 nzyme-node:22,22900);
	A -->| SSH | B;
	A -->| SSH, HTTPS | C;
	B -->| Wireless Data via HTTPS | C;

```

```json
	// Define the tags which can be applied to devices and by which users.
	"tagOwners": {
		// SNIP
		"tag:nzyme-node":     ["autogroup:admin"],
		"tag:nzyme-tap":      ["autogroup:admin"],
		"tag:nzyme-admin":    ["autogroup:admin"],
	},

	// SNIP
		// Allow nzyme endpoint WebUI and SSH access
		{
			"action": "accept",
			"src":    ["tag:nzyme-admin"],
			"dst":    ["tag:nzyme-node:22,22900", "tag:nzyme-tap:22"],
		},
		// Allow nzyme-taps to send data to nzyme-nodes
		{
			"action": "accept",
			"src":    ["tag:nzyme-tap"],
			"dst":    ["tag:nzyme-node:22900"],
		},
	//SNIP

```


## Tailscale Authkey

If you don't know what an authkey is, it allows you to authenticate a device to a Tailnet programmatically, without the browser involved. See the [official documentation](https://tailscale.com/kb/1085/auth-keys), or [Tailnet Access](../posts/wazuh-tailscale.md#tailnet-access) for steps to generate an authkey.

The resulting string looks like `tskey-auth-1234567890-ABCDEF1234567890ABCDEF1234567890`


## Write an OS Image

This is a brief overview of what you may need to write the OS images to an external device for your Raspberry Pi to boot.

- [Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager)
- [Rufus](https://github.com/pbatard/rufus)
- [Etcher](https://github.com/balena-io/etcher)

[**Ubuntu Server Images**](https://cdimage.ubuntu.com/releases/)

- Download the `.img.xz`, `SHA256SUMS`, and the `SHA256SUMS.gpg` files
- GPG Fingerprint: [8439 38DF 228D 22F7 B374  2BC0 D94A A3F0 EFE2 1092](https://keyserver.ubuntu.com/pks/lookup?search=8439+38DF+228D+22F7+B374++2BC0+D94A+A3F0+EFE2+1092&fingerprint=on&op=index)
- Verify the signature: `gpg --verify SHA256SUMS.gpg SHA256SUMS` (this validates the checksums)
- Verify the checksum: `sha256sum -c SHA256SUMS --ignore-missing` (these validate the image)

[**Raspberry Pi Images**](https://www.raspberrypi.com/software/operating-systems/)

- You'll want the "Raspberry Pi OS Lite" image
- Use the [Archive](https://downloads.raspberrypi.com/raspios_lite_arm64/images/) link to download the `.img.xz`, `.sha256`, and `.sig` files ([Example](https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2024-11-19/))
- GPG Fingerprint: [54C3 DD61 0D9D 1B4A F82A  3775 8738 CD6B 956F 460C](https://keyserver.ubuntu.com/pks/lookup?search=54C3DD610D9D1B4AF82A37758738CD6B956F460C&fingerprint=on&op=index)
- Verify the signature: `gpg --verify 2024-11-19-raspios-bookworm-arm64-lite.img.xz.sig 2024-11-19-raspios-bookworm-arm64-lite.img.xz`
- Unlike the Ubuntu images, you use the `.sig` file to validate the OS image file, not the checksum file
- Verify the checksum: `sha256sum -c 2024-11-19-raspios-bookworm-arm64-lite.img.xz.sha256`

!!! tip "Obtaining signing keys"

	You can generally obtain GPG keys from [https://keyserver.ubuntu.com/](https://keyserver.ubuntu.com/)

	If you have the fingerprint you can also try: `gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys '<fingerprint>'`

	The Raspbery Pi OS key is also indexed here: [https://www.raspberrypi.org/raspberrypi_downloads.gpg.key](https://www.raspberrypi.org/raspberrypi_downloads.gpg.key)

It's recommended to use either a small form-factor USB 3.2 drive (for example the SanDisk ultra fit thumb drives) with 32GB of space or more. This is the most convenient option. You can go further and have the OS installed on an SD card or USB stick, *and then add an external SSD*. This won't be covered here (yet).

This guide will focus primarily on the setup of each component and infrastructure involved. You can achieve this using the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) (for macOS, Windows, or Linux), something like [Rufus](https://github.com/pbatard/rufus) or [Etcher](https://github.com/balena-io/etcher), the `usb-creator-gtk` package on Ubuntu, or even `dd`.

However you write the  image to the external storage device, you'll want to browse or `cd` into the one of the following partitions after it's created on that device.

- `system-boot` (Ubuntu Server)
- `rootfs/home/pi/` and `bootfs` (Raspberry Pi OS)

!!! tip "xz utils"

	Etcher may do this automatically for you, however if you use `usb-creator-gtk` on Ubuntu, you'll need to extract the raw `.img` file first. You can decompress `.xz` files, while preserving the original xz archive, with progress printed to the terminal, using the following command.

	```bash
	xz -dkv ./2024-11-19-raspios-bookworm-arm64-lite.img.xz
	```


## Raspberry Pi OS

After writing the image file to external media, you'll see two partitions:

- `bootfs/`
- `rootfs/`

Raspbery Pi OS has no Debian style preseed.cfg or cloud-init system built in by default. Although there are projects out there and ways to do this, we'll take the most straight forward *and* universal approach, giving us the initial access we need to do really anything we'd want for any use case.

It's worth noting the [`rpi-imager`](https://github.com/raspberrypi/rpi-imager) utility can configure all of the following. Lets assume you want to stick with a built in tool like `dd`, or another utility for some reason. No matter how you write the RaspiOS image to disk, these steps will work (as of November 2024 / Bookworm).

!!! note "Auto-connect to WiFi?"

	[/bootfs/wpa_supplicant.conf](https://www.raspberrypi.com/documentation/computers/configuration.html#connect-to-a-wireless-network-2) no longer works as of Debian Bookworm (see the link)

	NetworkManager likely has a way to be preconfigured to do this. This post will be updated after a solution is found or not.

### Pre-boot Setup

There are a few files we can create within the `rootfs` and `bootfs` partitions.

[/bootfs/ssh.txt](https://www.raspberrypi.com/documentation/computers/configuration.html#ssh-or-ssh-txt) is an empty file that simply enables SSH on boot by being present.

```bash
touch bootfs/ssh.txt
```

[/bootfs/userconf.txt](https://www.raspberrypi.com/documentation/computers/configuration.html#remote-access) defines the initial user.

```bash
# Generate a password hash
openssl passwd -6

# Write this to the userconf.txt file
echo `pi:<password-hash>` | tee `bootfs/userconf.txt`
```

!!! tip "$HOME Path"

	`/rootfs/home/pi` is the path of the default "pi" user.

	Here you can drop scripts or files you'd like to have available, **they become owned by `pi:pi`**

Write the [Tailscale installer.sh](https://github.com/tailscale/tailscale/blob/main/scripts/installer.sh) file here.

```bash
cd ./rootfs/home/pi
curl -Lf https://github.com/tailscale/tailscale/blob/main/scripts/installer.sh > tailscale-installer.sh
```

Also write a key.txt file containing your Tailscale authkey, so you don't have to type it.

```bash
# Paste your tailscale authkey here for script use later
nano ./rootfs/home/pi/key.txt
```

Create `/home/pi/.ssh/authorized_keys` with your public key.

```bash
mkdir -p ./rootfs/home/pi/.ssh
nano ./rootfs/home/pi/.ssh/authorized_keys
```

Now move the USB over to the Raspberry Pi device and power it on.


### After First Boot

!!! note "Manual Setup"

	You will need a keyboard and monitor to interact with the device for the remaining steps. This may change in the future.

!!! tip "Keyboard Locale"

	You may run into issues with the keyboard locale, for instance when entering the WiFi password. To resolve this you'll need to ensure your keyboard is mapped to the expected country code, just like your WLAN setting.

	```bash
	sudo raspi-config
	# Choose "5 Localisation Options"
	# Choose "L3 Keyboard"
	# Choose "Keyboard, Generic" or simply hit Enter
	# Then eventually under "Keyboard layout" change to your locale using "Other" if it's not listed
	# TAB to OK, then OK, and allow all the defaults from here on
	```

The file generated at `/etc/NetworkManage/system-connections/<ESSID>.nmconnection` will contain your WiFi connection information once you setup networking. If you're planning to network the Pi via an ethernet cable, you can skip this part.

- [Enable Wireless Networking](https://www.raspberrypi.com/documentation/computers/configuration.html#enable-wireless-networking)
- [Connect to a Wireless Network](https://www.raspberrypi.com/documentation/computers/configuration.html#connect-to-a-network)

```bash
# Enable wireless networking by going to Localisation optons and choosing a WLAN country code
sudo raspi-config

# Check if your radio is enabled
nmcli radio wifi

# Enable your radio
nmcli radio wifi on

# Scan the local area for available wifi networks
nmcli dev wifi list

# Connect to a wireless network
sudo nmcli --ask dev wifi connect <example_ssid>
```

This is what the `<ESSID>.nmconnection` file looks like for reference, using WPA2:

```conf
[connection]
id=<essid-here>
uuid=<unique-uuid-here>
type=wifi
interface-name=wlan0

[wifi]
mode=infrastructure
ssid=<essid-here>

[wifi-security]
auth-alg=open
key-mgmt=wpa-psk
psk=<wifi-password-here>

[ipv4]
method=auto

[ipv6]
addr-gen-mode=default
method=auto

[proxy]
```

You can generate a UUID with:

```bash
uuidgen
# or
cat /proc/sys/kernel/random/uuid
```

!!! note "What is wlan0?"

	In this case `wlan0` will always be the built in wireless card of the Raspberry Pi. Since we're using an external wireless card as the nzyme-tap monitoring interface, we end up using the onboard card to provided us a way to connect to a network, and this provides us access from other devices either locally on that subnet, or remotely via Tailscale.

	Connecting over WiFi was used as an example 1) because it's trickier to visualize and execute without doing it before instead of simply connecting via an ethernet cable, and 2) to keep in the spirit of all-things-wireless, it wouldn't make sense to leave this out of the guide.

!!! note "Additional Steps"

	The remaining steps are mostly preference, but were part of the initial setup when drafting this guide.

On first login, you may have issues with the timesync being off.

```bash
sudo systemctl restart systemd-timesyncd
```

Once that's done, update the system.

```bash
sudo apt update; sudo DEBIAN_FRONTEND=noninteractive apt full-upgrade -y
sudo systemctl reboot
```

Finally, run the Tailscale installer and authenticate to your Tailnet.

```bash
cd /home/pi/
sudo bash ./installer.sh
sudo tailscale up --authkey $(cat key.txt)
sudo tailscale set --accept-dns=false
```

It's also not a bad idea to configure a firewall.

```bash
sudo apt install -y ufw
sudo ufw enable
sudo ufw allow ssh
# This can be configured further remotely over Tailscale
```

## Ubuntu

After writing the Ubuntu image to  your USB or SD card, you'll see two partitions were created; a `system-boot` and `writable` partition. Browse into the `system-boot` directory. Here you'll find two files we can use to automate the setup process; `user-data`, and `network-config`.


### cloud-config

**user-data**

You can either replace the `user-data` file completely, using  the contents below, or comment out the few existing lines and append the YAML contents (below). Either way, this file will contain all of our required changes. It will:

- Initialize our user, with our SSH public key for secure remote access
- Install Tailscale and authenticate to our Tailnet, by embedding a Tailscale authkey into the user-data file
- This is kept as generic as possible to be reusable for more than just an nzyme-tap or node

!!! tip "Make the config your own"

	Replace all values encapsulated in `{{ Jinja2 syntax }}` below with your own.

	See [https://cloudinit.readthedocs.io/en/latest/reference/examples.html](https://cloudinit.readthedocs.io/en/latest/reference/examples.html) for a complete reference of `user-data` examples.

```yml
# On first boot, set the (default) ubuntu user's password to "ubuntu" and
# expire user passwords
chpasswd:
	expire: true
	users:
	- name: {{ username }}
		password: ubuntu
		type: text


# Set system hostname
hostname: nzyme-tap


# Require public key auth
ssh_pwauth: false


# User definitions
users:
	- default
	- name: {{ username }}
		primary_group: {{ username }}
		groups:
			- users
			- sudo
		shell: /bin/bash
		# Allows sudo use, requiring a password
		sudo:
			- ALL=(ALL) ALL
		# Import an SSH public key from GH or LP
		ssh_import_id:
			- lp:{{ your-launchpab-username }}
			- gh:{{ your-github-username }}
		# Manually add a public SSH key if it's not available on GH or LP
		ssh_authorized_keys:
			- {{ your-public-key }}
		lock_passwd: false
		# Optional, create with mkpasswd --method=SHA-512 --rounds=4096
		passwd: {{ create-passwd-with-mkpasswd }}


## Update apt database and upgrade packages on first boot
package_update: true
package_upgrade: true


## Install additional packages on first boot, optional
#packages:
#- curl
#- pipx


## Write arbitrary files to system
write_files:
- path: /usr/local/bin/tailscale-installer.sh
	owner: root:root
	permissions: '0755'
	content: |
		{{ replace with raw, plaintext
		data of the installer.sh for
		tailscale }}


## Commands to run on first boot
runcmd:
	# Note: Don't write files to /tmp from cloud-init use /run/somedir instead.
	# Early boot environments can race systemd-tmpfiles-clean LP: #1707222.

	# Optional, if you didn't write the installer file as a base64 encoded file
	# to disk earlier in user-data
	#- wget "https://github.com/tailscale/tailscale/raw/refs/heads/main/scripts/installer.sh" -O /usr/local/bin/tailscale-installer.sh
	#- sha256sum /usr/local/bin/tailscale-installer.sh | grep -i "{{ sha256sum-of-tailscale-installer }}"


	# Install Tailscale
	- /usr/local/bin/tailscale-installer.sh


	# Authenticate to Tailnet, replace with a real authkey you generate just for this device
	- tailscale up --authkey {{ tailscale_authkey }}
	- tailscale set --accept-dns=false
```

**network-config**

Now open `network-config` and replace (or duplicate and modify) the baked-in example with your real connection information.

!!! note "What is wlan0?"

	In this case `wlan0` will always be the built in wireless card of the Raspberry Pi. Since we're using an external wireless card as the nzyme-tap monitoring interface, we end up using the onboard card to provided us a way to connect to a network, and this provides us access from other devices either locally on that subnet, or remotely via Tailscale.

	Connecting over WiFi was used as an example 1) because it's trickier to visualize and execute without doing it before instead of simply connecting via an ethernet cable, and 2) to keep in the spirit of all-things-wireless, it wouldn't make sense to leave this out of the guide.

Below is a slightly modified and commented version of that example block for you to use.

```yml
	wifis:
		wlan0:
			dhcp4: true
			optional: true
			access-points:
				# WPA2/3 Example
				myhomewifi:
					password: "S3kr1t"
				# EAP/TLS Example
				workssid:
					auth:
						key-management: eap
						method: peap
						identity: "me@example.com"
						password: "passw0rd"
						ca-certificate: /etc/my_ca.pem

			# Change to your local country code
			regulatory-domain: GB
```

!!! tip "Convert tabs to spaces"

	In VSCode it's easy, and going to help you after manually writing the YAML config files for cloud-init, to ensure there are no invisible tabs or spaces that will break your entire config.

	Simply follow the [advice in this post](https://stackoverflow.com/questions/33974681/how-can-i-convert-tabs-to-spaces-and-vice-versa-in-an-existing-file) to open the command list and type "Convert Indentation to Spaces", finally clicking that command to execute it.

Finally, you can [validate your config](https://cloudinit.readthedocs.io/en/latest/howto/debug_user_data.html) with:

```bash
cloud-init schema --config-file=/media/user/system-boot/user-data --annotate
# You want "Valid schema /media/user/system-boot/user-data" at the bottom of the output
```

Move the USB or SD card over to the Raspberry Pi. It's useful to have a video connection to a monitor available should anything go wrong. Otherwise you'll see your Pi connect to your Tailnet via the Tailscale web console.

At this point ensure you can SSH in, and update all packages, perform a reboot, then confirm you can still SSH in.


## Wireless Drivers

If you're using an external card mentioned [here](https://docs.nzyme.org/getting-started/requirements/#wifi-adapters) in the nzyme documentation, you should already be able to see it with the following commands.

```bash
lsusb
ip link
```

You should see `wlan0` connected to your WiFi network of choice (or not if you're using an ethernet cable), and an `wlx00...` device. This is your wireless usb adapter. If you don't see it listed in the output of the above two commands, you'll need to check which drivers are available in the Ubuntu package repos for your card.

If you *can* see your external wireless adapter, you can proceed.


## nzyme-node

If your plan is to provision a node, with or without the tap, follow the steps here:

- [Ubuntu 22.04](https://docs.nzyme.org/getting-started/installation/ubuntu-server-2204/#installing-a-nzyme-node)
- [Ubuntu 24.04](https://docs.nzyme.org/getting-started/installation/ubuntu-server-2404/#installing-a-nzyme-node)

Generally you're downloading and installing the deb file from [GitHub's latest release](https://go.nzyme.org/downloads), modifying the settings under `/etc/nzyme/nzyme.conf`, and starting the service.

The only point worth mentioning here is, when it comes to Tailscale, set both:

- `rest_listen_uri`
- `http_external_uri`

... to bind to your device's tailscale0 IP address if you want to lock access to the Tailnet. Otherwise bind to all interfaces with `0.0.0.0` and ensure the proper firewall rules are in place.


### proxmox

!!! warning "Under Construction"

	This section will be updated in the future after further testing.

	Generally you want:

	- 4 vCPU cores
	- 8 GB RAM
	- 32GB Storage


## nzyme-tap

!!! tip "Physical Hardware"

	Unlike an nzyme-node without a tap, any tap provisioned must have some type of hardware involved. Even if you're passing through an external wireless adapter to a VM, you will need the adapter listening in the real world to send data to the node.

	This is why the Raspberry Pi is ideal to act as a tap, whether you have another Pi as a node, or a dedicated VM as a node.

	If you're interesting in fully virtualizing and testing nzyme, look at [WiFi Challenge Lab](https://github.com/r4ulcl/WiFiChallengeLab-docker) which gives you a complete wireless environment to explore within a VM.

Once the tap device is up and running, [download and install the nzyme-tap deb package](https://go.nzyme.org/downloads) following these instructions:

- [Ubuntu 22.04](https://docs.nzyme.org/getting-started/installation/ubuntu-server-2204/#installing-a-nzyme-tap)
- [Ubuntu 24.04](https://docs.nzyme.org/getting-started/installation/ubuntu-server-2404/#installing-a-nzyme-tap)

Next configure the lines referenced in `/etc/nzyme/nzyme-tap.conf`. As mentioned above in the nzyme-node seciton, point this to the Tailnet address of your nzyme-node if using Tailscale.

Finally you may see that the service fails on the first start of `sudo systemctl start nzyme-tap`. What you'll want to do is modify the `wifi_interfaces` section of the /etc/nzyme/nzyme-tap.conf file to use what channels your external wireless adapter can see.

In other words, this assumes the built-in wireless card (`wlan0`) is being used to connect to a WiFi AP for internet access. Of course if you're connected over ethernet, you could use both cards to monitor wrieless activity.

[To generate a list of channels and bands your device can listen on, do](https://docs.nzyme.org/wifi/concepts/channel_hopping/#what-channels-do-my-wifi-adapters-support):

```bash
nzyme-tap --generate-channels
```

This will give you a configuration block that can be swapped in for the existing blocks under nzyme-tap.conf. You should add `channel_width_hopping_mode = "full"` [which is a recommended default](https://docs.nzyme.org/wifi/concepts/channel_hopping/#channel-width-hopping-mode), to have something that looks like this in your conf file.

```conf
[wifi_interfaces.wlan1]
active = true
channel_width_hopping_mode = "full"
channels_2g = [<list-of-channels>]
channels_5g = [<list-of-channels>]
channels_6g = [<list-of-channels>]
```

!!! tip "Debugging with journalctl"

	In my case I ran into an instance where even the generated channels list had a channel that didn't work when run as part of the nzyme-tap.conf file. Checking `sudo journalctl -xe` to see exactly what was wrong pointed to this channel not being understood by nzyme. Removing that channel from the config for that interface resulted in a successful start from nzyme-tap.

After making these adjustments, `sudo systemctl restart nzyme-tap` should result in a successful run.


## Bluetooth Monitoring

Follow the documentation [here](https://docs.nzyme.org/bluetooth/). You're obtaining your bluetooth interface information with `hciconfig` similar to `ifconfig`. Ensure any bluetooth interfaces you want monitoring have their own block in `/etc/nzyme/nzyme-tap.conf`, same as each wireless interface.

```conf
# Example
<SNIP>
[bluetooth_interfaces.hci0]
active = true
bt_classic_enabled = true
bt_le_enabled = true
discovery_period_seconds = 30
dbus_method_call_timeout_seconds = 2
<SNIP>
```

!!! tip "Bluetooth still isn't working?"

	You'll need to enable the bluetooth subsystem in three places for it to work: on the node itself at the system level, at the organization level, and at the tenant level.

	If you search the nzyme Discord for help with enabling bluetooth monitoring, for example you've set the appropriate changes in `/etc/nzyme/nzyme-tap.conf` (above) and still see something like the following in the journalctl or systemctl status logs:

	```bash
	[ERROR][2024-12-06 01:02:03][bluetooth::tables::bluetooth_table] Could not submit Bluetooth devices report: ...
	```

	...you'll find this post from the nzyme founder:

	> You have to enable it for the organization first and then it should  be possible to enable it for the tenant as well

To enable bluetooth for the Node at the system level:

- System > Subsystems
- Here you'll see a subsystem settings pane where you can enable bluetooth

To enable bluetooth at the Organization level:

- System > Authentication > Your-Organization-Name (Often "Default Organization")
- Choose "Edit Organization" on the top right
- Here you'll see a subsystem settings pane where you can enable bluetooth

To enable bluetooth at the Tenant level:

- System > Authentication > Your-Organization-Name (Often "Default Organization")
- Under "Tenants" click on your Tenant (Often "Default Tenant")
- Choose "Edit Tenant" on the top right
- Here you'll see a subsystem settings pane where you can enable bluetooth


## Going Forward

!!! warning "Under Construction"

	Sections to include in a future draft of this guide:

	- Configure monitoring
	- Additional automated provisioning of Raspberry Pi OS (Bookworm and later)
	- Compiling arm64 tap packages for Ubuntu Server on Raspberry Pi
	- Sending alerts to Discord and Slack from nzyme-nodes
	- How to set log retention
	- Possible SIEM integration (targeting Wazuh)