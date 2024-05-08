---
draft: false
date: 2024-05-07
categories:
  - openwrt
  - how-to
  - firewall
  - wifi
  - networking
  - home-lab
  - administration
---

# OpenWrt

A quick reference for general setup and usage.

<!-- more -->

## References

- Essentials
    * <https://openwrt.org/toh/ubiquiti/unifiac>
    * <https://openwrt.org/docs/guide-user/security/signatures>
    * <https://openwrt.org/docs/guide-user/installation/ar71xx.to.ath79>
    * <https://openwrt.org/docs/guide-user/installation/sysupgrade.cli>
    * <https://openwrt.org/docs/guide-user/troubleshooting/backup_restore>
    * <https://openwrt.org/docs/guide-user/troubleshooting/failsafe_and_factory_reset>

- UCI CLI Interface
    * <https://openwrt.org/docs/guide-user/base-system/uci>

- Wireless 
    * <https://openwrt.org/docs/guide-user/network/wifi/basic>
    * <https://openwrt.org/docs/guide-user/network/wifi/guestwifi/configuration_command_line_interface>
    * <https://openwrt.org/docs/guide-user/network/wifi/guestwifi/guest-wlan>
    * <https://openwrt.org/docs/guide-user/network/wifi/guestwifi/guestwifi_dumbap>

- Downloading
    * <https://downloads.openwrt.org/releases/>
    * `https://downloads.openwrt.org/releases/<version>/targets/<target>/<type>`

- Virtualization
    * <https://openwrt.org/docs/guide-user/virtualization/vmware>
    * <https://openwrt.org/docs/guide-user/virtualization/virtualbox-vm>

- Networking
    * <https://openwrt.org/docs/guide-quick-start/checks_and_troubleshooting>
    * <https://openwrt.org/docs/guide-user/base-system/basic-networking>
    * <https://openwrt.org/docs/guide-user/network/routing/routes_configuration>
    * <https://openwrt.org/docs/guide-user/base-system/dhcp>
    * <https://openwrt.org/docs/guide-user/base-system/dhcp_configuration>

- Changes regarding `ifname` -> `device`:
    * <https://forum.openwrt.org/t/mini-tutorial-for-dsa-network-config/96998>
    * <https://openwrt.org/docs/guide-user/network/dsa/converting-to-dsa>
    * <https://openwrt.org/docs/guide-user/network/dsa/dsa-mini-tutorial>

- Logging
    * <https://openwrt.org/docs/guide-user/perf_and_log/log.messages>

- UEFI / SecureBoot
    * <https://openwrt.org/docs/guide-developer/uefi-bootable-image>

## OpenWrt Wiki License:

- <https://creativecommons.org/licenses/by-sa/4.0/deed.en>


## Factory Reset

1. Power on the device
2. Hold reset button for 10 seconds
3. Release the reset button
4. Allow device to fully reboot

## Connecting

Connect to the device at OpenWrt's default: `ssh -p 22 root@192.168.1.1`

   1) Note that you'll likely need to reconfigure your machine's network interface to be an ip address on 192.168.1.0/24 using `ifconfig` or `ip`
   2) Use `sudo systemctl restart network-manager.service`
   3) On Windows, you will need to reboot unless a future update permits reconfiguring of a live network interface

## Troubleshoot Connecting

If you have issues connecting, try factory resetting the OpenWrt device again while it's still connected to your PC.

Also try restarting the PC you're managing the device from.

Alternatively walk through the steps below.

