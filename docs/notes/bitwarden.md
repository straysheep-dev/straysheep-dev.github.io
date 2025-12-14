---
draft: false
date:
  created: 2024-05-02
  updated: 2025-06-15
categories:
  - bitwarden
  - how-to
  - password-managers
  - cli
---

# :simple-bitwarden: bitwarden

Notes related to the [bitwarden password manager](https://bitwarden.com/).

<!-- more -->


## bitwarden SSH

[Bitwarden now has its own SSH agent](https://bitwarden.com/help/ssh-agent/#configure-bitwarden-ssh-agent). To enable it, go to File > Settings > Enable SSH agent.

!!! note "flatpak"

	The flatpak version **does not** have SSH agent support at the time of updating this post.

You can import existing SSH private keys, or even generate new ED25519 SSH keys.

This does not have to interfere with your existing configuration. For example, [configure_gnupg](https://github.com/straysheep-dev/ansible-configs/tree/main/configure_gnupg) still writes the block below to the `~/.bashrc` file. However, the commands to use bitwarden's SSH agent are included in the comments so that:

- You can copy and paste them into a single terminal pane to leverage per-session
- You can easily comment out the gpg line and replace it to use bitwarden by default

The configuration block is here for reference.

```conf
export GPG_TTY="$(tty)"
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
gpgconf --launch gpg-agent

# You can alternatively use Bitwarden's SSH agent, either temporarily per-session by copy and pasting
# one of the lines below into your terminal, or here in bashrc by commenting out the gpg SSH_AUTH_SOCK.
# File > Settings > Enable SSH agent
# https://bitwarden.com/help/ssh-agent/#configure-bitwarden-ssh-agent
#export SSH_AUTH_SOCK=/home/$USER/.bitwarden-ssh-agent.sock
#export SSH_AUTH_SOCK=/home/$USER/snap/bitwarden/current/.bitwarden-ssh-agent.sock
```


## bitwarden CLI

<https://bitwarden.com/help/cli/>

Essentials:
```bash
bw --help
bw <command> --help
bw login
bw sync
bw lock
bw logout
```

### Programmatically work with attachments

Without a way to do this from any of the GUI based clients, this will help you work with large numbers of attachments.

**WARNING**: *Do not do use `bw` if you have certain types of `PowerShell` logging enabled, for example transcript logging. Similar to enabling clipboard monitoring in Sysmon, you can potentially write your entire vault to disk in clear text.*

- [About Logging in PowerShell](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging?view=powershell-5.1)
- [Script Tracing and Logging](https://docs.microsoft.com/en-us/powershell/scripting/windows-powershell/wmf/whats-new/script-logging?view=powershell-7.2)
- [Transcript Logging](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.host/start-transcript?view=powershell-7.2)
- [About Script Blocks](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_script_blocks?view=powershell-7.2)
- [About PowerShell Profiles](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.2)

*Script tracing* and *script block logging* records the content of scripts themselves for auditing purposes. If a malicious script runs or if a script breaks something, what was executed exactly at the time of running that script is written to a file on disk.

To avoid issues here, use variables for `bw` queries as demonstrated below. The most that can be retrieved in some cases is the name or title of an entry, or it's `itemid` which is acceptable. By doing this the variable runs the query each time the script is invoked rather than hardcoding the variable iteself. You'd need to have a valid login session to return any information.

*Transcript logging* logs all of the text that appears in a terminal session window. This is what can potentially be dangerous if you're searching your vault from PowerShell while this setting is enabled. It can be enabled manually via the `Start-Transcript` cmdlet. This cmdlet can also be enabled in a profile to run every time you open PowerShell. Similar to bash profiles, this means it will invoke `Start-Transcript` anytime you run PowerShell.

To check your PowerShell profile settings to see if this is enabled, review the locations mentioned [here](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.2). Also [check your registry](https://github.com/carlospolop/hacktricks/tree/master/windows-hardening/windows-local-privilege-escalation#powershell-transcript-files) to see if it's enabled there as well.

#### Retrieve an `itemid`

```bash
bw list items --search <query> --pretty | grep -P "^\s+\"id\":\s\".*\",$" | cut -d '"' -f 4 | head -n1
```

Get the `itemid` of your entry named 'GitHub'

If mutiple results are found, remove the regex to see what's returned and refine your query
```bash
bw list items --search GitHub --pretty | grep -P "^\s+\"id\":\s\".*\",$" | cut -d '"' -f 4 | head -n1
```

Put the query for the `itemid` into a variable

This should be only one line, so double quoting `"$()"` is fine
```bash
ITEM_ID="$(bw list items --search <query> --pretty | grep -P "^\s+\"id\":\s\".*\",$" | cut -d '"' -f 4 | head -n1)"
```

#### Upload files

You'll receive continuous responses to the console window for each attachment uploaded.
```bash
for file in ../path/*.test; do bw create attachment --file "$file" --itemid "$ITEM_ID"; done
```

Confirm the files are listed under "$ITEM_ID" attachments
```bash
bw get item "$ITEM_ID" --pretty
```

#### Enumerate attachments

We grep for any matches *after* `-A` the list of attachments, as other entries above the attachments line (which we don't want) could also match this regex

10000 is arbitrary, just a guess to an upper limit on how many lines of json the attachments cover - update this if you need to

DO NOT double quote the command substitution `$()`, this way each result is it's own line vs the list of results being a single string
```bash
ATTACHMENT_IDS=$(bw get item "$ITEM_ID" --pretty | grep -A 10000 -Px "^\s+\"attachments\": \[$" | grep -Px "^\s+\"id\": \"\w+\",$" | grep -Fv '-' | cut -d '"' -f 4)
```

#### Download attachments

DO double quote the variable of each `"$file"`, or each line from the output above
```bash
for file in $ATTACHMENT_IDS; do bw get attachment "$file" --itemid "$ITEM_ID"; done
```

#### Delete attachments

Same as downloading, with `delete` instead of `get`
```bash
for file in $ATTACHMENT_IDS; do bw delete attachment "$file" --itemid "$ITEM_ID"; done
```