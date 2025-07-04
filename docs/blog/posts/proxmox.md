---
draft: false
date:
  created: 2024-08-13
  updated: 2025-06-15
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

📚 [Proxmox VE Documentation](https://pve.proxmox.com/pve-docs)

---

## ⏬ Download

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


## 💽 Install

Walk through the graphical or terminal installer. This is straight forward if you're used to installing hypervisor software.


**Email**

You'll need to choose an email address to send alerts to, this can be `root@localhost` or a real email.


## 🔑 Access

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


## ⚙️ Configure


### 🔐 MFA

First, add 2FA/MFA to your root login. This can be a software token saved to your password manager or authenticator app, plus a hardware key using webauthn.

---

### 📦 Package Repositories

Next, if you have a valid enterprise subscription key, you're good to go. If not, disable the enterprise repos and enable the pve-no-subscription repo.

- 🌐 [Proxmox VE Enterprise / No-Subscription Repository](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#sysadmin_package_repositories)

---

### 💻 System Updates

GUI: Click `Updates` then `Refresh` to run `apt-get update;` finally `>_ Upgrade` to run `apt-get dist-upgrade`

You can also run these from a bash shell on the Proxmox VE host.

- 🌐 [Proxmox VE System Software Updates](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#system_software_updates)

---

### 🐚 SSH Authentication

`root` is allowed to login with a password by default. You can disable this in `sshd_config` once you've added your key to `authorized_keys`:

```bash
curl https://github.com/USERNAME.keys | tee -a /root/.ssh/authorized_keys
# symlink to /etc/pve/priv/authorized_keys

sed -i_bkup 's/^.*PasswordAuthentication.*$/PasswordAuthentication no/g' /etc/ssh/sshd_config
systemctl restart sshd
```

---

### 🔥 Firewall

The firewall is off by default. To enable it:

- Datacenter > Firewall > Options > Click `Firewall ... No` to highlight it > Click `Edit` > Check `Firewall`
- Now `iptables -S` will reveal a set of default rules

If you enable it, the only default allow rules are to 22/tcp and 8006/tcp from the local subnet.

You can define specific rules to the Proxmox VE host and VM guests from the GUI or using a text editor on:

- `/etc/pve/nodes/<nodename>/host.fw` Host configuration
- `/etc/pve/firewall/<VMID>.fw` VM / Container configuration

- 🌐 [Proxmox VE Firewall](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#chapter_pve_firewall)

---

### 🔎 Suricata

You can install suricata IPS on the Proxmox host. Keep in mind REJECTED or DROP'd packets do not go to suricata.

```bash
apt-get install suricata
modprobe nfnetlink_queue
```

- 🌐 [Proxmox VE Firewall Tips and Tricks](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_tips_and_tricks)

---

### ⌨️ Users

To see all system users: Datacenter > Permissions > Users

User details for the PVE realm are defined on the filesystem under `/etc/pve/user.cfg` (note that passwords are under `/etc/pve/priv/shadow.cfg`).

Users in the PAM realm are still stored under `/etc/passwd`,`shadow` and the usual system paths.

- 🌐 [Proxmox VE User Management](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#pveum_users)
- 🌐 [Proxmox VE User Management: Examples](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_real_world_examples)


### 📇 Groups

To see, create, and modify groups: Datacenter > Permissions > Groups

These are effectively user defined, based on what your needs are. Groups are a way to tie multiple users to one or more permission sets.

- 🌐 [Proxmox VE Groups](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#pveum_groups)

Proxmox has a built-in set of permission "roles" that you can get started with. These are covered [below](#permissions).


### 📚 Realms

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


### 🪪 Permissions

See, create, and modify permission assignments: Datacenter > Permissions

- 🌐 [Proxmox VE Permission Management: List of Permissions](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#pveum_permission_management)

That link lists out all predefined permission sets and what they do. The most useful if you're just getting started:

- `Administrator`: has full privileges
- `PVEVMAdmin`: fully administer VMs
- `PVESysAdmin`: audit, system console and system logs
- `PVEVMUser`: view, backup, configure CD-ROM, VM console, VM power management


#### 📦 Packer User Example

[Mayfly277](https://github.com/Mayfly277), one of the contributers to [Game of AD](https://github.com/Orange-Cyberdefense/GOAD), has a [blog post on creating a separate user account in Proxmox to build packer VM's](https://mayfly277.github.io/posts/GOAD-on-proxmox-part2-packer/).

Using these steps as a reference, we can create a more "generic" user and group scope using the built-in permissions and roles tied to groups, then modify them from there if needed.

!!! example "Creating Administrators and DevOps Groups + Permissions"

	This walks through creating a dedicated Administrators group to not rely on the root account, and a DevOps group for packer use.

	- Datacenter > Permissions > Groups: Create "Administrators" and "DevOps" groups
	- Datacenter > Permissions: Add > Group Permission
		- Path: `/`
		- Group: Administrators
		- Role: Administrator
		- Propagate: ✅
	- Datacenter > Permissions: Add > Group Permission
		- Path: `/`
		- Group: DevOps
		- Role: PVEVMAdmin
		- Propagate: ✅
	- Datacenter > Permissions: Add > Group Permission
		- Path: `/`
		- Group: DevOps
		- Role: PVESysAdmin
		- Propagate: ✅
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


### 💾 Disk Encryption

***NOTE**: A safer approach instead of full disk encryption is to install the Proxmox OS on an unencrypted disk, and use additional storage as encrypted disks for your production VM's. Proxmox does not offer full disk encryption by default. There are some issues you can encounter with this depending on your use case.*


**Mount an Encrypted Disk**

- 🌐 [How to Mount Existing Disks Within proxmox](https://forum.proxmox.com/threads/how-to-mount-existing-disk-to-storage.66559/)
- 🌐 [Adding LUKS Encryption to proxmox](https://forum.proxmox.com/threads/adding-full-disk-encryption-to-proxmox.137051/)
- 🌐 [Add an Existing Disk to proxmox](https://forum.proxmox.com/threads/adding-an-existing-hdd-to-proxmox.118361/)
- 🌐 [Add a Dew Disk as LVM-thin](https://forum.proxmox.com/threads/adding-a-disk-and-set-it-as-lvm-thin-help-needed-please.111724/)
- 🌐 [proxmox Storage Wiki](https://pve.proxmox.com/wiki/Storage)
- 🌐 [Dr Duh Yubikey Guide: Backing Up Keys on Encrypted Storage](https://github.com/drduh/YubiKey-Guide?tab=readme-ov-file#backup-keys)

You can easily mount and add an encrypted disk to an existing Proxmox server over SSH or from the Proxmox server console in the Web UI. As mentioned above, a perfect example is you've installed Proxmox itself on an unencrypted drive, and have other internal drives you can encrypt with LUKS or ZFS encryption that you want to store VM's on.

!!! tip "Creating a LUKS Partition"

	The easiest way to do this (and the way I initially set this up) is from an Ubuntu or Debian live OS. Using the `disks` application in Ubuntu, you can easily format and create a LUKS encrypted drive. Poweroff the Proxmox server, reboot into a live USB, create  the drive, then continue.

    Alternatively, you can [do this all manually from the CLI](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Encrypting_devices_with_cryptsetup) within proxmox. ***NOTE**: this will need tested and verified.*

    ```bash
    cryptsetup luksFormat /dev/sdXY
    cryptsetup open --type luks /dev/sdXY <storage-id>
    # Enter a passphrase, store it in your password manager

    # Only create an ext4 filesystem if you need it, otherwise use lvm-thin for VM storage instead of ext4
    mkfs.ext4 /dev/mapper/luks3-lvm
    ```

Now that we have a LUKS partition, we can mount it.

Get a sense of which partition(s) you'll want to add. In this case, the device is `sdd` and the partition is `sdd1`.

```bash
lsblk
<SNIP>
sdd                            8:48   0   1.8T  0 disk
└─sdd1                         8:49   0   1.8T  0 part
```

**Unlock an Encrypted Disk**

Unlock and mount the LUKS partition. You can make up any name for a storage ID, just stick to alphanumeric characters and hyphens `-`. **Remember, you would only `mkdir` and `mount` a filesystem if you're using it as a filesystem. VM's benefit from lvm-thin data pools for more efficency than a filesystem**.

!!! note "Naming the Storage ID"

	If you look under Datacenter > Storage, you'll see the default ID's are `local` and `local-lvm`.

    In this example, lets say you already have two other internal disks that are also LUKS encrypted.

    Since this is the third disk, we'll call it `luks3-lvm` to stick to a similar naming convention.

```bash
# Example: cryptsetup open --type luks /dev/sddX <storage-id>
# This is all you need if the encrypted partition is acting as a storage pool for VMs
cryptsetup open --type luks /dev/sdd1 luks3-lvm

# If this will be a filesystem and NOT an lvm-thin storage pool
mkdir /mnt/2tb-internal3
mount /dev/mapper/luks3-lvm /mnt/2tb-internal3

# To unmount the filesystem, and close the the LUKS partition if you need to
umount /mnt/2tb-internal3
cryptsetup close <storage-id>
```

To add the encrypted partition to the Proxmox data pool (basically so you see it in the GUI and can make use of it), you can use:

- Datacenter > Storage > Add
- ID: `luks3-lvm` (storage identifier created with cryptsetup)

!!! tip "New vgname and thinpool"

	Create a new volume group and thinpool for LUKS drives to share. This is because adding them to the existing volume group and thinpool means VM's and data will still be written to the unencrypted drives (if you aren't using full disk encryption).

!!! note "Ai Usage"

    ChatGPT showed examples that led to posts and documentation detailing the following commands.

...or the CLI with `pvesm`:

```bash
# Helpful to visualize the pve storage
pvesm status

# Create a new volume group for LUKS drives
vgcreate luks-group /dev/mapper/luks3-lvm
WARNING: ext4 signature detected on /dev/mapper/luks3-lvm at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/mapper/luks3-lvm.
  Physical volume "/dev/mapper/luks3-lvm" successfully created.
  Volume group "luks-group" successfully created

# If you mess up, delete the group and remake it
vgremove luks-group

# Create a new thinpool for LUKS drives to share, leaving 5% of the drive for metadata
lvcreate --type thin-pool -l95%FREE -n <thinpool-name> <vgname>
lvcreate --type thin-pool -l95%FREE -n luks-data luks-group
  Thin pool volume with chunk size 1.00 MiB can address at most 254.00 TiB of data.
  WARNING: Pool zeroing and 1.00 MiB large chunk size slows down thin provisioning.
  WARNING: Consider disabling zeroing (-Zn) or using smaller chunk size (<512.00 KiB).
  Logical volume "luks-data" created.

# Add a LUKS thin-lvm data pool for VM storage
pvesm add lvmthin luks3-lvm --vgname pve --thinpool data --content images --content rootdir # lvm-thin is more efficient than a filesystem for VM storage

# Add a LUKS filesystem for backup, ISO, or other use
pvesm add dir luks3-lvm --path /mnt/2tb-internal3 --content backup # Only for backup, browsable filesystems
```

If you mess up and want to redo the pvesm, just use `pvesm remove <storage-id>`. Remember we came up with the storage ID when unlocking the LUKS partition during `cryptsetup open --type luks /dev/sddX <storage-id>`.

---

**Backup LUKS Headers**

- 🌐 [Fedora Project: Backup LUKS Headers](https://docs.fedoraproject.org/en-US/quick-docs/encrypting-drives-using-LUKS/#_backup_luks_headers)
- 🌐 [gitlab/cryptsetup: Backup and Data Recovery](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/FrequentlyAskedQuestions#6-backup-and-data-recovery)
- 🌐 [Arch Linux: LUKS Backup and Restore](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Backup_and_restore)

Once everything's working it's **recommended to backup the headers of your LUKS devices**. These are ~2MB or larger binary blobs, the first "area" of the LUKS disk containing the encrypted key slots and necessary data to unlock the drive. Having a backup of these is easy to store as an attachment in a password vault, and can recover a drive if anything corrupts the header.

Ideally you have two encrypted drives that both have copies of the data, for example two LUKS drives, one for running the VMs, the other for saving backups of the VMs. It's less likely both will fail at the same time, even less so if the "backup" drive is an external device that isn't always connected.

```bash
cryptsetup luksHeaderBackup --header-backup-file <file> <device>
cryptsetup luksHeaderBackup --header-backup-file /root/sddX-luks3-lvm.bin /dev/sddX
```

!!! note "Header Backup Storage"

	In most cases, it's fine to leave the header binary here in `/root` of the Proxmox unencrypted drive. An attacker would still need the key along with the header, and if the device itself is physically stolen, the LUKS volume itself also contains the header. This is a small trade off for having access to your header backup. Assume an attacker who could brute force your drive's key would already know how to extract the header, and has physical access.

    That said, you may choose to save the header to your password manager and delete it from proxmox.

    ```bash
    # Destroy the header file
    shred -n 7 -v /root/sddX-luks3-lvm.bin
    ```

Before you're done you can do two more things; if you're using this partition as a filesystem `type=dir`, or you want to use a keyfile to unlock your drives on boot, set autodecryption and set auto-mounting.

!!! warning "Keyfile Storage"

	Typically what you'd do is use full disk encryption on the OS drive, requiring a password over SSH with dropbear_initramfs on remote startup. This will decrypt the boot drive. This is where you would be safe to store keyfiles. Do not store keyfiles on unencrypted partitions, as it would be easy to decrypt your drives with physical or shell access.

```bash
# Without a keyfile, simply map the drive but still require a passphrase
echo "luks3-lvm UUID=$(blkid -s UUID -o value /dev/sdd1 | tr -d '\n') none luks" | tee -a /etc/crypttab

# With a keyfile stored in the filesystem root, literally as /keyfile
echo "luks3-lvm UUID=$(blkid -s UUID -o value /dev/sdd1 | tr -d '\n') /keyfile luks" | tee -a /etc/crypttab

# Mount the volume as a filesystem under /mnt (we created this path eariler)
echo 'UUID=$(blkid -s UUID -o value /dev/sdd1 | tr -d '\n') /mnt/2tb-internal3    ext4    defaults 0   2" >> /etc/fstab
```


**Unlocking the Drives**

From here, anytime you update and reboot the Proxmox server, you'll just need to paste this command into the terminal through the WebUI and enter your passphrase to unlock the LUKS disk. **Be sure to always use the same storage ID when mounting the LUKS partition, so pvesm can access it**.

```bash
cryptsetup open --type luks /dev/sddX <storage-id>
cryptsetup open --type luks /dev/sddX luks3-lvm
```

That's it. It's already mapped to your pve storage manager, and remembered there.

---

**Full Disk Encryption**

*One option is to install Debian with FDE, then install Proxmox on top of it. However it's recommended to just install Proxmox following one of the paths below.*

1. Install Proxmox with ZFS
2. Reboot into recovery mode from the installer USB
3. Follow [these steps](https://gist.github.com/yvesh/ae77a68414484c8c79da03c4a4f6fd55) to enable ZFS encryption
4. Follow [these steps](https://github.com/openzfs/zfs/tree/master/contrib/initramfs#unlocking-a-zfs-encrypted-root-over-ssh) to install `dropbear-initramfs` for remote decryption over SSH
5. Encrypt the dataset for VMs and containers (which is also step 5 in [this post](https://forum.proxmox.com/threads/encrypting-proxmox-ve-best-methods.88191/#post-387731))

ZFS + Dropbear, all of the resources below came from this post: [FDE with ZFS](https://forum.proxmox.com/threads/full-disk-encryption-with-zfs-using-proxmox-installer.127512/)

- 🌐 [Feature Request: Native ZFS encryption during Proxmox installation](https://bugzilla.proxmox.com/show_bug.cgi?id=2714)
- 🌐 [Encrypting Proxmox VE: Best Methods](https://forum.proxmox.com/threads/encrypting-proxmox-ve-best-methods.88191/#post-387731)
- 🌐 [gist: FDE with Proxmox and ZFS native encryption](https://gist.github.com/yvesh/ae77a68414484c8c79da03c4a4f6fd55)
- 🌐 [github/openzfs: Unlocking ZFS encrypted root over SSH](https://github.com/openzfs/zfs/tree/master/contrib/initramfs#unlocking-a-zfs-encrypted-root-over-ssh)
- 🌐 [Proxmox VE ZFS encryption](https://pve.proxmox.com/wiki/ZFS_on_Linux#zfs_encryption)

LUKS + Dropbear

- 🌐 [Adding LUKS FDE to Proxmox](https://forum.proxmox.com/threads/adding-full-disk-encryption-to-proxmox.137051/)

---

## 🔀 Migrating VMs

- 🌐 [Importing Virtual Machines](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#qm_import_virtual_machines)
- 🌐 [Migrating Hyper-V to Proxmox](https://forum.proxmox.com/threads/migrating-hyper-v-to-proxmox-what-i-learned.137664/)


**Hyper-V**

Follow [the link referenced above](https://forum.proxmox.com/threads/migrating-hyper-v-to-proxmox-what-i-learned.137664/). Essentially, with no snapshots on the Hyper-V VM you wish to migrate (you can export the VM and import it back into Hyper-V so it's separate from your "main" copy, to delete the snapshots, then export it again) export your Hyper-V VM so you have the .vhdx file. Move it over to the Proxmox filesystem.

!!! tip "Filesystem Space and Migrating VMs"

	An external drive or larger filesystem space within Proxmox is useful here, as by default Proxmox gives its default "filesystem" on the OS drive roughly 100GB while leaving the remaining space to lvm-thin pool storage for VMs.

    This is not explicitly talked about in the post, but if you cannot access the .vhdx file on your Hyper-V host by mounting a share on to the Hyper-V host on Proxmox, you should use an external drive to transfer the VM. **With the average size likely being over 40GB, it will take a long time to `scp`, `rsync`, or similar**.

    Since the OS drive has roughly 100GB on the root filesystem, the majority of the remaining space is lvm-thin space, which is a special type of storage format efficient for VM's but lacking a filesystem.

    Both an external drive and a fileshare allow you to import the VM directly to the lvm-thin space without accidentally overruning your /root filesystem's space which can happen if you copy the VM directly onto the OS drive filesystem before importing it.

**Create the VM in Proxmox**

Create the VM as you would if you were making it from scratch, following the steps in the linked forum post (q35 for the VM type, and then detach the default disk). Use the `qm disk import <vmid> <source> <storage> [OPTIONS]` command to import the .vhdx file as the disk image for your Hyper-V VM.

```bash
qm disk import 105 /mnt/wd500-external/Virtualization/Wazuh/Virtual\ Hard\ Disks/Wazuh.vhdx luks3-lvm
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
Successfully imported disk as 'unused1:luks3-lvm:vm-102-disk-3'
```

Add (attach) the disk under the VM's settings. **Be sure to go into "Options", not "Hardware" and set this disk first in the boot order**. In the event this is a Linux VM, even with SecureBoot enabled you'll be able to boot it, and get running.

That's really it, be sure to install the `spice-vdagent` service on the guest to leverage additional virtualization features if you want them, though this is not necessary.

Fix any issues (such as networking), poweroff, then take your first snapshot.

---


## ⚙️ VM Configuration

### 🛜 Virtual NIC

This is the equivalent of creating a virtual NIC (not to be confused with a VLAN) in VMware or VirtualBox for guest-to-guest communication (for example a LAN and OPT NIC to attach to a pfSense VM). First you need to create another Linux Bridge network device that isn't attached to any physical network ports.

- Under Datacenter, select your proxmox-ve hostname > System > Network > Create
- Name it, check `[x]` autostart, create
- Now click "Apply Configuration" at the top, so your new `vmbrX` shows as "Active: Yes"

Now when creating the VM:

- `[VM Name]` > Hardware > Add > Network Device
- Add two, the WAN side can be the default `vmbr0`, the LAN side is your new `vmbrX` with either `intel E1000` or `virtIO`
- One will receive DHCP / access to the outside world over the Proxmox virtual brdige
- The other will remain purely virtual (for example pfSense could then provide DHCP to that NIC)


### 💾 Disk Passthrough

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


## Troubleshooting

*This section details issues and how to resolve them.*


### UEFI Causes Boot Failure

Typically in Unix-like distros where UEFI is supported but Secure Boot is not, you must interrupt the boot process and turn off Secure Boot.

- During boot, when you see the Proxmox splash screen, hit `Esc`
- Select `Device Manager`
- Select `Secure Boot Configuration`
- Uncheck `Attempt SecureBoot` with the `Space` bar
- Back out to the main UEFI menu and select `Continue`