```bash
# Try this first
sudo systemctl restart network-manager.service
# Check to see if you were assigned an IP in the 192.168.1.x/24 range
ip a

# If not, continue below

# Run tcpdump in one window
sudo tcpdump -i eth0 -n -vv
# Optionally run with -Q in to ignore outbound packets, only looking at packets inbound to your PC
sudo tcpdump -i eth0 -n -vv -Q in

# Manage interface configuration in another window
ip address flush dev eth0 scope global
ip address add dev eth0 192.168.1.12/24 broadcast 192.168.1.255 scope global noprefixroute

# This line may fail, if it does skip it
ip route add default via 192.168.1.1 dev eth0

# This may reset your interface address from the ip command, if it does, repeat those steps then skip this step
sudo systemctl restart network-manager.service

# Once you have assigned an IP in the 192.168.1.x range to your interface, scan locally with nmap:
nmap -n -Pn -sT -p22 -e ethX 192.168.1.0/24 --open

```
From here continue to watch the output from `tcpdump`, see what the device is doing and adjust accordingly

When in doubt, factory reset the device, restart your PC, and then attempt a new connection.


## Installing OpenWrt on a UniFi AP:

The following link has extensive documentation on these devices and multiple installation methods:

- <https://openwrt.org/toh/ubiquiti/unifiac>

The following steps I've found to be the easiest and most reliable combination.

- Use the `<name>-sysupgrade.bin` image to do full upgrades of the device firmware
- UniFi default ssh creds: `ubnt::ubnt` `ubnt@192.168.1.20`
- OpenWrt default ssh creds: `root::root` `root@192.168.1.1`
- When using IPv6 ssh: `ssh ubnt@fe80::x:x:x%eth0`
- When using IPv6 scp: `scp /source/file/path ubnt@[fe80::x:x:x%eth0]:/dest/file/path`

**NOTE: Always use /tmp/ as the working directory for upgrades, backups, and restoring configurations**

---

