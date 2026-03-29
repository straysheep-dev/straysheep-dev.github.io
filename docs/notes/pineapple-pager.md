---
title: "Pineapple Pager"
icon: material/fruit-pineapple
draft: false
#date:
#  created: 2026-03-19
#  updated: 2026-03-29
categories:
  - hak5
  - pineapple
  - pager
  - openwrt
  - how-to
  - firewall
  - wifi
  - networking
  - home-lab
  - administration
---

# :material-fruit-pineapple: Pineapple Pager


## Overview

- [Hak5 Pager Documentation](https://docs.hak5.org/wifi-pineapple-pager/wifi-pineapple-pager-by-hak5/)

TODO


## Usage

This is OpenWrt under the hood. With CLI access over SSH or the Virtual Pager web UI, you can easily use a keyboard for things you'd be manually typing using the pager buttons.


### General

- [OpenWrt](./openwrt.md)
- [OpenWrt Configuration Files](https://github.com/straysheep-dev/linux-configs/tree/main/openwrt)

!!! tip "Configure Client-mode Wi-Fi"

    Instead of typing the SSID and passphrase using the pager buttons, you can do this over SSH.

    ```bash
    nano /etc/config/wireless
    # Modify the SSID, encryption type, and passphrase
    wifi reload
    ```

!!! warning "53/tcp+udp Open on 'WAN'"

    The Pager seems to leave port 53 open on the 'WAN' (public, connecting) interface to networks when in client mode. It also answers DNS queries made to this interface. This is curious, and could be fixed with an additional firewall rule block targeting the `option mark 0x4` interface group.

    - `option mark 0x2` is the USB-C interface
    - `option mark 0x3` is the Management AP interface
    - `option mark 0x4` covers all "public" or "WAN" side interfaces

    Add this block below the `Allow-Admin/SSH` and before the `Reject-Admin/SSH` blocks.

    ```conf
    config rule
        option name             'Reject-DNS'
        option src              'wan'
        option mark             '0x4'
        option proto            'tcpudp'
        option dest_port        '53'
        option target           'REJECT'
    ```

    Apply the rule change:

    ```bash
    fw4 reload
    ```

### Built-in Tools

This section covers the built-in pentesting tools that ship with the Pineapple Pager.

TODO


## Payloads

- [Hak5 Pager Payload Development](https://docs.hak5.org/wifi-pineapple-pager/payloads/introduction-to-payloads/)

TODO


## Third-party Packages

The Hak5 docs cover where and how you can and should install packages, via `opkg` or otherwise.

- [Pager: External Packages](https://docs.hak5.org/wifi-pineapple-pager/device/external-packages/)

The key is `/root` persists on a 4GB MMC storage partition. You can even point `opkg` to install there with `-d mmc`.

```bash
opkg update
opkg install -d mmc python3
```


### Installing Tailscale

There are a few ways to do this. This section aims to pull various options together to make choosing how you want to achieve this as simple as possible.

- [OpenWrt Docs: Tailscale](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start)
- [Tailscale: Static Bins Download](https://pkgs.tailscale.com/stable/#static)
- [Tailscale: Static Bins Usage](https://tailscale.com/download/linux/static)
- [Tailscale: Building Small Binaries (for OpenWrt)](https://tailscale.com/docs/how-to/set-up-small-tailscale?q=openwrt)
    - [GitHub Project (GuNanOvO/openwrt-tailscale) that automates this](https://github.com/GuNanOvO/openwrt-tailscale)


#### opkg

!!! warning "Work in Progress"

    This section is a work in progress and is not complete.

- [OpenWrt User Guide: Tailscale](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start)
- [github.com/openwrt/packages/net/tailscale](https://github.com/openwrt/packages/tree/master/net/tailscale)

TODO


#### Precompiled Binaries

!!! tip "Community Payloads"

    A set of community payloads already exist to install, configure, and run Tailscale on the Pager under [library/user/remote_access](https://github.com/hak5/wifipineapplepager-payloads/tree/master/library/user/remote_access).

This option uses the [statically compiled Tailscale binaries](https://pkgs.tailscale.com/stable/#static) for generic mipsle Linux.

!!! tip "mipsle"

    `uname -m` or `cat /proc/cpuinfo` shows the Pager is MIPS 24KEc V5.5 (`mips`). This requires the [statically compiled Tailscale binary](https://pkgs.tailscale.com/stable/#static) for "mipsle" instead of just "mips". This was missed originally until building Tailscale from source. Additionally, checking things like `file /bin/bash` or `opkg print-architecture` will hint at the correct mips architecture.

    Trying the mipsle binary on the Pager confirms it works with `tailscale --help`. If you try running the plain "mips" binary, it will potentially crash the system or more likely print:

    ```bash
    ./tailscale --help
    bash: ./tailscale: cannot execute binary file: Exec format error
    ```

The following commands will connect your Pager to your Tailnet.

```bash
# Create a path under /root where tailscale will be installed and persist
mkdir /root/tailscale
cd /root
wget https://pkgs.tailscale.com/stable/tailscale_1.96.2_mipsle.tgz
wget https://pkgs.tailscale.com/stable/tailscale_1.96.2_mipsle.tgz.sha256
sha256sum tailscale_1.96.2_mipsle.tgz | grep $(cat tailscale_1.96.2_mipsle.tgz.sha256)
tar xzvf tailscale_1.96.2_mipsle.tgz --strip-components=1 -C tailscale
cd tailscale

# Create symlinks to /usr/bin/ and /usr/sbin for convenience
# These will need recreated after updating the Pager firmware
ln -sf /root/tailscale/tailscale /usr/bin/tailscale
ln -sf /root/tailscale/tailscaled /usr/sbin/tailscaled

# Proof of concept, to confirm you can connect to your Tailnet
tailscaled --state=tailscaled.state &
tailscale up --accept-dns=false --authkey <authkey>

```

To persist and configure Tailscale as a real service, we can create `tailscale.conf` and `tailscale.init` from the existing [github.com/openwrt/packages/net/tailscale/files](https://github.com/openwrt/packages/tree/master/net/tailscale/files) by modifying the paths to point to our persistent MMC storage on `/root/tailscale`.

=== "/etc/init.d/tailscale"

    The only change here is pointing the `state_file` to the Pager's MMC partition under `/root`.

    ```conf
    #!/bin/sh /etc/rc.common

    # https://github.com/openwrt/packages/blob/master/net/tailscale/files/tailscale.init

    # Copyright 2020 Google LLC.
    # Copyright (C) 2021 CZ.NIC z.s.p.o. (https://www.nic.cz/)
    # SPDX-License-Identifier: Apache-2.0

    USE_PROCD=1
    START=80

    export TS_NO_LOGS_NO_SUPPORT=true

    start_service() {
        local state_file
        local port
        local std_err std_out

        config_load tailscale
        config_get_bool std_out "settings" log_stdout 1
        config_get_bool std_err "settings" log_stderr 1
        config_get port "settings" port 41641
        # Point to the Pager's MMC partition under /root
        config_get state_file "settings" state_file /root/tailscale/tailscaled.state
        config_get fw_mode "settings" fw_mode nftables

        /usr/sbin/tailscaled --cleanup

        procd_open_instance
        procd_set_param command /usr/sbin/tailscaled

        # Starting with v1.48.1 ENV variable is required to enable use of iptables / nftables.
        # Use nftables by default - can be changed to 'iptables' in tailscale config
        procd_set_param env TS_DEBUG_FIREWALL_MODE="$fw_mode"

        # Disable logging to log.tailscale.com
        procd_append_param env TS_NO_LOGS_NO_SUPPORT=true

        # Set the port to listen on for incoming VPN packets.
        # Remote nodes will automatically be informed about the new port number,
        # but you might want to configure this in order to set external firewall
        # settings.
        procd_append_param command --port "$port"
        procd_append_param command --state "$state_file"

        procd_set_param respawn
        procd_set_param stdout "$std_out"
        procd_set_param stderr "$std_err"

        procd_close_instance
    }

    stop_service() {
        /usr/sbin/tailscaled --cleanup
    }
    ```

=== "/etc/config/tailscale"

    The only change here is pointing the `state_file` to the Pager's MMC partition under `/root`.

    ```conf
    # https://github.com/openwrt/packages/blob/master/net/tailscale/files/tailscale.conf
    config settings 'settings'
        option log_stderr '1'
        option log_stdout '1'
        option port '41641'
        # Point to the Pager's MMC partition under /root
        option state_file '/root/tailscale/tailscaled.state'
        # default to using nftables - change below to 'iptables' if still using iptables
        option fw_mode 'nftables'
    ```

Finally, allowing management via Tailscale in the [firewall rules](https://docs.hak5.org/wifi-pineapple-pager/connecting/firewall/) will tie everything together.

=== "/etc/firewall.d/admin"

    This script is called by the `hak5admin` block at the top of the `/etc/config/firewall` rules.

    We're just adding additional "bridge" and "ip" rules for the `tailscale0` interface by mimicking what's already there. `tailscale0` will be referenced as `0x5` from `mark set 0x5`.

    Here's the full script:

    ```bash
    #!/bin/bash

    nft delete table bridge ifdetect

    nft add table bridge ifdetect
    nft add chain bridge ifdetect prerouting_mangle '{type filter hook prerouting priority -150; }'

    nft add rule bridge ifdetect prerouting_mangle iifname "eth0" counter meta mark set 0x1
    nft add rule bridge ifdetect prerouting_mangle iifname "wlan0mgmt" counter meta mark set 0x3

    # group arbitrary external "client" interfaces together
    nft add rule bridge ifdetect prerouting_mangle iifname "wlan0cli" counter meta mark set 0x4
    nft add rule bridge ifdetect prerouting_mangle iifname "wwan*" counter meta mark set 0x4
    nft add rule bridge ifdetect prerouting_mangle iifname "eth1" counter meta mark set 0x4
    nft add rule bridge ifdetect prerouting_mangle iifname "tailscale0" counter meta mark set 0x5

    nft delete table ip ifdetect
    nft add table ip ifdetect
    nft add chain ip ifdetect prerouting_mangle '{type filter hook prerouting priority -150; }'
    nft add rule ip ifdetect prerouting_mangle iifname "eth0" counter meta mark set 0x1
    nft add rule ip ifdetect prerouting_mangle iifname "wlan0mgmt" counter meta mark set 0x3

    nft add rule ip ifdetect prerouting_mangle iifname "wlan0cli" counter meta mark set 0x4
    nft add rule ip ifdetect prerouting_mangle iifname "wwan*" counter meta mark set 0x4
    nft add rule ip ifdetect prerouting_mangle iifname "eth1" counter meta mark set 0x4
    nft add rule ip ifdetect prerouting_mangle iifname "tailscale0" counter meta mark set 0x5

    ```

=== "/etc/config/firewall"

    Add these two blocks just above the `Reject-Admin` / `Reject-SSH` blocks. You don't need to edit the "wan" zone because this is tied together using the option mark of `0x5` just like the USB-C and management network blocks.

    Do not include a `hak5ver` line, this is [an internal versioning option for the Pager](https://docs.hak5.org/wifi-pineapple-pager/connecting/firewall/#turning-off-the-firewall) that we can safely leave out.

    ```conf
    config rule
            option name             'Allow-Admin-Tailscale'
            option src              'wan'
            option mark             '0x5'
            option proto            'tcp'
            option dest_port        '1471'
            option target           'ACCEPT'

    config rule
            option name             'Allow-SSH-Tailscale'
            option src              'wan'
            option mark             '0x5'
            option proto            'tcp'
            option dest_port        '22'
            option target           'ACCEPT'
    ```

Enable and start the service:

```bash
chmod +x /etc/init.d/tailscale
/etc/init.d/tailscale enable
/etc/init.d/tailscale start
tailscale up --accept-dns=false --authkey <key>
```

Reload the firewall rules:

```bash
fw4 restart
```

With at least two Tailscale ACL tags, one for the Pineapple Pager, and one for your management device, you can configure access to the machine via your Tailnet over 22/tcp and 1471/tcp for both SSH and web-based access.

Once all of these steps are done you can either `ssh` or browse to your pager using its Magic DNS name.


#### Building from Source

!!! example "Cross Compiling in Go"

    Three environment variables allow you to easily cross-compile in go: `GOOS`, `GOARCH`, and `CG0_ENABLED`. Unique to mips is the `GOMIPS` env variable, allowing us to specify either `softfloat` or `hardfloat`.

    See the [go environment variable options here when building go from source](https://go.dev/doc/install/source#environment), for reference (we don't need to build go itself from source). `go tool dist list` enumerates all valid OS/ARCH combos you can pass to go for cross-compiling.

    | ENV | Description |
    | --- | --- |
    | `GOOS` | The target os (e.g. linux) |
    | `GOARCH` | The target architecture (e.g. mipsle) |
    | `CGO_ENABLED` | [Enabled for local builds, disabled for cross-compiling](https://pkg.go.dev/cmd/cgo), used in [`build_dist.sh`](https://github.com/tailscale/tailscale/blob/931fe5658610e8ba49015fd291aefcbe39f40975/build_dist.sh#L19) |
    | `GOMIPS` | Set to "softfloat" to use soft floating point. ("hardfloat" is the default) |

The following commands build Tailscale from source for mipsle.

```bash
# Install go
cd ~/Downloads
wget https://go.dev/dl/go1.26.1.linux-amd64.tar.gz
sha256sum go1.26.1.linux-amd64.tar.gz | grep -i 031f088e5d955bab8657ede27ad4e3bc5b7c1ba281f05f245bcc304f327c987a
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.26.1.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version

# Clone Tailscale, and build a single static bin for mipsle
mkdir ~/src && cd ~/src
git clone git@github.com:tailscale/tailscale.git
cd ~/src/tailscale
GOOS=linux GOARCH=mipsle GOMIPS=softfloat CGO_ENABLED=0 \
  ./build_dist.sh \
  --extra-small \
  -o tailscale.combined \
  ./cmd/tailscaled

# On the Pager, create /root/tailscale
mkdir /root/tailscale

# Move the binary to the Pager
scp tailscale.combined root@<pager-ip>:/root/tailscale/tailscale.combined

# On the Pager, create symlinks to the combined bin and run Tailscale
ln -sf /root/tailscale/tailscale.combined /usr/bin/tailscale
ln -sf /root/tailscale/tailscale.combined /usr/sbin/tailscaled
chmod +x /root/tailscale/tailscale.combined
tailscaled --state=tailscaled.state &
tailscale up --accept-dns=false --authkey <authkey>

```

Here you can repeat the same steps in the [precompiled binaries](#precompiled-binaries) section to configure Tailscale with the right firewall rules, and to run as a system service.
