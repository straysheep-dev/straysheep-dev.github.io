---
title: "Proxmox"
icon: simple/proxmox
draft: false
#date:
#  created: 2024-08-13
#  updated: 2025-12-14
categories:
  - how-to
  - automation
  - ansible
  - virtualization
  - home-lab
  - proxmox
  - server
  - administration
  - qemu
---

# :simple-proxmox: Proxmox

Get started with Proxmox. Installation considerations, networking, disk encryption options, and migrating existing virtual machines.

Everything's presented in a useful order to follow if you're used to Hyper-V, VMware Workstation, or VirtualBox and want to jump in by moving to a Proxmox server.

What's most interesting about Proxmox is the Web UI gives you full console and GUI (yes, GUI desktop) access to your VM's through either noVNC or SPICE, *even via a smart phone*.

<!-- more -->

:lucide-book-copy: [Proxmox VE Documentation](https://pve.proxmox.com/pve-docs)

!!! note "AI Usage"

    The following models and tools were used for drafts, examples, or research:

    - Claude (`claude-sonnet-4.6`) via web
    - ChatGPT (`gpt-4o`) via web

    Posts and documentation discovered organically or compiled by the model, were reviewed and are linked as resources.

---

## :lucide-download: Download

!!! tip "Verifying Signatures"

    The ISO URL points to an `/iso/` folder on the Proxmox webiste. Browsing this manually reveals the following files:

    - <https://enterprise.proxmox.com/iso/SHA256SUMS.txt>
    - <https://enterprise.proxmox.com/iso/SHA256SUMS.asc>

    You can use these along with the following public key to verify the ISO's integrity.

    ```bash
    gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys 'F4E136C67CDCE41AE6DE6FC81140AF8F639E0C39'

    # list keys
    pub   rsa4096/0x1140AF8F639E0C39 2022-11-27 [SC] [expires: 2032-11-24]
        Key fingerprint = F4E1 36C6 7CDC E41A E6DE  6FC8 1140 AF8F 639E 0C39
    uid                   [ unknown] Proxmox Bookworm Release Key <proxmox-release@proxmox.com>
    ```


## :lucide-hard-drive-download: Install

Walk through the graphical or terminal installer. This is straight forward if you're used to installing hypervisor software.


**Email**

You'll need to choose an email address to send alerts to, this can be `root@localhost` or a real email.


## :lucide-key-round: Access

Proxmox listens on 22/tcp (SSH) and 8006/tcp (Web UI).

You can browse directly to your Proxmox machine's `https://<IP>:8006` or use SSH. SSH allows you to do local port forwarding which will help you access the web interface from a jump host.


!!! tip "Jump Hosts"

    **Tailscale**

    You can install Tailscale on proxmox, since it's Debian under the hood. This is a great way to isolate and access the management interface from authorized tailnet nodes.


    **From WSL to Jump Host**

    This example has a Yubikey connected to WSL on Windows:

    - The Yubikey is required to SSH into Proxmox
    - SSH Windows:8006 > WSL:8006
    - SSH WSL:8006 (with Yubikey) > JumpHost:SSH-PORT > Proxmox:8006
    - Now point your Windows browser to https://127.0.0.1:8006
    - It will forward to WSL which uses a jump host to forward again to proxmox's localhost:8006

    ```bash
    # On Windows Host
    $wsl_user = wsl.exe whoami
    $wsl_ipv4 = wsl.exe ip addr show eth0 | sls "(?:(?:192\.168|172\.16|10\.(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?))\.)(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?\.)(?:25[0-4]|2[0-4][0-9]|[01]?[0-9][0-9]?)" | ForEach-Object { $_.Matches.Value }
    PS> ssh -L 127.0.0.1:8006:127.0.0.1:8006 $wsl_user@$wsl_ipv4

    # On WSL
    JUMP_HOST_USER=''
    JUMP_HOST_IP=''
    JUMP_HOST_PORT=''
    PROXMOX_USER=''
    PROXMOX_IP=''
    $ ssh -L 127.0.0.1:8006:127.0.0.1:8006 -J $JUMP_HOST_USER@$JUMP_HOST_IP:$JUMP_HOST_PORT $PROXMOX_USER@$PROXMOX_IP
    ```


## :lucide-settings: Configure


### :lucide-clock-2: MFA

First, add 2FA/MFA to your root login. This can be a software token saved to your password manager or authenticator app, plus a hardware key using webauthn.

---

### :lucide-package: Package Repositories

Next, if you have a valid enterprise subscription key, you're good to go. If not, disable the enterprise repos and enable the pve-no-subscription repo.

- :lucide-external-link: [Proxmox VE Enterprise / No-Subscription Repository](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#sysadmin_package_repositories)

!!! tip "[Debian Firmware Repository](https://pve.proxmox.com/wiki/Package_Repositories#sysadmin_debian_firmware_repo)"

    Additionally, you may consider inclduing the Debian `non-free-firmware` repository. This provides microcode updates. As of Proxmox 9 this is enabled by default.

    On Proxmox 8, you can simply append `non-free-firmware` to all `debian.org` sources in `/etc/apt/sources.list`

---

### :lucide-laptop: System Updates

GUI: Click `Updates` then `Refresh` to run `apt-get update;` finally `>_ Upgrade` to run `apt-get dist-upgrade`

You can also run these from a bash shell on the Proxmox VE host.

- :lucide-external-link: [Proxmox VE System Software Updates](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#system_software_updates)

!!! tip "Systemd Service"

    All of this including safely handling VM's has been automated with [ansible-role-configure_updates](https://github.com/straysheep-dev/ansible-role-configure_updates) as a systemd service.

For major version upgrades (e.g. 8 to 9) you can perform an in-place upgrade. Follow all of the guidance based on your major version number in the Proxmox Wiki. Most notable is the "version-to-version" script, that will check for any issues you may need to address prior to upgrading. The script for Proxmox 8 is run like this: `pve8to9 --full`.

- :lucide-external-link: [Upgrading from 8 to 9](https://pve.proxmox.com/wiki/Upgrade_from_8_to_9)

---

### :lucide-square-terminal: SSH Authentication

`root` is allowed to login with a password by default. You can disable this in `sshd_config` once you've added your key to `authorized_keys`:

```bash
curl https://github.com/USERNAME.keys | tee -a /root/.ssh/authorized_keys
# symlink to /etc/pve/priv/authorized_keys

sed -i_bkup 's/^.*PasswordAuthentication.*$/PasswordAuthentication no/g' /etc/ssh/sshd_config
systemctl restart sshd
```

---

### :lucide-brick-wall-fire: Firewall

The firewall is off by default. To enable it:

- Datacenter > Firewall > Options > Click `Firewall ... No` to highlight it > Click `Edit` > Check `Firewall`
- Now `iptables -S` will reveal a set of default rules

If you enable it, the only default allow rules are to 22/tcp and 8006/tcp from the local subnet.

!!! tip "Firewall Isolation"

    You can enable the firewall at the Proxmox node level, as well as per-VM to isolate lateral traffic between VM's on the same bridge interface. This requires nothing more than simply enabling the firewall for each VM, the default rules allow outbound but drop inbound. Keep in mind this will also prevent you from remoting into VM's directly without some type of reverse tunnel or proxy.

You can define specific rules to the Proxmox VE host and VM guests from the GUI or using a text editor on:

- `/etc/pve/nodes/<nodename>/host.fw` Host configuration
- `/etc/pve/firewall/<VMID>.fw` VM / Container configuration

- :lucide-external-link: [Proxmox VE Firewall](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#chapter_pve_firewall)

---

### :lucide-search-alert: Suricata

You can install suricata IPS on the Proxmox host. Keep in mind REJECTED or DROP'd packets do not go to suricata.

```bash
apt-get install suricata
modprobe nfnetlink_queue
```

- :lucide-external-link: [Proxmox VE Firewall Tips and Tricks](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_tips_and_tricks)

---

### :lucide-circle-user: Users

To see all system users: Datacenter > Permissions > Users

User details for the PVE realm are defined on the filesystem under `/etc/pve/user.cfg` (note that passwords are under `/etc/pve/priv/shadow.cfg`).

Users in the PAM realm are still stored under `/etc/passwd`,`shadow` and the usual system paths.

- :lucide-external-link: [Proxmox VE User Management](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#pveum_users)
- :lucide-external-link: [Proxmox VE User Management: Examples](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_real_world_examples)


### :lucide-users-round: Groups

To see, create, and modify groups: Datacenter > Permissions > Groups

These are effectively user defined, based on what your needs are. Groups are a way to tie multiple users to one or more permission sets.

- :lucide-external-link: [Proxmox VE Groups](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#pveum_groups)

Proxmox has a built-in set of permission "roles" that you can get started with. These are covered [below](#permissions).


### :lucide-component: Realms

Proxmox understands two authentication "realms", which are "pve" and "pam".

!!! tip "PVE & PAM Realms are Two Separate Things"

	- **PVE**: Authentication and permissions tied directly to Proxmox-specific functionality (GUI/API)
		- A user created in the PVE realm doesn't necessarily have direct Linux shell access outside of the Proxmox GUI/API
	- **PAM**: Authentication to, and controlled by, the underlying OS (for shell or SSH access)
		- PAM can still be used as an auth realm via the GUI
		- In other words, a PAM account can be expanded to have GUI/API access, but a PVE account must have a separate PAM account for shell access

!!! note "Testing User Access"

	You can try and verify how this works by creating users however you'd like, and seeing what level of access they have to the UI, API, and console / SSH, one step at a time. This is the best way to visualize and understand what's required and what the authentication mechanisms expect.

!!! example "Users and Realms Examples"

	- A user created in the web UI, under the PAM realm, doesn't have a shell account automatically; you must create this
	- There isn't even a way to set a password like this either; the expectation is the user account already exists on the OS underneath
	- A user created via a utility like `useradd` from the shell, does not automatically have an account in the web UI
	- In this case, create your user with `useradd`, then create them in the web UI tied to the PAM realm, and you can login to the web UI with that user
	- Similarly you must create a shell account for a web UI user that you'd like to have shell access, say a unique admin account
	- Users can even have the `Sys.Console` permission in Proxmox, but won't be able to use the shell or SSH unless they know a valid login to the OS underneath

**Example Continued :material-wrench-cog-outline:**

Here we're using the Proxmox utility `pveum user add` to create a user in the Proxmox "side" of the system, but tie their authenticaiton to PAM:

```bas
root@pve:~# pveversion
pve-manager/8.4.1/2a5fa54a8503f96d (running kernel: 6.8.12-11-pve)
root@pve:~#
root@pve:~# pveum user add testuser@pam
root@pve:~#
root@pve:~# pveum user list
┌──────────────┬─────────────────────────────────┬────────────────┬────────┬────────┬───────────┬────────┬──────┬──────────┬────────────┬──────────────────┬──
│ userid       │ comment                         │ email          │ enable │ expire │ firstname │ groups │ keys │ lastname │ realm-type │ tfa-locked-until │ t
╞══════════════╪═════════════════════════════════╪════════════════╪════════╪════════╪═══════════╪════════╪══════╪══════════╪════════════╪══════════════════╪══
│ admin@pve    │ Standard administrative user.   │                │ 1      │      0 │           │        │      │          │ pve        │                  │
├──────────────┼─────────────────────────────────┼────────────────┼────────┼────────┼───────────┼────────┼──────┼──────────┼────────────┼──────────────────┼──
│ builder@pam  │ User to build packer templates. │                │ 0      │      0 │           │        │      │          │ pam        │                  │
├──────────────┼─────────────────────────────────┼────────────────┼────────┼────────┼───────────┼────────┼──────┼──────────┼────────────┼──────────────────┼──
│ root@pam     │                                 │ root@localhost │ 1      │      0 │           │        │      │          │ pam        │                  │
├──────────────┼─────────────────────────────────┼────────────────┼────────┼────────┼───────────┼────────┼──────┼──────────┼────────────┼──────────────────┼──
│ testuser@pam │                                 │                │ 1      │      0 │           │        │      │          │ pam        │                  │
└──────────────┴─────────────────────────────────┴────────────────┴────────┴────────┴───────────┴────────┴──────┴──────────┴────────────┴──────────────────┴──
root@pve:~#
```

We can see even though we created this user in Proxmox, they don't really exist (yet):

```bash
root@pve:~# su testuser
su: user testuser does not exist or the user entry does not contain all the required fields
root@pve:~#
root@pve:~#
root@pve:~# pveum passwd testuser@pam
Enter new password: ************************************
Retype new password: ************************************
change password failed: user 'testuser' does not exist
```

Once they've been added to the underlying OS, we can both, run as this user via a shell, AND login to Proxmox's web UI or API with them since it's aware of, and accepting, their authentication through PAM:

```bash
root@pve:~# useradd -m -s /bin/bash testuser
root@pve:~# passwd testuser
root@pve:~#
root@pve:~# su testuser
testuser@pve:/root$ cd ~
testuser@pve:~$ pwd
/home/testuser
testuser@pve:~$ id
uid=1001(testuser) gid=1001(testuser) groups=1001(testuser)
```

!!! note "Setting Passwords"

	Interestingly, both `passwd user` and `pveum passwd user@pam` can manage a shell user's login password, once they exist under PAM.

Finally, you'll need to scope each user their permissions, either individually, or by giving permissions to a group and tying user(s) to groups, which is the recommended way to manage permissions at scale.


### :lucide-id-card-lanyard: Permissions

See, create, and modify permission assignments: Datacenter > Permissions

- :lucide-external-link: [Proxmox VE Permission Management: List of Permissions](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#pveum_permission_management)

That link lists out all predefined permission sets and what they do. The most useful if you're just getting started:

- `Administrator`: has full privileges
- `PVEVMAdmin`: fully administer VMs
- `PVESysAdmin`: audit, system console and system logs
- `PVEVMUser`: view, backup, configure CD-ROM, VM console, VM power management


#### :lucide-package: Packer User Example

[Mayfly277](https://github.com/Mayfly277), one of the contributers to [Game of AD](https://github.com/Orange-Cyberdefense/GOAD), has a [blog post on creating a separate user account in Proxmox to build packer VM's](https://mayfly277.github.io/posts/GOAD-on-proxmox-part2-packer/).

Using these steps as a reference, we can create a more "generic" user and group scope using the built-in permissions and roles tied to groups, then modify them from there if needed.

!!! example "Creating Administrators and DevOps Groups + Permissions"

	This walks through creating a dedicated Administrators group to not rely on the root account, and a DevOps group for packer use.

	- Datacenter > Permissions > Groups: Create "Administrators" and "DevOps" groups
	- Datacenter > Permissions: Add > Group Permission
		- Path: `/`
		- Group: Administrators
		- Role: Administrator
		- Propagate: :lucide-square-check-big:
	- Datacenter > Permissions: Add > Group Permission
		- Path: `/`
		- Group: DevOps
		- Role: PVEVMAdmin
		- Propagate: :lucide-square-check-big:
	- Datacenter > Permissions: Add > Group Permission
		- Path: `/`
		- Group: DevOps
		- Role: PVESysAdmin
		- Propagate: :lucide-square-check-big:
	- Datacenter > Permissions > Users: Edit User > Assign groups from the drop-down menu

	[You can do the same using the `pveum` command line utility](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_real_world_examples):

	```bash
	# Define the groups
	pveum group add Administrators -comment "System Administrators"
	pveum group add DevOps -comment "Infrastructure-as-code Users"
	# Define the roles
	pveum acl modify / -group Administrators -role Administrator
	pveum acl modify / -group DevOps -role PVEVMAdmin
	pveum acl modify / -group DevOps -role PVESysAdmin
	# Assign users, assumes these users exist in PAM
	pveum user modify admin@pam -group Administrators
	pveum user modify builder@pam -group DevOps
	```

!!! tip "Permissions Scopes and Privilege Escalation"

	These permissions could be locked down further, however in the example above we give the `builder` user `Sys.Console` access.

	Effectively you have sudo-less console access despite the scope for the permission path itself being `/`.

	- You cannot run any `pve*` binaries, or update apt packages
	- You cannot read files you do not have access to read
	- There's no `sudo` by default

	You'll need something like a kernel exploit, misconfiguration in scheduled tasks, the filesystem, or services, a secret being visible in process memory with pspy, or a unique attack path on your local Proxmox configuration to "break out" of this shell context and have higher privileges.

	There are likely other methods not listed above, and this isn't even taking into account attacking Proxmox via the UI or API. The point is, it's not a perfect sandbox, but it's better than running devops workflows as the root account.

	To properly audit unique admin users, you'll need to install `sudo` and configure each admin user that requires sudo access.


---


### :lucide-hard-drive: Disks

It's recommended to have at least one or more:

- Disks dedicated as a storage pool
- Disks dedicated as a file system (VM backups + files and settings)

!!! abstract "Disk Encryption"

    There are really two paths to take here based on your requirements:

    **Option 1: Full Disk Encryption**

    - Not offered by Proxmox by default
    - Creates a number of reboot + remote access issues
    - Requires configuration and testing that may change across major versions

    **Option 2: Encrypt the Storage Disks**

    - Safer and easier
    - Will not prevent the system from booting
    - Unlock can be automated, tied to TPM or Yubikey
    - ZFS is trickier here than LUKS


#### :lucide-lock-keyhole: Full Disk Encryption

*One option is to install Debian with FDE, then install Proxmox on top of it. However it's **recommended** to just install Proxmox normally.*

1. Install Proxmox with ZFS
2. Reboot into recovery mode from the installer USB
3. Follow [these steps](https://gist.github.com/yvesh/ae77a68414484c8c79da03c4a4f6fd55) to enable ZFS encryption
4. Follow [these steps](https://github.com/openzfs/zfs/tree/master/contrib/initramfs#unlocking-a-zfs-encrypted-root-over-ssh) to install `dropbear-initramfs` for remote decryption over SSH
5. Encrypt the dataset for VMs and containers (which is also step 5 in [this post](https://forum.proxmox.com/threads/encrypting-proxmox-ve-best-methods.88191/#post-387731))

ZFS + Dropbear, all of the resources below came from this post: [FDE with ZFS](https://forum.proxmox.com/threads/full-disk-encryption-with-zfs-using-proxmox-installer.127512/)

- :lucide-external-link: [Feature Request: Native ZFS encryption during Proxmox installation](https://bugzilla.proxmox.com/show_bug.cgi?id=2714)
- :lucide-external-link: [Encrypting Proxmox VE: Best Methods](https://forum.proxmox.com/threads/encrypting-proxmox-ve-best-methods.88191/#post-387731)
- :lucide-external-link: [gist: FDE with Proxmox and ZFS native encryption](https://gist.github.com/yvesh/ae77a68414484c8c79da03c4a4f6fd55)
- :lucide-external-link: [github/openzfs: Unlocking ZFS encrypted root over SSH](https://github.com/openzfs/zfs/tree/master/contrib/initramfs#unlocking-a-zfs-encrypted-root-over-ssh)
- :lucide-external-link: [Proxmox VE ZFS encryption](https://pve.proxmox.com/wiki/ZFS_on_Linux#zfs_encryption)

LUKS + Dropbear

- :lucide-external-link: [Adding LUKS FDE to Proxmox](https://forum.proxmox.com/threads/adding-full-disk-encryption-to-proxmox.137051/)

---


#### :lucide-lock-keyhole: Storage Disk Encryption

This will walk you through encrypting and managing other internal drives on Proxmox.

**Creating a LUKS Partition**

Assume you have at least one additional internal drive connected to Proxmox.

!!! note "Unlock & Name the Storage ID"

	If you look under Datacenter > Storage, you'll see the default ID's are `local` and `local-lvm`.

    In this example, we'll call it `luks1-lvm` to stick to a similar naming convention.

    You can make up any name for a storage ID, just stick to alphanumeric characters and hyphens `-`.


```bash
# Format the disk, fdisk is available on Proxmox, interactive but straightforward.
# If you do not partition the disk, you'll only see /dev/sdX, not /dev/sdX1.
# https://wiki.archlinux.org/title/Fdisk
fdisk /dev/sdX1

# Commands inside fdisk:
# g  → create GPT partition table
# n  → new partition (accept all defaults for single full-disk partition)
# w  → write and exit

# Format the partition to be LUKS encrypted
cryptsetup luksFormat /dev/sdX1
# Enter a passphrase, store it in your password manager

# Unlock it, assign a storage-id
cryptsetup open --type luks /dev/sdX1 <storage-id>  # You create the storage ID string, e.g. luks1-lvm

# Only create a filesystem if you need it, otherwise use lvm-thin pools for VM storage.
mkfs.ext4 /dev/mapper/luks1-lvm
```


##### Storage Pools

!!! tip "New `vgname` and `thinpool`"

	Create a new volume group and thinpool for LUKS drives to share. **This is because adding them to the existing volume group and thinpool means VM's and data will still be written to the unencrypted drives** (if you aren't using full disk encryption).

    Example naming convention to create unique entries per-disk:

    - Storage ID: `luks1-lvm`
    - vgroup (Volume Group): `luks1-group`
    - thinpool (Data Pool): `luks1-data`

    For LUKS, the intent is more clear if you do this per-disk, so sdb1 gets luks1, sdc1 gets luks2, etc. If one disk fails, there's no overlapping storage pools between them. ZFS is potentially easier to work with in this sense.

To make Proxmox aware of the new storage pool:

- Datacenter > Storage > Add
- ID: `luks1-lvm` (storage identifier created with cryptsetup)

Alternatively you can use `pvesm` to do everything (examples originally from `gpt-4o`).

```bash
# Helpful to visualize the pve storage
pvesm status

# Create a new volume group for LUKS drives
vgcreate luks1-group /dev/mapper/luks1-lvm

# Create a new thinpool called "luks1-data" for the luks1-lvm storage drive, leaving 5% of the drive for metadata
lvcreate --type thin-pool -l95%FREE -n luks1-data luks1-group

# Add a LUKS thin-lvm data pool for VM storage
# lvm-thin is more efficient than a filesystem for VM storage
pvesm add lvmthin luks1-lvm \
    --vgname luks1-group \
    --thinpool luks1-data \
    --content images \
    --content rootdir

```

If you mess up, you can remove or undo almost anything at this stage. For example:

```bash
vgremove luks1-group

```


##### File Systems

!!! note "LVM-Thin vs Filesystem"

    **Remember, you would only `mkdir` and `mount` a filesystem if it's not a storage pool. VM's benefit from lvm-thin data pools for more efficency than a filesystem**.

Get a sense of which partition(s) you'll want to add. In this case, the device is `sdc` and the GPT partition we created with `fdisk` is `sdc1`.

```bash
lsblk
<SNIP>
sdc                            8:48   0   1.8T  0 disk
└─sdc1                         8:49   0   1.8T  0 part
```

!!! note "Skipped Partitioning?"

    Whether you use `parted`, `fdisk`, or otherwise, if you didn't create a disk partition some programs and systems will complain about this. However, LUKS encrypting the entire disk still works.

In this example, assume you have another internal LUKS disk used for scheduled backups, `luks2-lvm`.

```bash
# If this will be a filesystem and NOT an lvm-thin storage pool
# Create a mount path (can be anywhere, /mnt/ is easy to understand)
# and use mount to connect that storage id to the path.
mkdir /mnt/luks2-backup
mount /dev/mapper/luks2-lvm /mnt/luks2-backup

# Make Proxmox aware of the filesystem for VM backup schedules, ISOs, or other use
pvesm add dir luks2-lvm \
    --path /mnt/luks2-backup \
    --content backup
```

If you need to unmount and close the LUKS disk:

```bash
# To unmount the filesystem, and close the the LUKS partition if you need to
umount /mnt/luks2-backup
cryptsetup close luks2-lvm
```

#### :lucide-cpu: TPM Unlocking Disks

Unlock Encrypted Storage via a TPM.

**References**

- :lucide-external-link: [github.com/tpm2-software](https://github.com/tpm2-software)
- :lucide-external-link: [tpm2-tools Maintainers](https://github.com/tpm2-software/tpm2-tools/blob/master/docs/MAINTAINERS.md)
- :lucide-external-link: [Intel: Open Sourcing the TPM2 Software Stack](https://www.intel.com/content/www/us/en/developer/articles/technical/tpm2-software-stack-open-source.html)
- :lucide-external-link: [Arch Linux Wiki: TPM](https://wiki.archlinux.org/title/Trusted_Platform_Module)
- :lucide-external-link: [Arch Linux Wiki: systemd-cryptenroll](https://wiki.archlinux.org/title/Systemd-cryptenroll)
- :lucide-external-link: [github.com/systemd tpm2-util.c](https://github.com/systemd/systemd/blob/main/src/shared/tpm2-util.c) (search github.com/tpm2-software)

Essential packages:
```bash
# On Proxmox 8
sudo apt update; sudo apt install -y \
  cryptsetup \
  tpm2-tools

# On Proxmox 9 or later
sudo apt update; sudo apt install -y \
  cryptsetup \
  tpm2-tools \
  systemd-cryptsetup
```

Setup on Proxmox:

=== "Custom Service"

    !!! note "Why not use `systemd-cryptenroll`?"

        This is different than using `systemd-crypt*`, which will block the machine from booting without manual (physical) intervention if anything changes with the PCRs you're tying the unlock condition to. For example, `--tpm2-pcrs=0+7` ties the unlock to the firmware and Secure Boot state. If you update the firmware of the host, or Secure Boot gets a new revocation list, unlock fails, and in the case of `systemd-cryptenroll`, the system is stuck between GRUB and booting, asking for your LUKS password as a fallback.

        Instead, we create our own custom systemd-service that runs the unlock commands *after* boot, and if it fails, you'll still have remote access to Proxmox to assess the situation.

    Identify your LUKS device.

    ```bash
    lsblk  # Use this to determine <storage-name>
    lsblk -f | grep crypto_LUKS  # If this has a number appended, like sdb1, include the number below too
    ```

    Enroll the TPM (***appends*** data to an open keyslot).

    ```bash
    # --tpm2-pcrs=0+7 == firmware + Secure Boot state, is generally auto-patch safe
    systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7 /dev/sdX1
    ```

    Do ***NOT*** use crypttab, this will block boot if anything goes wrong. This block below is only for reference.

    ```bash
    # Format: <name> <device> <keyfile> <options>
    # echo "<storage-name> UUID=$(blkid -s UUID -o value /dev/sdX) none tpm2-device=auto" \
    #    >> /etc/crypttab
    ```

    Create a systemd service to run the unlock commands at boot. If the TPM is present and the firmware or Secure Boot state has not changed, this will automatically unlock the disks.

    If you're mounting a filesystem, ensure the path exists and you've registered it with Proxmox.  For example:

    ```bash
    mkdir -p /mnt/luks3-backup
    pvesm add dir luks3-backup --path /mnt/luks3-backup --content backup
    ```

    We use `crytpsetup` because it should work across all recent Proxmox verisons (8 and 9). Additionally, using the wwn (World Wide Name) of storage devices is encouraged over block names like `/dev/sdb1`.

    - :lucide-external-link: [Arch Linux Wiki: Persistent Block Device Names](https://wiki.archlinux.org/title/Persistent_block_device_naming#World_Wide_Name)

    ```bash
    echo "
    [Unit]
    Description=TPM Unlock Disks
    After=systemd-modules-load.service
    Before=pve-storage.target

    [Service]
    Type=oneshot
    # Prepend a "-" to the command path, to ignore the exit code
    # so the system doesn't stall on boot if there's a change.
    # You'll want to pin the drive to to the unique drive id.
    # Obtain it with: ls -l /dev/disk/by-id/
    # So /dev/sdb1 == /dev/drive/by-id/wwn-0x0123456789abcdef-part1
    ExecStart=-/usr/sbin/cryptsetup open /dev/drive/by-id/wwn-0x0123456789abcdef-part1 <storage-name>
    # ExecStart=-/usr/sbin/cryptsetup open /dev/drive/by-id/wwn-0x1111111111111111-part1 <storage-name>
    # Repeat for each LUKS volume to be auto-unlocked by TPM.
    # For filesystems, you must also include the mount commands here too.
    # ExecStart=-/usr/bin/mount /dev/mapper/<storage-name> /mnt/some-existing-path
    RemainAfterExit=yes

    SuccessExitStatus=0

    [Install]
    WantedBy=multi-user.target" | tee /etc/systemd/system/tpm-unlock-disks.service
    ```

    Reload systemd to pick up crypttab changes

    ```bash
    systemctl daemon-reload
    systemctl enable tpm-unlock-disks.service
    # You should also be able to confirm the service unlocks the disks
    # by restarting it manually
    systemctl restart tpm-unlock-disks.service
    ```

    Verify the new TPM-based keyslot exists alongside your password

    ```bash
    cryptsetup luksDump /dev/sdX1
    ```

=== "systemd-cryptenroll"

    !!! warning "Manual Intervention"

        This method will block Proxmox from booting if any of the PCR values change (generally we use 0+7, meaning firmware and Secure Boot states), and will require physical keyboard + monitor access to resolve, even if the OS drive is not encrypted.

    !!! warning "TODO"

        This section is still under construction.


#### :lucide-microchip: Backup LUKS Headers

- :lucide-external-link: [Fedora Project: Backup LUKS Headers](https://docs.fedoraproject.org/en-US/quick-docs/encrypting-drives-using-LUKS/#_backup_luks_headers)
- :lucide-external-link: [gitlab/cryptsetup: Backup and Data Recovery](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/FrequentlyAskedQuestions#6-backup-and-data-recovery)
- :lucide-external-link: [Arch Linux: LUKS Backup and Restore](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Backup_and_restore)

Once everything's working it's **recommended to backup the headers of your LUKS devices** in the extremely rare case a drive header corrupts. These are ~2MB or larger binary blobs, the first "area" of the LUKS disk containing the encrypted key slots and necessary data to unlock the drive.

- Store these on a Proxomox backup filesystem drive
- Store these in your password manager
- If these are stolen, they can be brute-forced offline

```bash
cryptsetup luksHeaderBackup --header-backup-file <header-file> <device>
cryptsetup luksHeaderBackup --header-backup-file /mnt/luks2-backup/luks2-header-"$(date +%F)".bin /dev/sdc1
```

**Additional Resources**

- :lucide-external-link: [How to Mount Existing Disks Within proxmox](https://forum.proxmox.com/threads/how-to-mount-existing-disk-to-storage.66559/)
- :lucide-external-link: [Adding LUKS Encryption to proxmox](https://forum.proxmox.com/threads/adding-full-disk-encryption-to-proxmox.137051/)
- :lucide-external-link: [Add an Existing Disk to proxmox](https://forum.proxmox.com/threads/adding-an-existing-hdd-to-proxmox.118361/)
- :lucide-external-link: [Add a Dew Disk as LVM-thin](https://forum.proxmox.com/threads/adding-a-disk-and-set-it-as-lvm-thin-help-needed-please.111724/)
- :lucide-external-link: [proxmox Storage Wiki](https://pve.proxmox.com/wiki/Storage)
- :lucide-external-link: [Dr Duh Yubikey Guide: Backing Up Keys on Encrypted Storage](https://github.com/drduh/YubiKey-Guide?tab=readme-ov-file#backup-keys)

---


## :lucide-shuffle: Migrating VMs

- :lucide-external-link: [Importing Virtual Machines](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#qm_import_virtual_machines)
- :lucide-external-link: [Migrating Hyper-V to Proxmox](https://forum.proxmox.com/threads/migrating-hyper-v-to-proxmox-what-i-learned.137664/)

### :lucide-database-backup: Between Storage Pools

!!! warning "TODO"

    This section is still under construction.

### :lucide-monitor-cog: From Hyper-V

Follow [the link referenced above](https://forum.proxmox.com/threads/migrating-hyper-v-to-proxmox-what-i-learned.137664/). Essentially, with no snapshots on the Hyper-V VM you wish to migrate (you can export the VM and import it back into Hyper-V so it's separate from your "main" copy, to delete the snapshots, then export it again) export your Hyper-V VM so you have the .vhdx file. Move it over to the Proxmox filesystem.

!!! tip "Filesystem Space and Migrating VMs"

	An external drive or larger filesystem space within Proxmox is useful here, as by default Proxmox gives its default "filesystem" on the OS drive roughly 100GB while leaving the remaining space to lvm-thin pool storage for VMs.

    This is not explicitly talked about in the post, but if you cannot access the .vhdx file on your Hyper-V host by mounting a share on to the Hyper-V host on Proxmox, you should use an external drive to transfer the VM. **With the average size likely being over 40GB, it will take a long time to `scp`, `rsync`, or similar**.

    Since the OS drive has roughly 100GB on the root filesystem, the majority of the remaining space is lvm-thin space, which is a special type of storage format efficient for VM's but lacking a filesystem.

    Both an external drive and a fileshare allow you to import the VM directly to the lvm-thin space without accidentally overruning your /root filesystem's space which can happen if you copy the VM directly onto the OS drive filesystem before importing it.

**Create the VM in Proxmox**

Create the VM as you would if you were making it from scratch, following the steps in the linked forum post (q35 for the VM type, and then detach the default disk). Use the `qm disk import <vmid> <source> <storage> [OPTIONS]` command to import the .vhdx file as the disk image for your Hyper-V VM.

```bash
qm disk import 105 /mnt/wd500-external/Virtualization/Wazuh/Virtual\ Hard\ Disks/Wazuh.vhdx luks1-lvm
importing disk '/mnt/wd500-external/Virtualization/Wazuh/Virtual Hard Disks/Wazuh.vhdx' to VM 105 ...
  Logical volume "vm-105-disk-3" created.
transferred 0.0 B of 127.0 GiB (0.00%)
transferred 1.3 GiB of 127.0 GiB (1.00%)
transferred 2.5 GiB of 127.0 GiB (2.00%)
transferred 3.8 GiB of 127.0 GiB (3.00%)
transferred 5.1 GiB of 127.0 GiB (4.00%)
<SNIP>
transferred 125.9 GiB of 127.0 GiB (99.12%)
transferred 127.0 GiB of 127.0 GiB (100.00%)
transferred 127.0 GiB of 127.0 GiB (100.00%)
Successfully imported disk as 'unused1:luks1-lvm:vm-102-disk-3'
```

Add (attach) the disk under the VM's settings. **Be sure to go into "Options", not "Hardware" and set this disk first in the boot order**. In the event this is a Linux VM, even with SecureBoot enabled you'll be able to boot it, and get running.

That's really it, be sure to install the `spice-vdagent` service on the guest to leverage additional virtualization features if you want them, though this is not necessary.

Fix any issues (such as networking), poweroff, then take your first snapshot.

---


## :lucide-settings: VM Configuration

### :lucide-ethernet-port: Virtual NIC

This is the equivalent of creating a virtual NIC (not to be confused with a VLAN) in VMware or VirtualBox for guest-to-guest communication (for example a LAN and OPT NIC to attach to a pfSense VM). First you need to create another Linux Bridge network device that isn't attached to any physical network ports.

- Under Datacenter, select your proxmox-ve hostname > System > Network > Create
- Name it, check `[x]` autostart, create
- Now click "Apply Configuration" at the top, so your new `vmbrX` shows as "Active: Yes"

Now when creating the VM:

- `[VM Name]` > Hardware > Add > Network Device
- Add two, the WAN side can be the default `vmbr0`, the LAN side is your new `vmbrX` with either `intel E1000` or `virtIO`
- One will receive DHCP / access to the outside world over the Proxmox virtual brdige
- The other will remain purely virtual (for example pfSense could then provide DHCP to that NIC)


### :lucide-hard-drive: Disk Passthrough

Say you've installed Proxmox on an existing server or workstation, either as the primary OS or next to an existing OS. You can mount any of the internal drives to VM's without any special modifications to Proxmox's kernel or boot parameters (tested on PVE-8.3.0).

!!! note "PCIe Passthrough?"

    This does *not* cover PCIe passthrough, see [this video](https://www.youtube.com/watch?v=_hOBAGKLQkI) for details on passing through entire storage controllers and GPUs.

    This type of hardware passthrough *does* require modifications to the kernel and reboots. It will have its own section in this guide after testing.

The commands below are summarized from this article, on [how to pass through a physical disk to a virtual machine](https://pve.proxmox.com/wiki/Passthrough_Physical_Disk_to_Virtual_Machine_(VM)). This can be done while the VM is live, and without rebooting the VM or Proxmox.

In summary, first obtain the MODEL (e.g. ata-ST1000DM001-1ABCDEF) plus the SERIAL string (e.g. Z1A2B3C4) of all disks:

```bash
lsblk -o +MODEL,SERIAL,WWN
```

If you're looking for a specific disk, you'll have to recognize it by MODEL name, or if they're all the same, you may have to mount each one and review what's on it.

Together those strings create a full path to the disk itself. Find the full path under `/dev/disk/by-id/` by searching for the SERIAL string.

```bash
ls -l /dev/disk/by-id | grep Z1A2B3C4
```

Hot-plug the disk as a new SCSI device to the VM.

```bash
# Existing SCSI disks are listed as scsi0, scsi1, etc. Add this as a new disk
# to <vm-id>. Use the full path to the disk like below.
qm set <vm-id> --scsi<int> /dev/disk/by-id/<model>_<serial>

# Example:
qm set 105 --scsi2 /dev/disk/by-id/ata-ST1000DM001-1ABCDEF_Z1A2B3C4
```

Hot-unplug or remove the disk.

```bash
qm unlink 105 --idlist scsi2
```


## :lucide-wrench: Maintenance

Use the following tabs to understand the overall activity of your node(s):

- Datacenter > Summary
- `<node>` > Summary

For the best data and visualization, you'll want to enable the Metric Server, to ship logs to a VM running [InfluxDB + Grafana](https://docs.influxdata.com/influxdb/v2/tools/grafana/#Copyright).


### :lucide-hard-drive-download: Snapshots

VM snapshots are valuable, but more brittle than backups:

- Tied to the storage pool they were taken in
- Are not backed up
- Must be deleted to fully migrate a VM


### :lucide-rotate-ccw: Backups

System backups can be taken by archiving `/etc/pve`:

```bash
tar czf /mnt/luks3-backup/pve-config-$(date +%Y%m%d).tar.gz /etc/pve/
```

VM backups are the best recovery and long term storage method. They can be scheduled easily from:

- Datacenter > Backup > Add
- Select the VMs
- Set the cadence / time to run backups
- `Mode: Stop` will shutdown the VMs before running `vzdump` to archive them
- Set retention policy, e.g. keep up to 3 backups of each VM

!!! tip "Backup `jobs.cfg`"

    You can see the raw config this creates, for later automation, under `/etc/pve/jobs.cfg`.


## :lucide-puzzle: Troubleshooting

*This section details issues and how to resolve them.*


### :lucide-octagon-minus: UEFI Causes Boot Failure

Typically in Unix-like distros where UEFI is supported but Secure Boot is not, you must interrupt the boot process and turn off Secure Boot.

!!! tip "Proxmox Support"

    Proxmox 8 and later support Secure Boot.

- During boot, when you see the Proxmox splash screen, hit `Esc`
- Select `Device Manager`
- Select `Secure Boot Configuration`
- Uncheck `Attempt SecureBoot` with the `Space` bar
- Back out to the main UEFI menu and select `Continue`