**If UniFi device still has stock firmware higher than version 3.7.58**:
- [Download your device's UniFi firmware image for version 3.7.58](https://openwrt.org/toh/ubiquiti/unifiac#non-invasive_method_using_mtd)
- Copy it to the UniFi device: `scp <firmware-name>.bin ubnt@[fe80::x:x:x%eth0]:/tmp/`
- Connect to the UniFi device:`ssh ubnt@fe80::x:x:x%eth0`
- Flash the version 3.7.58 firmware:`fwupdate.real -m /tmp/ubnt.bin`

If you happen to disconnect during the process, the firmware write will still contiue on the UniFi device as long as it has power.

Proceed to the next step once the device is running firmware version 3.7.58

**If UniFi device has stock firmware version 3.7.58**:
- Copy the OpenWrt firmware to the UniFi-device `scp openwrt-ath79-generic-ubnt_unifiac-XXX-squashfs-sysupgrade.bin ubnt@192.168.1.20:/tmp/`
- Connect to the UniFi device:
- Write the OpenWrt firmware to the first kernel partition: `mtd write /tmp/openwrt-xxxxx-squashfs-sysupgrade.bin kernel0`
- Erase the second kernel partition: `mtd erase kernel1`
- `cat /proc/mtd | grep "bs"`, expected result is `mtd4`, if it's not, make note and use that value in the next command
- Use the obtained partition name for bs here if it was not `mtd4`: `dd if=/dev/zero bs=1 count=1 of=/dev/mtd4`
- Reboot

Now you can connect to using the default OpenWrt values: `ssh -L 8000:127.0.0.1:80 root@192.168.1.1`

## Upgrading to New Release Targets

**NOTE: Always use /tmp/ as the working directory for upgrades, backups, and restoring configurations**

### 1. Backup your configuration

Edit the backup configuration, note that the contents may be different from that of `sysupgrade -l` output.
```ash
vi /etc/sysupgrade.conf

/etc/config/dropbear
/etc/config/firewall
/etc/config/network
/etc/config/system
/etc/config/uhttpd
/etc/config/wireless
/etc/dropbear/
/etc/passwd
/etc/shadow
```

Verify the backup configuration
```ash
sysupgrade -l
```

Generate a backup
```ash
umask go=
sysupgrade -b /tmp/backup-${HOSTNAME}-$(date +%F).tar.gz

# Get the hash of the backup archive
sha256sum /tmp/backup-*

# Download this backup to your PC before power cycling / upgrading the OpenWrt device or it will be erased
scp root@<ip>:/tmp/backup-*.tar.gz .

# Check the hash locally to make sure the backup configuration is OK
sha256sum ./backup-* | grep <hash>
```

### 2. Perform the upgrade

A note on keeping configurations:

You can choose to keep your current configuration, from OpenWrt v21.x and later you may never have an issue with this.

However, if a configuration becomes corrupt during an upgrade, or the configuration syntax changes in the future, it's recommended to erase the configuration during the upgrade, and resore it afterwords.

**IMPORTANT**: This can create an impossible situation when performing upgrades remotely, without physical access to reprovision the device, it will not be reachable over the wire

Two possible solutions for entirely remote upgrades to OpenWrt AP's:

- Maintain the configuration
	* There's always a risk it will break, as with any remote upgrade
	* No additional work needed once the upgrade is complete
	* You'll encounter the least issues with same version upgrades (22.01 -> 22.02)
- Create an OpenWrt image set to a static IP in the range you require upon factory reset
	* This is because bridged AP's would never have the 192.168.1.1 address.
	* Requires creating a custom OpenWrt firmware image
	* Will work automatically and be reachable over the wire after factory resets

---

Performing the upgrade:
```ash
scp <sysupgrade.bin> root@<ip>:/tmp/      # must be /tmp/ as flash storage is unmounted during upgrades
ssh root@<ip>
sha256sum /tmp/*.bin                      # be sure the files's signature uploaded to the AP matches your local copy
sysupgrade -v /tmp/*.bin                  # preserve settings
sysupgrade -n -v /tmp/*.bin               # factory default settings
```

If you're conducting this upgrade remotely or over WiFi you'll likely see:

```
<datetime> upgrade: Commencing upgrade. Closing all shell sessions.
Command failed: Connection failed
root@OpenWrt:/tmp# Connection to <openwrt-ip> closed by remote host.
Connection to <openwrt-ip> closed.
```

If you're physically near the AP, you may see the device's indicator lights signalling an upgrade is happening.

If you cannot physically see the AP, watch the list of access points on the machine you'll be reconnecting from until it comes back online.

- 'TRX header' errors can be safely ignored
- If the device is unresponsive after an upgrade, wait 5 minutes before dis/re-connecting power to the device

In cases where you're either 1) connected directly to the AP or 2) have a shell on a device directly connected to the AP and the AP can obtain a DHCP lease in that subnet:

- Using `tcpdump` with `-Q in` you'll be able to see when the device is back online based on the traffic and MAC addresses (you may need to know the MAC address).

In all cases, once the device is back online you'll be able to reconnect over ssh to confirm the upgrade succeeded when the ssh banner prints the current version.

### 3. Restore your configuration

Automated restore process:
```ash
scp <backup-*.tar.gz> root@<ip>:/tmp/
ssh root@<ip>
sysupgrade -r /tmp/backup-*.tar.gz
```

---

You can manually restore a configuration over the CLI with `vi` and `copy/paste`.

In the future if either:
1) A configuration file becomes corrupted and is causing issues with the system, or 
2) There's been a change in the way OpenWrt reads configuration files as part of a difference in upgrade to a newer verison

**NOTE**: configuration files can be edited freely, until you run `/etc/init.d/<service> restart` your changes won't be loaded.

- Leave the default `lan` interface in `/etc/config/network` alone until the end
- `vi /etc/config/network`
	* Drop in your configurations (you can copy / paste over ssh)
	* Make any adjustments to what you've just added based on what you see in the defaults
- `vi /etc/config/wireless`
	* Optionally change SSID names
	* Optionally leave both radio's disabled
- `vi /etc/dropbpar/authorized_keys`
	* Add any ssh public keys
- `vi /etc/config/uhttpd`
	* Modifying the listening interface(s) of the uhttpd
- `/etc/init.d/network restart`

---

Ensure your settings were restored:
```ash
cat /etc/config/wireless
cat /etc/dropbear/authorized_keys
```
---

