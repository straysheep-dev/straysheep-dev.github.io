---
title: "USB Reasearch"
icon: fontawesome/brands/usb
draft: true
#date:
#  created: 2022-08-29
#  updated: 2025-12-25
categories:
  - usb
  - malware
  - research
  - threat-hunting
  - security
  - awareness
---

# :fontawesome-brands-usb: USB Research

This post is a compilation of my notes covering all things related to USB and my research into the security considerations surrounding the USB protocol and devices. It's been updated to present the notes in a more practical order, one you could follow if you were simply curious on the topic. This research exists to augment everything that's already in MITRE ATT&CK.

After reading, you should have an understanding of:

- How USB works (protocol and devices)
- USB-based attacks
- Safely enumerating USB devices
- Baselining, detection, and mitigating USB threats

This is primarily targeting Linux and Windows machines.

<!-- more -->


!!! abstract "References"

    This research was started using the following references.

    - [MITRE ATT&CK Removeable Media Overview](https://attack.mitre.org/techniques/T1091/)
    - [USB Device Class Codes](https://www.usb.org/defined-class-codes)
    - [USBGuard](https://github.com/USBGuard/usbguard)
    - [USBGuard Class Codes Rule Implementation](https://usbguard.github.io/documentation/rule-language.html)
    - [USB DFU Specifications](https://usb.org/sites/default/files/DFU_1.1.pdf)
    - [Example Showing Steps Taken In Writing BadUSB Firmware to RubberDucky Device](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Flashing-ducky)
    - [Flashrom (Read, Write, Edit firmware)](https://github.com/flashrom/flashrom)
    - [Linux Kernel USB Authorization](https://www.kernel.org/doc/html/latest/usb/authorization.html)
    - [Systemd-Mount](https://www.freedesktop.org/software/systemd/man/systemd-mount.html#)
    - [Kali forensic menu hooks](https://gitlab.com/kalilinux/build-scripts/kali-live/-/blob/fb799cf349c1c72ab6a9bd9804ef346376f2c8d3/kali-config/common/hooks/live/forensic-menu.binary)
    - [xfce Docs: Removable Media](https://docs.xfce.org/xfce/thunar/using-removable-media)
    - [autofs daemon](https://www.kernel.org/doc/html/latest/filesystems/autofs.html)
    - [CISA: Removeable media best practices](https://www.cisa.gov/news-events/news/using-caution-usb-drives)
    - [CISA: 0-Day - Downloading a payload when Windows Explorer processes a crafted shortcut](https://us-cert.cisa.gov/ics/advisories/ICSA-10-201-01C)

!!! tip "Utilities to Get Started"

    USBGuard is not strictly necessary, but is universally available regardless of the desktop (or lack of) environment.

    ```bash
    # Install, on Debian-family OS's it should automatically approve
    # all actively connected USB controllers and devices at the time
    # of install
    sudo apt install usbguard

    # List all devices and their information
    usbguard list-devices

    # Monitor bus activity
    usbguard watch

    # Permanently allow a device after reviewing its capabilities
    sudo usbguard allow-device -p <id>
    ```

    These Linux utilities are used to interface with and manipulate USB devices and their embedded firmware:

    ```bash
    sudo apt install dfu-util
    sudo apt install dfu-programmer
    sudo apt install flashrom
    ```

    The following filesystem locations relate to USB logs or devices:

    ```bash
    # Serial Devices
    /dev/ttyUSB*

    # Kernel Files
    /sys/bus/usb/devices/*

    # Logs
    /var/log/kern.log
    /var/log/syslog
    /var/log/user.log
    ```


## How USB Works

Wikipedia has a full history on [USB](https://en.wikipedia.org/wiki/USB) and [USB connections](https://en.wikipedia.org/wiki/USB_communications).

Generally, USB is handled first via the machine's firmware, then the OS's kernel, before anything in user-land sees it. [USB device class codes](https://www.usb.org/defined-class-codes) describe what the device can do. DFU mode is one thing that's unique here.


### DFU Mode

[Device Firmware Upgrade (DFU) mode](https://usb.org/sites/default/files/DFU_1.1.pdf) exposes the following class codes: `{ FE:*:* }`, `{ FE:01:* }`, `{ FE:01:01 }`

| Base Class  | Sub Class   | Protocol    |
| ----------- | ----------- | ----------- |
|   `FE`      |   `01`      |  ` 01`      |

This is how DFU mode works:

- While being upgraded, the USB device cannot act as a USB device (if it was a keyboard, it loses keyboard functionality during DFU mode)
- Linux requires dfu instrumentation locally (`dfu-programmer`, `dfu-util`) to flash chips
    - These tools can be embedded in other tools (e.g. a program to flash updates to your USB device)
    - They often require root privileges to perform their functions
    - In any case, the tools must exist locally in some way for the device to enter DFU and be modified
- The device hash will change in USBGuard
- The device class code(s) will change if the firmware alters functionality
- Vendor, manufacturer, serial number, and other index string descriptors can be changed

---


## USB-based Attacks

!!! quote "[Replication Through Removable Media: T1091](https://attack.mitre.org/techniques/T1091/)"

    Adversaries may move onto systems, possibly those on disconnected or air-gapped networks, by copying malware to removable media and taking advantage of Autorun features when the media is inserted into a system and executes. In the case of Lateral Movement, this may occur through modification of executable files stored on removable media or by copying malware and renaming it to look like a legitimate file to trick users into executing it on a separate system. In the case of Initial Access, this may occur through manual manipulation of the media, modification of systems used to initially format the media, or modification to the media's firmware itself.

    Mobile devices may also be used to infect PCs with malware if connected via USB. This infection may be achieved using devices (Android, iOS, etc.) and, in some instances, USB charging cables. For example, when a smartphone is connected to a system, it may appear to be mounted similar to a USB-connected disk drive. If malware that is compatible with the connected system is on the mobile device, the malware could infect the machine (especially if Autorun features are enabled).

!!! quote "[Communication Through Removable Media: T1092](https://attack.mitre.org/techniques/T1092/)"

    Adversaries can perform command and control between compromised hosts on potentially disconnected networks using removable media to transfer commands from system to system. Both systems would need to be compromised, with the likelihood that an Internet-connected system was compromised first and the second through lateral movement by Replication Through Removable Media. Commands and files would be relayed from the disconnected system to the Internet-connected system to which the adversary has direct access.


### AutoPlay/Exec/Run

AutoRun, AutoExec, and AutoPlay are features that will automatically execute data from removable media ***when it's mounted***. This primarily affects Windows and some Linux Desktop environments. macOS [does](https://github.com/drduh/macOS-Security-and-Privacy-Guide) [not](https://support.apple.com/en-us/102522) [appear](https://www.reddit.com/r/macsysadmin/comments/zk9145/disabling_autorun_from_removable_drives_macos/) [to](https://learn.microsoft.com/en-us/defender-endpoint/mac-device-control-overview) [have](https://superuser.com/questions/316414/autorun-file-on-usb-drive-in-os-x) [this](https://support.apple.com/en-us/102282) [issue](https://docs.velociraptor.app/artifact_references/).

On Windows this behavior is split into both, [AutoRun and AutoPlay](https://learn.microsoft.com/en-us/windows/win32/shell/autorun-and-autoplay-bumper).

[Using and Configuring Autoplay](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/legacy/cc144212(v=vs.85)) is specifically for media files (e.g. music) rather than applications.

[AutoRun](https://en.wikipedia.org/wiki/Autorun.inf) is the mechanism through which, if enabled, external media can execute anything described in the [Autorun.inf Entries](https://learn.microsoft.com/en-us/windows/win32/shell/autorun-cmds), which is a text file in the root of the external media drive. These are just ini-like key/values interpretted by Windows if it's allowed to AutoRun external media. [This example](https://learn.microsoft.com/en-us/windows/win32/shell/autorun-cmds#usage-example) illustrates what these files can contain, and what each key means. This is useful for both forensics and understanding their offensive capabilities.

With Linux, this behvaior depnends on your desktop environment.

| Desktop Environment Docs | Autorun Optional | Default Setting | Prompt Required |
| --- | --- | --- | --- |
| [Windows: AutoRun.INF](https://learn.microsoft.com/en-us/windows/win32/shell/autoplay-reg) | ⚠️ | [Disabled by Default via Patch](https://en.wikipedia.org/wiki/Autorun.inf#Abuse) | NO |
| [GNOME: Files & Autorun](https://help.gnome.org/gnome-help/files-autorun.html) | ✅ | OFF | YES |
| [KDE: Removable Devices](https://docs.kde.org/stable5/en/plasma-desktop/kcontrol/solid-device-automounter/index.html) | ❌ | N/A | N/A |
| [xfce: Managing Removable Drives and Media](https://docs.xfce.org/xfce/thunar/using-removable-media#managing_removable_drives_and_media) | ✅ | OFF | YES |



### BadUSB

!!! quote "From the Flipper Docs"

    A BadUSB device can change system settings, open backdoors, retrieve data, initiate reverse shells, or do basically anything that can be achieved with physical access. It is done by executing a set of commands written in the Rubber Ducky Scripting Language, also known as DuckyScript. This set of commands is also called a payload.

[BadUSB](https://docs.flipper.net/zero/bad-usb) attacks were made popular by [Hak5's Rubber Ducky](https://github.com/hak5/usbrubberducky-payloads?tab=readme-ov-file#about-the-new-usb-rubber-ducky), which is a USB drive designed to act with HID (keyboard and mouse) capabilities via the device's firmware. The dead giveaway is a device having the class codes for both, something else that's the intended purpose of the device (e.g. mass storage, WiFi adapater, etc) and HID (`03:xx:xx`) when you aren't expecting it to have typing and mouse capabilities.

BadUSB devices can include anything with a USB port:

- External drives
- KB/Mouse (yes, with hidden payload injection)
- WiFi, bluetooth adapters
- Power brick with USB ports
- USB hub
- ...and more!

DuckyScript being a programmable language means you can embed logic for delayed execution. A sneaky vector would be to delay for hours, days, or weeks depending on the device, and only make small modifications. This is why access-controls based on device class codes and hashes is the best detection and mitigation here.

!!! tip "Can My USB Become Bad?"

    An existing USB device **can become a BadUSB if the firmware is modifiable**. Typically, this requires that the device must support DFU mode, and subsequently be set to DFU mode to receive a firmware controller update before reverting from DFU mode back to "normal" operating mode, only now that includes HID payload injection.

    Flashing firmware often requires the flashing tools exist locally, or via web-USB in your browser, and prompt you for root or sudo privileges.

    If there's an undocumented way to modify the USB device's firmware, that's of course also possible, but becomes way less common and requires a targeted effort even with all the right tools and privileges locally available to modify the device, either in a target environment or ahead of time by the adversary.

You can see how common these are in [GitHub topics for BadUSB payloads](https://github.com/topics/badusb-payloads).


### Firmware Attacks

These were described briefly in the previous section on BadUSB, and include the notes above on [DFU mode](#dfu-mode).

It's less likely, but possible to do remotely, within a target environment, if all the right tools and access are available.

Even more unlikely is the firmware of the USB device having a payload designed for a target's hardware + firmware layer, or the kernel components that handle USB devices. More research is needed here.

---


## Examining Untrusted Devices

### Usbguard

On a system running [USBguard](https://github.com/USBGuard/usbguard), you can simply connect the external device to see if it has a USB interface and what it's capable of. This is an easy way to confirm devices like chargers or power bricks are just that, and not anything more.

```bash
usbguard watch
```

Once connected, details and capabilities will be printed to your terminal window without giving the device any USB capabilities on your system. Details also include a hash of the device's interface, so you could even detect firmware changes to devices previously approved in USBguard this way; or establish a baseline hash for new and trusted devices.


### Kali Linux

Kali makes this easy by having forensics tools and good defaults built-in via forensics mode.

- [Kali live usb -> forensic mode](https://www.kali.org/docs/general-use/kali-linux-forensics-mode/)
- Live environment kernel and files are read-only via [Squashfs](https://www.kernel.org/doc/html/latest/filesystems/squashfs.html)
- Does not automatically mount any storage devices by default

!!! tip "Risk"

    Regarding the sample device under examination attacking the Kali live USB or other firmware during review:

    - Kali Live in forensic mode does not do anything without user interaction (think AutoRun or AutoMount)
    - `fwupdmgr get-devices --show-all-devices > devices.out` can generate a summary of the internal device details before you start
    - `usbguard list-devices` and commands like `lsusb` can also give you a baseline before starting
    - Ideally you're on a dedicated test machine, like a [Raspberry Pi 5, that has a signed chain of firmware](https://github.com/raspberrypi/usbboot/blob/master/docs/secure-boot-chain-of-trust-2712.pdf)

When you're ready to start ensure [USBGuard](https://github.com/USBGuard/usbguard) is installed:

```bash
sudo apt install -y usbguard
```

Ensure after install there is no `5: allow id *:* label "GNOME_SETTINGS_DAEMON_RULE"` or similar present.

```bash
sudo usbguard list-rules

```

In the above example the rule id is `5`. Remeove this rule with:

```bash
sudo usbguard remove-rule 5
```

Open a new terminal window and set usbguard daemon to watch:

```bash
usbguard watch
```

From here you're ready to mount the device you're examining. This will print the devices manufacturer ID, serial number, name, a hash of the device tied to its firmware, hash of the parent connector (the usb port on the system), a list of **interfaces**, and connection type if available.

Example output:

```
2: block id 1234:abcd serial "123456789" name "USB Worm Device" hash "1234567890abcdefghijklmnopqrstuvwxyz=" parent-hash "1234567890abcdefghijklmnopqrstuvwxyz=" via-port "1-1" with-interface {08:01:02 03:01:23} with-connect-type "unknown"
```

As mentioned above, seeing interfaces `08:*:*` and `03:00:*` aka 'USB Mass Storage' and 'HID/Keyboard' together on a simple flash drive indicates a keyboard interface or similar functionality was added to the firmware. It's fair to assume this device can or will attempt to autotype commands once mounted.


#### Forensic Mode (noautomount)

What is Kali doing behind the scenes to achieve this? And can Ubuntu live do the same? (Short answer is yes)

Looking at [kali-config/common/hooks/live/forensic-menu.binary](https://gitlab.com/kalilinux/build-scripts/kali-live/-/blob/fb799cf349c1c72ab6a9bd9804ef346376f2c8d3/kali-config/common/hooks/live/forensic-menu.binary) shows that `noautomount` is a live hook, but where is that parameter being read and by what?

Is the following the same as setting...

```
/org/gnome/desktop/media-handling/automount -> false
/org/gnome/desktop/media-handling/automount-open -> false
/org/gnome/desktop/media-handling/autorun-never -> true
```

...in a GNOME environment? Or is more happening at the kernel level to prevent any read/write operations on external media?

From [systemd-mount](https://www.freedesktop.org/software/systemd/man/systemd-mount.html#), the `--automount` and `--discover` flags seem to handle removable media, however it's unclear if these options are about the same thing kali is doing in forensic mode, as they facilitate connections with the kernel happening AFTER user opts to open the media device in the file system (my interpretation).

**Manually enabling auto-mounting of media in the Thunar file explorer enables this feature once you log out and back in during a live session** - this seems to indicate the `noautomount` hook sets a preference, similar to setting site preferences with dconf in `/etc/dconf/db/site.d/locks`

Looking in other relevant places, there are no configurations on Kali Live for the following:

- `/etc/fstab`
- `/etc/udev/rules.d/`
- `noauto` is not a kernel parameter or a udisks2 parameter...

See:

- `man systemd.mount`
- `man systemd.automount`
- `man systemd-mount`

!!! example "Example from the systemd Manual"

    Use a udev rule like the following to automatically mount all USB storage plugged in:

    ```
    ACTION=="add", SUBSYSTEMS=="usb", SUBSYSTEM=="block", ENV{ID_FS_USAGE}=="filesystem", \
            RUN{program}+="/usr/bin/systemd-mount --no-block --automount=yes --collect $devnode"
    ```

Configurations to `/etc/udisks2/udisks2.conf` are the same for Kali Live and Ubuntu's defaults

How does this all compare with an Ubuntu / GNOME environment configured with the following?

```bash
cat /etc/dconf/db/site.d/00_media-handling
# Disable autorun and automount of software and external media
[org/gnome/desktop/media-handling]
autorun-never=true
automount=false
automount-open=false

cat /etc/dconf/db/site.d/locks/media-handling
# Disable autorun and automount of software and external media
/org/gnome/desktop/media-handling/automount
/org/gnome/desktop/media-handling/automount-open
/org/gnome/desktop/media-handling/autorun-never
```

Can the same be achieved? We can review the relevant log files to confirm:

Running the following...

- `tail -f /var/log/kern.log`
- `tail -f /var/log/syslog`
- `tail -f /var/log/user.log`

...reveals filesystems are not mounted until explicitly mounted by the user.

**Both Kali in forensics mode and Ubuntu with AutoMount/Open disabled, output the same logging data.**

You can further verify this in the GUI by launching `disks` and reviewing the status of the device, this will show it as unmounted or (encrypted if applicable).

---


## Detection & Mitigation

This section covers defensive strategies for combatting USB threats.


### USBGuard

[USBGuard](https://github.com/USBGuard/usbguard) is by far the most robust tool for protecting your unix-like system(s) from USB-based threats. It will prevent unknown, or unauthorized devices from being seen by any system driver by using the kernel to allow or deny it based on attributes and rules.

The [Arch Linux Wiki](https://wiki.archlinux.org/title/USBGuard) has a similar write up to this one, with references to the implementation in GNOME that will be shared below.

```bash
# Install, on Debian-family OS's it should automatically approve
# all actively connected USB controllers and devices at the time
# of install
sudo apt install usbguard

# List all devices and their information
usbguard list-devices

# Monitor bus activity
usbguard watch

# Permanently allow a device after reviewing its capabilities
sudo usbguard allow-device -p <id>
```

That's all there is to using it. It's recommended to perform this install on a **test system first** using the Linux distribution you intend to run on real hardware. This way you can catch if anything goes wrong during install or configuration. USBGuard runs just fine on QEMU / virt-manager, VMWare, VirtualBox, Hyper-V, and similar virtualization software that supports USB pass-through.

!!! warning "USB != SD"

    Interestingly enough this does ***not*** block SD cards, or anything that isn't a USB device.


#### USBGuard on GNOME

GNOME had a [research project in 2018 design to increase USB protection](https://wiki.gnome.org/Internships(2f)2018(2f)Projects(2f)USB(2d)Protection.html). This appears to have resulted in the following settings being added to GNOME's system key/value store.

- [Key: /org/gnome/desktop/privacy/usb-protection-level](https://documentation.ubuntu.com/adsys/stable/reference/policies/User%20Policies/Ubuntu/Desktop/Shell/Privacy/usb-protection-level/)
- [Key: /org/gnome/desktop/privacy/usb-protection](https://documentation.ubuntu.com/adsys/stable/reference/policies/User%20Policies/Ubuntu/Desktop/Shell/Privacy/usb-protection/)

USBGuard seems to be required for this work. This just allows you to handle USBGuard via GNOME's settings if you want. To enforce these via dconf or gsettings, follow similar steps to those detailed below in the [Disabling Autorun, Autoplay, and Automount Linux section](#linux-gnome).


### Device Management via Group Policy

[Preventing installation of prohibited devices via Group Policy](https://learn.microsoft.com/en-us/windows/client-management/client-tools/manage-device-installation-with-group-policy#scenario-steps--preventing-installation-of-prohibited-devices) can be used to emulate what USBGuard does, but on Windows, with the native Windows tools. This can be complicated, but detailed instructions are available for a number of scenarios.

- [Scenario #1: Prevent installation of all printers](https://learn.microsoft.com/en-us/windows/client-management/client-tools/manage-device-installation-with-group-policy#scenario-1-prevent-installation-of-all-printers)
- [Scenario #2: Prevent installation of a specific printer](https://learn.microsoft.com/en-us/windows/client-management/client-tools/manage-device-installation-with-group-policy#scenario-2-prevent-installation-of-a-specific-printer)
- [Scenario #3: Prevent installation of all printers while allowing a specific printer to be installed](https://learn.microsoft.com/en-us/windows/client-management/client-tools/manage-device-installation-with-group-policy#scenario-3-prevent-installation-of-all-printers-while-allowing-a-specific-printer-to-be-installed)
- [Scenario #4: Prevent installation of a specific USB device](https://learn.microsoft.com/en-us/windows/client-management/client-tools/manage-device-installation-with-group-policy#scenario-4-prevent-installation-of-a-specific-usb-device)
- [Scenario #5: Prevent installation of all USB devices while allowing an installation of only an authorized USB thumb-drive](https://learn.microsoft.com/en-us/windows/client-management/client-tools/manage-device-installation-with-group-policy#scenario-5-prevent-installation-of-all-usb-devices-while-allowing-an-installation-of-only-an-authorized-usb-thumb-drive)

!!! quote "Layers of Connectivity"

    Some device in the system have several layers of connectivity to define their installation on the system. USB thumb-drives are such devices. Thus, when looking to either block or allow them on a system, it's important to understand the path of connectivity for each device. There are several generic Device IDs that are commonly used in systems and could provide a good start to build an 'Allow list' in such cases. See below for the list:

    `PCI\CC_0C03`; `PCI\CC_0C0330`; `PCI\VEN_8086`; `PNP0CA1`; `PNP0CA1&HOST` (for Host Controllers)/ `USB\ROOT_HUB30`; `USB\ROOT_HUB20` (for USB Root Hubs)/ `USB\USB20_HUB` (for Generic USB Hubs)/

    Specifically for desktop machines, it's very important to list all the USB devices that your keyboards and mice are connected through in the above list. Failing to do so could block a user from accessing its machine through HID devices.

    Different PC manufacturers sometimes have different ways to nest USB devices in the PnP tree, but in general this is how it's done.

Generally you'll need to enumerate USB devices with either `Get-WmiObject Win32_USBControllerDevice ...` or `cmd.exe /c pnputil`. This is detailed over in the [Device Management section of the Windows notes](./windows.md#device-management).


### Disabling Autorun, Autoplay, and Automount

#### Windows

[SANS Blog: Configure Windows Forensic Workstations](https://www.sans.org/blog/digital-forensics-how-to-configure-windows-investigative-workstations/) is a great reference for Windows here in general.

Disable AutoRun:

```powershell
# HKLM
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\policies\Explorer /v NoDriveTypeAutoRun /t REG_DWORD /d 0xFF /f

# HKCU
reg add HKCU\Software\Microsoft\Windows\CurrentVersion\policies\Explorer /v NoDriveTypeAutoRun /t REG_DWORD /d 0xFF /f
```

Disable AutoPlay:

```powershell
# Per user, AutoPlay setting
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers" -Name "DisableAutoplay" -Type DWord -Value 1
```

Disable Automount

```powershell
diskpart.exe; automount disable;
# or
mountvol /N
# or
reg add HKLM\SYSTEM\CurrentControlSet\Services\MountMgr /v NoAutoMount /t REG_DWORD /d 1 /f
```


#### Linux (xfce)

Mentioned in the earlier section on [AutoRun.INF](#autoplayexecrun), by default, xfce does *not* auto-mount or auto-run any media. [xfce: Managing Removable Drives and Media](https://docs.xfce.org/xfce/thunar/using-removable-media#managing_removable_drives_and_media) describes this behavior. Even if you allow this, it appears to still prompt you for confirmation when an app tries to launch.


#### Linux (KDE)

The [KDE: Removable Devices](https://docs.kde.org/stable5/en/plasma-desktop/kcontrol/solid-device-automounter/index.html) docs don't appear to have any details on an AutoRun/Open/Exec feature.


#### Linux (GNOME)

See this [example guide](https://help.gnome.org/admin/system-admin-guide/stable/desktop-background.html.en) and [dconf lockdown explaination](https://help.gnome.org/admin/system-admin-guide/stable/dconf-lockdown.html.en).

1. Create the following user profile:

`/etc/dconf/profile/user`

```
user-db:user
system-db:site
```

!!! note "dconf Database Names"

    "***site***" is the name of the system database.

2. For each database other than `user` you listed in the profile, create `/etc/dconf/db/<databse>.d/` if it doesn't already exist

`sudo mkdir /etc/dconf/db/site.d`

3. Set the keys within a keyfile within the `local` database directory:

`sudo nano /etc/dconf/db/local.d/00_media-handling`:

```
[org/gnome/desktop/media-handling]
automount=false
automount-open=false
autorun-never=true
```
4. Create the lock directory, and a lock file with one path to lock per line, relating to the keyfile just set:

`sudo mkdir /etc/dconf/db/local.d/locks`

`sudo nano /etc/dconf/db/local.d/locks/media-handling`:

```
/org/gnome/desktop/media-handling/automount
/org/gnome/desktop/media-handling/automount-open
/org/gnome/desktop/media-handling/autorun-never
```

5. Update the database

`sudo dconf update`

6. Users must logout and back in for the system-wide settings to take effect.

7. Confirm by trying to change a locked key. You should receive the following output:

```bash
gsettings set org.gnome.desktop.media-handling automount true

"The key is not writable"
```

### Filesystem Protections

What this means is any process or executable spawning from a mounted or removable share or drive is suspicious, and should generally be outright blocked.


#### Windows (Filesystem ASR Rules)

On Windows, you'll want to enable the attack surface reduction (ASR) rule to [block untrusted and unsigned processes that run from USB](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-reference), or your security platform's equivalent.


#### Linux (Filesystem Capabilities)

On Linux, you can configure partitions (including mount points) to not execute binaries via the `noexec` directive in `/etc/fstab`. You should also configure `nodev` and `nosuid` on these paths as well, but they're often the default.

The [Ubuntu 22.04 CIS L2 Server Guide: noexec on /dev/shm](https://static.open-scap.org/ssg-guides/ssg-ubuntu2204-guide-cis_level2_server.html#xccdf_org.ssgproject.content_rule_mount_option_dev_shm_noexec) has Ansible and Bash snippets, ready-to-use, that show you how to do this. You're adding the `noexec` flag to partitions under `/etc/fstab`.

To test this, you can copy an existing binary out of the OS path:

```bash
# Copy the id bin to /dev/shm
cp /usr/bin/id /dev/shm/

# Execute it to confirm you can
/dev/shm/id
uid=1000(user1) gid=1000(user1) groups=1000(user1),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd)
```

Now if you were to copy the remediation script block above into a `noexec.sh` and run it, you'll see binaries no longer execute on that path:

```bash
# Create the remediation script from https://static.open-scap.org/ssg-guides/ssg-ubuntu2204-guide-cis_level2_server.html#xccdf_org.ssgproject.content_rule_mount_option_dev_shm_noexec
nano ./noexec.sh
sudo bash ./noexec.sh

# Confirm id no longer executes
/dev/shm/id
-bash: /dev/shm/id: Permission denied
```

!!! warning "Limitations"

    This does NOT prevent script execution (`.py`, `.sh`, `.rb`, `.ps1`), if they're executed via the relevant binaries already installed within the OS (e.g. `/usr/bin/*`).

    Here's a test you can do to confirm this, write a small python script that writes "pwned" to terminal, following up from our `id` test earlier:

    ```bash
    # This does not work
    chmod +x /dev/shm/test.py
    /dev/shm/test.py
    -bash: /dev/shm/test.py: Permission denied

    # This does work
    python3 /dev/shm/test.py
    pwned

    # Binary files are still blocked, even when using a system binary to run them
    bash /dev/shm/id
    /dev/shm/id: /dev/shm/id: cannot execute binary file
    ```

    See [Securing Debian: Chapter 04, Section 10](https://www.debian.org/doc/manuals/securing-debian-manual/ch04s10.en.html) for more on setting `noexec` `nodev` and `nosuid` on filesystem partitians.

`cat /proc/mounts` or `mount -l` will show the filesystem and device capabilities:

```txt
tmpfs on /run type tmpfs (rw,nosuid,nodev,noexec,relatime[...]
```

Notable syscalls for auditd rules:

- `execve`, `execl`, `execlp`, `execle`, `execv`, `execvp`, `execvpe`: Execute a file (all front-ends for execve, syscall 59)
- `fork`: Process fork/clone
- `clone`: Process fork/clone
- `ptrace`: Process injection
- `openat`: Open or create a file
- `connect`: Connection, can be local or remote

!!! tip "Monitor your Mount Path"

    Mount paths like `/media` are defined either as a default, or by the user. You could mount an external filesystem *anywhere*. Because of this, you need to take one of two things into account:

    - Either have dynamic auditing rules that will catch all process creation (you can filter based on path then)
    - Only permit mounting to specific paths, that have unique auditing rules watching them for process creation

Example auditd policy for detection:

```md
# /etc/audit/rules.d/40-usb.rules
-a always,exit -F arch=b32 -F dir=/media/ -S socketcall -F key=T1091.1_Replication_Through_Removable_Media
-a always,exit -F arch=b64 -F dir=/media/ -S connect -F key=T1091.1_Replication_Through_Removable_Media

-a always,exit -F arch=b32 -F dir=/media/ -S open -F key=T1091.2_Replication_Through_Removable_Media
-a always,exit -F arch=b64 -F dir=/media/ -S openat -F key=T1091.2_Replication_Through_Removable_Media

-a always,exit -F arch=b32 -F dir=/media/ -S clone -S fork -F key=T1091.3_Replication_Through_Removable_Media
-a always,exit -F arch=b64 -F dir=/media/ -S clone -S fork -F key=T1091.3_Replication_Through_Removable_Media

-a always,exit -F arch=b32 -F dir=/media/ -S execve -F key=T1091.4_Replication_Through_Removable_Media
-a always,exit -F arch=b64 -F dir=/media/ -S execve -F key=T1091.4_Replication_Through_Removable_Media
```

Relevant excerpts from the exec, execve, capabilities, and mount man pages:

!!! quote "execve"

    `execve(2)`

    `execve()`  executes  the  program referred to by pathname.  This causes the program that is currently being run by the calling process to be replaced with a new program, with newly initialized stack, heap, and (initialized and uninitialized) data  segments.

!!! quote "capabilities"

    `capabilities(7)`

    If a thread drops a capability from its permitted set, it can never reacquire that capability  (unless  it  execve(2)s either a set-user-ID-root program, or a program whose associated file capabilities grant that capability).

!!! quote "exec"

    `exec(3)`

    `p` - `execlp()`, `execvp()`, `execvpe()`

    These  functions  duplicate  the  actions of the shell in searching for an executable file if the specified filename does not contain a slash (/) character.  The file is sought in the colon-separated list of directory pathnames specified in  the  PATH environment  variable.   If  this  variable isn't defined, the path list defaults to a list that includes the directories returned by confstr(_CS_PATH) (which typically returns the value "/bin:/usr/bin") and possibly also the current working  directory; see NOTES for further details.

    If  the  specified  filename includes a slash character, then PATH is ignored, and the file at the specified pathname is executed.

    In addition, certain errors are treated specially.

    If permission is denied for a file (the attempted execve(2) failed with the error  EACCES),  these  functions  will  continue searching the rest of the search path.  If no other file is found, however, they will return with errno set to EACCES.

    If  the  header of a file isn't recognized (the attempted execve(2) failed with the error ENOEXEC), these functions will execute the shell (/bin/sh) with the path of the file as its first argument.  (If this attempt fails, no  further  searching  is done.)

    All  other  exec()  functions  (which do not include 'p' in the suffix) take as their first argument a (relative or absolute) pathname that identifies the program to be executed.

!!! quote "mount"

    `mount(8)`

    noexec Do not permit direct execution of any binaries on the mounted filesystem.


### Physical & Firmware Security

These suggestions fall slightly outside of USB itself, but consider attacks that can be done with physical access and a USB.

!!! tip "Set a Firmware / Boot / UEFI Password"

    Requiring this password during administrative UEFI actions / boot sequences can prevent adversaries with physical access from booting via unauthorized live USB's. Kali Linux for example could be run from a USB and used to mount the drives, modify data on unencrypted boot partitions, modify the device firmware, and more.

!!! tip "Enforce Secure Boot"

    If combined with a firmware password, and other low-tech physical security techniques that can help you detect chassis intrusion, you can have a reasonable level of confidence in your boot chain integrity if the CMOS and TPM values have not been cleared and your Secure Boot signatures have not changed.
