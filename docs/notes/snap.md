---
draft: false
date:
  created: 2024-05-07
  updated: 2025-06-15
categories:
  - snap
  - guide
  - package-managers
  - cli
---

# :simple-snapcraft: snap

Notes related to the [snap package manager](https://snapcraft.io/about).

<!-- more -->

*This file is originally from [straysheep-dev/linux-configs](https://github.com/straysheep-dev/linux-configs/tree/main/snap).*

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