# UCI

```ash
uci [<options>] <command> [<args>]
```

**NOTE**: Applying changes via the `uci` command will wipe any comments from configuration files being modified.

To maintain any notations or `#` comments modify the file manually with `vi`

```ash
uci set uhttpd.main.listen_http='8080'
uci commit uhttpd
/etc/init.d/uhttpd restart
```

Viewing configurations for a select subsystem:

Available subsystems are:
- defaults
- dnsmasq
- dropbear
- firewall
- fstab
- net
- qos
- samba
- system
- wireless

```ash
uci show <subsystem>

uci show dropbear
```

Setting a configuration in a subsystem:

```ash
uci add firewall rule
uci set firewall.@rule[-1].src='wan'
```

Showing the not-yet-saved modified values

```ash
uci changes
```

Saving modified values of a single subsystem
```ash
uci commit SUBSYSTEM_NAME
reload_config
```

Saving all modified values
```ash
uci commit
reload_config
```

Generate a UCI section via batch script that can be copy/pasted into terminal:

```ash
# https://openwrt.org/docs/guide-user/base-system/uci#generating_a_full_uci_section_with_a_simple_copy-paste

rule_name=$(uci add firewall rule)
uci batch << EOI
set firewall.$rule_name.enabled='1'
set firewall.$rule_name.target='ACCEPT'
set firewall.$rule_name.src='wan'
set firewall.$rule_name.proto='tcp'
set firewall.$rule_name.dest_port='22'
set firewall.$rule_name.name='SSH'
EOI
uci commit
```

# Administration

A factory install / reset of OpenWrt has open ssh / http access on the LAN, but wireless radios are disabled by default.

This prevents taking over an unconfigured device wirelessly.

Set the root password
```ash
passwd
```

Protect the serial TTY
```ash
uci set system.@system[0].ttylogin='1'
uci commit system
/etc/init.d/system restart
```

View configuration
```ash
cat /etc/config/network
cat /etc/config/wireless
# or
uci show wireless
uci show network
```

View wireless card's [regulatory information](https://openwrt.org/docs/guide-user/network/wifi/wifi_countrycode#how_does_openwrt_known_what_wi-fi_channel_settings_are_ok_for_my_country)
```ash
iw reg get
```

List network interfaces
```ash
ifconfig
ip a
ubus list network.interface.*
```

Enumerate a specific interface
```ash
ifstatus <iface>
```

Enumerate sockets (BusyBox specific syntax, no `-A` option)
```ash
netstat -antup
```

View authorized ssh keys
```ash
cat /etc/dropbear/authorized_keys
```

Enable MFA for SSH
```ash
# https://openwrt.org/docs/guide-user/services/ssh/ssh.mfa.auth
opkg install google-authenticator-libpam openssh-server-pam
```

Manage services
```ash
service list
service <service> stop
service <service> start
service <service> reload
service <service> disable
service <service> enable

service rpcd disable
service dnsmasq disable
service dropbear start
service uhttp reload
```

Reload and reinitialize configurations
```ash
service <service> reload
service network reload
# or
/etc/initi.d/<service> restart
/etc/init.d/network restart
```

**NOTE**: some `/etc/config/wireless` settings will reload in real time when edited with `vi`, such as `option isolate '1'`

Other settings such as MAC address filtering `list maclist` will require `/etc/init.d/network restart`

Regenerate factory wireless configuration
```ash
# https://openwrt.org/docs/guide-user/network/wifi/basic#regenerate_configuration
rm -f /etc/config/wireless
wifi config
```

Enumerate free memory space
```ash
free
```

Verify read-only SquashFS root partition exits
```ash
grep squash /proc/mounts
```

Harden Web-UI access and authentication via ssh tunneling
```ash
# Add ssh public key to /etc/config/dropbear/authorized_keys

# Replace the http(s) listening addresses of [::] with [::1] and 0.0.0.0 with 127.0.0.1
vi /etc/config/uhttpd

service uhttpd reload

# confirm changes:
netstat -antup
```

