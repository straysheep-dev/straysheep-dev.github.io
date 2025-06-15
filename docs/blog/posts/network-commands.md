---
draft: false
date:
  created: 2025-04-18
  updated: 2025-06-15
categories:
  - how-to
  - networking
  - wireless
  - cli
  - linux
  - windows
  - reference
---

# :material-console-network: Network Commands

This post is meant to be a single point of reference for all of the random ways of interacting with and diagnosing network(s) from a machine.

It was started after spending a significant amount of time working in packer, across both the Debian and Red Hat family OS's. There are so many ways to handle networking it's hard to remember these notes when I don't have access to them, and each tool had a dedicated page that has been built upon for 5+ years. Porting each of these notes to this page is an opportunity to clean up, review, and expand on each tool (and add new ones).

As of the latest update, this includes Linux (Debian / RedHat), BSD (pfSense), and Windows.

<!-- more -->

Each tool or section is presented with a quick overview of the tool, how to install it if that's ever necessary, and a set of useful examples starting with what I feel is the most practical command structure(s) to know as go-to's, followed by arbitrary useful and expanded examples.

!!! warning "AI Usage"

	Some code snippets and concepts were originally discovered by conversing with ChatGPT, in many cases either o1-preview or 4o. These were always validated by sourcing the official documentation and testing any examples shared.

---


## ping

!!! note "The ping Command"

	The `ping` command is one of the most basic but useful tools to test network connectivity, reliability, and name resolution. It works by sending ICMP echo request packets to a specified host and waiting for a reply. It's available by default on nearly all Unix-like systems and is built-in to Windows as ping.exe.

	References:

	- <https://github.com/iputils/iputils/>
	- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ping>
	- <https://developers.google.com/speed/public-dns/docs/troubleshooting>
	- <https://en.wikipedia.org/wiki/Ping_(networking_utility)>


!!! tip "Reading ping Output"

	Ping is meant to be a connectivity and diagnostic tool. Sometimes you'll need to use it to determine the frequency of connectivity drops in a network. This makes more sense if you run ping for a number of minutes before reviewing the results.

	```bash
	$ ping -I lo 127.0.0.1
	PING 127.0.0.1 (127.0.0.1) from 127.0.0.1 lo: 56(84) bytes of data.
	64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.057 ms
	64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.049 ms
	64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.049 ms
	64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.056 ms
	^C
	--- 127.0.0.1 ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 3077ms
	rtt min/avg/max/mdev = 0.049/0.052/0.057/0.003 ms
	```

	This resuls in two summary lines:

	- Packets *sent*, and how many were *lost*
	- `min/avg/max/mdev` is showing you the **latency statistics**

	The TTL can indicate the type of operating system:

	- *nix systems often have `ttl=64`
	- Windows often has `ttl=128`

**Installation :material-package-variant-plus:**

```bash
# On Ubuntu ping is part of the iputils package and installed by default
sudo apt install -y iputils-ping
```

**Practical Usage :material-wrench-cog-outline:**

Ping a hostname or IP address until `Ctrl+c` canceled:

```bash
ping example.com
ping 1.1.1.1
ping fe80::abcd:ef01:2345:1234%enp1s0
```

```cmd
ping.exe /t 192.168.1.1
```

Set a maximum number of ping packets to send:

```bash
ping -c 4 example.com
```

```cmd
ping.exe /n 4 example.com
```

!!! tip "Count on Linux"

	Linux by default will ping forever if `-c <number>` isn't set. The default behavior on Windows is to send 4 pings.


Set the source interface, interface is either an address, an interface name or a VRF name (Linux only):

```bash
ping -I eth0 google.com
ping -I 10.40.8.182 google.com
```

This is an easy way to obtain a public IPv6 address to ping:

```bash
dig @1.1.1.1 google.com -t AAAA +tls +short
```

Request reverse name resolution (Windows only):

```cmd
ping.exe /a 1.1.1.1
```

**Expanded Usage :octicons-stack-16:**

Set the TTL of a ping packet:

```bash
ping -t 128 example.com
```

