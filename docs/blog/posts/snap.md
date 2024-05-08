---
draft: false
date: 2024-05-07
categories:
  - snap
  - guide
  - package-managers
  - cli
---

# snap

Files related to the [snap package manager](https://snapcraft.io/about).

<!-- more -->

---

## Snap Security

A quick overview of the security behind `snap` packages.

### Application Integrity

If you download a snap with `snap download <package-name>` you will get two files:

| File                    | Description                                                                                         |
| ----------------------- | --------------------------------------------------------------------------------------------------- |
| `<package-name>.snap`   | The snap application itself                                                                         |
| `<package-name>.assert` | Plain text file containing mulitple signatures required to validate the integrity of the .snap file |

This is a good way of archiving or backing up specific versions of snap packages.

Install the snap package with:

```bash
sudo snap ack <package-name>.assert
sudo snap install ./<package-name>.snap
```

Any attempt to modify or tamper with the `.assert` file will result in snapd failing to acknowledge those signatures, ultimately preventing the install of that `.snap` package.

### Runtime Isolation

Snap packages are isolated, almost like chroot jails, in that they cannot access / read anything owned by `root`, or anything belonging to another `snap` within `/snap` or `~/snap` as well as any `.hidden` files and directories (among other optional / user defined locations and resources).

They also have limited access to binaries available on the host.

**NOTE**: This can create issues with screensharing windows of other snap applications while conferencing.

It's always **recommended** to run your browser in some form of additional system level sandbox (snap or flatpak for example) due to the nature of web-based threats. Windows does this in Microsoft Edge with [Application Guard](https://docs.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-application-guard/md-app-guard-overview). This opens any resources defined as untrusted to your organization in a temporary Hyper-V container, essentially a temporary VM to further isolate those processes from the host in addition to a number of other protections.

In the case of snaps, sharing windows of other snap applications is no longer possible due to the sandbox (which is both good and bad). You can share your entire screen, which is not always the best option. However if your other applications are not snap or flatpak applications, there's no issue sharing these windows via a snap-based web browser.

One trick is opening a separate browser tab as it's own window, and opening the files you need to share in that browser tab while sharing that window.

Additionally with many resources being web based, it's likely you have the files stored with a cloud provider. Sharing a folder of specific files in a single browser tab window like this can also work well.

#### Exceptions to Isolation

Any package that requires the `--classic` argument during install will **NOT** be sandboxed, and is exactly like a traditional `.deb` package installed with `apt`.

#### Reviewing Isolation

To visualize how the runtime isolation works we can open shells in the context of snap applications, as if we were an attacker that compromised a particular snap application's process:

```bash
$ snap run --shell firefox
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

$ pwd
~/

$ whoami
bash: /usr/bin/whoami: Permission denied

$ sudo -l
bash: /usr/bin/sudo: Permission denied

$ cat /etc/shadow
cat: /etc/shadow: Permission denied

$ cat ~/.ssh/id_rsa
cat: ~/.ssh/id_rsa: Permission denied
```

We can limit what snaps can access, for example to block access to the entire home directory:

```bash
$ echo 'test' > ~/Documents/read.txt

$ snap run --shell firefox

$ cat ~/Documents/read.txt
test

$ exit
exit

$ sudo snap disconnect firefox:home
[sudo] password for user:

$ snap run --shell firefox

$ cat ~/Documents/read.txt
cat: read.text: Permission denied
```

In the above case, you would save downloads directly to the ~/snap/firefox/* directories instead of ~/Downloads.

Next, we'll look at network connections.

Elements from this section largely come from HackTricks:

<https://github.com/carlospolop/hacktricks/blob/master/linux-unix/useful-linux-commands/bypass-bash-restrictions.md>

```bash
$ nc -nvlp 8080
Listening on 0.0.0.0 8080

# Checking listening ports from another shell
$ sudo ss -anp -A inet
tcp  LISTEN   0   1    0.0.0.0:8080    0.0.0.0:*     users:(("nc",pid=1234,fd=3))

# Connecting via curl from another shell
Connection received on 127.0.0.1 50482
GET / HTTP/1.1
Host: 127.0.0.1:8080
User-Agent: curl/7.68.0
Accept: */*

# Connecting out to another resource
$ nc -nv 127.0.1.1 8080
Connection to 127.0.1.1 8080 port [tcp/*] succeeded!

$ exit
exit
```

Now we'll remove network access from the application:

```bash
# Disconnecting network access
$ sudo snap disconnect firefox:network
[sudo] password for user:

$ snap run --shell firefox
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

# Try to open a listener
$ nc -nvlp 8080
bash: /usr/bin/nc: Permission denied

# Try to connect to a listener
$ nc -nv 127.0.1.1 8080
bash: /usr/bin/nc: Permission denied

$ (sh)0>/dev/tcp/127.0.1.1/8080
bash: socket: Permission denied
bash: /dev/tcp/127.0.1.1/8080: Permission denied

# Try another connection
$ python3 -c 'import pty; pty.spawn("/bin/sh")' ; nc -nv 127.0.1.1 8080

$ nc -nv 127.0.1.1 8080
/bin/sh: 3: nc: Permission denied

$ exit
bash: /usr/bin/nc: Permission denied
```

Interestingly with snap you can enumerate whether or not a local resource exists:

```bash
$ cat /etc/ufw/user.rules
cat: /etc/ufw/user.rules: Permission denied

$ cat /etc/ufw/user.rulesbutnotreally
cat: /etc/ufw/user.rulesbutnotreally: No such file or directory
```

## Tools

Additional snap related tools in this repository.

#### snap-up.sh

This tool attempts to collect and backup all current user created snap files.

Saving snap files in their own 'home' folders under `~/snap/<snap-package>/*` prevents other snaps from reading or accessing those files. This tool is an automated way of backing up all of those files to a directory that has the same protections under `~/snap/backups/`.

Files are named based on backup date and time, and copied to `/home/$USERNAME/snap/backups/<snap-package>/<backup-filename>`

It's essentially mirroring that data for recovery purposes.

You can set an additional mirror destination with rsync.

There's also a builtin option to safely empty that default `/home/$USERNAME/snap/backups/` folder used by this tool, if for example you've already copied those backups to the cloud or external storage and want to free the disk space.

Usage:

```
$ snap-up --help

NAME
       snap-up - Shell script to backup snap files created by the user

SYNOPSIS
       snap-up [OPTIONS]

DESCRIPTION
       Without any options, this will backup all non-hidden files in the latest (current) directory of each snap package.
       
       If a snap has been uninstalled the backups currently existing for it will not be deleted without -c|--clean.

       Specifically it looks under: /home/$USERNAME/snap/<package> for /home/$USERNAME/snap/<package>/current/<files>
       to backup <files> to: /home/$USERNAME/snap/backups/<package>/<date/time><file>

       Example: /home/ubuntu/snap/chromium/current/example.data  ->  /home/ubuntu/snap/backups/chromium/2022-05-01_11:22:33_example.data
       Example: /home/ubuntu/snap/gedit/current/file with spaces.txt  ->  /home/ubuntu/snap/backups/gedit/2022-05-01_11:22:33_file_with_spaces.txt

       ~/snap/backups is used because snaps cannot read from other snap directories, even with the "home" connection.

OPTIONS
    -c, --clean
       Clean allows you to empty the backup directory located in "/home/$USERNAME/snaps/backups". Considered dangerous, the operation has two prompts for
       confirmation before doing anything. This is typically run after backing up the latest files to external media or cloud storage to save disk space.

    -l, --list
       Print an easily readable list of all currently backed up files in "/home/$USERNAME/snaps/backups". This will also check if the environment variable 
       SET_SYNC_PATH has a path, and read from there as well.

    -s, --sync
       The sync option will let you specify a path to rsync the "/home/$USERNAME/snaps/backups" directory to. This can be run multiple times with the 
       same sync location, and rsync automatically updates only what was changed, added, or removed from the source "/home/$USERNAME/snaps/backups" folder.
       
       If you wanted to automate this, you can export the sync arguments as environment variables. The script will walk through prompts to do this for you.
       
       export SYNC_CHOICE="y"
       export SET_SYNC_PATH="/tmp"
       snap-up -s
       
       Add them to bashrc for persistence.
       Running with the -s|--sync switch will automatically sync to the target folder. Remove the environment variables interactively with:
       
       unset SYNC_CHOICE
       unset SET_SYNC_PATH
       or
       export SYNC_CHOICE=""
       export SET_SYNC_PATH=""
       
       or remove the lines from ~/.bashrc

    -t, --test
       Perform a test run without making any changes. This will show all discovered files to be backed up, and their destination.
```
