---
title: "bind9"
icon: material/dns
draft: false
#date:
#  created: 2024-07-19
#  updated: 2025-06-15
categories:
  - how-to
  - administration
  - guide
  - home-lab
  - git
  - source
  - dns
  - bind
  - chroot
  - apparmor
  - linux
description: How to install, maintain, and run a BIND9 DNS server.
---

# :material-dns: BIND9 DNS

How to install, maintain, and run a BIND9 DNS server (`named`). Covers building from source, configuring, hardening, and DNS over TLS as well as DNSSEC.

<!-- more -->

> BIND (Berkeley Internet Name Domain) is a complete, highly portable implementation of the Domain Name System (DNS) protocol.

- [Website](https://www.isc.org/bind/)
- [Git](https://gitlab.isc.org/isc-projects/bind9)
- [ISC BIND9 Knowledge Base](https://kb.isc.org/docs/using-this-knowledgebase)
- [Administrator Reference Manual](https://bind9.readthedocs.io/en/latest/index.html)

In a few cases, to have a better understanding of what's required when managing and working with `bind` / `named`, the documentation is spread across various platforms and git issue trackers; each with their own subtle differences. This guide tries to put some of that together in a useful sequence. Both Ubuntu and Fedora are touched on in each section, but this is primarily written from the perspective of building bind from scratch, without any pre-existing binaries, service files, mandatory access controls, or configurations, and installing it on Ubuntu.



## Install

[BIND9  Documentation: Installing and upgrading BIND](https://kb.isc.org/docs/aa-00648)

=== "Ubuntu"

	Ensure your firewall is configured (the defaults **will** allow external DNS access).

	```bash
	sudo ufw status verbose
	sudo iptables -S
	sudo ip6tables -S
	```

	Installing through `apt` preconfigures everything, the systemd service file, ~~chroot~~, and apparmor.

	```bash
	sudo apt update; sudo apt install -y bind9

	# Stop, disable systemd-resolved
	sudo systemctl disable systemd-resolved.service
	sudo systemctl stop systemd-resolved.service

	# Start, enable bind
	sudo systemctl enable bind
	sudo systemctl start bind

	# Confirm apparmor confinement
	sudo aa-status | grep named

	# Dump version information
	/usr/sbin/named -V
	```

=== "Fedora"

	Ensure your firewall is configured (the defaults won't allow external DNS access).

	```bash
	sudo firewall-cmd --list-all
	```

	Installing through `dnf` preconfigures everything, the systemd service file, chroot, and selinux.

	```bash
	sudo dnf install -y bind

	# Stop, disable systemd-resolved
	sudo systemctl disable systemd-resolved.service
	sudo systemctl stop systemd-resolved.service

	# Start, enable named
	sudo systemctl enable named
	sudo systemctl start named

	# Confirm selinux status
	getsebool -a | grep named

	# Dump version information
	/usr/sbin/named -V
	```

=== "Source"

	When cloning the source from git, instructions for building BIND9 are located under [main/doc/arm/build.inc.rst](https://gitlab.isc.org/isc-projects/bind9/-/blob/main/doc/arm/build.inc.rst).

	You have [a few options when building to install or upgrade BIND9](https://kb.isc.org/docs/aa-00648#bind-is-built-so-now-what):

	- Running the new binaries from their new paths based on `--prefix=/usr/local` (difficult, system-wide changes)
	- Script replacing the existing binaries with new versions (stop `named`, replace / upgrade, restart `named`)
	- **Recommended**: Symlink the to the new `bin/` and `sbin/` paths (good for version changes and testing)

	For other features available when building, a small txt report is printed to screen after the build completes. Some essentials might include DNSSEC support through the openssl development package. Same for improved performance with the jemalloc development package.

	Install the required packages.

	```bash
	# Install required dev packages (ubuntu)
	sudo apt install -y make autoconf automake libtool libuv1-dev libcap-dev libssl-dev libxml2-dev libjson-c-dev libjemalloc-dev libnghttp2-dev liburcu-dev dns-root-data

	# Install required dev packages (fedora)
	sudo dnf install -y autoconf automake libtool libuv-devel libcap-devel openssl-devel libxml2-devel json-c-devel jemalloc-devel libnghttp2-devel userspace-rcu-devel
	```

	Clone the source, checkout a specific version.

	```bash
	mkdir ~/src
	cd ~/src
	git clone https://gitlab.isc.org/isc-projects/bind9.git
	cd bind9
	git tag               # choose a version tag
	version='stable'      # leave off the 'v' if using a numbered version
	git checkout $version # prepend the 'v' if using a numbered version
	```

	Build and `make install` using a unique local prefix path. This way you can build and maintain different versions.

	!!! quote "Build Options"

		The option `--sysconfdir` can be specified to set the directory where configuration files such as [`named.conf`](https://bind9.readthedocs.io/en/stable/manpages.html#std-iscman-named.conf) go by default; `--localstatedir` can be used to set the default parent directory of `run/named.pid`. `--sysconfdir` defaults to `$prefix/etc` and `--localstatedir` defaults to `$prefix/var`.

	!!! example "Build Options Continued"

		Basically, on Ubuntu we can use `--sysconfdir=/etc/bind --localstatedir='/var'`, since `/var/run` is a symlink to `/run`.

		It's entirely up to you if you would like to create and use `/etc/named/` for the conf files, you'll need to be sure to change this in your AppArmor profile.

	```bash
	autoreconf -fi  # (only if building from the git repository)

	# Useful for in-place upgrades and new installs
	version='stable'
	./configure --prefix=/usr/local/bind-$version --sysconfdir=/etc/bind --localstatedir='/var'

	make
	sudo make install
	```

	To illustrate how `--sysconfdir` and `--localstatedir` affect bind, we can run `named -V`.

	```bash
	default paths:
	named configuration:  /etc/bind/named.conf          # --sysconfdir=/etc/bind
	rndc configuration:   /etc/bind/rndc.conf           # --sysconfdir=/etc/bind
	DNSSEC root key:      /etc/bind/bind.keys           # --sysconfdir=/etc/bind
	nsupdate session key: /var/run/named/session.key    # --localstatedir=/var
	named PID file:       /var/run/named/named.pid      # --localstatedir=/var
	named lock file:      /var/run/named/named.lock     # --localstatedir=/var
	```

	**Optional**: Backup any existing, currently installed binaries.

	```bash
	# Backup existing binaries from package manager
	mkdir /usr/local/bind-pkg-mgr/{bin,sbin,lib} -p
	for i in /usr/local/bind-$version/sbin/*; do cp /usr/sbin/$(basename $i) /usr/local/bind-pkg-mgr/sbin/; done
	for i in /usr/local/bind-$version/bin/*; do cp /usr/bin/$(basename $i) /usr/local/bind-pkg-mgr/bin/; done
	sudo cp -r /usr/lib/named/* /usr/local/bind-pkg-mgr/lib/
	```

	[Create symlinks to the new binaries to "install" them](https://kb.isc.org/docs/aa-00648#bind-is-built-so-now-what).

	```bash
	# Symlink to new binaries compiled from source
	for i in /usr/local/bind-$version/sbin/*; do sudo ln -f -s $i /usr/sbin; done
	for i in /usr/local/bind-$version/bin/*; do sudo ln -f -s $i /usr/bin; done
	sudo rm -rf /usr/lib/named
	sudo ln -f -s /usr/local/bind-$version/lib /usr/lib/named
	```

	You will also need to manually create everything the prebuilt package typically installs if you're not updating an existing bind service:

	- `bind` (Ubuntu) /`named` (Fedora) user & group, it's up to you to choose which, it doesn't matter
	- Configuration files and folders, (chroot) paths
	- Systemd service files
	- AppArmor / SELinux policy

	All of this is covered below.



## Uninstall

Follow these steps to uninstall and remove the built `named` binaries, for instance if you want to rebuild them with different configuration arguments.

Stop `named` if it's running.

```bash
sudo systemctl stop named
```

Remove the built binaries from the prefix path.

```bash
sudo make uninstall
```

Remove any symlinks.

```bash
version='stable'
for i in /usr/local/bind-$version/sbin/*; do rm -f /usr/sbin/$(basename $i); done
for i in /usr/local/bind-$version/bin/*; do rm -f /usr/sbin/$(basename $i); done
sudo rm -f /usr/lib/named
```

Restore the original binaries.

```bash
for i in /usr/local/bind-pkg-mgr/sbin/*; do rm -f /usr/sbin/$(basename $i); cp -f $i /usr/sbin/; done
for i in /usr/local/bind-pkg-mgr/bin/*; do rm -f /usr/bin/$(basename $i); cp -f $i /usr/bin/; done
sudo mkdir /usr/lib/named
sudo cp -r /usr/local/bind-pkg-mgr/lib/* /usr/lib/named/
```

**Optional**: Clean the project directory.

```bash
make clean
```



## Initial Setup

To ensure bind9's `named` service is working out of the box after a fresh install:

=== "Ubuntu"

	Point your system to `bind`'s service on localhost.

	```bash
	sudo rm -f /etc/resolv.conf
	echo 'nameserver 127.0.0.1' | sudo tee /etc/resolv.conf
	```

	If name resolution isn't working, and you see `RRSIG validity period has not begun resolving './DS/IN'`errors in `journalctl -u named`, ensure the date / time is correct.

	```bash
	sudo systemctl restart systemd-timesyncd
	sudo date --set="2024-01-31 12:00:00 PM PDT"
	```

=== "Fedora"

	Point your system to `bind`'s service on localhost.

	```bash
	sudo rm -f /etc/resolv.conf
	echo 'nameserver 127.0.0.1' | sudo tee /etc/resolv.conf
	```

	If name resolution isn't working, and you see `RRSIG validity period has not begun resolving './DS/IN'`errors in `journalctl -u named`, ensure the date / time is correct.

	```bash
	sudo systemctl restart chronyd
	sudo date --set="2024-01-31 12:00:00 PM PDT"
	```

=== "Source"

	Add a user and group. If you built from source, it's your choice whether to pick `named` or `bind`.

	```bash
	# Ubuntu default (not required)
	sudo groupadd bind
	sudo useradd -g bind --system -M -s /usr/sbin/nologin bind

	# Fedora default (best for compatability)
	sudo groupadd named
	sudo useradd -g named --system -M -s /usr/sbin/nologin named
	```



## Firewall

`named` listens on 53/udp and 53/tcp as well as 953/tcp.

=== "Ubuntu"

	```bash
	# By application name
	sudo ufw allow bind9

	# Defined ports and protocols
	sudo ufw allow in on eth0 to any port 53 comment 'bind9'
	```

=== "Fedora"

	```bash
	sudo firewall-cmd --add-service dns --permanent
	sudo firewall-cmd --reload
	```


## named.conf

BIND9 is highly configurable. The options and deployments possible are kind of overwhelming, if you're more used to something like `unbound` or especially if you've stuck with `systemd-resolved`. This section shows the bare minimum required files to get `named` running as a fully functioning DNS service.

- [BIND9  Documentation: Configuration File Reference](https://bind9.readthedocs.io/en/stable/reference.html)
- [ISC KB: How to check the default option values in named.conf](https://kb.isc.org/docs/aa-00704)
- [Ubuntu Source: bind /etc Files](https://git.launchpad.net/ubuntu/+source/bind9/tree/debian/extras/etc?h=ubuntu/jammy-devel)
- [Fedora Source: named.conf.sample](https://src.fedoraproject.org/rpms/bind/blob/rawhide/f/named.conf.sample)

!!! tip "`named-checkconf` and `-C`"

	Similar to `unbound-checkconf` bind also has a tool called `named-checkconf` to validate the configuration file(s).

    `named` also has `named -C` to dump a complete example of the default option values for `named.conf`.

Create the directories for bind's [working paths](https://bind9.readthedocs.io/en/stable/reference.html#namedconf-statement-directory) you define in In `named.conf`. This also depends on what you choose for `--localstatedir="...` during the build process. Remember that `/var/run` symlinks to `/run`so we need to make the following paths.

```bash
# Assumes these are in named.conf, and everything else defined is within "/var/cache/named"
#   pid-file "/run/named/named.pid";
#   directory "/var/cache/named";

bind_paths='/var/run/named
/var/cache/named'

for path in $bind_paths; do
	sudo mkdir $path
	sudo chown -R bind:bind $path
done
```

!!! note "Choosing an `/etc` File Path"

	As mentioned above in the [install](#install) section about build options, this depends on what you used for `--sysconfdir=` during the build process.

	```bash
	sudo mkdir /etc/bind
	sudo chown -R bind:bind /etc/bind
	```

At minimum, we need two files to get `named` / bind running. Bind handles everything else (as of `BIND 9.18.26 (Extended Support Version) <id:936d80b>`).

- [named.conf](https://src.fedoraproject.org/rpms/bind/blob/rawhide/f/named.conf.sample)
- [named.rfc1912.zones](https://src.fedoraproject.org/rpms/bind/blob/rawhide/f/named.rfc1912.zones)

This is a combination of both Ubuntu and Fedora's defaults, with a few options from the BIND9 docs.

```conf
// named.conf

options {
	pid-file "/var/run/named/named.pid";

	directory          "/var/cache/named";
	dump-file          "/var/cache/named/data/cache_dump.db";
	statistics-file    "/var/cache/named/data/named_stats.txt";
	memstatistics-file "/var/cache/named/data/named_mem_stats.txt";
	secroots-file      "/var/cache/named/data/named.secroots";
	recursing-file     "/var/cache/named/data/named.recursing";

	listen-on port 53 { 127.0.0.1; };
	listen-on-v6 port 53 { ::1; };

	allow-query { localhost; };
	allow-query-cache { localhost; };

	recursion yes;

	dnssec-validation auto;
};

zone "." IN {
	type hint;
	file "/usr/share/dns/root.hints"; // sudo apt install -y dns-root-data
};

include "/etc/bind/named.rfc1912.zones";
```

Next, the rfc1912.zones file.

```conf
// named.rfc1912.zones:
//
// Provided by Red Hat caching-nameserver package
//
// ISC BIND named zone configuration for zones recommended by
// RFC 1912 section 4.1 : localhost TLDs and address zones
// and https://tools.ietf.org/html/rfc6303
// (c)2007 R W Franks
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// Note: empty-zones-enable yes; option is default.
// If private ranges should be forwarded, add
// disable-empty-zone "."; into options
//

zone "localhost.localdomain" IN {
	type primary;
	file "named.localhost";
	allow-update { none; };
};

zone "localhost" IN {
	type primary;
	file "named.localhost";
	allow-update { none; };
};

zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
	type primary;
	file "named.loopback";
	allow-update { none; };
};

zone "1.0.0.127.in-addr.arpa" IN {
	type primary;
	file "named.loopback";
	allow-update { none; };
};

zone "0.in-addr.arpa" IN {
	type primary;
	file "named.empty";
	allow-update { none; };
};
```

At this point you can run bind as the `bind` / `named` user.

```bash
# -g runs bind in the foreground
sudo /usr/local/bind-stable/sbin/named -g -L /tmp/bind.log -u bind
```

You can continue on to the following sections to build your `named.conf` file(s), or skip ahead to get this running through [systemd](#systemd-service).



### DNSSEC

- [BIND9  Documentation: DNSSEC](https://bind9.readthedocs.io/en/stable/chapter5.html#dnssec)



**Querying**

You may have noticed `dnssec-validaton auto;` in the `named.conf` file. This tells bind to validate DNS using the built in trust anchor data (`bind.keys`) which is now just part of BIND9.

You can manage your own keys by setting `dnssec-validation yes;`, however keep in mind you need to manually update the key to that subdomain when it's scheduled to rotate. This is something you might do for an internal domain, or a test network.

!!! quote "DNSSEC History"

	The [BIND9 docs suggest](https://bind9.readthedocs.io/en/stable/dnssec-guide.html#trusted-keys-and-managed-keys) setting this to `auto` for general use, as 90% of the top-level domains have now been signed since the root zone was signed in 2010. With the `bind.keys` data file built into the package, all of this can be handled automatically now.



**Replying**

BIND9 also has automated and manual options for [Zone Signing](https://bind9.readthedocs.io/en/stable/chapter5.html#zone-signing). It's recommended to make this fully automatic outside of test networks and specific use cases.

!!! quote "Zone Signing Example"

	To sign a zone, add the `dnssec-policy` definition to a zone block in your conf files.

	```conf
	zone "dnssec.example" {
		type primary;
		file "dnssec.example.db";
		dnssec-policy default;    ⬅️
		inline-signing yes;
	};
	```

	There are then three files created for zone keys:

	- `Kdnssec.example.+013+12345.private` private key for generating sigatures
	- `Kdnssec.example.+013+12345.key` the public key for validating the signatures
	- `Kdnssec.example+013+12345.state` state file used to track key timings and perform rollovers

The rest of the [DNSSEC chapter](https://bind9.readthedocs.io/en/stable/chapter5.html#dnssec) covers details like defining the DNSSEC policy parameters, and more. This guide may be updated with examples of this in the future.



### Access Control

`named` will listen on *all* interfaces by default on Ubuntu and only localhost (`::1`,`127.0.0.1`) on Fedora. Define ACL's to limit what resources can access the bind server.

=== "Ubuntu"

	Define ACL's in `/etc/bind/named.conf.options`.

	```conf
	acl "trusted" {
			::1;
			127.0.0.1;
			10.55.55.0/24;
	};

	options {
			listen-on port 53 { 127.0.0.1; 10.55.55.29; };
			listen-on-v6 port 53 { ::1; 10.55.55.29; };
			allow-query     { trusted; };
			allow-recursion { trusted; };
	<SNIP>
	```

=== "Fedora"

	Define ACL's in `/etc/named.conf`.

	```conf
	acl "trusted" {
			::1;
			127.0.0.1;
			10.55.55.0/24;
	};

	options {
			listen-on port 53 { 127.0.0.1; 10.55.55.29; };
			listen-on-v6 port 53 { ::1; 10.55.55.29; };
			allow-query     { trusted; };
			allow-recursion { trusted; };
	<SNIP>
	```


### Logging

The [logging statement](https://bind9.readthedocs.io/en/v9.19.24/reference.html#namedconf-statement-logging) configures the channel (output method) and category (classes of messages) to be logged. Only one logging statement is used to define as many channels and categories as desired. If there is no logging statement, a built in default configuration is used.

Query logging can be enabled for `bind`/`named` with the following block.

First create the log file path with the correct ownership and permissions.

```bash
sudo mkdir -p /var/log/named
sudo touch /var/log/named/query.log
sudo chown -R bind:bind /var/log/named
```

!!! tip "AppArmor Compatability"

	AppArmor lists the following paths for logging.

	```c
	/var/log/named/** rw,
	/var/log/named/ rw,
	```

	Use these, or modify the AppArmor policy to point to `/var/log/bind` instead.

Add or [include](https://bind9.readthedocs.io/en/stable/reference.html#include-grammar) the following **logging block** in `named.conf`.

```c
logging {
	// https://bind9.readthedocs.io/en/latest/reference.html
	// The channel defines the file to send messages to
    channel query_channel {
        file "/var/log/bind/query.log" versions 8 size 8m suffix increment;
        print-category yes; // Includes category in logs
        print-severity yes; // Includes severity in logs
        print-time yes; // Includes a timestamp in logs (iso8601 | iso8601-utc | local | <boolean>)
        severity info; // info is the default and lowest log level, meaning everything gets logged

        // Tells named to write to syslog as a specific syslog facility (your choice)
        // You must choose either syslog, or writing to a file, not both
        //syslog [<syslog_facility>];  // Options include daemon, syslog, kern, and many more
    };
    // The category defines *what* is sent to the log file, there can be multiple defined
    // https://bind9.readthedocs.io/en/latest/reference.html#namedconf-statement-category
    category queries { query_channel; };
    category query-errors { query_channel; };
    category security { query_channel; };
    category dnssec { query_channel; };
};
```



### dnstap

The [RHEL 9 documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_networking_infrastructure_services/assembly_setting-up-and-configuring-a-bind-dns-server_networking-infrastructure-services#proc_recording-dns-queries-using-dnstap_assembly_setting-up-and-configuring-a-bind-dns-server) covers recording DNS queries using `dnstap`. Requires `bind-9.16.15-3` or later. This won't be covered here for now, as it isn't always available.



### DNS over TLS

This section focuses on **making queries** over TLS, not running a server that accepts DNS over TLS

!!! warning "Minimum Version Requirements"

	Only certain recent versions of BIND9 (9.19+) have this capability. Interestingly the absolute latest version available did not have this feature enabled at the time of writing. You can easily test this by checking out a specific version, compiling the binaries, and running `named-checkconf` to assess the example blocks below. If it throws an error, DNS over TLS isn't supported in that build.

	- [BIND9: named.conf Forwarders](https://bind9.readthedocs.io/en/latest/reference.html#namedconf-statement-forwarders)
	- [How to use DNS over TLS with BIND9 Forwarders](https://unix.stackexchange.com/questions/735368/how-to-use-dns-over-tls-with-bind9-forwarders)
	- [gitlab.isc.org: merge Resolve "Forward queries via DoT"](https://gitlab.isc.org/isc-projects/bind9/-/merge_requests/7199)

	According to the git history, this feature will be available in versions 9.19 or later, and by default in 9.20 stable.



**Cloudflare**

- [View the CA used for DNS over TLS with kdig](https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-tls/#example)
- [BIND9: tls block example](https://bind9.readthedocs.io/en/latest/reference.html#namedconf-statement-session-tickets)

The [tls block](https://bind9.readthedocs.io/en/latest/reference.html#tls-block-grammar) in `named.conf` requires a .pem file for the CA related to the hostname we're using. Without this, there's no way to verify the hostname is actually who they say they are. We need to know what certificate authority to point to in the `tls` block of the configuraiton file. It's possible to view the certificate used for Cloudflare by checking the returned CN with `kdig`.

```bash
dnf install knot-utils
kdig -d @1.1.1.1 +tls-ca +tls-host=one.one.one.one example.com | grep CN
```

Example configuration blocks:

```conf
// https://bind9.readthedocs.io/en/latest/reference.html#tls-block-grammar
// You can define multiple tls blocks, one for every provider
tls Cloudflare {
	ca-file "/etc/ssl/certs/DigiCert_Global_Root_G2.pem";
	remote-hostname "one.one.one.one";
};
#tls Quad9 {
#	ca-file "/etc/ssl/certs/?";
#	remote-hostname "dns.quad9.net";
#};

options {
	// https://bind9.readthedocs.io/en/latest/reference.html#namedconf-statement-forwarders
	forwarders {
		1.1.1.1 port 853 tls Cloudflare;
		1.0.0.1 port 853 tls Cloudflare;
		2606:4700:4700::1111 port 853 tls Cloudflare;
		2606:4700:4700::1001 port 853 tls Cloudflare;

#		9.9.9.9 port 853 tls Quad9;
	};
};
```

!!! warning "Quad9's CA File"

	It's unclear which (if any) of the CA files from the `ca-certificates` package will work for Quad9. The returned string is `CN=DigiCert TLS Hybrid ECC SHA384 2020 CA1` which the closest match on Fedora is `DigiCert_TLS_ECC_P384_Root_G5.pem`. However this CA doesn't work, and breaks name resolution.



## Systemd Service

Now that we have bind functioning and configured, we can create a service file so it will run automatically.

!!! warning "Daemon Security"

	At this point in the guide, the only hardening feature implemented is dropping privileges from root to the `bind` / `named` user via `-u`. You'll want to implement one of the [security](#security) options when finally using bind in a production sense.

Create the systemd service file. This assumes you've been following the recommendation of creating symlinks in `/usr/{sbin,bin}` to your `make install` path.

- `EnvironmentFile=-$sysconfdir/default/named` is Ubuntu specific
- Fedora uses `EnvironmentFile=-$sysconfdir/sysconfig/named`
- The entire block below is meant to be safe to copy and paste over the previous files for version testing

```bash
# Change these as needed, especially if there's a chroot path (default is no chroot)
envfilepath='default' # Ubuntu=default, Fedora=sysconfig

if (systemctl is-active named); then sudo systemctl stop named; fi

sleep 3

echo "[Unit]
Description=BIND Domain Name Server
Documentation=man:named(8)
After=network.target
Wants=nss-lookup.target
Before=nss-lookup.target

[Service]
Type=forking
EnvironmentFile=-/etc/$envfilepath/named
ExecStart=/usr/sbin/named \$OPTIONS
ExecReload=/usr/sbin/rndc reload
ExecStop=/usr/sbin/rndc stop
Restart=on-failure

[Install]
WantedBy=multi-user.target
Alias=bind9.service" | sudo tee /lib/systemd/system/named.service

sudo mkdir /etc/default
echo '# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-u bind"' | sudo tee /etc/$envfilepath/named

sudo systemctl daemon-reload
sudo systemctl enable named
sudo systemctl restart named

systemctl status named

# It should find these
echo ""
echo "[*]cache Files"
find /var/cache/named -type f
echo ""
echo "[*]--localstatedir Files"
find /var/run/named -type f
```


### Environment File

The script block above created the default environment file, with only the `-u bind` option. More options can be defined here.

- Ubuntu: `/etc/default/named`
- Fedora: `/etc/sysconfig/named`


**IPv4 or IPv6 Only**

To run only IPv4 *or* IPv6 (meaning not both), the `-4` or `-6` arguments must be passed to `named`.

```bash
# Only use IPv6
OPTIONS="-u bind -6"
```



## Security

Hardening the bind9 / `named` process itself (meaning outside of its runtime configuration) can take one of three approaches; AppArmor, SELinux, or chroot.



### AppArmor

On Debian-based systems (Ubuntu) bind9 ships [an AppArmor policy](https://git.launchpad.net/ubuntu/+source/bind9/tree/debian/extras/apparmor.d/usr.sbin.named?h=ubuntu/jammy-devel) that's enabled by default.

You could obtain this policy from the debian source package.

```bash
sudo sed -i_bkup 's/# deb-src/deb-src/g' /etc/apt/sources.list
sudo apt update

cd ~/src
apt download bind9
dpkg-deb -x ./bind9_1%3a9.18.24-0ubuntu0.22.04.1_amd64.deb bind9-deb
cd bind9-deb/etc/apparmor.d/
```

You can also obtain a copy of the policy file directly from Ubuntu's git repo (change the Ubuntu version to match yours).

```bash
cd ~/src/bind9  # Change into the bind9 upstream git folder
curl -Lf 'https://git.launchpad.net/ubuntu/+source/bind9/plain/debian/extras/apparmor.d/usr.sbin.named?h=ubuntu/jammy-devel' > usr.sbin.named
```

Create a local override so `named` can access either `/var/cache/{bind,named}`, whichever one exists. You could also manually edit `usr.sbin.named` to make this change, it's up to you.

```bash
echo '# Local overrides for usr.sbin.named
  /var/cache/named/** lrw,
  /var/cache/named/ rw,' | sudo tee /etc/apparmor.d/local/usr.sbin.named
```

!!! info "AppArmor Includes"

	In any case, if the main `usr.sbin.named` AppArmor profile includes `local/usr.sbin.named`, you will at least need to `touch` that path to ensure it exists.

	```bash
	sudo touch /etc/apparmor.d/local/usr.sbin.named
	```

Finally, install the AppArmor profile and reload both AppArmor and named to ensure everything works.

```bash
sudo cp ./usr.sbin.named /etc/apparmor.d/

# Load the new profile into AppArmor
sudo apparmor_parser -a /etc/apparmor.d/usr.sbin.named

# Ensure everything works, use `journalctl -xeu named` to diagnose any errors
sudo systemctl restart named
systemctl status named  # Should have no errors
echo ""
echo "[*]AppArmor Status"
sudo aa-status | grep named
```

To disable the profile:

```bash
sudo ln -s /etc/apparmor.d/usr.sbin.named /etc/apparmor.d/disable/usr.sbin.named
sudo apparmor_parser -R /etc/apparmor.d/usr.sbin.named
```

To remove the profile completely:

```bash
sudo apparmor_parser -R /etc/apparmor.d/usr.sbin.named
find /etc/apparmor.d/ -name 'usr.sbin.named' -print0 | xargs -0 sudo rm -f
```

This profile locks `named`'s ability to access or modify data to the following paths (where `lrw` means read,write,link).

```txt
  /var/lib/bind/** rw,
  /var/lib/bind/ rw,
  /var/cache/bind/** lrw,
  /var/cache/bind/ rw,
  /var/cache/bind/_default.nzd-lock rwk,
  /var/lib/dnscvsutil/compiled/** rw,
  owner @{PROC}/@{pid}/task/@{tid}/comm rw,
  /{,var/}run/named/named.pid w,
  /{,var/}run/named/session.key w,
  /var/log/named/** rw,
  /var/log/named/ rw,
  /{,var/}run/slapd-*.socket rw,
  /var/tmp/DNS_* rw,
  /var/lib/samba/bind-dns/dns/** rwk,
  /var/lib/samba/private/dns/** rwk,
  /dev/urandom rwmk,
  owner /var/tmp/krb5_* rwk,
  /var/lib/sss/pipes/nss  rw,
  @{run}/.nscd_socket   rw,
  @{run}/nscd/socket    rw,
  @{run}/avahi-daemon/socket rw,
```


!!! example "Reviewing Confinement"

	- [`aa-exec` manpage](https://gitlab.com/apparmor/apparmor/-/wikis/manpage_aa-exec.1#synopsis)

	AppArmor has a tool called `aa-exec` that allows you to run a process under the confinement of a (currently loaded) profile. When asking ChatGPT if there's a similar method for AppArmor to using `snap run --shell firefox` or `flatpak run --command=bash firefox` to explore what the snap/flatpak-confined firefox process can see and do, it pointed to `aa-exec`. This is essentially the same thing, allowing you to explore the AppArmor confinement of regular deb packages that have a profile.

	Ensure the profile is loaded into AppAmor.

	```bash
	sudo apparmor_parser -a /etc/apparmor.d/usr.sbin.named
	```

	Specify the profile name based on what `sudo aa-status` returns for that executable. For example you would use `named` and not `usr.sbin.named` here. To open a `bash` shell in `named`'s AppArmor profile:

	```bash
	sudo aa-exec -p named -- /bin/bash
	```

	This results in a bash process confined by the `usr.sbin.named` profile.

	```bash
	server@ubuntu2404:~$ sudo aa-exec -p named -- /bin/bash
	bash: /etc/bash.bashrc: Permission denied
	bash: /root/.bashrc: Permission denied
	bash-5.1#
	bash-5.1# cat /etc/shadow
	bash: /usr/bin/cat: Permission denied
	bash-5.1#
	bash-5.1# echo 'bad-key' >> /root/.ssh/authorized_keys
	bash: /root/.ssh/authorized_keys: Permission denied
	bash-5.1#
	bash-5.1# sh -c whoami
	bash: /usr/bin/sh: Permission denied
	bash-5.1# bash
	bash: /usr/bin/bash: Permission denied
	bash-5.1#
	bash-5.1# nc
	bash: /usr/bin/nc: Permission denied
	bash-5.1#
	bash-5.1# curl http://127.0.0.1:8080/exploit.py | python3
	bash: /usr/bin/python3: Permission denied
	bash: /usr/bin/curl: Permission denied
	bash-5.1#
	bash-5.1# touch /tmp/test.txt
	bash: /usr/bin/touch: Permission denied
	```


### SELinux

*⚠️ TO DO, check back later!*



### chroot and setuid

This option comes with a number of caveats.

!!! warning "Chroot vs Mandatory Access Control"

	The [RHEL 9 documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_networking_infrastructure_services/assembly_setting-up-and-configuring-a-bind-dns-server_networking-infrastructure-services#ref_considerations-about-protecting-bind-with-selinux-or-running-it-in-a-change-root_environment_assembly_setting-up-and-configuring-a-bind-dns-server) suggests using a mandatory access control mechanism like SELinux or AppArmor is more robust than a chroot jail. Only use this method if you cannot use AppArmor of SELinux.

	Additionally, if you want to use AppArmor on top of a chroot environment, [you will need to modify the AppArmor profile to use the chroot base path](https://wiki.ubuntu.com/DebuggingApparmor#AppArmor_and_chroot_environments).

!!! info "bind-chroot.x86_64"

	This functionality is available as an rpm package, `bind-chroot` (on Fedora) and `named-chroot` (on RHEL).

	- [BIND9  Documentation: Chroot and Setuid](https://bind9.readthedocs.io/en/v9.19.24/chapter7.html#chroot-and-setuid)
	- [Ubuntu Community: Chrooting BIND9](https://help.ubuntu.com/community/BIND9ServerHowto#Chrooting_BIND9)

!!! quote "The chroot environment"

	Unlike with earlier versions of BIND, [named](https://bind9.readthedocs.io/en/v9.19.24/manpages.html#std-iscman-named) does *not* typically need to be compiled statically, nor do shared libraries need to be installed under the new root. However, depending on the operating system, it may be necessary to set up locations such as `/dev/zero`, `/dev/random`, `/dev/log`, and `/etc/localtime`.

!!! tip "Building on What Exists"

	The Fedora repo for bind contains a number of files you could use to script setting up a chroot instance.

    - [named-chroot-setup.service](https://src.fedoraproject.org/rpms/bind/blob/rawhide/f/named-chroot-setup.service), executes `setup-named-chroot.sh`, `After=named-setup-rndc.service`
    - [setup-named-chroot.sh](https://src.fedoraproject.org/rpms/bind/blob/rawhide/f/setup-named-chroot.sh), this creates the necessary directories on Fedora using [named-chroot.files](https://src.fedoraproject.org/rpms/bind/blob/rawhide/f/named-chroot.files)
    - [named-chroot.service](https://src.fedoraproject.org/rpms/bind/blob/rawhide/f/named-chroot.service), the main service file, running `named` in a chroot environment
    - [named-setup-rndc.service](https://src.fedoraproject.org/rpms/bind/blob/rawhide/f/named-setup-rndc.service), executes `generate-rndc-key.sh`
    - [generate-rndc-key.sh](https://src.fedoraproject.org/rpms/bind/blob/rawhide/f/generate-rndc-key.sh), generates `/etc/rndc.key` if doesn't exist AND if there is no `rndc.conf`

Without a kernel-based mandatory access control system, you can still chroot the `bind`/`named` process into its own "sandbox" directory, and run bind unprivileged with `-u <user>`.

**Setting Up the Chroot**

To do this as simply as possible on Ubuntu, start by making the chroot paths:

```bash
# Add any paths you use that are missing here
sudo mkdir -p /chroot/bind/{etc/{bind,default},var/{cache/named/data,run/named,log/bind},usr/share/dns}
find /chroot/ -type d
# The bind user can only write to /chroot/bind/var, read the rest
sudo chown -R root:bind /chroot/bind
sudo chown -R bind:bind /chroot/bind/var
```

Note that you don't need to copy any libraries or other binaries into the chroot, only files `named` will read or write to.

Copy the existing configuration files over.

```bash
sudo cp /etc/bind/* /chroot/bind/etc/bind/
sudo cp /etc/default/named /chroot/bind/etc/default/
sudo cp /usr/share/dns/root.hints /chroot/bind/usr/share/dns/
```

For compatability, we'll create a separate Systemd service called named-chroot.

```bash
envfilepath='default' # Ubuntu=default, Fedora=sysconfig

echo "[Unit]
Description=BIND Domain Name Server (Chroot)
Documentation=man:named(8)
After=network.target
Wants=nss-lookup.target
Before=nss-lookup.target

[Service]
Type=forking
EnvironmentFile=-/etc/$envfilepath/named
ExecStart=/usr/sbin/named \$OPTIONS -t /chroot/bind
ExecReload=/usr/sbin/rndc reload
ExecStop=/usr/sbin/rndc stop
Restart=on-failure

[Install]
WantedBy=multi-user.target
Alias=bind9-chroot.service" | sudo tee /lib/systemd/system/named-chroot.service
```

Stop the non-chroot service, then start the chroot service.

```bash
sudo systemctl stop named
sudo systemctl start named-chroot
systemctl status named-chroot

echo ""
echo "[*]Chroot runtime files"
find /chroot/ -type f
```

`named` is now running in a chroot.


!!! example "Reviewing Confinement"

	- [GNU Manual: Coreutils/Chroot](https://www.gnu.org/software/coreutils/chroot)
	- [HackTricks: Chroot Escapes](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/escaping-from-limited-bash#chroot-escapes)

	You can interactively explore a chroot environment with the `chroot <path>` command. Specify the `<path>` to your chroot, then a command or shell to execute. If you don't specify a shell, it defaults to `/bin/sh -i`.

	!!! quote "man chroot"

		If no command is given, run '"$SHELL" -i' (default: '/bin/sh -i').

	You can see below with the chroot created for bind, there aren't any binaries available to execute. There's no way to explore this interactively in our case, which is exactly what we want. For reference, `find` was used to list all avaialble files under the chroot. Assume that a rogue `named` process could at the very least "see" these files, which is fine.

	```bash
	server@ubuntu2404:~$ sudo chroot /chroot/bind
	chroot: failed to run command ‘/bin/bash’: No such file or directory
	server@ubuntu2404:~$
	server@ubuntu2404:~$ sudo find /chroot/ -type f
	/chroot/bind/etc/bind/named.conf
	/chroot/bind/etc/bind/rndc.key
	/chroot/bind/etc/bind/named.rfc1912.zones
	/chroot/bind/etc/bind/bind.keys
	/chroot/bind/etc/bind/rndc.conf
	/chroot/bind/usr/share/dns/root.hints
	/chroot/bind/var/run/named/session.key
	/chroot/bind/var/run/named/named.pid
	/chroot/bind/var/cache/named/managed-keys.bind
	/chroot/bind/var/cache/named/managed-keys.bind.jnl
	```


## Administration

- [BIND9 Administrative Tools](https://bind9.readthedocs.io/en/stable/chapter4.html#administrative-tools)

First create an `rndc.key` file.

```bash
sudo rndc-confgen -a
```

This file holds the shared secret used to connect to the server, in our case localhost. You'll need to make sure `named` can read this file. If you're running as `-u bind` this means something like:

```bash
# Remember to copy these into the chroot if you're using one
sudo chown root:bind /etc/bind/rndc.key
sudo chmod 640 /etc/bind/rndc.key
```

Then write a conf file, [including](https://bind9.readthedocs.io/en/stable/reference.html#include-grammar) the `rndc.key` file.

```bash
// https://bind9.readthedocs.io/en/stable/chapter4.html#rndcconf-statement-options
echo 'options {
	default-key rndc-key;
	default-port 953;
	default-server "localhost";
};

include "/etc/bind/rndc.key"' | sudo tee /etc/bind/rndc.conf
```

Restart `named` to load the key.

```bash
sudo systemctl restart named
```

Diagnosing a misbehaving BIND server is detailed [here](https://kb.isc.org/docs/aa-00341).

```bash
rndc status         # Obtain a snapshot of `named`'s status

rndc recursing      # Generate a list of the client queries that named is currently handling (named.recursing)

rndc dumpdb -all    # Get a snapshot of the current state of named's cache (named_dump.db)

rndc querylog       # Toggle query logging on

rndc trace 3        # Temporarily increase the level of server logging
```

Follow journal message with:

```bash
sudo journalctl -f -u bind  # or named, if you used named instead
```

Run `named` in the foreground with:

```bash
sudo named -g -L /tmp/named.log -u bind -c /etc/bind/named.conf
```







