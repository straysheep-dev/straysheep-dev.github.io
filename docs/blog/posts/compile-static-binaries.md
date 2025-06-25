---
draft: false
date:
  created: 2025-06-15
  updated: 2025-06-23
categories:
  - how-to
  - ctf
  - htb
  - linux
  - cups
---

# :fontawesome-solid-square-binary: Compile Static Binaries

Creating static binaries easily, repeatably, and on your own.

<!-- more -->

Each section was originally part of other notes taken over the last few years. Initially these were put together to build static binaries when competing in a live cyber attack-defense event, where systems are already assumed to be compromised in some way. Bringing your own static binaries along with [uac](https://github.com/tclahr/uac) is a really effective IR technique.

Eventually static builds came up again, recently with [pspy](https://github.com/DominicBreuker/pspy), just to understand golang and using Docker to build software. This is an effort to centralize all of these notes for public reference.

!!! question "Why?"

    The first time ever using [naabu](https://github.com/projectdiscovery/naabu) and seeing how it could work after being moved to other systems, was what prompted creating the notes in this post. It's also a great way to learn more about reviewing and modifying code.

    1. Binaries linked to shared libraries will often only run on the system(s) they were compiled on. Static binaries bake in those dependencies and have a level of portability to them.
    2. Static binaries of common tools are often most useful for pivoting, where you have shell access to a system and you need the capability to do something local to that machine without the ability to compile anything on it.
    3. Customization and signature evasion
    4. Compiling for other architectures and systems
    5. Most importantly, to trust the tool. For some tools, you can pull the source directly from the Linux repositories which already have to be trusted to some degree. Almost none of those projects release static binaries by default, and this is easier than reverse engineering existing unofficial binaries from third-parties if any exist.

## :octicons-play-16: Getting Started

!!! warning "Work in Progress"

    The notes in this post as of right now are close to their original state, and need to be updated to use musl and Docker in the build process.

These resources are necessary starting points to succeed here.

- [musl-libc](https://musl.libc.org/)
- [andrew-d/static-binaries](https://github.com/andrew-d/static-binaries)
- [How to compile a static binary](https://stackoverflow.com/questions/983515/how-to-statically-link-a-complex-program#983604)
- [Docker](https://docs.docker.com/)
- [Kali Linux Source Repos](https://www.kali.org/docs/general-use/kali-linux-sources-list-repositories/#source-repositories)

The [andrew-d/static-binaries](https://github.com/andrew-d/static-binaries) repository has a great approach that's fairly easy to understand and learn from. Familiarity with docker, CICD, and devops in general also helps; a lot of modern projects will include a [dockerfile](https://docs.docker.com/reference/dockerfile/) among other build tools to help you compile things from source. Part of the purpose of this post is to help unwind and determine how these are working, to modify and use them, and ultimately build our own.

**Using makefiles :material-wrench-cog-outline:**

Essentially, these are the steps you'll encounter with makefiles:

```bash
./configure # There's often a configuration shell script
make        # Compiles the code
make clean  # Revert project directory to state before running make, great if you encounter errors or want to reconfigure and build again
```

!!! tip "Handling Compile Errors"

    When handling errors note:

    - Which header file, or library, is causing the error(s)
    - Does it require different dependancies
    - Does something specific in the configuration of the build need to be changed

**musl-libc :material-wrench-cog-outline:**

!!! note "What uses musl?"

    In addition to the andrew-d/static-binaries project, [threathunters-io/laurel](https://github.com/threathunters-io/laurel) also uses musl to build static binaries for laurel.

⚠️ TO DO ⚠️


## :simple-openssl: openssl

OpenSSL is a protocol implementation and library for cryptography. Specifically from the project README:

!!! quote ""

    OpenSSL is a robust, commercial-grade, full-featured Open Source Toolkit for the TLS (formerly SSL), DTLS and QUIC protocols.

    The protocol implementations are based on a full-strength general purpose cryptographic library, which can also be used stand-alone. Also included is a cryptographic module validated to conform with FIPS standards.

    OpenSSL is descended from the SSLeay library developed by Eric A. Young and Tim J. Hudson.

    The official Home Page of the OpenSSL Project is www.openssl.org.

These are the main resources we'll need for reference to build OpenSSL:

- [openssl.org](https://www.openssl.org/)
- [openssl-library.org](https://openssl-library.org/) / [github.com/openssl/openssl](https://github.com/openssl/openssl)
- [Compilation Options](https://wiki.openssl.org/index.php/Compilation_and_Installation#Configure_Options)
- [andrew-d/static-binaries Example for Compiling nmap](https://github.com/andrew-d/static-binaries/blob/master/nmap/build.sh)
- [OpenSSL PGP Signing Key Information](https://openssl-library.org/source/)
- [OpenSSL PGP Signing Keys](https://openssl-library.org/source/pubkeys.asc)

!!! note "Where to download from?"

    The downloads on <https://openssl-library.org/source/> point to <https://github.com/openssl/openssl/releases>.

Clone the library source, detached signature, checksums, and finally obtain the signing key to verify the file integrity.

```bash
# Install jq, if you want to use this block
if ! command -v jq; then sudo apt update; sudo apt install -y jq; fi

# Obtain the latest release info
gh_release_info="$(curl -Lf https://api.github.com/repos/openssl/openssl/releases/latest)"
latest_version="$(echo "$gh_release_info" | jq -r '.tag_name')"

# Download the files
curl -LfO "https://github.com/openssl/openssl/releases/download/${latest_version}/${latest_version}.tar.gz"
curl -LfO "https://github.com/openssl/openssl/releases/download/${latest_version}/${latest_version}.tar.gz.asc"
curl -LfO "https://github.com/openssl/openssl/releases/download/${latest_version}/${latest_version}.tar.gz.sha256"

# Obtain the signing key
# pub   rsa4096/0x216094DFD0CB81EF 2024-04-08 [SC] [expires: 2026-04-08]
#       Key fingerprint = BA54 73A2 B058 7B07 FB27  CF2D 2160 94DF D0CB 81EF
# uid                   [ unknown] OpenSSL <openssl@openssl.org>
gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys 'BA5473A2B0587B07FB27CF2D216094DFD0CB81EF'

# Integrity checks
sha256sum -c "${latest_version}.tar.gz.sha256" --ignore-missing
gpg --verify "${latest_version}.tar.gz.asc" "${latest_version}.tar.gz"

# Build the library
tar -xvzf "${latest_version}.tar.gz"
cd "${latest_version}"
./Configure LDFLAGS=-static no-shared
make
```

This will result in a static binary for OpenSSL under `apps/openssl`,


## :fontawesome-solid-sitemap: nmap

!!! note "OpenSSL Dependancy"

    You may want to compile a static version of OpenSSL first and point to it with `--with-openssl=<directoryname>` when building a static nmap binary.

- [nmap: Release and Build Information](https://nmap.org/download.html)
- [nmap: Compile Instructions](https://nmap.org/book/inst-source.html)
- [andrew-d/static-binaries Example for Compiling nmap](https://github.com/andrew-d/static-binaries/blob/master/nmap/build.sh)
- [nmap: Release Archive](https://nmap.org/dist/)
- [nmap: Detached Signatures](https://nmap.org/dist/sigs/)
- [nmap: Verifying archive integrity](https://nmap.org/book/install.html#inst-integrity)
- [nmap GitHub Issue 2648: Fails to build with lua 5.4.6](https://github.com/nmap/nmap/issues/2648)
- [nmap GitHub Issue 145: --with-liblua=included fix](https://github.com/nmap/nmap/issues/145)
- [nmap GitHub Issue 425: FEATURE REQUEST: single-exe 'portable' nmap](https://github.com/nmap/nmap/issues/425)
- [askubuntu: Decompress an .xz tarball in one command](https://askubuntu.com/questions/92328/how-do-i-uncompress-a-tarball-that-uses-xz)

**:material-file-download: Obtain and verify source (nmap.org)**

```bash
# Set which versions you'd like to compile
nmap_version='7.97'
openssl_version='3.5.0'

# Download the source archive, and signature information
wget "https://nmap.org/dist/nmap-${nmap_version}.tar.bz2"
wget "https://nmap.org/dist/sigs/nmap-${nmap_version}.tar.bz2.asc"
wget "https://nmap.org/dist/sigs/nmap-${nmap_version}.tar.bz2.digest.txt"

# If you don't already have nmap's signing key, you'll see the abbreviated signature you can use to obtain it
# Check these places to verify the signature fingerprint:
# https://nmap.org/book/install.html#inst-integrity
# https://svn.nmap.org/nmap/docs/nmap_gpgkeys.txt
# https://keyserver.ubuntu.com/pks/lookup?search=436D66AB9A798425FDA0E3F801AF9F036B9355D0&fingerprint=on&op=index
#
# pub   dsa1024/0x01AF9F036B9355D0 2005-04-24 [SC]
#       Key fingerprint = 436D 66AB 9A79 8425 FDA0  E3F8 01AF 9F03 6B93 55D0
# uid                   [ unknown] Nmap Project Signing Key (http://www.insecure.org/)
# sub   elg2048/0x44AEF5D7A50A6A94 2005-04-24 [E]

# Obtain the key with:
gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys '436D 66AB 9A79 8425 FDA0 E3F8 01AF 9F03 6B93 55D0'

# Verify:
gpg --verify "nmap-${nmap_version}.tar.bz2.asc" "nmap-${nmap_version}.tar.bz2"

# Compare hashes:
cat "nmap-${nmap_version}.tar.bz2.digest.txt"
gpg --print-md sha256 "nmap-${nmap_version}.tar.bz2"
```

**:material-file-download: Obtain and verify source (apt)**

*This may not work, try obtaining nmap from nmap.org if it fails.*

!!! tip "Instructions in Kali"

    Interestingly, as of Kali in 2025 and possibly earlier, you'll get a message for some packages when downloading the source code via `apt`:

    ```bash
    $ apt source nmap
    NOTICE: 'nmap' packaging is maintained in the 'Git' version control system at:
    https://gitlab.com/kalilinux/packages/nmap.git
    Please use:
    git clone https://gitlab.com/kalilinux/packages/nmap.git
    to retrieve the latest (possibly unreleased) updates to the package.
    ```

```bash
# Add the source repo in Kali, adjust this based on the Linux distro
echo "deb-src http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware" | sudo tee -a /etc/apt/sources.list
sudo apt update

# Download the source code
cd ~/Download
apt source nmap
sudo apt-get build-dep nmap

# Check the gpg and sha256 signatures:
gpg --verify "nmap_${nmap_version}+dfsg1-1kali1.dsc"
# If you're missing the key, get it from the ubuntu keyserver and verify again
gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys '<key-id>'
gpg --verify "nmap_${nmap_version}+dfsg1-1kali1.dsc"
cat "nmap_${nmap_version}+dfsg1-1kali1.dsc" # Take the signature for the archive you want to verify
sha256sum nmap_<version>.tar.xz | grep <sha256sum>
```

Prepare and compile nmap:

```bash
# If you obtained it from apt:
tar -xf "nmap_${nmap_version}+dfsg1.orig.tar.xz"
# If you obtained it from nmap.org:
bzip2 -cd "nmap-${nmap_version}.tar.bz2" | tar xvf -

cd "nmap-${nmap_version}"

# Don't build the libpcap.so file, this was discovered from andrew-d/static-binaries
sed -i -e 's/shared\: /shared\: #/' libpcap/Makefile

./configure --help # See the configuration options
./configure LDFLAGS=-static --with-liblua=included --with-openssl=../openssl-${openssl_version} --with-pcap=linux --without-ndiff --without-zenmap --withou-nmap-update
make -j4
```

If you encounter an error and need to attempt another build be sure to run:

```bash
make clean
```


## :material-vector-line: Socat

- [Debian Package](https://packages.debian.org/bookworm/socat)
- [Kali Linux Source Repos](https://www.kali.org/docs/general-use/kali-linux-sources-list-repositories/#source-repositories)
- [How to compile a static binary](https://stackoverflow.com/questions/983515/how-to-statically-link-a-complex-program#983604)

Add the Kali source repo:

```bash
echo "deb-src http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware" | sudo tee -a /etc/apt/sources.list
sudo apt update
```

Download the source code

```bash
cd ~/Download
apt source socat
sudo apt-get build-dep socat
```

Check the gpg and sha256 signatures:

```bash
gpg --verify socat_1.7.4.4-2.dsc
# If you're missing the key, get it from the ubuntu keyserver and verify again
gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys '<key-id>'
gpg --verify socat_1.7.4.4-2.dsc
cat socat_1.7.4.4-2.dsc # Take the signature for the archive you want to verify
sha256sum socat_<version>.tar.gz | grep <sha256sum>
```

Compile a static binary:

```bash
rm -rf socat_<version> # We want to unpack and use files from the archive we verified hashes on
tar -xzf socat_<version>.tar.gz
cd socat_<version>
./configure LDFLAGS=-static # This works here, while --enable-static does not
make
```

After some errors and warnings, you'll have a static `socat` binary for transfer.


## :fontawesome-solid-magnifying-glass-chart: pspy

- [github.com/DominicBreuker/pspy](https://github.com/DominicBreuker/pspy)
- [kali.org/tools/pspy](https://www.kali.org/tools/pspy/)

⚠️ TO DO ⚠️


## :fontawesome-solid-bug-slash: yara

⚠️ TO DO ⚠️


## :octicons-terminal-16: busybox

⚠️ TO DO ⚠️
