---
draft: false
date:
  created: 2022-08-29
  updated: 2025-06-15
categories:
  - vulnerability
  - platform
  - scanning
  - network
  - nessus
---

# :material-magnify-scan: Nessus

!!! abstract "What does this post cover?"

    Nessus is possibly the most well-known vulnerability scanner out there. [It was originally open source before going closed source / commercial](https://en.wikipedia.org/wiki/Nessus_(software)#History). At that point, development split off into the [OpenVAS](https://en.wikipedia.org/wiki/OpenVAS) project.

    Having tried OpenVAS first, after configuring and setting it up, it was clear when moving over to Nessus, that also had similar processes to install and configure it. For example, back when these notes were taken, the web interface was accessible from any network interface by default (this still appears to be the case). There's also a setting related to checking the signatures of the scanner plugins after downloading which wasn't often talked about.

    Installing Nessus regularly was more or less required for a number of pentesting courses, so this page was created to document those steps a long time ago. This is just a port of those notes with some review of the currrent documentation.

!!! warning "Snapshot in Time"

    These are my notes from 2022 and earlier on setting up Nessus in a lab environment. They may be outdated, and were only checked against the latest documentation when porting to this blog post. They have not been tested.


<!-- more -->


## Install

- <https://www.tenable.com/products/nessus>
- [Download](https://www.tenable.com/downloads/nessus)
- [Documentation](https://docs.tenable.com/Nessus.htm)

```bash
sudo dpkg -i ./Nessus-10.5.1-debian10_amd64.deb
# or
sudo apt install ./Nessus-10.5.1-debian10_amd64.deb
```

You can start the Nessus Scanner by typing
```bash
sudo /bin/systemctl start nessusd.service
/bin/systemctl is-active nessusd
# or
sudo /etc/init.d/nessusd start
```

Then go to `https://<hostname>:8834/` to [complete the initial setup](https://docs.tenable.com/nessus/10_8/Content/ConfigureNessus.htm).

!!! info "Nessus Essentials Registration"

    You can easily re-register a Nessus Essentials Trial directly from the application if you need a new one / or reinstalled Nessus.


## Configure

!!! note ""

    <https://docs.tenable.com/nessus/Content/ConfigureNessus.htm>

It's [recommended to configure which interface address `nessusd` accepts connectons on](https://docs.tenable.com/nessus/Content/NessusService.htm?Highlight=listen_address#Considerations):

```bash
sudo /opt/nessus/sbin/nessuscli fix --secure --set listen_address=127.0.0.1
```

By default, the web interface binds to all network interfaces.

!!! tip ""

    Alternatively you can set the listening address from the WebUI under settings/advanced, as sometimes the `nessuscli` won't work.

Apply configuration changes:

```bash
sudo systemctl restart nessusd.service
```

Review all advanced settings:

```bash
sudo nessuscli fix --show
```

!!! important "Plugin Signature Checking"

    Ensure `nasl_no_signature_check` = `FALSE` (or no). This requires all nasl files downloaded to be valid and signed by Tenable.

    <https://www.tenable.com/plugins/nessus/179042>


## Updating

[Update both the nessus core components and scanner plugins](https://docs.tenable.com/nessus/Content/NessusCLI.htm#Software_Update_Commands):

```bash
sudo systemctl stop nessusd.service
sudo /opt/nessus/sbin/nessuscli update --all
sudo systemctl start nessusd.service
```

This means you can install a recent version of the `.deb` or `.rpm` binary you may have stored locally, and fetch the latest updates from the cli.


## nessuscli

!!! note "Nessuscli"

    A command line tool exists under `/opt/nessus/sbin/nessuscli` on Linux to run and administer Nessus.

    <https://docs.tenable.com/nessus/Content/NessusCLI.htm>

    The majority of this section was taken from the documentation (linked above) based on what commands appeared to be the most useful or important to remember for setup or regular use **in a lab scenario**.

General usage:

```bash
nessuscli <command> [<options>]
nessuscli <command> help
```

Bug Reporting Commands:

```bash
bug-report-generator
bug-report-generator --quiet [--full] [--scrub]
```

User Commands:

```bash
rmuser [username]
chpasswd [username]
adduser [username]
lsuser
```

Fix Commands:

```bash
# Lists all possible settings
fix --show

# To navigate and modify the settings
fix [--secure] --list
fix [--secure] --set <name=value>
fix [--secure] --get <name>
fix [--secure] --delete <name>
```

Backup Tool (backs up your Nessus settings, not scan data):

```bash
backup --create </path/to/filename>
backup --restore </path/to/filename>
```

Software Update Commands:

```bash
update
update --all
update --plugins-only
update <plugin archive>
```


## Scanning

!!! note ""

    <https://docs.tenable.com/nessus/Content/Scans.htm>

**Setting the Source IP**

- Settings > Miscilleneous > Scan Source IP(s)
- Source IPs to use when running on a multi-homed host. If multiple IPs are provided, Nessus will cycle through them whenever it performs a new connection.

**Port Ranges**

- Port ranges can be specified like with `nmap`
- 80-5000
- 22,25,80,445,8080

**Pause / Abort**

- Pausing a scan will abort it if you sign out before resuming


## Results

You can disable grouping to display findings individually (rather than sets of vulnerabilities in groups like "MIXED" or "CRITICAL").
