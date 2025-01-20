---
draft: false
date:
  created: 2024-12-08
  updated: 2024-12-15
categories:
  - kvm
  - qemu
  - how-to
  - home-lab
  - virtualization
  - infrastructure-as-code
  - packer
  - virt-manager
  - libvirt
---

# KVM (Kernel-based Virtual Machine)

KVM is a type 1 hypervisor technology built into the Linux kernel, using components like QEMU, libvirt, and virt-manager to orchestrate virtual machines. This reference aims to cover everything you'd need to have a basic understanding of KVM and QEMU, how to use `virt-manager`, networking configuration options, and the various components such as SPICE.

!!! abstract "[What is KVM?](https://www.redhat.com/en/topics/virtualization/what-is-KVM)"

    - `virt-manager` acts as one possible GUI / CLI frontend to managing VM's.
    - `qemu` is the virtualization / emulation technology that technologies like virt-manager call to run VM's.
    - QEMU can be used via the CLI on its own.

    KVM relates to QEMU when you run VM's with hardware acceleration rather than pure software emulation, KVM provides the hardware translation and acceleration component. This is the difference when starting VM's with `kvm` instead of `qemu-system-x86_64`.

    `kvm` is the equivalent of running `qemu-system-x86_64 -machine accel=kvm:tcg ` (see `man kvm`).

    `libvirt` is an API, daemon, and management tool that ties all of the components you want to use together, and is used by KVM, Xen, VMware ESXi, and QEMU when performing virtualization tasks.

<!-- more -->


## Virt-Manager

*Virt-Manager is a GUI application. Think of Virt-Manager as an equivlent to Hyper-V, VMware Workstation, or VirtualBox.*


### Install

Install the utilties and add your user(s) you want to access the VM's to the libvirtd group.

```bash
sudo apt install -y virt-manager
sudo usermod -aG libvirtd $USER
```


### Paths

| Path | Description |
| --- | ---
| `/etc/libvirt/qemu` | VM XML configs |
| `/var/lib/libvirt/images` | VM disks |
| `/var/lib/libvirt/qemu/nvram` | EFI Vars |
| `/var/lib/libvirt/qemu/snapshot/<vm-name>/<snapshot-name>.xml` | Snapshot XML configs |
| `/etc/libvirt/qemu/networks/` | Network configurations |
| `/etc/libvirt/nwfilter/` | Network filtering configurations |


### Commands

`virsh` allows you to automate and control virtual machine behavior from the CLI.

| Command | Description |
| --- | --- |
| `virsh list --all` | List all VMs |
| `virsh [start|stop|shutdown|reset] <vm-name>` | Control VM state |


#### Snapshot Commands

!!! note "Snapshot Requirements"

    VMs must have a disk image associated with them to take and modify snapshots.*

| Command | Description |
| --- | --- |
| `virsh snapshot-list <vm-name>` | List all snapshots of `<vm-name>` |
| `virsh snapshot-create-as <vm-name> <snapshot-name> --description "<description>"` | Take a snapshot |
| `virsh snapshot-revert <vm-name> <snapshot-name>` | Revert to `<snapshot-name>` |
| `virsh snapshot-delete <vm-name> <snapshot-name>` | Delete a snapshot |


#### Networking Commands

| Command | Description |
| --- | --- |
| `virsh net-list --all` | List all available `<network-name>`s |
| `virsh net-dhcp-leases <network-name>` | List all current DHCP leases on `<network-name>` if it's managed via virt-manager / dnsmasq |
| `virsh domiflist <vm-name>` | List all network interfaces attached to `<vm-name>` |
| `virsh domifaddr <vm-name> [--source agent|arp|lease]` | Get IP information from `<vm-name>`, optionally using one of three methods |
| `virsh net-dumpxml <network>` | Output the virtual network information as an XML dump to stdout. |
| `virsh net-create <network> [--validate]` | Create a transient (temporary) virtual network from an XML file and instantiate (start) the network. |
| `virsh net-define <network> [--validate]` | Define an inactive persistent virtual network or modify an existing persistent one from the XML file. |
| `virsh net-undefine <network>` | Undefine the configuration for a persistent network. If the network is active, make it transient. |
| `virsh net-edit <network>` | Edit the XML configuration file for a network. |
| `virsh net-list [--all]` | Returns the list of active networks. |
| `virsh net-start <network>` | Start a (previously defined) inactive network. |
| `virsh net-destroy <network>` | Destroy (stop) a given transient or persistent virtual network specified by its name or UUID. This takes effect immediately. |
| `virsh nwfilter-define <filter> [--validate]` | Define or update a network filter from an XML file |
| `virsh nwfilter-undefine <filter>` | Undefine a network filter |
| `virsh nwfilter-dumpxml <filter>` | Network filter information in XML |
| `virsh nwfilter-list` | List network filters |
| `virsh nwfilter-edit <filter>` | Edit XML configuration for a network filter |