```cmd
ping.exe /i 128 example.com
```

!!! quote "TTL Details"

	 The TTL value of an IP packet represents the maximum number of IP routers that the packet can go through before being thrown away. In current practice you can expect each router in the Internet to decrement the TTL field by exactly one.

	 This summary is directly from the `ping` manpage.

Set the interval between packets sent (in seconds):

```bash
ping -i 5 example.com
```

```cmd
ping.exe /i 5 example.com
```

Set the packet size (in bytes), default=64, 8 for the ICMP header + 56:

```bash
ping -s 1337 example.com
```

```cmd
ping.exe /l 1337 example.com
```

:octicons-history-16: See also: [Ping of Death](https://en.wikipedia.org/wiki/Ping_of_death)

Ping flood (requires root):

- For every ECHO_REQUEST sent a period `.` is printed
- For every ECHO_REPLY received a backspace is printed

```bash
sudo ping -f 10.0.0.1
```

---

## tracepath

!!! note "tracepath"

	Effectively `traceroute`, but without any special options and does not require root privileges.

	> It traces the network path to destination discovering MTU along this path. It uses UDP port `<port>` or some random port.

	References:

	- <https://github.com/iputils/iputils/>
	- [traceroute vs tracepath](https://www.redhat.com/en/blog/traceroute-tracepath-network-troubleshooting)

**Installation :material-package-variant-plus:**

```bash
# tracepath is often installed by default on Ubuntu
sudo apt install -y iputils-tracepath
```

**Practical Usage :material-wrench-cog-outline:**

Trace a network path and print both, hostnames and IP addresses:

```bash
tracepath -b google.com
tracepath -b 10.0.0.1
```

Do not resolve hostnames:

```bash
tracepath -n 1.1.1.1
```

Set an initial destination port:

```bash
tracepath -p 8000 192.168.40.22
```

---


## traceroute

!!! note "traceroute"

	> Traces the route taken by packets over a network.

	References:

	- <https://www.gnu.org/software/inetutils/manual/inetutils.html#traceroute-invocation>
	- <https://packages.debian.org/bookworm/inetutils-traceroute>
	- [traceroute vs tracepath](https://www.redhat.com/en/blog/traceroute-tracepath-network-troubleshooting)

	See also:

	- [tcptraceroute](https://github.com/mct/tcptraceroute)

**Installation :material-package-variant-plus:**

If you try to run `traceroute` on Ubuntu when it's not installed, you'll get:

```bash
# Command 'traceroute' not found, but can be installed with:
# sudo apt install inetutils-traceroute  # version 2:2.2-2ubuntu0.1, or
# sudo apt install traceroute            # version 1:2.1.0-2
```

!!! tip ""

	See [traceroute vs inetutils-traceroute](https://askubuntu.com/questions/1017286/what-is-the-difference-between-traceroute-from-traceroute-and-inetutils-tracerou).

Install one of them with:

```bash
sudo apt install -y traceroute

# GNU version
sudo apt install -y inetutils-traceroute
```

!!! tip ""

	traceroute is often *not* installed by default on Linux systems like `ping` is. Instead, you may find `tracepath` or `mtr`.

**Practical Usage :material-wrench-cog-outline:**

```bash
traceroute example.com
```

---


## tracert

!!! note "tracert"

	tracert.exe is a Windows tool.

	> This diagnostic tool determines the path taken to a destination by sending Internet Control Message Protocol (ICMP) echo Request or ICMPv6 messages to the destination with incrementally increasing time to live (TTL) field values.

	> To trace a path and provide network latency and packet loss for each router and link in the path, use the `pathping.exe` command.

	- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tracert>

**Practical Usage :material-wrench-cog-outline:**

Trace the path to a host:

```cmd
tracert www.microsoft.com
```

Do not attempt name resolution:

```cmd
tracert /d www.microsoft.com
```

---


## mtr

!!! note "My Traceroute"

	`mtr` (My Traceroute) combines the functionality of the `traceroute` and `ping` programs into a single network diagnostic tool.

	- <https://github.com/traviscross/mtr>
	- [Red Hat: How to use the Linux mtr command ](https://www.redhat.com/en/blog/linux-mtr-command)


**Installation :material-package-variant-plus:**

```bash
# CLI + GUI tool
sudo apt install -y mtr

# CLI version only, installed by default on Ubuntu
sudo apt install -y mtr-tiny

# RedHat/Fedora
sudo dnf install mtr

# macOS (Homebrew)
brew install mtr
```

Available manpages:

```bash
man mtr
man mtr-packet
```

**Practical Usage :material-wrench-cog-outline:**

Start an interactive (curses-based) trace using an `-I` interface, include `-b` IPs and hostnames:

```bash
mtr -I enp1s0 -b example.com
```

Run a trace for `-c 20` pings, that prints `-r` results to the terminal, for optional redirection to a file:

```bash
mtr -I enp1s0 -b -c 20 -r 192.168.0.1
```

!!! tip ""

	`-c` is not very useful for interactive mode as the curses interface auto-quits after reaching the count limit, and you'll lose the visibile results.

!!! tip "Output Formats"

	`mtr` has a number of useful output formats:

	- `--csv`
	- `--json`
	- `--xml`

	These are invoked *separately* from `-r` / `-w` report modes, which prints the default output format to your console non-interactively.

Do not do name resolution:

```bash
mtr -n example.com
```

**Expanded Usage :octicons-stack-16:**

!!! tip "Destination Port Options"

	`mtr` has some really interesting destination port options (taken directly from the manpage):

	`-u`, `--udp`

	Use UDP datagrams instead of ICMP ECHO.

	`-T`, `--tcp`

	Use TCP SYN packets instead of ICMP ECHO.  PACKETSIZE is ignored, since SYN packets  can not contain data.

	`-S`, `--sctp`

	Use Stream Control Transmission Protocol packets instead of ICMP ECHO.

	`-P PORT`, `--port PORT`

	The target port number for TCP/SCTP/UDP traces.

	`-L LOCALPORT`, `--localport LOCALPORT`

	The source port number for UDP traces.

	`-Z SECONDS`, `--timeout SECONDS`

	The  number  of  seconds  to keep probe sockets open before giving up on the connection. Using large values for this, especially combined with a short interval, will  use  up  a lot of file descriptors.

	`-M MARK`, `--mark MARK`

	Set the mark for each packet sent through this socket similar to the netfilter MARK target but socket-based.  MARK is 32 unsigned integer.  See socket(7) for full  description of this socket option

---


## tcpdump

!!! note "tcpdump"

	The standard command-line network traffic analyzer. It's an excellent tool for monitoring, debugging, troubleshooting, or creating network capture files for later analysis.

	Website: <https://www.tcpdump.org/>

	If you're looking for an alternative, it's likely `tshark`.


**Installation :material-package-variant-plus:**

```bash
# Debian / Ubuntu
sudo apt install -y tcpdump
```

!!! tip ""

[Daniel Miessler's cheat sheet on tcpdump](https://danielmiessler.com/blog/tcpdump) should be your first stop.

**Practical Usage :material-wrench-cog-outline:**

Monitor all traffic crossing interface `-i eth0`, dropping process privileges to `nobody`:

```bash
sudo tcpdump -i eth0 -n -vv -Z nobody
```

!!! tip "Useful Options"

	Essentials:

	- Use `-D` to list available interfaces
	- Use `-n` to prevent name resolution
	- Use `-nn` to prevent name resolution *and* port resolution
	- Use `-v[vv]` to increase verbosity in output, such as what gets decoded
	- Use `-Q [in|out]` to specify direction
	- Use `-Z USER` to drop prvilieges to those of USER, often `nobody` or `tcpdump`
		- Ensure USER can write to the output directory
		- It's recommended to use a limited privilege user like `nobody` or `tcpdump` for ongoing capture processes

	Console Output:

	- `-A` to print ASCII data
	- `-X[X]` to print hex and ASCII data (similar to Wireshark)

	File Output:

	- `-w FILE` write data to a pcap file instead of stdout, can use a strftime(3) fromatted datetime string
		- Example: `-w $(hostname -s).%Y%m%d%H%M%S.pcap`
		- For commands executing as systemd service tasks, use double `%%` escaping: `$(hostname -s).%%Y%%m%%d%%H%%M%%S.pcap`
	- `-r FILE` reads a file written with `-w FILE`
	- `-G SECONDS` rotates the `-w` written FILE every SECONDS
		- Overwrites the FILE if a strftime(3) fromatted datetime string isn't used
	- `-C SIZE` used with `-w FILE`, rotates files based on SIZE, appending an integer to the FILE string each time

	Much of this is summarized from the tcpdump manpage.

**Expanded Usage :octicons-stack-16:**

For the expression syntax, see `man pcap-filter`.

⚠️ TO DO ⚠️

---


## tshark

⚠️ TO DO ⚠️

---

## ipconfig

!!! note "ipconfig"

	`ipconfig` is possibly the most well-known Windows utility to obtain your networking information.

	> Displays all current TCP/IP network configuration values and refreshes Dynamic Host Configuration Protocol (DHCP) and Domain Name System (DNS) settings. Used without parameters, ipconfig displays Internet Protocol version 4 (IPv4) and IPv6 addresses, subnet mask, and default gateway for all adapters.

	The latest online documentation does a great job showcasing the usage:

	- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig>

**Practical Usage :material-wrench-cog-outline:**

Display all TCP/IP configuration information for all adapters:

```cmd
ipconfig /all
```

Release and renew a DHCP lease, either per-adapter or globally:

```cmd
ipconfig /release [<adapter>]
ipconfig /release6 [<adapter>]
ipconfig /renew [<adapter>]
ipconfig /renew6 [<adapter>]
```

List the contents of the DNS resolver cache:

```cmd
ipconfig /displaydns
```

Flush DNS resolver cache:

```cmd
ipconfig /flushdns
```

---


## arp

⚠️ TO DO ⚠️


---


## netstat

⚠️ TO DO ⚠️

```bash
# Linux
sudo netstat -antup

# BSD
TO DO
```

```cmd
netstat.exe -abno
```

---


## ss

!!! note "Socket Statistics"

	`ss` is used to dump socket statistics. It allows showing information similar to `netstat`, and in fact is slowly replacing it.  It can display more TCP and state information than other tools.

	`ss` is part of the `iproute2` package.

**Practical Usage :material-wrench-cog-outline:**

Display all "inet" (IPv4/6) network connections regardless of state, and the related process.

```bash
sudo ss -anp -A inet | less -S
```

!!! tip "Other socket tables"

	`sudo ss -anp -A all | less -S` will display `-A all` socket tables on the system, inlcuding Unix sockets. This behavior can be controlled with:

	`-A QUERY`, `--query=QUERY`, `--socket=QUERY`

	> List of socket tables to dump, separated by commas. The following identifiers are understood: all, inet, tcp, udp, raw, unix, packet, netlink, unix_dgram, unix_stream, unix_seqpacket, packet_raw, packet_dgram, dccp, sctp, vsock_stream, vsock_dgram, and xdp. Any item in the list may optionally be prefixed by an exclamation mark (!) to exclude that socket table from being dumped.

	Taken from the manpage for `ss`.

⚠️ TO DO ⚠️

---


## route

⚠️ TO DO ⚠️


---


## netsh

Regarding packet captures with native Windows tools, see: <https://github.com/microsoft/etl2pcapng>

⚠️ TO DO ⚠️

---


## ip

!!! note "The ip Command"

	> Show or manipulate routing, network devices, interfaces and tunnels.

	`ip` is meant as a more feature-rich replacement to `ifconfig` and `route`, which are being deprecated. This is similar to [`iw`](https://wireless.wiki.kernel.org/en/users/documentation/iw) replacing `iwconfig`.

	Much of this information was gleaned and adapted from Ubuntu's server documentation.

	References:

	- [Main Project Page and Source Code](https://wiki.linuxfoundation.org/networking/iproute2)
	- [Ubuntu Server: Configuring Networks](https://ubuntu.com/server/docs/network-configuration)

**Installation :material-package-variant-plus:**

!!! quote ""

	iproute2 is usually shipped in a package called iproute or iproute2 and consists of several tools, of which the most important are `ip` and `tc`.

```bash
# Debian / Ubuntu
sudo apt install -y iproute2
```

!!! tip "manpages"

	*If you want to see the built in examples, search "EXAMPLES" under the `man` pages:*

	```bash
	man ip
	man ip-address
	man ip-link
	man ip-route
	```

	Interestingly if you read both manpages for `ping` or `tracepath` and `ip`, you'll see they were originally written by the same author.

**Practical Usage :material-wrench-cog-outline:**

Get information (basics):

```bash
ip addr      # Protocol address info
ip link      # Network device info
ip route     # Route info
ip -6 <cmd>  # IPv6 equivalent command
```

!!! tip ""

	*Nearly all of the `ip` commands can be abbreviated, so `ip address` is the same as `ip addr` which is the same as `ip a`.*

Get information (expanded):

```bash
ip address show up
ip link show dev ens33
```

!!! tip "Changes are Temporary"

	The majority of the changes and settings you can make with the `ip` command(s) are not persistent across reboots. These often require (on modern systems) a [netplan configuration](https://documentation.ubuntu.com/server/explanation/networking/configuring-networks/index.html#dynamic-ip-address-assignment-dhcp-client).

	See also: <https://netplan.readthedocs.io/en/stable/netplan-yaml/#>

Add / delete default gateway (route to any):

```bash
sudo ip route add default via <gateway-ip>
sudo ip route del default via <gateway-ip>
```

Add / delete a defined static route (route to a subnet):

```bash
sudo ip route add <dst-net> via <gateway-ip> dev <dev>
sudo ip route add 192.168.1.0/24 via 10.0.0.1 dev eth0
sudo ip route delete 192.168.1.0/24 via 10.0.0.1 dev eth0
```

Add / delete an interface:

```bash
sudo ip link add dev eth2 type bridge
sudo ip link del dev eth2
```

!!! note "Adding Interfaces"

	Say you have `eth0`. To truly add another interface, for instance an `eth1`, you would need another NIC attached to the device.

	In other words, the only interfaces you'd be creating with this command are virtual or bridged or VLAN style interfaces.

	Alternatively you could use these commands to completely recreate the information for your existing `eth0` interface from scratch if necessary.

Enable / disable an interface:

```bash
sudo ip link set wlan0 down
# Set the interface to monitor mode, or change the mac address
sudo ip link set wlan0 up
```

Add / remove an IP address on a device:

```bash
sudo ip addr add <subnet/cidr> dev <dev>
sudo ip addr del <subnet/cidr> dev <dev>
```

!!! question "Why would you do this?"

	In some cases, particularly with local VM's on laptops cycling through sleep and wake modes, stale network information can persist on the interface. Rather than rebooting, if you're not familiar with the network configuration the system is using, you can manually remove old information this way.

Purge all IP information from a device:
```bash
sudo ip address flush dev eth4
```

To configure an interface from scratch:

!!! warning ""

	You can get disconnected over SSH by doing this.

```bash
sudo ip addr flush dev eth0
sudo ip addr add 10.55.55.99/24 dev eth0
sudo ip route add default via 10.55.55.1
sudo nano /etc/resolv.conf  # Add DNS information
```

---


## iw

⚠️ TO DO ⚠️

---

## netplan

!!! note "netplan"

	> YAML network configuration abstraction for various backends.

	- [netplan.io (Maintained by Canonical)](https://netplan.io/)
	- [readthedocs.io YAML Configuration Reference](https://netplan.readthedocs.io/en/stable/netplan-yaml/)
	- [Ubuntu Server Guide: Networking](https://documentation.ubuntu.com/server/explanation/intro-to/networking/)

!!! quote "Network Renderer"

	> During early boot, the netplan "network renderer" runs which reads `/{lib,etc,run}/netplan/*.yaml` and writes configuration to `/run` to hand off control of devices to the specified networking daemon. Configured devices get handled by systemd-networkd by default, unless explicitly marked as managed by a specific renderer (NetworkManager).

	Valid values are `networkd` and `NetworkManager`.  Defaults to `networkd` if not defined. This is the most minimal netplan configuration for most desktop environments:

	```yml
	# /etc/netplan/01-network-manager-all.yaml
	network:
	  version: 2
	  renderer: NetworkManager
	```

	Much of this is quoted from the manpages.

**Practical Usage :material-wrench-cog-outline:**

Display the current configuration of an `[<interface>]` or `-a` all interfaces:

```bash
sudo netplan status [<interface>]
sudo netplan status -a
```

!!! tip "netplan status and networkd"

	Currently, netplan status depends on systemd-networkd as a source of data and will try to start it if it's not masked.

Create and apply a configuration:

```bash
# Examples from https://netplan.io/
sudo netplan generate  # Use /etc/netplan to generate the required configuration for the renderers
sudo netplan try       # Will roll back if networking is broken or without confirmation
sudo netplan apply     # Applies all configuration for the renderers, restarting them as necessary
```

---


## /etc/network/interfaces

⚠️ TO DO ⚠️

---

## systemd-networkd

!!! note "systemd-networkd"

	> systemd-networkd is a system service that manages networks. It detects and configures network devices as they appear, as well as creating virtual network devices.

	> - `systemd-networkd.service`
	> - `/lib/systemd/systemd-networkd`

	> To configure low-level link settings independently of networks, see systemd.link(5).

	Summary taken from the systemd-networkd manpage.

**Practical Usage :material-wrench-cog-outline:**

Get DNS settings:

```bash
resolvectl status
```

!!! info "resolvectl"

	`resolvectl` may be used to resolve domain names, IPv4 and IPv6 addresses, DNS resource records and services with the systemd-resolved.service(8) resolver service.



⚠️ TO DO ⚠️

---


## NetworkManager (nmcli)

!!! note "NetworkManager Command Line Utility"

	NetworkManager Command Line, or `nmcli`. Much of this information was gleaned and adapted from RedHat's documentation linked below.

	This is the standard utility for managing and interacting with NetworkManager for networking in desktop environments, where [servers primarily use netplan with systemd-networkd as the backend renderer](https://documentation.ubuntu.com/server/explanation/intro-to/networking/).

	- [gnome.org NetworkManager Project Page](https://wiki.gnome.org/Projects/NetworkManager)
	- [RedHat: Configuring IP Networking with `nmcli`](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_nmcli)
	- [RedHat Legal Notice](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/legal-notice)
	- [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/)

**Practical Usage :material-wrench-cog-outline:**

To enable and disable connections based on network interface (device):

```bash
nmcli device
nmcli dev down enp1s0
nmcli dev up eth0
```

To enable and disable connections based on profile:

```bash
nmcli con show
nmcli con down 'Wired connection 1'
nmcli con up 'hyper-v-lab'
```

!!! tip "The Default Profile"

	The default `Wired connection 1` is the most generic and versitile profile that accepts DHCP when it's available.

To configure an interface profile from scratch for an ethernet connection with static values using `nmcli` (adapted from the nmcli-examples manpage):

```bash
CON_NAME='hyper-v-lab'
IFNAME='eth0'
ADDR_IPV4='10.55.55.11/24'
GTWY_IPV4='10.55.55.1'
DNS_IPV4='1.1.1.1 1.0.0.1'
ADDR_IPV6=''
GTWY_IPV6=''
DNS_IPV6='2606:4700:4700::1111 2606:4700:4700::1001'
nmcli con add type ethernet con-name "$CON_NAME" ifname "$IFNAME" ip4 "$ADDR_IPV4" gw4 "$GTWY_IPV4"
nmcli con mod "$CON_NAME" ipv4.dns "$DNS_IPV4"
nmcli con mod "$CON_NAME" ipv6.dns "$DNS_IPV6"
nmcli con up "$CON_NAME" ifname "$IFNAME"
nmcli device status
nmcli -p con show "$CON_NAME"  # -p "pretty" output is more readable
```

!!! tip "Managed and Unmanaged Networking"

	If you get the error that "device is strictly unmanaged" check `/etc/NetworkManager/` and `/etc/NetworkManager/conf.d`.

	Look for any configurations lines under `[keyfile]` or `[ifupdown]` sections in files that might be setting interfaces as unmanaged or strictly managed.

	In Kali, you need to change `managed=false` to `managed=true` in `/etc/NetworkManager/NetworkManager.conf` to use `nmcli`.

To delete a network configuration:

```bash
nmcli con show

# Delete based on profile-name
nmcli con delete id <profile-name>

# Delete based on uuid
nmcli con delete uuid <profile-uuid>
```

Scan for wireless networks:

```bash
nmcli device wifi list [--rescan yes] [ifname wlan0] [bssid <bssid>]
```

Connect to or disconnect from a wireless network:

```bash
# Prompt for the password
nmcli --ask device wifi connect <ssid> ifname <wlanX>

# Password will appear in bash history
nmcli device wifi connect <ssid> password <password> ifname <wlanX>

nmcli device disconnect <wlanX>
```

**Expanded Usage :octicons-stack-16:**

Create a wireless networking profile from scratch that does the following:

- Ignore local DHCP server's suggested DNS servers and set your own.
- Does not use mDNS or LLMNR
- Randomizes your MAC address
- See also:
	- [NetworkManager Docs: nmcli Settings](https://networkmanager.dev/docs/api/latest/nm-settings-nmcli.html)
	- [serverfault.com How to Manage DNS via nmcli](https://serverfault.com/questions/810636/how-to-manage-dns-in-networkmanager-via-console-nmcli)

!!! warning "AI Usage"

	Some of these settings were originally suggested as part of a larger script built with help from ChatGPT (GPT-o1-preview) from OpenAI.

	The nmcli settings can be verified in the documentation linked above.

```bash
CONN_NAME='Some-ESSID'
SSID="$CONN_NAME"
INTERFACE='wlan0'
nmcli connection add type wifi ifname "$INTERFACE" con-name "$CONN_NAME" ssid "$SSID"

nmcli connection modify "$CONN_NAME" ipv4.dns "127.0.0.1"
nmcli connection modify "$CONN_NAME" ipv4.ignore-auto-dns yes

nmcli connection modify "$CONN_NAME" ipv6.dns "::1"
nmcli connection modify "$CONN_NAME" ipv6.ignore-auto-dns yes

# Disable LLMNR and mDNS
nmcli connection modify "$CONN_NAME" connection.llmnr "no"
nmcli connection modify "$CONN_NAME" connection.mdns "no"

# Set MAC address to a random value
nmcli connection modify "$CONN_NAME" 802-11-wireless.cloned-mac-address "random"

# You will be prompted for the password
nmcli connection up "$CONN_NAME"
```

!!! tip "This is not a global setting"

	You will need to set this **per-connection name** meaning you likely need to run this for every wireless network you connect to, as they all use unique connection profile names.

---

## PowerShell

Networking commands available to PowerShell, including both Windows and Linux versions.


### Test-NetConnection

Tests a TCP or UDP connection to a remote host:port. `ComputerName` can be an IP or hostname.

```powershell
Test-NetConnection -ComputerName 1.1.1.1 -Port 853

# Linux
Test-Connection -ComputerName example.com -Port 443
```

!!! tip ""

	On Linux, PowerShell may only have `Test-Connection`, which functions the same way but only returns `True` or `False`:

---


## netcat

⚠️ TO DO ⚠️

---


## socat

⚠️ TO DO ⚠️

---


## nmap

⚠️ TO DO ⚠️

---


## zmap

⚠️ TO DO ⚠️

---


## naabu

⚠️ TO DO ⚠️

---


## masscan

⚠️ TO DO ⚠️

---


## chisel

⚠️ TO DO ⚠️

---


## proxychains

⚠️ TO DO ⚠️

---
