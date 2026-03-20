---
title: "Pineapple Pager"
icon: material/fruit-pineapple
draft: false
#date:
#  created: 2026-03-19
#  updated: 2026-03-19
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

## Third-party Packages

The Hak5 docs cover where and how you can and should install packages, via `opkg` or otherwise.

- [Pager: External Packages](https://docs.hak5.org/wifi-pineapple-pager/device/external-packages/)

The key is `/root` persists on a 4GB MMC storage partition. You can even point `opkg` to install there with `-d mmc`.

```bash
opkg update
opkg install -d mmc python3
```


### Installing Tailscale

!!! warning "Work in Progress"

    This section is a work in progress and is not complete.

There are a few ways to do this, and there's a decent amount of documentation out there for a couple options. This section aims to pull those all together to be as simple as possible.

- [OpenWrt Docs: Tailscale](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start)
- [Tailscale: Static Bins Download](https://pkgs.tailscale.com/stable/#static)
- [Tailscale: Static Bins Usage](https://tailscale.com/download/linux/static)
- [Tailscale: Building Small Binaries (for OpenWrt)](https://tailscale.com/docs/how-to/set-up-small-tailscale?q=openwrt)
    - [GitHub Project (GuNanOvO/openwrt-tailscale) that automates this](https://github.com/GuNanOvO/openwrt-tailscale)

**Installing via opkg**

- [OpenWrt User Guide: Tailscale](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start)
- [github.com/openwrt/packages/net/tailscale](https://github.com/openwrt/packages/tree/master/net/tailscale)

TODO

**Precompiled Binaries**

A `uname -m` or `cat /proc/cpuinfo` shows the Pager is MIPS 24KEc V5.5 (`mips`). This is requires the [statically compiled Tailscale binary](https://pkgs.tailscale.com/stable/#static) for just "mips".

```bash
mkdir /root/tailscale/
cd /root/tailscale
wget https://pkgs.tailscale.com/stable/tailscale_1.96.2_mips.tgz
wget https://pkgs.tailscale.com/stable/tailscale_1.96.2_mips.tgz.sha256
sha256sum tailscale_1.96.2_mips.tgz | grep $(cat tailscale_1.96.2_mips.tgz.sha256)
tar xzvf tailscale_1.96.2_mips.tgz
cd tailscale_1.96.2_mips

# This will crash the pager using the precompiled binary above, needs investigated
tailscaled --state=tailscaled.state
tailscale up --accept-dns=false --authkey <authkey>

```

**Building from Source**

TODO