### Networking

!!! warning "Tailscale and Exit Nodes"

	If you use tailscale and force all traffic out of an exit node, this may break routing on your libvirtd instance. Simply unset the exit node to verify this.

    ```bash
    sudo tailscale set --exit-node=''
    ```

libvirtd VM's appear under `ip a` as `vnetX`, which you can observe under `kern.log` when a VM comes online.


#### Internal Networking

These can be created under Edit > Connection Details > Virtual Networks

!!! tip "pfSense with Virtual Networks"

    The examples below are for a pfSense router VM to have it's own internal LAN and OPT interfaces that are purely virutal, and cannot be seen by or talk to the host's networking interfaces.

- Choose "Add"
- Name: `pfsense_lan` (`pfsense_opt1`, `pfsense_opt2`...)
- Mode: Isolated
- Enable IPv4
	- Disable DHCP
	- Delete all the other IPv4 information (this can be provisioned by pfSense)
	- DNS can be "Custom: `<blank>`"
- *IPv6 must remain disabled for now, it requires a hard-coded address*
- Once created you should see "Autostart On Boot" checked as well

Ultimately your pfSense VM is routing via NAT through the `default` virt-manager network, and has its own, virtual (isolated) pfsense_lan, pfsense_opt1, and pfsense_opt2, etc. addresses available to it. To connect a VM to pfSense, attach it to any of the virtual (isolated) network interfaces.


#### Troubleshooting

This example deletes the generated xml config under `/etc/libvirt/qemu`, and uses the built in template from `/usr/share/libvirt` to reprovision a totally new default NAT network.

Once you've powered down or paused all VMs, quit `virt-manager` and its system tray icon. The entire bash block below is safe to copy and paste.

```bash
sudo systemctl restart libvirtd
sudo virsh net-list --all
sudo virsh net-undefine default
sudo virsh net-destroy default
sudo rm -f /etc/libvirt/qemu/networks/default.xml
sudo virsh net-define /usr/share/libvirt/networks/default.xml
sudo virsh net-start default
sudo virsh net-autostart default
sudo virsh net-list --all
systemctl status libvirtd
```


#### Filtering Network Traffic

!!! abstract "nwfilter vs Host Firewall"

    Instead of using a host utility like `iptables` or `ufw` to manage virtual machine network filtering, it's better to use libvirt's built-in `nwfilter` directly to ensure rules are being applied to machines correctly. This is most useful when VM's are either bridged, or NAT'd without a router / firewall in front of them.

