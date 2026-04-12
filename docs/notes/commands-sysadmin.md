---
title: "Commands: Systems"
icon: material/keyboard
draft: true
#date:
#  created: 2025-08-24
#  updated: 2026-04-10
categories:
  - how-to
  - sysadmin
  - users
  - login
  - management
  - devops
  - logging
  - bash
  - powershell
  - reference
---


# :material-keyboard: System Administration

Useful commands for managing systems.

<!-- more -->


## Linux / Unix


### General

!!! note "What's in this section?"

	These are commands you might find on any Linux, BSD, or even embedded systems.

```bash
w             # Show who's logged in, what they're doing
last | head   # List of last logged in users
```


### apt

Essential maintenance.

```bash
sudo apt update -q
	sudo PATH="$PATH":/usr/bin \
	DEBIAN_FRONTEND=noninteractive \
	NEEDRESTART_MODE=a \
	apt full-upgrade -y \
	-o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold'
	sudo apt autoremove --purge -yq
	sudo apt-get clean
```

!!! info "Apt Options"

    `DEBIAN_FRONTEND=noninteractive` sets apt to run without live input from a user or admin.

    `NEEDRESTART_MODE=a` is an Ubuntu-specific setting (from 22.04 LTS and later) that automatically restarts services when necessary.

    Options are `a` to auto-restart services unattended, `l` to only list services needing a restart, or `i` to prompt interactively.

    - <https://github.com/liske/needrestart/issues/109>
    - <https://github.com/liske/needrestart/blob/master/man/needrestart.1>

    Even with the previous variables set, you may be prompted to manage configuration file changes via dpkg, for example when you've modified a file and an update ships a new one. `-o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold'` will automate this.

    - <https://wiki.debian.org/AutomatedUpgrade>
    - <https://manpages.debian.org/bullseye/debconf-doc/debconf.7.en.html#Frontends>
    - <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html#parameter-dpkg_options>

Package review commands.

```bash
apt-mark showauto         # Lists all packages installed automatically / by default
apt-mark showmanual       # Lists all packages YOU installed
apt-cache stats           # Show apt-cache stats
apt-cache show <pkg>      # Show all information about a package
apt-cache depends <pkg>   # Lists all dependencies of a package
apt-cache rdepends <pkg>  # Lists what package(s) depend on this one

dpkg -S <pattern>  # Search for a filename from installed packages
dpkg -l <pattern>  # List packages matching given pattern
dpkg -s <pkg>      # Report status of specified package
dpkg -L <pkg>      # List files installed to your system from package-name

```

---


### Systemd

**Systemd Commands**

> systemd-analyze may be used to determine system boot-up performance statistics and retrieve other state and tracing information from the system and service manager, and to verify the correctness of unit files. It is also used to access special functions useful for advanced system manager debugging.

```bash
systemd-analyze security [<unit>]  # Reviews sandbox settings of units, calculates risk score
```

**Systemd `*ctl` Commands**

- **Pattern:** all `*ctl` tools share `status`, `list`, and tab-completion on unit/resource names.
- Some commands use `--json=[short|pretty]` (hostnamectl, networkctl, etc) or `-o json` (journalctl) to dump the output to JSON
- Some commands use `--no-pager` to dump results to stdout instead of a pager
- Add `-l` to most commands to avoid truncated output.

`systemctl` - Service/unit management

```bash
systemctl status [<unit>|<pid>]  # Runtime status + latest journal logs
systemctl list-units [--failed]  # List units systemd has loaded in memory
systemctl list-timers            # All timers in memory, orderd by next run time
systemctl cat <unit>             # Cat the systemd unit file
systemctl show <unit>            # Shows low-level properties of units
systemctl is-active <unit>       # Returns true if the unit is active
systemctl is-enabled <unit>      # Returns true if the unit is enabled
systemctl is-failed <unit>       # Returns true if the unit has failed
```

`journalctl` - Log viewer

```bash
journalctl -b [<id>]             # Show logs from "BOOT_ID=<id>" or current boot if none
journalctl -u <unit> -f          # Tail a systemd service unit in real time
journalctl --since "10 min ago"  # Search by time
journalctl -p err                # Show errors only
journalctl <args>  --no-pager    # Print to stdout, not to a pager
journalctl <args>  -o json       # JSON output
```

`hostnamectl` - System hostname data

```bash
hostnamectl                       # Show hostname and device details (virtualization, chassis info, etc)
hostnamectl hostname <hostname>   # Set the hostname
hostnamectl <args> --json=pretty  # JSON oputput
```

`networkctl` - systemd-networkd status (only complete if networkd is running)

```bash
networkctl                       # Overview
networkctl status [<iface>]      # Overview + details
networkctl reload                # Reload .netdev and .network files
```

`resolvectl` - DNS data (only sees systemd-resolved and resolvconf info)

```bash
resolvectl status                # Show DNS settings currently in effect
resolvectl statistics            # DNSSEC status, resolution and validation stats
resolvectl query <hostname>      # Can use --type= or --class= to resolve records
resolvectl flush-caches          # Flush all local caches
```

`timedatectl` - Timezone, NTP data

```bash
timedatectl status                # Current time, timezone, NTP sync status
timedatectl list-timezones        # Lists all available timezone options
timedatectl set-timezone <tz>     # Sets timezone
timedatectl set-ntp <true|false>  # Enable or disable existing time synchronization service
```

`loginctl` - Session/user/seat management

```bash
loginctl list-sessions           # List current sessions
loginctl show-session <id>       # Show details of a session
loginctl lock-session <id>       # Locks a session
loginctl unlock-session <id>     # Unlocks a session
loginctl terminate-session <id>  # Kills all processes and deallocates resources of a session
```

`coredumpctl` - Retrieve and process core dumps

```bash
coredumpctl list                 # Lists a table of core dumps captured in the journal
coredumpctl info <pid|exe>       # Show details about <pid|exe>'s last core dump
coredumpctl debug <exe>          # Invoke gdb on <exe>'s last core dump
```

`machinectl` - VM / container management (sudo apt install systemd-containers)

```bash
machinectl list                  # List all active machines
machinectl list -o json          # JSON output
machinectl shell <machine>       # Open an interactive shell within <machine>
machinectl status <machine>      # Shows runtime status and journal log of <machine>
```

---


### Xorg

!!! warning "TODO"

	Mirror Wayland commands here if possible


### Wayland

Wayland and modern Linux systems may use systemd-logind to handle sessions and power / suspend decisions. `acpid` may still be present on some systems, but is more of a legacy component that's been replaced by `logind`.

Confirm you're on wayland:

```bash
echo $XDG_SESSION_TYPE
```

Get a list of sessions:

```bash
loginctl session-status
```

Dump the system's default session configuration:

```bash
loginctl show-session
```

List all system power action inhibitors; in other words, what-does-what when a poweroff / suspend / or hibernate request takes place.

```bash
systemd-inhibit --list
```