Find all non-volatile files that have changed on the filesystem in the last 24 hours:
```ash
find / -type f -mtime -1 | grep -Ev "/(tmp|sys|proc)"
```

Package management
```ash
# update all packages, do this after every power cycle
# and before installing or searching for new packages
opkg update

# search for a package
opkg list | grep <regex-for-package-name>
opkg list | grep 'nmap*'

# show package details
opkg info nmap-full

# install packages
opkg install nmap-full netcat tcpdump nano wireguard-tools kmod-wireguard git python3 scapy tmux audit
```

## Logging

Syslog writes to a small membuffer, which is why you won't find the typical (large) `/var/log/*` files where they normally are.

Use `logread` to review logs for the system applications, and `dmesg` for kernel level logs.

View all dropbear activity:
```ash
logread -e 'dropbear'
```

View dnsmasq logs:
```ash
logread -e 'dnsmasq'
```

**TO DO**: further logging configuration.

As of now, it seems syslog and kern.log are only accessible via the LuCI web interface.

You can copy/paste the syslog output into a text file to parse locally:
```ash
grep 'auth' <openwrt.log>
grep 'succeed' <openwrt.log>
grep 'fail' <openwrt.log>
```

**Log Examples**

uhttpd logs:
```
Tue Jan  1 08:30:00 2021 daemon.err uhttpd[2000]: luci: failed login on / for root from 127.0.0.1
Tue Jan  1 08:30:00 2021 daemon.err uhttpd[2000]: luci: accepted login on / for root from 127.0.0.1
```

Dropbear logs:
```
Tue Jan  1 08:30:00 2021 authpriv.notice dropbear[3348]: Pubkey auth succeeded for 'root' with key sha1!! aa:bb:cc:dd:ee:ff:11:22:33:44:55:66:77:88:99:00:a1:a2:a3:a4 from 192.168.1.82:53878
Tue Jan  1 08:30:00 2021 authpriv.info dropbear[3348]: Exit (root) from <192.168.1.82:53878>: Disconnect received
```

---

## Networking

- <https://openwrt.org/docs/guide-quick-start/checks_and_troubleshooting>
- <https://openwrt.org/docs/guide-user/base-system/basic-networking>
- <https://openwrt.org/docs/guide-user/network/routing/routes_configuration>
- <https://openwrt.org/docs/guide-user/base-system/dhcp>
- <https://openwrt.org/docs/guide-user/base-system/dhcp_configuration>

The following three sections are one network configuration file, broken into three parts to show each part as a template.

The default loopback and globals:
```conf
config interface 'loopback'
	option device 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'
	option ula_prefix 'f123:xxxx:xxxx::/48'
```

A phyiscal interface:
```conf
config device
	option name 'br-lan'
	option type 'bridge'
	list ports 'eth0'

config interface 'lan'
	option device 'br-lan'
	option proto 'static'
	option ipaddr '192.168.1.1'
	option netmask '255.255.255.0'
	option ip6assign '60'
```

A vlan:
```conf
config device
	option name 'br-vlan1'
	option type 'bridge'
	list ports 'eth0.110'

config device
	option type '8021q'
	option name 'eth0.110'
	option ifname 'eth0'
	option vid '110'

config interface 'vlan1'
	option device 'br-vlan1'
	option proto 'static'
	option ipaddr '172.16.1.100'
	option netmask '255.255.255.0'
	option gateway '172.16.1.1'
	list dns '172.16.1.1'
```

---

### DNS

You can either setup OpenWrt to resolve DNS queries with `dnsmasq` (default built in resolver daemon), or add a remote DNS server to `/etc/resolv.conf`.

**1. Adding an entry to /etc/resolv.conf**

Taking our DNS server of 10.0.2.105 from before, add it to the top of `/etc/resolv.conf` beneath 'search lan'
```conf
nameserver 10.0.2.105
```

