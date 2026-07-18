---
title: "Commands: Processes"
icon: material/family-tree
draft: true
#date:
#  created: 2025-06-17
#  updated: 2026-07-18
categories:
  - how-to
  - process
  - system
  - cli
  - linux
  - windows
  - reference
  - ebpf
  - trace
---

# :material-family-tree: Process Tracing Commands

Similar to [Network Commands](../posts/network-commands.md), this post is meant to be a single point of reference for all the random ways of interacting with and tracing processes.

<!-- more -->

---

## Windows

### Essentials

From the video "[1st 3 Windows IR Commmands](https://www.youtube.com/watch?v=ilhzt-Hw_sY)" by BHIS:

```powershell
netstat -abno
wmic process where processid=<pid> get commandline
tasklist.exe /m /fi "pid eq <pid>"
```

---

## Linux

### Essentials

!!! info "Go-to's for General Use"

    These are commands I tend to rely on from muscle memory.

BSD-style process listing:

```bash
ps aux
```

Process tree:

```bash
ps axjf
```

Securtiy info (AppArmor / SELinux):

```bash
ps axZ
```

Live process and resource usage:

```bash
# Not as fancy as htop but just as good (and seemingly always available)
top
# "m" cycles through memory usage dispaly options
# "u" will prompt you for a user to filter for
# "c" toggles full command line and paths
# "z" + "b" toggle color/mono and bold display options
# "s" lets you enter the refresh speed (e.g. 2.0, 0.1, all in seconds, 0.5 is a good speed)
# "M" sort by memory usage
# "N" sort by PID number
# "P" sort by CPU usage
# "T" sort by TIME+
# Use "R" to reverse the sort order

# More robust process and resource usage, likely needs installed via package manager
htop
```

---

### bpftrace

!!! quote "Trace new processes via `exec()` syscalls. Uses bpftrace/eBPF"

    `bpftrace` is a way to interact with the Linux BPF subsystem through LLVM.

    - [github.com/iovisor](https://github.com/iovisor/bpftrace), redirects to [github.com/bpftrace](https://github.com/bpftrace/bpftrace)
    - [bpftrace.org/tutorial-one-liners](https://bpftrace.org/tutorial-one-liners)

    Below are a number of `*.bt` commands. These commands are effectively [scripts](https://bpftrace.org/docs/release_026/language) wrapping around `bpftrace` to monitor a specific thing (they literally start with `#!/usr/bin/env bpftrace`).

    You can see everything available on the system with:

    ```bash
    find /usr/sbin/ -type f -name "*.bt"
    ```

    This doesn't include the `*-bpfcc` commands that are also under `/usr/sbin/`. Depending on the system, these may be called directly, or indirectly (pending review).

    These are excellent debugging and **threat hunting** tools.

**Installation :material-package-variant-plus:**

```bash
# https://github.com/bpftrace/bpftrace#quick-start
sudo apt install bpftrace
sudo dnf install bpftrace

# Nightly AppImage
declare -A suffixes=([x86_64]="X64" [amd64]="AMD64");
declare prefix="bpftrace/bpftrace/workflows/binary/master/bpftrace";
declare url="https://nightly.link/${prefix}-${suffixes[$(uname -m)]}.zip";
curl -L -o bpftrace.zip "${url}" && unzip bpftrace.zip
```

**Practical Usage :material-wrench-cog-outline:**

Detect and trace execution events on a system.

```bash
# Trace new processes via exec() syscalls.
sudo execsnoop.bt

# This focuses more narrowly than `execsnoop.bt` by printing all `bash` commands system-wide.
sudo bashreadline.bt
```

Detect and trace filesystem activity.

```bash
# Trace all open(),openat(),openat2() syscalls. (Noisy!)
sudo opensnoop.bt
sudo opensnoop.bt | grep -F '/etc/some/path'

# Trace the stat() syscall, showing which processes are attempting to stat which files.
sudo statsnoop.bt

# Trace directory entry cache (dcache) lookups. (Noisy!)
sudo dcsnoop.bt
```

Detect and trace network activity.

```bash
# Trace TCP active connections (connect())
sudo tcpconnect.bt

# Trace TCP passive connections (accept())
sudo tcpaccept.bt
```

**Expanded Usage :octicons-stack-16:**

You'll notice the `.bt` scripts themselves are not too long to read, but may be complicated to understand and write. The [language is fully documented here](https://bpftrace.org/docs/release_026/language), which means we can start learning interactively with something like Claude in a Virtual Machine.

!!! note "AI Usage"

    claude-sonnet-5 helped research and modify the code below.

When tracing `connect()` calls with `tcpconnect.bt`, I noticed the COMM did not always resolve to the "thing" that's creating it. For example, Firefox network calls would display as `Socket Thread` instead of `firefox`.

To patch `/usr/sbin/tcpconnect.bt` to change this behavior, at the very bottom of the script, add `$task` and `$pname` variables that will resolve the "comm" string to the "group_leader" string, which should be the "thing" that spawns it. This almost always is the plane name of the "thing", such as `firefox`, `chrome`, or `wget`. Finally just swap in `$pname` for comm in the print statement, and that's it.

It's recommended to just make a copy of the original file under `/usr/local/bin/` for your modifications, so you keep your version separate.

```c
    //SNIP

    // Add the two lines below, trace up to process name, not just the comm
    $task = (struct task_struct *)curtask;
    $pname = $task->group_leader->comm;

    time("%H:%M:%S ");
    printf("%-8d %-16s ", pid, $pname); /* replace comm with $pname */
    printf("%-39s %-6d %-39s %-6d\n", $saddr, $lport, $daddr, $dport);
  }
}
```

Optionally, add `#include <linux/sched.h>` to the include directives. The changes above may work fine without this, but it seems this may be necessary on some older systems (pending review).

```c
#ifndef BPFTRACE_HAVE_BTF
#include <linux/socket.h>
#include <linux/sched.h> /* add this line */
#include <net/sock.h>
```

Two things that would make this even better, resolving IPs to hostnames and getting the process path.
