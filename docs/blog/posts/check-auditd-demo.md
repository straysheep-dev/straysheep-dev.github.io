---
draft: false
date: 2024-05-15
categories:
  - check-auditd
  - auditd
  - logging
  - monitoring
  - cli
  - linux
  - compliance
  - threat-hunting
  - scripting
---

# Triage auditd logs with check-auditd

*Recently updated to mirror and support `ausearch` arguments. This post showcases those changes and how the tool works.*

<!-- more -->

## Overview

This script was written to help reduce time in reviewing systems running auditd for suspicious activity, or anomalies worth investigating further. Auditd logs are dense and rich in information, but the format is uniquely difficult to parse and read on its own.

With [check-auditd](https://github.com/straysheep-dev/linux-configs/blob/main/check-auditd.sh), logs are summarized and sorted based on frequency, and are printed in full color to highlight noteworthy data. There are three areas of focus for finding needles (splinters?) in the log haystack: command (binary or script) activity, files, and network activity.

It's important to note that in order for this tool to pull data from the system, auditd must be logging that data to begin with. Check [Neo23x0's auditd rules](https://github.com/Neo23x0/auditd), or my fork [here](https://github.com/straysheep-dev/auditd) to get started. I also have an [ansible role](https://github.com/straysheep-dev/ansible-configs/tree/main/install-auditd) and a [shell script](https://github.com/straysheep-dev/setup-auditd) to automate installing and setting up auditd, with [laurel](https://github.com/threathunters-io/laurel) as an option.

<video width="1080" controls>
  <source src="/blog/media/check-auditd-demo/Check-Auditd-Demo.mp4" type="video/mp4">
</video>

<!-- How to embed video:
https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video
https://github.com/squidfunk/mkdocs-material/discussions/3984
-->

## Usage

*This section contains the full `-h`, `--help` manual.*

check-auditd.sh; a wrapper to summarize auditd logs.

This script can do the following:

- Parse active auditd logs or a take a path to offline logs
- Prints a summary of UIDs appearing in events
- Search for command events matching a built in list of living-off-the-land binaries
- Match any entries of a specific [COMMAND]
- Show each unique command line string, sorted by frequency
- Shows log entries related to a [FILE], or a built in list of special files, sorted by frequency
- Summarizes logged network activity, such as dest port, dest IP, URL patterns, or applications making connections
- Network activity is also sorted by frequency

An attempt is made to filter out command events using specific 'ausearch' and 'aureport' strings to prevent queries from flooding the results.
Activity must already be logged by audit rules for this script to parse it out of a log file.

```
[*]Usage: ./check-auditd.sh -ts today [-if FILE] [OPTIONS...]
```

**MAIN ARGUMENTS**

`-ts`, `--start [start-date|start-time|keyword]`
> The time frame to begin searching in logs. Example date: 01/01/2024. Example time: 18:00:00. You can also use keywords.
> Keywords include: now, recent, this-hour, boot, today, yesterday, this-week, week-ago, this-month, this-year, or checkpoint.
> Currently can only use either a date or a time, not both.

`-te`, `--end [end-date|end-time|keyword]`
> The time frame to end searching in logs. If blank, default is 'now'. Like start-time, you can also use keywords.
> Currently can only use either a date or a time, not both.

`-if`, `--input`
> Path to an offline audit log file.

`-h`, `--help`
> Print this help menu.

**OPTIONAL ARGUMENTS**

> Only one of these arguments will work at a time. If multiple are present, the first one wins.

`-ae`, `--all-events`
> Display command, file, and network events.

`-ne`, `--net-events`
> Only display network related information.

`-fe`, `--file-events`
> Only display file related information.

`-ce`, `--cmd-events`
> Only display command related information.

`-c`, `--command`
> Match events that include `[COMMAND]`.

`-f`, `--file`
> Match events that include `[FILE]`.


## Next Steps

There's still more that can be done to make this a universally usable tool.

The built in lists for lolbins (`CMD_LIST`) and files (`FILE_LIST`) for example could be changed with arugments that allow you to bring your own list of strings to search for.

One downside to this shell script is the `ausearch` and `aureport` binaries need to be installed on the system you're working from. This is not a huge issue since these binaries typically come with auditd, and logs can be taken offline. The next evolution of this tool will need to be a programming language that can undestand each field as an object, and return or analyze data from there.

[Laurel](https://github.com/threathunters-io/laurel) restructures auditd logs into json format, meaning they can be parsed effectively with [`jq`](https://github.com/jqlang/jq). This has become a popular utility to have running on systems using auditd. Having the ability to parse raw audit.log files, and files generated by laurel would be a good milestone for this project.