**2. Telling dnsmasq to use the upstream dns server**

```ash
# https://openwrt.org/docs/guide-user/base-system/dhcp_configuration#dns_forwarding

uci -q delete dhcp.@dnsmasq[0].server
uci add_list dhcp.@dnsmasq[0].server="10.0.2.105"
uci commit dhcp
/etc/init.d/dnsmasq restart
```
---

The `/etc/config/network` example at: 

<https://openwrt.org/docs/guide-user/network/dsa/dsa-mini-tutorial#multiple_bridged_networks>

illustrates exactly how to configure a network with multiple isolated subnets.

The idea with DSA is devices and interfaces are now broken into separate configuration blocks within the config file.

Subnets, or bridges in this case, cannot talk to eachother without firewall rules allowing the traffic to happen.

You'll note the `wan` interface is less explicit in it's parameters since most of that information will come from the downstream router / gateway.

The internal `lan` and `guest` interfaces have more prescriptive definitions.

Below is a working example of an OpenWrt VM with 3 NICs attached.

- `eth0` = `wan`
- `eth1` = `lan`
- `eth2` = `guest`

```config
config interface 'loopback'
	option device 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'
	option ula_prefix 'fd8a:xxxx:xxxx::/48'

config device
	option name 'br-wan'
	option type 'bridge'
	list ports 'eth0'

config device
	option name 'br-lan'
	option type 'bridge'
	list ports 'eth1'

config device
	option name 'br-guest'
	option type 'bridge'
	list ports 'eth2'

config interface 'wan'
	option device 'br-wan'
	option proto 'dhcp'

config interface 'wan6'
	option device 'br-wan'
	option proto 'dhcpv6'

config interface 'lan'
	option device 'br-lan'
	option proto 'static'
	option ipaddr '192.168.111.101'
	option netmask '255.255.255.0'
	option gateway '192.168.111.1'
	option dns '192.168.111.1'

config interface 'guest'
	option device 'br-guest'
	option proto 'static'
	option ipaddr '192.168.222.101'
	option netmask '255.255.255.0'
	option gateway '192.168.222.1'
	option dns '192.168.222.1'
```

## Wireless

- <https://openwrt.org/docs/guide-user/network/wifi/basic>
- <https://en.wikipedia.org/wiki/List_of_WLAN_channels>
- <https://www.ecfr.gov/current/title-47>

#### Country Code and Regulatory Requirements

