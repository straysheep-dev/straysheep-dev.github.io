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


# :material-keyboard: System Administration Commands

*Useful commands for managing systems.*

<!-- more -->
---


## Linux / Unix


### General

!!! note "What's in this section?"

	These are commands you might find on any Linux, BSD, or even embedded systems.

```bash
w             # Show who's logged in, what they're doing
last | head   # List of last logged in users
```


### apt

**Essential Maintenance**

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

**Package Review Commands**

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

**OS Major Version Upgrades**

*Commands and detached systemd unit trick provided by Claude Sonnet 5.5*

!!! note "Major Versions Tested"

    The steps below were tested on:

    - amd64 Nzyme node, successfully upgrading from 22.04 LTS to 26.04 LTS
    - arm64 Nzyme tap, successfully upgrading from Debian 12 (bookworm) to 13 (trixie)

    The upgrade is slower than rebuilding in `packer`, but may be the only option in some cases. It can take roughly 30-60 minutes per major version jump.

!!! note "Detached Systemd Unit"

    This is ideal instead of a tmux session because it writes the output to the system journal and won't go away if the shell (or tmux) dies mid-upgrade.

=== "Ubuntu"

    ```bash
    # Pre-flight: apt will refuse a major version upgrade until everything is patched.
    # Optionally, hold any third party packages.
    sudo apt update && sudo apt full-upgrade -y
    sudo apt-mark hold tailscale wazuh-agent
    grep ^Prompt /etc/update-manager/release-upgrades   # want Prompt=lts
    sudo dpkg --audit
    sudo reboot
    ```

    Run the upgrade as a detached systemd unit (writes to journal).

    Read the command below; adjust it as needed based on the third-party packages you have installed.

    ```bash
    # Detached upgrade, no prompts. Edit THIRD_PARTY to the repo files you want restored.
    # Uses environment variables to dynamically handle the OS codenames.
    sudo systemd-run --unit=osupgrade --collect \
      --setenv=DEBIAN_FRONTEND=noninteractive \
      --setenv=NEEDRESTART_MODE=a \
      --setenv=OLD_CODENAME="$(. /etc/os-release; echo $VERSION_CODENAME)" \
      --setenv=THIRD_PARTY="tailscale.list" \
      bash -c '
        echo "Dpkg::Options { \"--force-confold\"; \"--force-confdef\"; };" \
          > /etc/apt/apt.conf.d/99-upgrade-noprompt
        trap "rm -f /etc/apt/apt.conf.d/99-upgrade-noprompt" EXIT
        do-release-upgrade -f DistUpgradeViewNonInteractive || exit 1
        . /etc/os-release            # now reports the new VERSION_CODENAME
        for f in $THIRD_PARTY; do
          sed -i "s/$OLD_CODENAME/$VERSION_CODENAME/g" \
            /etc/apt/sources.list.d/$f
        done
      '

    journalctl -fu osupgrade     # watch; then reboot manually when it exits 0
    ```

=== "Debian"

    Official instructions: [debian.org/releases/trixie/release-notes](https://www.debian.org/releases/trixie/release-notes/upgrading.en.html)

    This process is very similar to upgrading Proxmox major versions, you're literally pointing the apt sources to the next major version codename. This is different from Ubuntu's `do-release-upgrade` which will comment out third-party repos and handle this for you.

    ```bash
    # Pre-flight
    sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y
    sudo apt-mark hold tailscale wazuh-agent
    sudo dpkg --audit && sudo reboot
    ```

    Run the upgrade as a detached systemd unit (writes to journal).

    Read the command below; adjust it as needed based on the third-party packages you have installed.

    ```bash
    sudo systemd-run --unit=osupgrade --collect \
      --setenv=DEBIAN_FRONTEND=noninteractive \
      --setenv=NEEDRESTART_SUSPEND=1 \
      --setenv=OLD_CODENAME="$(. /etc/os-release; echo $VERSION_CODENAME)" \
      --setenv=NEW_CODENAME=trixie \
      bash -c '
        set -e
        echo "Dpkg::Options { \"--force-confold\"; \"--force-confdef\"; };" \
          > /etc/apt/apt.conf.d/99-upgrade-noprompt
        trap "rm -f /etc/apt/apt.conf.d/99-upgrade-noprompt" EXIT

        cp -a /etc/apt "/root/apt-backup-$OLD_CODENAME"

        # THIS is the step that changes releases (covers .list and .sources)
        grep -rl "$OLD_CODENAME" /etc/apt/sources.list /etc/apt/sources.list.d 2>/dev/null \
          | xargs -r sed -i "s/$OLD_CODENAME/$NEW_CODENAME/g"

        apt update
        apt upgrade --without-new-pkgs -y
        apt full-upgrade -y
      '

    journalctl -fu osupgrade     # watch; reboot manually when it exits 0
    ```

When the upgrade completes, you'll want to see what errors need addressed, if anything:

```bash
systemctl status osupgrade   # want inactive, exit 0
# Determine what to do next based on the output.

# Sometimes you'll need to reconfigure a broken package install.
sudo dpkg --configure -a
sudo apt -f install

# In many cases, a reboot will resolve any package errors.
# Be sure you know you will still have (remote) access to the system after rebooting.
# For example, Tailscale is a static go binary, and the service will still come right
# back after upgrades. Check this on a test system first.
sudo systemctl reboot
```

Check the system health after reboot.

!!! note "Disabled Repos"

    Be sure to review everything under `/etc/apt/sources.list.d/` to fix any issues with third-party repos that were disabled for the upgrade.

```bash
sudo apt-mark unhold tailscale wazuh-agent
systemctl --failed
systemctl status
```

---


### snap

!!! info "Expanded Notes"

    See the [existing note dedicated to snap](./snap.md).

!!! tip "Snap Author Verification"

    When searching for packages with `snap find`, verified authors and publishers have a `✓` in their name. These accounts have been trusted by the snap project maintainers.

Update all snaps, or a specific snap `[package]`:

```bash
sudo snap refresh [package]
```

Revert a snap package to a previous version, if an update broke functionality:

```bash
# Get all versions available
snap list <package-name> --all

# Refresh to another version
sudo snap refresh <package-name> --revision=<number>
```

Pin the package version indefinitely, until the issue is fixed:

```bash
# Apply the hold
sudo snap refresh --hold <package-name>

# Remove the hold
sudo snap refresh --unhold <package-name>
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
systemctl list-timers            # All timers in memory, ordered by next run time
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
hostnamectl <args> --json=pretty  # JSON output
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