- [libvirt.org/firewall](https://libvirt.org/firewall.html#the-network-filter-driver)
- [libvirt.org/formatnwfilter](https://libvirt.org/formatnwfilter.html)
- [`nwfilter` CLI](https://libvirt.org/formatnwfilter.html#command-line-tools)
- [RedHat: Apply Network Filtering](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-virtual_networking-applying_network_filtering)

There are a number of pre-built example rules. According to the documentation:

!!! quote ""

    They are all stored in `/etc/libvirt/nwfilter`, but don't edit the files there directly. Use `virsh nwfilter-define` to update them. This ensures the guests have their iptables/ebtables rules recreated.

To apply nwfilter rules, we need to:

- Create the XML file, defaults are under `/etc/libvirt/nwfilter/<rule-name>.xml`
- Define the new file as a rule with `virsh nwfilter-define`
- Add a `<filterref/>` reference to this rule in a VM's network configuration XML
- Verify the rules are working

---

##### Example: Block Tailnet Access

This is useful if for example your host has access to Tailnet resources that an untrusted guest VM should not be able to reach.

Create a custom rule by referencing [the custom rule examples](https://libvirt.org/formatnwfilter.html#writing-your-own-filters). We can build one that uses the existing `clean-traffic` filter as a base reference, and adds a "drop" rule to all outbound traffic destined for `100.64.0.0/10` to prevent VM's from accessing Tailnet endpoints they shouldn't be able to see.

```bash
sudo nano /etc/libvirt/nwfilter/no-tailnet-traffic.xml
```

!!! tip "Generate a UUID"

    In Linux you can create a UUID using a proc path. Credit to [rpinz](https://github.com/rpinz) for this:

    ```bash
    cat /proc/sys/kernel/random/uuid
    dc48344d-99fa-42c2-8db8-4eeaea0116f6
    ```

The priority values are left as the defaults described in the documentation.

```xml
<filter name='no-tailnet-traffic' chain='ipv4' priority='-700'>
  <uuid>dc48344d-99fa-42c2-8db8-4eeaea0116f6</uuid>
  <!-- Reference the clean traffic filter to prevent
       MAC, IP and ARP spoofing. By not providing
       and IP address parameter, libvirt will detect the
       IP address the VM is using.

       https://libvirt.org/formatnwfilter.html#writing-your-own-filters
  -->
  <filterref filter='clean-traffic'/>

  <!-- Drop all outbound traffic to the CGNAT ranges, or
       100.64.0.0/10, which is also used in Tailnets.
       This prevents VM's from talking to endpoints that
       hosts can reach over Tailnets.

       By referencing the clean-traffic filter above, the
       rule is dynamically obtaining each VM's $IP address
       when this filter applies. That's why a srcipaddr and
       srcipmask aren't used here.

       Since CGNAT ranges are exclusively IPv4, this only
       applies to the <ip/> protocol in libvirt's nwfilter.

       https://libvirt.org/formatnwfilter.html#reserved-variables
  -->
  <rule action='drop' direction='out' priority='500'>
    <ip dstipaddr='100.64.0.0' dstipmask='255.192.0.0'/>
  </rule>
</filter>
```

!!! tip "Nested Filter References"

    As you can probably tell from the example, you can build rules by referencing other rules, that reference other rules, and so on.

Save it, then define it as a new `nwfilter` rule. `--validate` ensures there are no syntax issues.

```bash
sudo virsh nwfilter-define /etc/libvirt/nwfilter/no-tailnet-traffic.xml --validate
```

Add the following into the `<interface/>` block within a guest VM's NIC XML data.

```xml
  <filterref filter='no-tailnet-traffic'/>
```

Reboot the VM if necessary and verify that the rules are working.

!!! note "Adding Filters to Networks"

    Initially adding a `<filterref/>` block to a network such as the `default` network was tried. It appears these rules must be applied per-machine. Until a way is found to apply this globally per-network, assume this is true.


## SPICE

Some machines will install the `spice-vdagent` by default, however others (Kali) may require you install it after you install the guest.

```bash
sudo apt update; sudo apt install -y spice-vdagent
```

Sometimes the SPICE connection will go completely blank after some time. This could either be from **Power Saver settings on the guest**, or an issue with the SPICE agent / server.

!!! tip "Blank or frozen SPICE connection on guest sleep"

    You should configure QEMU guests to never fall asleep or go blank if this is happening frequently. Alternatively pause the guest when it's not in use.

!!! tip "Blank or frozen SPICE connection on host wake"

    Simply leave QEMU VM's running when putting the host to sleep. They'll continue to be running and operating as normal when the host wakes again.

Restart the SPICE server on the host for that VM:

```bash
# TO DO
```


### Clipboard

If you're concerned about guests being able to steal the clipboard data from your host during use, you can temporarily pause the guest while you copy / paste credentials from your clipboard. Once a guest is paused it will no longer have access to the clipboard contents.

You can also of course just close the SPICE window and reopen it later.


### Windows Guests

Windows does not come with the drivers for libvirt or the spice-agent process installed. This means guest enhancements don't work out of the box.

- [SPICE Guest Tools (Windows, Latest)](https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe)
- [SPICE User Manual (See "Windows guest" section)](https://www.spice-space.org/spice-user-manual.html)
- [chef/bento KVM/QEMU Support for Windows Templates](https://github.com/chef/bento?tab=readme-ov-file#kvmqemu-support-for-windows)

If you follow the links through the Fedora project, eventually it points from [here](https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/index.html) to [here](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md). Basically, without a RedHat machine + subscription, you won't have access to signed drivers for Windows to load without allowing test drivers to load (not ideal if this VM will be battle-tested or exposed to threats).

!!! tip "Windows GUI Alternatives"

	Given the requirement for the unsigned drivers to leverage the SPICE connection, there are a few clever alternatives:

    - The new Windows App may offer some interesting solutions for networked or cloud machines on your Microsoft account
    - You could use remmina to RDP into the Windows guest (remmina can be installed as a snap or flatpak for security sandboxing the client)
    - You could SSH into Windows
    - In virtual networks where you'd need a jump host to connect from your host to the guest, you could RDP from a neighboring Linux guest in that subnet


#### RDP + Remmina

Enable RDP on the Windows Guest.

- Settings > System > Remote Desktop > Enabled Remote Desktop: "On"

Install Remmina on your host:

```bash
sudo snap install remmina
```

- Configure user, password, and server IP
- Set resolution to "Use initial window size" and after connecting for the first time, resize the window as needed, then reconnect once more
- Advanced > Quality > Set to "Best" (great for local VM's)
- Optionally disable Clipboard synchronization if you don't want the remote machine to "see" your clipboard


#### OpenSSH-Server

An alternative solution is installing [OpenSSH-Server on Windows](https://github.com/straysheep-dev/windows-configs/blob/main/Manage-OpenSSHServer.ps1).

!!! note "winget with the program"

    If installing winget on an evaluation copy of Windows Server, you will need to download the msixbundle file and license xml file from GitHub's release page. Otherwise you'll get a license error.

    See [winget-cli issue #700](<https://github.com/microsoft/winget-cli/issues/700#issuecomment-874084714>).

```powershell
# Download the msixbundle and xml license file from the latest releases.
# https://github.com/microsoft/winget-cli/releases
# Use Add-AppxProvisionedPackage to use the -LicensePath argument.
Add-AppxProvisionedPackage -Online -PackagePath .\Microsoft.DesktopAppInstaller_*.msixbundle -LicensePath .\*_License1.xml -Verbose
```

Install the latest PowerShell + OpenSSH Server, optionally Windows Terminal

```powershell
winget install --id Microsoft.PowerShell -e --source winget
winget install --id Microsoft.OpenSSH.Beta -e --source winget
winget install --id Microsoft.WindowsTerminal -e
```

Allow SSH through the firewall.

```powershell
New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22 > $null
```


## QEMU

QEMU is what virt-manager uses under the hood to run Virtal Machines, however you can use QEMU entirely on its own.

- [QEMU Introduction and Essential Commands](https://www.qemu.org/docs/master/system/introduction.html)
- [superuser: Forwarding Ports to QEMU Guest](https://superuser.com/questions/1706286/qemu-configuration-to-host-a-linux-with-ssh-and-http-on-windows)


### Install

To install QEMU on Debian-based systems:

```bash
my_arch=$(uname -m | cut -d '_' -f 1)
sudo apt install -y qemu-system-"$my_arch"
```

!!! note "Windows and macOS Support"

    See the documentation for Windows and macOS instructions.


### Usage

There may be reasons why you want to create and launch a VM entirely via the CLI interface with KVM / QEMU.

Use these steps to do so.

!!! note "Credit & Thanks"

    Much of this section was learned from the steps under [Ubuntu: Autoinstall Quick Start](https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/autoinstall-quickstart.html#autoinstall-quick-start). This guide expands upon those, to get more options working like EFI boot and SecureBoot.

Create a virtual disk.

```bash
truncate -s 10G ubuntu.img
```

First launch, walk through the installer.

```bash
kvm -no-reboot -m 2048 \
    -drive file=ubuntu.img,format=raw,cache=none,if=virtio \
    -cdrom ~/path/to/ubuntu-<version-number>-live-server-amd64.iso

```

Second launch, start the installed machine.

```bash
kvm -no-reboot -m 2048 \
    -drive file=ubuntu.img,format=raw,cache=none,if=virtio
```

#### Networking

You can choose between numerous networking interface devices. List all available to you with the following. Note that some OS's are only compatible with certain devices.

Generally, `e1000` seems to work across any guest, while `virtio-net-pci` is the most performant, but may only work on unix-like guests.

```bash
qemu-system-x86_64 -nic model=help
Supported NIC models:
e1000
e1000-82544gc
e1000-82545em
e1000e
i82550
i82551
i82557a
i82557b
i82557c
i82558a
i82558b
i82559a
i82559b
i82559c
i82559er
i82562
i82801
ne2k_pci
pcnet
pvrdma
rtl8139
tulip
virtio-net-pci
virtio-net-pci-non-transitional
virtio-net-pci-transitional
vmxnet3
```


#### SecureBoot and QEMU

!!! tip "Required Settings"

	You'll need to specify the same settings you'd use in virt-manager to configure a VM for SecureBoot.

    - [superuser/qemu-kvm-uefi-secure-boot-doesnt-work](https://superuser.com/questions/1703377/qemu-kvm-uefi-secure-boot-doesnt-work)
    - [debian.org/SecureBoot/VirtualMachine](https://wiki.debian.org/SecureBoot/VirtualMachine)

Practically what you need are 3 items:

- The Q35 chipset
- The EFI firmware ROM (read-only)
- The EFI variables (writable)

The EFI variables file can be copied to a place for use with that VM from `/usr/share/OVMF/OVMF_VARS_*`.

Which of those files you need depends on which EFI ROM you used to provision the VM.

- `-machine q35,smm=on` handles the chipset, and supports SecureBoot
- `accel=kvm:hvf:whpx:tcg` tells QEMU what accelerator to use in that order, depending on what's available
- `-drive if=pflash,format=raw,readonly=on,file=/usr/share/OVMF/OVMF_CODE_4M.secboot.fd` points to the EFI ROM
- `-drive if=pflash,format=raw,file=efivars.fd` points to a local, writable, copy of the EFI variables.

```bash
cd /path/to/ubuntu-2204.qcow2 && \
kvm -no-reboot -m 4096 -smp 4 \
    -cpu host \
    -machine q35,smm=on,accel=kvm:hvf:whpx:tcg \
    -global driver=cfi.pflash01,property=secure,value=on \
    -device virtio-net-pci,netdev=net0 \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -object rng-random,filename=/dev/urandom,id=rng0 \
    -drive file=ubuntu-2204.qcow2,format=qcow2,if=virtio \
    -drive if=pflash,format=raw,readonly=on,file=/usr/share/OVMF/OVMF_CODE_4M.secboot.fd \
    -drive if=pflash,format=raw,file=efivars.fd \
    -vga virtio \
    -display gtk

```


#### Autoinstall

Autoinstall, cloud-init, and user-data together can be confusing to delineate. The following links (in order) do a good job of framing what autoinstall is, and how it uses cloud-init.

- [Subiquity autoinstaller uses cloud-init](https://cloudinit.readthedocs.io/en/latest/reference/faq.html#autoinstall)
- [Intro to Subiquity autoinstaller](https://canonical-subiquity.readthedocs-hosted.com/en/latest/intro-to-autoinstall.html)
- [Cloud-init and autoinstaller interaction](https://canonical-subiquity.readthedocs-hosted.com/en/latest/explanation/cloudinit-autoinstall-interaction.html#cloudinit-autoinstall-interaction)
- [Providing autoinstall](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/providing-autoinstall.html#providing-autoinstall)
- [The NoCloud datasource](https://docs.cloud-init.io/en/latest/reference/datasources/nocloud.html#datasource-nocloud)
- [Cloud-config user-data formats](https://docs.cloud-init.io/en/latest/explanation/format.html#user-data-formats-cloud-config)
- [Subiquity autoinstaller user-data ✨](https://canonical-subiquity.readthedocs-hosted.com/en/latest/reference/autoinstall-reference.html#user-data)

**That final link (star'd) has the syntax reference you need for an autoinstall user-data file.**

!!! tip "Subiquity autoinstaller user-data?"

    This is the best way I can summarize this; `autoinstall:` is a "parent" to the `user-data:` file of a cloud-init cloud-config. Replace the user-data file with your `autoinstall:` data, which has `user-data:` embedded within it.

    In other words, you still have a user-data file. It's required for cloud-init to successfully provision a machine. However, all of your original user-data text now lives under a `user-data:` yaml block, along side other optional `autoinstall:` blocks.

    This can be incredibly confusing when trying to debug what's wrong with your config. More often than not, you're looking at the wrong examples.

Autoinstall has been available since 20.04.

- [Cloud-init QEMU Tutorial](https://cloudinit.readthedocs.io/en/latest/tutorial/qemu.html) (this section builds off of these steps)
- Ubuntu Server 20.04+ ([It's possible to use server to install `ubuntu-desktop`](https://github.com/canonical/autoinstall-desktop))
- Ubuntu Desktop 24.04+

The GitHub project referenced is specifically for using Ubuntu's Autoinstall provisioning method. There are reasons why you might want to try this method yourself and observe the process via the console.

Here are some additional copy-paste-ready snippets to help you replicate those steps that are more implied or specific to your environment, and aren't mentioned in that tutorial. This example installs Ubuntu 22.04 desktop, which requires the Ubuntu 22.04 server image to start with since Desktop doesn't have Autoinstall until 24.04.

!!! tip "How does this work with QEMU?"

    This example prefers serving the autoinstall data over the (localhost) network instead of using the `cloud-image-utils` package, which is exclusive to Ubuntu. This is so you can replicate the same steps on other OS's. You can create any directory for serving the autoinstall files, **as long as you point to them via `kvm`/`qemu` or `packer`**.

    QEMU will default to creating an ephemeral NAT network on `10.0.2.2/24` when you spin up the VM. When you bind a python web server to `0.0.0.0` or all interfaces, it will be reachable on `10.0.2.2:<port>`.

    The syntax for running `kvm` / `qemu` interactively are basically the same when doing this through `packer`'s `boot_command`.

    - QEMU: `'autoinstall ds=nocloud-net;s=http://_gateway:3000/'`
    - Packer: `" autoinstall ds=nocloud-net\\;s=http://10.0.2.2:{{ .HTTPPort }}/",`

First, make a directory you'll serve the autoinstall files from.

```bash
mkdir -p ~/Public/http; \
cd ~/Public/http; \
touch meta-data; \
wget https://github.com/canonical/autoinstall-desktop/raw/refs/heads/main/autoinstall.yaml -O user-data
# This autoinstall.yaml file is short enough to manually inspect and modify
```

Modify and save your `user-data` content, then serve it.

!!! note "What goes in `user-data`?"

    This is a complete reference to writing the user-data file.

    - [All cloud-config examples](https://cloudinit.readthedocs.io/en/latest/reference/examples.html)
    - [Examples library](https://cloudinit.readthedocs.io/en/latest/reference/examples_library.html#examples-library)

    The example here is a very minimal config. You'll eventually want to modify this to fit your needs.

    ```yaml
    #cloud-config
    autoinstall:
        # version is an Autoinstall required field.
        version: 1

        # User creation can occur in one of 3 ways:
        # 1. Create a user using this `identity` section.
        # 2. Create users as documented in cloud-init inside the user-data section,
        #    which means this single-user identity section may be removed.
        # 3. Prompt for user configuration on first boot.  Remove this identity
        #    section and see the "Installation without a default user" section.
        identity:
            realname: 'ubuntu'
            username: ubuntu
            # A password hash is needed. `mkpasswd --method=SHA-512` can help.
            # mkpasswd can be found in the package 'whois'
            # The hash below represents a password of just "ubuntu" without the quotes
            password: '$6$IEXNBzIlhRd2lR2e$ygPi2XMmQtKDKm3Z40xKUtEc00HIL9rUzJx7Hx9Xe.wqpiSIvHPC3RGEgvy7jfrydTNdlVsoHmUCShzQQtl5B0'
            hostname: ubuntu-2204

        # Controls password authentication with the SSH daemon; the default here
        # prevents logging into SSH with a password. Changing this is a security risk
        # and you should at the very least ensure a different default password is
        # specified above
        ssh_pwauth: true
    ```

!!! tip "Firewall rules"

	If you have default-deny inbound rules in place on your firewall, you'll need to make an allowance for the `10.0.2.0/24` subnet, or whatever static IP you assign the VM, to be able to reach `10.0.2.2` on an optionally specified port to obtain the autoinstall files.

Ensure your inbound firewall rules are scoped to the default QEMU network, so only the VM can talk to your python web server. Of course you'll need to adjust the QEMU default network if your LAN overlaps with 10.0.2.0/24.

```bash
your_ip=''
www_port=''
qemu_subnet='10.0.2.0/24' # Common default subnet used by KVM/QEMU
sudo ufw allow in to "$your_ip" proto tcp port "$www_port" from "$qemu_subnet" comment 'qemu autoinstall'
```

!!! tip "Serving autoinstall files"

    Packer will automatically serve those files to itself using a webserver tied to the packer gateway IP.

    If you use QEMU on its own, you will have to spin up a python3 webserver to serve the autoinstall files.

    ```bash
    cd ~/Public/http
    python3 -m http.server 3000
    ```


Create a mount point to reference the vmlinuz and initrd files within the ISO. This only appears to be necessary in some cases, when running QEMU itself outside of Packer.

```bash
sudo mkdir /mnt/ubuntu_iso
sudo mount -r ~/iso/ubuntu-22.04/ubuntu-22.04.4-desktop-amd64.iso /mnt/ubuntu_iso
```

Create a virtual disk.

```bash
cd ~/src/packer_tutorial
truncate -s 10G ubuntu.img
```

Now boot the ISO with KVM, and direct it to use the proper options for Autoinstall.

```bash
cd ~/src/packer_tutorial

kvm -no-reboot -m 4096 \
  -drive file=ubuntu.img,format=raw,cache=none,if=virtio \
  -cdrom ~/iso/ubuntu-22.04/ubuntu-22.04.4-live-server-amd64.iso \
  -kernel /mnt/ubuntu_iso/casper/vmlinuz \
  -initrd /mnt/ubuntu_iso/casper/initrd \
  -append 'autoinstall ds=nocloud-net;s=http://_gateway:3000/' \
  -boot d
```

!!! tip "vmlinuz and initrd files"

	In older versions of Ubuntu, where you'd use preseed.cfg instead of Autoinstall, the vmlinuz and initrd files may be named differently (e.g. initrd.gz) and in different directories.

    Do a `find` to see where they are and change those paths accordingly.

This will start QEMU, and kick off the Autoinstall process for the live server 22.04 image, which ultimately installs all of the desktop tools and environment for 22.04. This is a way around 22.04 having no Autoinstall capability. The entire process can be observed in the QEMU console window.

When the installation completes the system will shutdown and you can boot into the new install.

```bash
kvm -no-reboot -m 4096 \
  -drive file=ubuntu.img,format=raw,cache=none,if=virtio

```

Unless you configure persistent networking, it's not always possible to route from your host into the QEMU subnet. If you want to forward localhost:2222 into the guest:22, you can do this:

```bash
kvm -no-reboot -m 4096 \
  -drive file=ubuntu.img,format=raw,cache=none,if=virtio \
  -netdev user,id=net0,hostfwd=tcp::2222-:22 \
  -device e1000,netdev=net0
  # Here e1000 is used as the network card instead of virtio
```

!!! warning "Gave up waiting for root file system device"

	This warning appears to happen anytime a QEMU VM is installed via Autoinstall, where initrd and vmlinuz need specified from the mounted ISO.

    ```
    Gave up waiting for root file system device. Common problems:
    - Boot args (cat /proc/cmdline)
    	- Check rootdelay= (did the system wait long enough?)
    - Missing modules (cat /proc/modules: ls /dev)P <unique-string>
    ALERT! /dev/mapper/ubuntu--vg--ubuntu--lv does not exist. Dropping to a shell!

    BusyBox v1.30.1 (Ubuntu 1:1.30.1-7ubuntu3.1) built-in shell (ash)
    Enter 'help' for a list of built-in commands.

    (initramfs) _
    ```

    It's unclear how to resolve this, or why it's happening. The ubuntu.img disk can be chrooted into from the host, so this needs investigated further.


### Security

Under the hood, virt-manager and libvirt are run by QEMU / KVM.

!!! note "Security Overview"

    [QEMU considers the following to be *untrusted*](https://www.qemu.org/docs/master/system/security.html):

    - Guest
    - User-facing interfaces (e.g. VNC, SPICE, WebSocket)
    - Network protocols (e.g. NBD, live migration)
    - User-supplied files (e.g. disk images, kernels, device trees)
    - Passthrough devices (e.g. PCI, USB)

    It's recommended to read through that entire page if you plan to leverage QEMU / KVM or similar technologies like Proxmox to run untrusted code.


### Backup and Restore

!!! warning "*Section under construction*"

    This section is informational and untested. Check back later for more information.

- [Reddit: How do you make a backup of virt-manager?](https://www.reddit.com/r/VFIO/comments/l4mpph/how_do_you_make_a_full_backup_of_a_virtmanager/)
- [github: virtnbdbackup](https://github.com/abbbi/virtnbdbackup)

- `/var/lib/libvirt/images` VM disks
- `/etc/libvirt/qemu`: VM xml configs

```bash
sudo rsync -arv --delete --safe-links /var/lib/libvirt/images/ /media/user/external/Virtual-Machines/libvirt/var_lib_libvirt_images/
sudo rsync -arv --delete --safe-links /etc/libvirt/qemu/ /media/user/external/Virtual-Machines/libvirt/etc_libvirt_qemu/
```