As mentioned above under [administration](#administration) for how to obtain your wireless card's [regulatory info](https://openwrt.org/docs/guide-user/network/wifi/wifi_countrycode#how_does_openwrt_known_what_wi-fi_channel_settings_are_ok_for_my_country), once the country code is set under `/etc/config/wireless` the device will only allow settings in compliance with that country's regulations.

OpenWrt uses the CRDA database to do this which is part of the Linux kernel.

- <https://wireless.wiki.kernel.org/en/developers/Regulatory#crda> 

```
config wifi-device 'radio0'
	...
	option country '<country-code>'
```

Be aware you must still comply with the maximum transmission power (`txpower`) for your locale, taking into account [indoor and outdoor operation](https://openwrt.org/docs/guide-user/network/wifi/wifi_countrycode#wi-fi_outdoor_operation).

#### Wireless Channels

In many devices, setting the channel to `auto` will select the lowest available channel regardless of local signal density.

```
config wifi-device 'radio0'
	...
	option channel '<channel>'
```

You can use the script detailed in the following link to automatically select the channel with the least density:

- <https://openwrt.org/docs/guide-user/network/wifi/iwchan>

Alternatively from the WebUI you can view the wireless density in your area as a graph under `System > Status > Channel Analysis`.

This scan will also create a list of all observed wireless devices operating under the same 2.4/5 GHz frequency.

From here change your wireless radio('s) channel to the least populated.


## Virtualization

- <https://openwrt.org/docs/guide-user/virtualization/vmware>
- <https://openwrt.org/docs/guide-user/virtualization/virtualbox-vm>

#### qemu-utils

Using qemu-utils is the quickest way to create an OpenWrt VM.

```bash
sudo apt install -y qemu-utils
```

Obtain the OpenWrt signatures for x86/64, replacing <verison> with the one you want:
```bash
curl -LfO 'https://downloads.openwrt.org/releases/<version>/targets/x86/64/sha256sums.asc'
curl -LfO 'https://downloads.openwrt.org/releases/<version>/targets/x86/64/sha256sums'
```

On the `https://downloads.openwrt.org/releases/<version>/targets/x86/64/` page you'll see a few different images to choose from:

- generic-ext4-combined-efi.img.gz    <- this is the image we want
- generic-ext4-combined.img.gz
- generic-ext4-rootfs.img.gz
- generic-kernel.bin
- generic-squashfs-combined-efi.img.gz
- generic-squashfs-combined.img.gz
- generic-squashfs-rootfs.img.gz
- rootfs.tar.gz

If you want the UEFI compatible image (recommended), use `generic-ext4-combined-efi.img.gz` instead of `generic-ext4-combined.img.gz`.
```bash
curl -LfO 'https://downloads.openwrt.org/releases/21.02.1/targets/x86/64/openwrt-21.02.1-x86-64-generic-ext4-combined-efi.img.gz'
```

Obtain the signing key, verifying the signatures:
```bash
# https://openwrt.org/docs/guide-user/security/signatures

# 21.02
gpg --keyid-format long --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys '6672 05E3 79BA F348 863A 5C66 88CA 59E8 8F68 1580'
gpg --verify --keyid-format long ./sha256sums.asc ./sha256sums

# 22.03
gpg --keyid-format long --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys 'BF85 6781 A012 93C8 409A BE72 CD54 E82D ADB3 684D'
gpg --verify --keyid-format long ./sha256sums.asc ./sha256sums
```

Verify the firmware hash:
```bash
sha256sum -c ./sha256sums --ignore-missing
```

You must use `gunzip` to properly unzip the data. The GUI archive utilites don't handle the trailing empty data correctly:
```bash
gunzip openwrt-21.02.1-x86-64-generic-ext4-combined-efi.img.gz
```

**TIP**: Once the .img file is decompressed, you can `right-click > mount image filesystem` (this uses the Image Disk Utility on Ubuntu) and browse the firmware's filesystem under `/media/$USERNAME/rootfs` and `/media/$USERNAME/kernel`

Convert the raw image to a vmdk file:

```bash
qemu-img convert -f raw -O vmdk <input-file>.img <output-file>.vmdk
```

### Starting the OpenWrt VM

**VirtualBox**
- `Tools > Add` and browser to the .vmdk file
- If the OpenWrt .vmdk file is not available to select here, create a new `Tools > New` Linux / Other x86-64 virtual machine, and when selecting a disk image use the existing OpenWrt .vmdk.
- Power on. When the boot sequence looks like it's paused, pressing [Enter] will bring you to the root shell / login screen.

**VMware**
- Follow a similar procedure in VMware, creating a new Linux / Other x86-64 VM, choosing the Linux Kernel that your version of OpenWrt has, and selecting the .vmdk file as an existing disk when you get to that option.
- It may also help organize your virtual machine's files on your hard drive by moving the .vmdk file into the VM's folder after creating it in VMware.
- You'll need to create a new hard disk under settings and point it at the .vmdk file's new path, then delete the old one (you cannot just update the path on your host).

In both cases, remember to enable EFI firmware if it's an EFI enabled image.

You'll also receive a notice on VMware Workstation for Linux the VM is entering promiscuous networking mode.

**After boot, the startup log will eventually stop scrolling - press [Enter] here to be at the console menu**

You will likely need to configure `/etc/config/network` manually if the output of `ip a` or `ifconfig` indicates a hardcoded address of 192.168.1.1 on a bridged interface without receiving dhcp from the local network.

To do this:

```ash
vi /etc/config/network

# ensure the following exists
config device
	option name 'br-wan'
	option type 'bridge'
	list ports 'eth0'

config interface 'wan'
	option device 'br-wan'
	option proto 'dhcp'
```

The `option proto dhcp` will allow OpenWrt to obtain an IP address from the upstream router.

If you run into errors, double check the syntax of your `/etc/config/network` for single `'` quotes where needed, and no repeated lines in the wrong code blocks.

You may also want to add a second networking interface to act as the 'LAN' side, following the configuration example above:

```ash
vi /etc/config/network

# assumes 'wan' is eth0, so 'lan' will be eth1
config device
	option name 'br-lan'
	option type 'bridge'
	list ports 'eth1'

config interface 'lan'
	option device 'br-lan'
	option proto 'static'
	option ipaddr '192.168.10.1'
	option netmask '255.255.255.0'
```

Alternatively, set a single NIC to be a 'client' and receive a DHCP lease from a downstream router.

Instead of those above, the only changes are removing the following lines from `config interface 'lan'`:
```
	option proto 'static'
	option ipaddr '192.168.1.1'
	option netmask '255.255.255.0'
```

And replacing them with:
```
	option proto 'dhcp'
```

Run `/etc/init.d/network restart` to apply the config.

---

## Bridged AP

You do not *need* to configure firewall rules, routes, dns, or non-wireless settings when OpenWrt is acting as a bridged access point.

All of these services are handled by the downstream router.

You can safely disable the following services:

Disable dnsmasq

- `/etc/init.d/dnsmasq disable`
- `System > Startup > dnsmasq`

Disable odhcp

- `/etc/init.d/odhcpd disable`
- `System > Startup > odhcp`

Apply changes
```bash
/etc/init.d/network reload
ifup wifi
wifi
```

## Firewall

The default firewall rules can be found in `/etc/config/firewall`.

Dump them with `iptables` or `nftables`

- <https://openwrt.org/docs/guide-user/firewall/misc/nftables>

Example configuration syntax enabling SSH access from the WAN side:

```conf
# Allow external SSH access
config rule
    option name         Allow-SSH-WAN
    option src          wan
    option proto        tcp
    option dest_port    22
    option target       ACCEPT
    option family       ipv4
```

Run `/etc/init.d/firewall restart` to apply the rule.


## VLAN

- <https://openwrt.org/docs/guide-user/network/vlan/switch_configuration>

Example configurations of VLANS:

```
# /etc/config/network

config device
	option name 'br-lan'
	option type 'bridge'
	list ports 'lan1'
	list ports 'lan2'
	list ports 'lan3'
	list ports 'lan4'

config bridge-vlan
	option device 'br-lan'
	option vlan '1'
	list ports 'lan1'
	list ports 'lan2'

config bridge-vlan
	option device 'br-lan'
	option vlan '2'
	list ports 'lan3'
	list ports 'lan4'

config interface 'home'
	option device 'br-lan.1'
	option proto 'static'
	option ipaddr '192.168.1.1'
	option netmask '255.255.255.0'

config interface 'office'
	option device 'br-lan.2'
	option proto 'static'
	option ipaddr '192.168.13.1'
	option netmask '255.255.255.0'
```

Example configurations of VLANS with VLAN Tagging:

```
# /etc/config/network
config device
	option name 'br-lan'
	option type 'bridge'
	list ports 'lan1'
	list ports 'lan2'
	list ports 'lan3'
	list ports 'lan4'

config bridge-vlan
	option device 'br-lan'
	option vlan '1'
	list ports 'lan1'
	list ports 'lan2'
	list ports 'lan3'
	list ports 'lan4:t'

config bridge-vlan
	option device 'br-lan'
	option vlan '2'
	list ports 'lan4:u*'

config interface 'lan'
	option device 'br-lan.1'
	option proto 'static'
	option ipaddr '192.168.1.1'
	option netmask '255.255.255.0'
```

