---
draft: false
date:
  created: 2024-08-13
categories:
  - wazuh
  - edr
  - xdr
  - how-to
  - compliance
  - ossec
  - alerts
  - defense
  - blue-team
  - purple-team
  - automation
  - ansible
---

# :material-server-security: Wazuh all your things with Tailscale

This guide shows you how to get a Wazuh instance running over Tailscale on both Windows and Linux, using Sysmon(+forLinux), auditd, and all the tweaks you'll want to get started. This is ideal for a low resource, low budget, or lab scenario. You could eventually migrate this Wazuh data to a distributed cluster (proxmox), or real hardware if you grow with it.

<!-- more -->

---

!!! abstract "Overview"

    - 🌐 [Creating a tailnet](https://login.tailscale.com/start), [defining an ACL policy, connecting machines using tags](https://tailscale.com/kb/1192/acl-samples#restrict-based-on-purpose-tags)
    - Installing Ubuntu Server + Wazuh on a Hyper-V VM
        - 🌐 [Shell script installer](https://documentation.wazuh.com/current/quickstart.html#installing-wazuh)
        - 🌐 [wazuh-ansible](https://github.com/wazuh/wazuh-ansible/tags) [installation method](https://documentation.wazuh.com/current/deployment-options/deploying-with-ansible/index.html) (based on tag of latest stable version)
        - 🌐 [Docker](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html)
    - Applying and maintaining the [CIS L2 benchmark](https://static.open-scap.org/ssg-guides/ssg-ubuntu2204-guide-cis_level2_server.html) using [ansible tags](https://github.com/straysheep-dev/ansible-configs/tree/main/inventory_openscap_utils)
    - Ingesting logs: 3 components
    - Reading and understanding logs
    - How to write a custom integration, [including a ready-to-use Discord integration for shipping alerts](https://github.com/straysheep-dev/alert-service/tree/main/wazuh-integrations)

---

## Tailnet Access

[Create a Tailscale account](https://login.tailscale.com/start) if you already don't have one ([it's free for up to 3 users and 100 devices](https://tailscale.com/pricing)).

### 🏷️ Defining ACLs and Tags

This allows agents to communicate with ports on the manager node(s), but nothing else in your tailnet, not even other endpoints.

!!! tip "Tailnet ACLs"

    Tailscale's ACL rules work on a default-deny-all policy. You must define what can talk to what.

```json
	// Define the tags which can be applied to devices and by which users.
	"tagOwners": {
		<SNIP>
		"tag:wazuh-agent": ["autogroup:admin"],
		"tag:wazuh-node":  ["autogroup:admin"],
	},
    "acls": [
	<SNIP>
		// Allow Wazuh agents to communicate with Wazuh nodes
		{
			"action": "accept",
			"src":    ["tag:wazuh-agent"],
			"dst":    ["tag:wazuh-node:1514,1515,55000"],
		},
    ],
    <SNIP>
```

*Be sure your Wazuh nodes ingesting data also have their firewalls open on tcp/1515 tcp/1514 tcp/55000 **for their tailnet interface only**.*


### 🏷️ Tagging Devices

This can be done when generating an authkey, or once a device is already in your tailnet.

If you've already enrolled some endpoints, and have an existing tailnet, you can add a tag to them from the Tailnet dasboard.

- Click the `...` three dots
- Choose `Edit ACL Tags...`
- Choose `Add tags`, or go to `Manage tags in Access Controls` if you haven't created tags yet
- Create a tag called something like `wazuh-agent` (it can be anything, but has to match what's in your ACLs)
- Create another tag called `wazuh-node` for the server components (this tag could be used on each server component if they're distributed)

![](../media/wazuh-tailscale/wazuh_tailscale_tailnet_dashboard.png)

If you're still enrolling more endpoints, tag your auth keys as either `wazuh-node` or `wazuh-agent`.

- `Settings` > `Keys` > `Generate auth key ...`
- Tag:`wazuh-node` for server components or Tag:`wazuh-agent` for endpoints

---

## Manager VM Requirements

Check the [Quick Start](https://documentation.wazuh.com/current/quickstart.html) first.

A headless server is the ideal environment for Wazuh to live for performance sake, even in a lab scenario. To avoid any exposed web interfaces, use SSH local portforwarding to access the Wazuh manager dashboard after installing it. So your localhost:8443 can reach the Wazuh node's localhost:443 to login.

```bash
ssh -L 127.0.0.1:8443:127.0.0.1:443 user@wazuh-ip
```

- Ubuntu 22.04 Server
- 80 GB Disk Space (50GB for logs, 30GB for OS)
- 4 vCPU
- 8 GiB RAM (will exceed 8 if you run this with a GUI)


### Get an ISO

[Grab the latest server image and signatures](https://releases.ubuntu.com).

```bash
url_list='
https://releases.ubuntu.com/jammy/SHA256SUMS
https://releases.ubuntu.com/jammy/SHA256SUMS.gpg
https://releases.ubuntu.com/jammy/ubuntu-22.04.4-live-server-amd64.iso
'

for remote_file in $url_list
do
	curl -LfO "$remote_file"
done

gpg --verify SHA256SUMS.gpg SHA256SUMS || exit 1
sha256sum -c SHA256SUMS --ignore-missing || exit 1
```


### Install the OS (Hyper-V)

!!! info "Hyper-V"

    This guide uses Hyper-V, since it's less often covered than VMware and VirtualBox, which are heavily documented and more beginner friendly.

If Hyper-V isn't already enabled on your system, [enable it](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v).

```powershell
# Run as administrator
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
shutdown.exe /t 0 /r
```

Once rebooted, you're ready to continue:

- Run Hyper-V Manager as admin (search for Hyper-V manager in the Start Menu)
- Right-Click your hostname in the Hyper-V Manger window, New > Virtual Machine...
- Name: Wazuh
- Check "Store in a different location"
- `PS> mkdir "C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines\Wazuh"`
- Select this folder
- Choose "Next"
- Select "Generation 2"
- Startup memory: 8192, leave "Use Dynamic Memory" checked
- Choose "Next"
- Connection: "CustomNATSwitch" ([which you should create if you're doing a lot with Hyper-V and home labs](https://github.com/straysheep-dev/windows-configs#hyper-v-default-switch-has-no-dhcp))
- Choose "Next"
- Create a virtual hard disk, set the Location: `C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines\Wazuh\`
- Choose "Next"
- Choose Install an operating system later
- Choose "Next", then "Finish"

!!! info "Custom NAT Switch"

    A custom NAT switch solves a lot of problems in Hyper-V networking.

    - The default Hyper-V network "regenerates" every time you reboot, so it's considered a new interface
    - This is an issue [if you have strict firewall rules on the host (AllowInboundRules: False)](https://learn.microsoft.com/en-us/windows/security/operating-system-security/network-security/windows-firewall/tools#shields-up-mode-for-active-attacks)
    - If you do set AllowInboundRules: False (which is generally a good idea) this will also break DHCP

    With a custom switch, **while you can only have one custom NAT switch per host**, it's persistent, you define the IP range, and firewall rules persist. If you configure WSL to talk to this internal IP range, it will continue to work

Make the custom NAT switch:

```powershell
New-VMSwitch -SwitchName "CustomNATSwitch" -SwitchType Internal
$ifindex = (Get-NetAdapter -IncludeHidden | where { $_.Name -eq "vEthernet (CustomNATSwitch)" }).ifIndex
New-NetIPAddress -IPAddress 10.55.55.1 -PrefixLength 24 -InterfaceIndex $ifindex
New-NetNat -Name CustomNATNetwork -InternalIPInterfaceAddressPrefix 10.55.55.0/24
```

If you want WSL to be able to reach this switch, [allow both interfaces to forward packets](https://github.com/straysheep-dev/windows-configs?tab=readme-ov-file#communicating-with-hyper-v):

```powershell
# Apply
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL (Hyper-V firewall))' -or $_.InterfaceAlias -eq 'vEthernet (CustomNATSwitch)'} | Set-NetIPInterface -Forwarding Enabled

# Remove
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL (Hyper-V firewall))' -or $_.InterfaceAlias -eq 'vEthernet (CustomNATSwitch)'} | Set-NetIPInterface -Forwarding Disabled
```

!!! warning "If WSL and Hyper-V can't talk after a reboot..."

    You may have to re-enable this after every reboot.


Next, modifying the settings before we start:

- Right-Click the "Wazuh" VM name from the list, choose Settings
- Security > Secure Boot > Template: `Microsoft UEFI Certificate Authority`
- Memory > Minimum RAM: `4096`
- Memory > Maximum RAM: `8192`
- Processor > Number of virtual processors: 4 (6 if you can)
- Click "SCSI Controller", select "DVD Drive", click "Add"
- You'll be on the DVD drive tab automatically, select "Image file:" and choose your ubuntu server ISO
- Integration Services > You can safely disable everything, for isolation
- Checkpoints > uncheck "Use automatic checkpoints"
- Checkpoints > Checkpoint File Location: `C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines\Wazuh`
- Smart Paging File Location > `C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines\Wazuh`
- Apply, then OK
- Take a checkpoint (snapshot) here so you don't have to do this again if anything goes wrong

![](../media/wazuh-tailscale/wazuh_tailscale_hyper-v_conf.png)

Now install Ubuntu as you would normally. You'll need to manually configure `eth0`.

- Subnet: 10.55.55.0/24
- IPv4: 10.55.55.X
- Gateway: 10.55.55.1
- Name Servers: [1.1.1.1](https://www.cloudflare.com/learning/dns/dns-over-tls/),[9.9.9.9](https://docs.quad9.net/)

Use the entire disk as LVM (default), you do not necessarily *need* encryption if your host has full disk encryption. Set the server's name to something like `wazuh-node` or `wazuh-standalone`. When you get to SSH access, check "Install OpenSSH server" and then "Import SSH Key" to provide your GItHub username. It will pull your public key (if you have one) from the GitHub API (you can do this with `curl` by the way) into the `~/.ssh/authorized_keys` file. Let the install complete.

When it's done, follow the prompts as usual to "eject" the ISO and reboot. SSH in and ensure the packages are up to date with a `sudo apt update; sudo apt upgrade -y; sudo snap refresh`. Then `sudo systemctl poweroff` and take another checkpoint.

---

### Configure State (CIS L2)

!!! abstract "CIS L2 Benchmark for Ubuntu Server"

    Here's where you can configure the system into a state, such as the [CIS L2 benchmark for Ubuntu server](https://static.open-scap.org/ssg-guides/ssg-ubuntu2204-guide-cis_level2_server.html).

    Apply any additional changes you normally would to secure or configure a server here. These changes need to avoid getting in the way of Wazuh functioning. You may need to test and debug any other hardening policies you apply outside of those mentioned here.

    The tags and tasks applied in the steps below will still allow Wazuh to function.

Install Ansible.

```bash
sudo apt install -y python3-pip
python3 -m pip install --user ansible
# logout, then back in to update your PATH
```

Clone my [ansible-configs](https://github.com/straysheep-dev/ansible-configs) repo.

```bash
mkdir ~/src
cd ~/src
git clone git@github.com:straysheep-dev/ansible-configs.git
cd ansible-configs/
gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys '9906 9EB1 2D40 9EA9 3BD1 E52E B09D 00AE C481 71E0'
git verify-commit $( git log | head -n1 | awk '{print $2}')

# You could loop through every commit and verify it, if you can't download over SSH
for signature in $(git log | grep -P "^commit" | awk '{print $2}'); do if ! (git verify-commit "$signature" 2>&1 | grep 'Good signature'); then echo "[WARNING]: $signature"; fi ; done
```

Uncomment the following roles in `playbook.yml` to gather some essential packages and tools:

```yml
    - role: "install_unbound"
#    - role: "install_ykman"
#    - role: "install_chrome"
#    - role: "build_wireguard_server"
    - role: "build_tailscale_node"
#    - role: "build_ubuntu_desktop"
#    - role: "install_auditd"
#    - role: "hyperv_guest_tools"
    - role: "configure_gnupg"
    - role: "configure_microsoft_repos"
#    - role: "install_vscode"
    - role: "install_powershell"
    - role: "install_sysmon"
```

!!! tip "Unbound DNS"

    My unbound DNS role is usually always added so the machine can leverage DNS over TLS. [You can do this with stubby instead](https://github.com/straysheep-dev/linux-configs/tree/main/dns), or if you prefer another method altogether that's fine too. `systemd-resolved` has not had full support for DNS over TLS or HTTPS, and is just starting to support it as of 2023/2024.

[Generate an authkey](https://tailscale.com/kb/1085/auth-keys). From your Tailnet dashboard go to `Settings` > `Keys` > `Genereate auth key...`

![](../media/wazuh-tailscale/wazuh_tailscale_tailnet_authkey.png)

Add the authkey to an ansible-vault.

```bash
cd ~/src/ansible-configs
rm auth.yml
ansible-vault create auth.yml
```

Contents of `auth.yml`, *with* the double quotes around each value:

```yml
ansible_become_password: "<your_sudo_password>"
tailscale_authkey: "<tskey-auth-abcdef0123456789abcdef0123456789>"
```

Run the playbook.

```bash
echo "Enter Vault Password"; read -s vault_pass; export ANSIBLE_VAULT_PASSWORD=$vault_pass
# paste your vault password, hit enter

ansible-playbook -i inventory/inventory.ini -e "@auth.yml" --vault-pass-file <(cat <<<$ANSIBLE_VAULT_PASSWORD) -v ./playbook.yml
```

![](../media/wazuh-tailscale/wazuh_tailscale_ansible_tailnet_enrollment.png)

If everything succeeded, poweroff the machine and take a checkpoint. Otherwise read the Ansible output and revise any potential issues. It's possible some tasks may hang, especially if your network drops the connection. Easiest thing here is to `ctrl+c` and just run the playbook again. It's configuring a state and the roles were written to safely run more than once.

---

**⚙️ Applying the SCAP Content**

SSH back in, this time `cd ~/src/ansible-configs/inventory_openscap_utils`.

!!! tip "Ansible OpenSCAP Utils"

    *This section is from [OpenSCAP Practical Usage](http://127.0.0.1:8000/blog/2024/05/29/openscap-practical-usage/).*

    If you haven't already, clone the latest [ansible-configs](https://github.com/straysheep-dev/ansible-configs):

    ```bash
    sudo apt update; sudo apt install -y curl unzip

    mkdir ~/src
    cd ~/src
    git clone https://github.com/straysheep-dev/ansible-configs
    cd ansible-configs/inventory_openscap_utils
    ```

    and follow the steps below.

Run `./download-content.sh` to pull the latest OpenSCAP policy release from GitHub.

![](../media/openscap-ansible/openscap_ansible_1.png)

It will automatically `unzip` policy files matching the current OS. To specify another OS, use `-u <os-name>`.

![](../media/openscap-ansible/openscap_ansible_2.png)

You can list all available policy files with `-l`.

![](../media/openscap-ansible/openscap_ansible_3.png)

The wrapper script is written to interpret posix-extended regex. Combine rules from multiple policies like this.

![](../media/openscap-ansible/openscap_ansible_4.png)

Comment out any rules in the tags-*.txt files you don't want to apply, or find break the deployment.

![](../media/openscap-ansible/openscap_ansible_5.png)

!!! example "Why Ansible?"

    Running `./apply-tags.sh` with the `-d|--diff` options will run Ansible with `--check --diff`, showing you the changes without making them, and failing if a change cannot be made correctly. This is the strength of this approach. With states maintained as tags you can more easily isolate and debug what could have broken a system, especially if you're testing tags in groups.

---

**📚 Script Usage**

The wrapper script has built in `-h|--help` information. You can pass it all the arguments you will usually need to either test a policy on the localhost, or use an inventory + vault.

![](../media/openscap-ansible/openscap_ansible_6.png)

When the script executes a playbook, the raw command with all of the tags listed will be printed to your screen. This is copy / paste-able to repeat manually if necessary.

![](../media/openscap-ansible/openscap_ansible_7.png)

**🏷️ Premade Tags**

There are also folders in the same directory of premade tag sets that will apply as many rules as possible without breaking a system, exceptions being `aide` and `auditd` rules. The reason being these rules often endlessly loop, need tuned to your environment, or break the deployment. Use the [`aide`](https://github.com/straysheep-dev/ansible-configs/tree/main/aide) and [`install_auditd`](https://github.com/straysheep-dev/ansible-configs/tree/main/install_auditd) roles instead.


Once all the loops finish, `sudo systemctl reboot` and SSH back in. Check the system with `systemctl status` to ensure `State:  running`. Check `journalctl -xe` for any errors or issues. If all looks good, poweroff and once again take a checkpoint.

Now you're ready to install Wazuh.

!!! tip "`tmux`"

    When you SSH into the Wazuh server, start a `tmux` session to easily return to your console if you're disconnected. You can also monitor the install with a second pane using `prefix+"` then in the new pane run `htop`.

---


## Install Wazuh

Installation can take 20-30 minutes, or even more depending on your setup and connection speed. Kick off the installer script or playbook and take a break.

No matter which method you choose, you can start by ensuring ports tcp/1515, tcp/1514, tcp/55000 allow inbound connections from the Tailnet IP ranges:

```bash
tailnet_ip=$(ip addr show tailscale0 | grep -oP "\b100(\.(\d){1,3}){3}\b")
sudo ufw allow in on tailscale0 to $tailnet_ip proto tcp port 1515,1514,55000 comment wazuh
```

!!! quote "[Wazuh Docs: Large Scale Deployment](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html)"

    If you are deploying Wazuh in a large environment, with a high number of servers or endpoints, keep in mind that this deployment might be easier using automation tools such as [Puppet](https://documentation.wazuh.com/current/deployment-options/deploying-with-puppet/index.html), [Chef](https://github.com/wazuh/wazuh-chef), SCCM, or [Ansible](https://documentation.wazuh.com/current/deployment-options/deploying-with-ansible/installation-guide.html).

---

### Agent Authentication

!!! abstract "Verifying Wazuh Connections"

    This section details steps you can use to lock down connections to and from your Wazuh instance. You may want to do this for a few reasons.

    1. Prevent a rogue Wazuh server from intercepting data from endpoints
    2. Prevent rogue endpoints from sending data into your Wazuh instance.

    An adversary-controlled Wazuh server having full visibility into an endpoint is less than ideal for a number of reasons, and of these two points is likely the easier to achieve in a real world scenario where there's no authentication.

    Rogue endpoints enrolling would ideally have little effect on your Wazuh instance, unless a known exploit is available for one of the backend server components, and this exploit can be leveraged by an agent sending crafted data to the data ingestors. You're already using Wazuh to detect and respond to emerging threats, so a malicious endpoint is baked-in to the potential threat model. Practically this would only be used in a more targeted scenario.

There are effectively three options. The [Wazuh User Manual: Agent Enrollment Security Options](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/security-options/index.html) page details all of these.

#### 🔑 Password Auth

*This method requires using a password during the enrollment process to ensure that Wazuh agents enrolled in the Wazuh manager are authenticated.*

- [Instructions](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/security-options/using-password-authentication.html)
- It's recommended to use the same password across all nodes in a multi-node deployment
- Agents and managers will no longer communicate without the password set on both ends
- This will break exisitng connections to your manager
- Configure the manager to expect password auth
- Configure each agent to use the password during deployment or by adding the `authd.pass` file later

Generally, if you want to get your Wazuh instance protected with minimal work, this is the way to go. Not only does it prevent unauthorized agents from enrolling, but a rogue server would need to know the password the agent requests when during enrollment. In other words, both the server and agents are protected.

This output is from the agent's side when providing an enrollment password on the agent, for a Wazuh server that does not require a password to authenticate.

```bash
root@bind9-node:~# tail -F /var/ossec/logs/ossec.log
2025/01/01 11:22:33 wazuh-agentd: INFO: Requesting a key from server: 10.0.0.20
2025/01/01 11:22:33 wazuh-agentd: INFO: Using agent name as: bind9-node
2025/01/01 11:22:33 wazuh-agentd: INFO: Waiting for server reply
2025/01/01 11:22:33 wazuh-agentd: ERROR: Invalid request for new agent (from manager)
2025/01/01 11:22:33 wazuh-agentd: ERROR: Unable to add agent (from manager)
```

These steps enforce password auth on the server:

```bash
sudo su -
echo "Enter an enrollment password:"; read -s wazuh_auth_pass
sed -E -i 's/<use_password>.*<\/use_password>/<use_password>yes<\/use_password>/' /var/ossec/etc/ossec.conf
echo "$wazuh_auth_pass" > /var/ossec/etc/authd.pass
chmod 640 /var/ossec/etc/authd.pass
chown root:wazuh /var/ossec/etc/authd.pass
systemctl restart wazuh-manager
```

These steps configure an agent to use password auth:

=== "Linux/Unix"

    ```bash
    sudo su -
    WAZUH_MANAGER="<server-ip-or-fqdn>" WAZUH_REGISTRATION_PASSWORD="<password>" apt-get install wazuh-agent
    # Installing the agent with these env variables will make the necessary configurations
    # for password auth to occur.

    # If you have existing agents, you will need to deploy the password
    # to each of them:
    sudo su -
    echo "<password>" > /var/ossec/etc/authd.pass
    chmod 640 /var/ossec/etc/authd.pass
    chown root:wazuh /var/ossec/etc/authd.pass
    systemctl restart wazuh-agent.service
    ```

=== "Windows"

    ```powershell
    .\wazuh-agent-4.9.2-1.msi /q WAZUH_MANAGER="<server-ip-or-fqdn>" WAZUH_REGISTRATION_PASSWORD="<password>"
    Restart-Service -Name wazuh
    # Installing the agent with these env variables will make the necessary configuraitons
    # for password auth to occur.

    # If you have existing agents, you will need to deploy the password
    # to each of them:
    # 32-bit
    echo "<password>" > "C:\Program Files\ossec-agent\authd.pass"
    # 64-bit
    echo "<password>" > "C:\Program Files (x86)\ossec-agent\authd.pass"
    # Restart the agent
    Restart-Service -Name wazuh
    ```


#### 🎫 Agent Verification

!!! quote ""

    *This method uses SSL certificates to verify that a Wazuh agent is authorized to enroll in the Wazuh manager.*


#### 🎫 Manager Verification

!!! quote ""

    *This method uses SSL certificates to verify the identity of the Wazuh manager before a Wazuh agent sends the enrollment request.*


---

### Tailnet Enrollment

*How to enroll multiple endpoints each with their own tailnet_authkey*

This was covered above in [Configure State (CIS L2)](#configure-state-cis-l2) with screenshots enrolling just the Wazuh manager machine into a tailnet. Enrolling multiple devices at once is as simple as giving each endpoint a unique variable with a pregenerated authkey.

Here's what your vault could contain:

```yml
client01_tsauthkey: "tskey-abcdef0123456789abcdef0123456789"
server01_tsauthkey: "tskey-abcdef0123456789abcdef0123456789"
<SNIP>
```

...and the corresponding inventory file:

```yml
managed_group:
  hosts:
    172.16.20.20:
      ansible_port: 22
      ansible_user: user
      tailscale_authkey: "{{ client01_tsauthkey }}"
    172.16.20.21:
      ansible_port: 22
      ansible_user: server
      tailscale_authkey: "{{ server01_tsauthkey }}"
      is_exit_node: "true"
      <SNIP>
```

---

### Shell Scripting

#### 📦 Deploy a Single Node

[There's a shell script that will deploy the SIEM on a single node](https://documentation.wazuh.com/current/quickstart.html).

```bash
cd ~/src
curl -sO https://packages.wazuh.com/4.8/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

![](../media/wazuh-tailscale/wazuh_tailscale_wazuh_installer_script.png)


#### 📦 Deploy Agents

[Deploy agents with a few lines](https://documentation.wazuh.com/current/quickstart.html#next-steps).

!!! note "Wazuh's Public Key"

    ```bash
    $ gpg --with-fingerprint /usr/share/keyrings/wazuh.gpg
        gpg: WARNING: no command supplied.  Trying to guess what you mean ...
        pub   rsa4096/0x96B3EE5F29111145 2016-08-01 [SC] [expires: 2027-05-15]
            Key fingerprint = 0DCF CA55 47B1 9D2A 6099  5060 96B3 EE5F 2911 1145
        uid                             Wazuh.com (Wazuh Signing Key) <support@wazuh.com>
        sub   rsa4096/0x417F3D5A664FAB32 2016-08-01 [E] [expires: 2027-05-15]
            Key fingerprint = 7C74 8627 7A6A 9681 DF3F  3D8A 417F 3D5A 664F AB32
    ```

=== "Linux"

    ```bash
    curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && sudo chmod 644 /usr/share/keyrings/wazuh.gpg
    echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list
    sudo apt-get update
    # Change to root
    sudo su -
    WAZUH_MANAGER="<server-ip-or-fqdn>" WAZUH_REGISTRATION_PASSWORD="<password>" apt-get install wazuh-agent
    ```

=== "Windows"

    ```powershell
    cd $env:TEMP
    iwr -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.8.0-1.msi -OutFile wazuh-agent.msi
    Start-Process msiexec '/quiet /i wazuh-agent.msi WAZUH_MANAGER="<server-ip-or-fqdn>" WAZUH_REGISTRATION_PASSWORD="<password>"'
    ```

!!! tip "Deploy agents as root / admin"

    `sudo su -` or run from an administrator powershell session when using the `WAZUH_MANAGER="<ip>" apt-get install wazuh-agent` otherwise the environment variable isn't read correctly.

    If this happens, manually add your manager's tailnet IP in `/var/ossec/etc/ossec.conf` or `C:\Program Files (x86)\ossec-agent\ossec.conf` **on the endpoint**:

    ![](../media/wazuh-tailscale/wazuh_tailscale_wazuh-agent.png)

---


### Ansible

The Ansible repo allows for a distributed cluster or a single node / agent deployment. **This guide recommends using Ansible to help you automate your infrastructure and endpoint enrollment. Create your own roles or plays for custom configurations.**

- 🌐 [wazuh-ansible](https://github.com/wazuh/wazuh-ansible)
- 🌐 [You'll want to use the latest release's tag](https://github.com/wazuh/wazuh-ansible/releases/latest)


Obtain the latest release version:

```bash
RESPONSE=$(curl -s https://api.github.com/repos/wazuh/wazuh-ansible/releases/latest)
URL=$(echo "$RESPONSE" | awk -F '"' '/zipball_url/{print $4}')
TAG=$(echo "$RESPONSE" | awk -F '"' '/tag_name/{print $4}')

curl --silent -L "$URL" --output wazuh-ansible-"$TAG".zip
```

Alernatively you could clone the entire repo and checkout the latest tag:

```bash
git clone https://github.com/wazuh/wazuh-ansible
cd wazuh-ansible
git tag
git checkout tags/v4.8.0
```

If you want to verify the latest tag signature, look for the `Good signature from "GitHub <noreply@github.com>" [unknown]` line.

```bash
# Obtain GitHub's signing key
# https://github.com/web-flow.gpg
# 5DE3E0509C47EA3CF04A42D34AEE18F83AFDEB23
gpg --keyserver hkps://keyserver.ubuntu.com:443 --recv-keys '5DE3E0509C47EA3CF04A42D34AEE18F83AFDEB23'
git verify-commit v4.8.0
gpg: Signature made Thu Jun  6 07:09:59 2024 UTC
gpg:                using RSA key B5690EEEBB952194
gpg: Good signature from "GitHub <noreply@github.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 9684 79A1 AFF9 27E3 7D1A  566B B569 0EEE BB95 2194
```

!!! tip "Verifying Git Signatures: Tags vs Releases vs Commits"

    If commit signing is enforced for a project, and it's on GitHub, commits will either be signed by the committer's key, or GitHub's signing key.

    Tags are sometimes a more "complete" snapshot of a repo's contents, where commits only validate the changes they relate to. It's better to check the tag or even a release signature when possible, else you'll need to loop through every commit if they've all been signed. A release signature is often it's own .asc file released along side the other release files, so you can check the integrity of each file with the .asc signature using the related public key.

    In the case of wazuh-ansible the tags are actually commits. Basically you have a few options:

    - Clone the repo over SSH (trusting GitHub's keys)
    - Ensure your DNS is trustworthy (use 1.1.1.1 or 9.9.9.9 or 8.8.8.8 over TLS / HTTPS)
    - Loop through the commit signatures

    In all cases you'll need to place trust primarily into the project (which your're already doing by using it) and GitHub's infrastructure. This is an overly paranoid take on verifying integrity, but something to consider when an EDR / SIEM has complete access to your infrastructure. If you'd rather use Wazuh's apt / dnf installation options, they rely on the Wazuh signing key which has a known fingerprint of `0DCF CA55 47B1 9D2A 6099  5060 96B3 EE5F 2911 1145`. Use this fingerprint to cross reference it against [the GPG key from Wazuh.com](https://packages.wazuh.com/key/GPG-KEY-WAZUH) and [keyserver.ubuntu.com](https://keyserver.ubuntu.com/pks/lookup?search=0DCF+CA55+47B1+9D2A+6099++5060+96B3+EE5F+2911+1145&fingerprint=on&op=index)


#### 📦 Deploy a Single Node

`cd` into the project folder and create a new inventory file.

```bash
nano inventory.ini
```

Let's assume your ansible control node in this case is also the one you're installing Wazuh's server / node components onto. If not, then replace the IP in the inventory file with the IP you'd use to SSH into the machine where Wazuh will be installed.

```ini
[aio]
<your server host's IP>

[all:vars]
ansible_ssh_user=root
ansible_ssh_private_key_file=/path/to/ssh/key.pem    # Comment this line if you're using a YubiKey
ansible_ssh_extra_args='-o StrictHostKeyChecking=no'
```

If you are in fact doing this "locally" it will still try to ssh into 127.0.0.1. You'll need a public key your own localhost will accept ssh connections over.

```bash
sudo su -
ssh-keygen  # Enter all defaults, no password
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
ssh-add -L >> ~/.ssh/authorized_keys
```

!!! info "Temporary SSH key for deployment"

    You should delete this public / private key pair once you're done installing Wazuh. It's only available on the server, so it's not a huge risk, but should not be left behind.

Execute with

```bash
ansible-playbook -i inventory.ini -b -v ./wazuh-single.yml
```

---

#### 📦 Deploy Agents

!!! info "Tailnet connections"

    Your endpoints should already be enrolled, or get unrolled, before the wazuh-agent is installed so it can reach the wazuh-manager.

`cd` into the project folder and edit the `wazuh-agent.yml` playbook.

```bash
cd playbooks
nano wazuh-agent.yml
```

Replace `hosts:` with either the keyword `localhost` to run locally or create a group name to reference in an inventory file. Replace `address:` with your Wazuh manager's Tailnet IP.

```yml
---
- hosts: tailnet_group
  become: yes
  become_user: root
  roles:
    - ../roles/wazuh/ansible-wazuh-agent
  vars:
    wazuh_managers:
      - address: <your Wazuh manager's Tailnet IP>
        port: 1514
        protocol: tcp
        api_port: 55000
        api_proto: 'https'
        api_user: wazuh
        max_retries: 5
        retry_interval: 5
```

We'll use an inventory file, assuming you may want to deploy the agent on a number of endpoints.

- `tailnet_group` is the group name we came up with for these hosts
- All you need is the tailnet IP of the endpoint
- We'll be authenticating and running as root to avoid having to create a vault with sudo passwords for each endpoint
- `/root/.ssh/authorized_keys` must have a public key to access your connection

```yml
tailnet_group:
  hosts:
    172.16.20.20:
      ansible_port: 22
      ansible_user: user
    172.16.20.21:
      ansible_port: 22
      ansible_user: server
<SNIP>
```

!!! tip "Windows Endpoints"

    You effectively have two options for opening Windows endpoints to Ansible provisioning:

    - WinRM (Domain-Joined, ideally with Kerberos auth)
    - 🌐 [SSH (Best for non-domain-joined endpoints)](https://github.com/straysheep-dev/windows-configs/blob/main/Manage-OpenSSHServer.ps1)

    First, change the playbook to support the become method `runas`, and specify an admin user.

    ```yml
    - hosts: tailnet_group
      become: yes
      become_user: admin
      become_method: runas
      <SNIP>
    ```

    Next, update your `inventory.yml` file by appending the following options to your Windows endpoints:

    - `cmd` is the default shell for SSH on Windows
    - Change this to `powershell` if you've defined PowerShell as the default login shell
    - Ensure `C:\Temp` exists, or specify a different temp directory

    ```ini
    <SNIP>
      vars:
        ansible_port: 22
        ansible_user: admin
        ansible_connection: ssh
        ansible_shell_type: cmd
        become_method: runas
        shell_type: cmd
        remote_tmp: "C:\Temp\ansible"
    <SNIP>
    ```

    See the following references:

    - 🌐 [Ansible Playbook Fails on Windows](https://devops.stackexchange.com/questions/16532/ansible-playbook-fails-on-windows-server)
    - 🌐 [Ansible Playbook Become Error](https://stackoverflow.com/questions/66671945/ansible-playbook-error-the-powershell-shell-family-is-incompatible-with-the-sud)

Execute with:

```bash
~/.local/bin/ansible-playbook -i inventory.ini -b -v ./wazuh-agent.yml
```

---

### Docker

*⚠️ TO DO: This section is still under construction, check back later! ⚠️*

---

## Disk Usage

*⚠️ TO DO: This section is still under construction, check back later! ⚠️*

Paths that will contain a large amount of data:

- `/var/ossec/queue/vd/feed` Vulnerability data, set a retention policy in `vulnerability-detection` of `ossec.conf` on the manager
- `/var/ossec/logs` Logs from your endpoints

---

## Central Management

*⚠️ TO DO: This section is still under construction, check back later! ⚠️*

[Wazuh User Manual: Central Configuration](https://documentation.wazuh.com/current/user-manual/reference/centralized-configuration.html)

- Config per OS?
- Multiple config templates?

---

## Ingesting Logs

*For reference, the post from Wazuh's blog below is a great starting point for auditd, sysmon, sysmonforlinux, and a repo to use for custom decoders and rules.*

- 🌐 [Detecting the SysJoker malware with Wazuh](https://wazuh.com/blog/detecting-sysjoker-backdoor-malware-with-wazuh/)
- 🌐 [github/socfortress/Wazuh-Rules](https://github.com/socfortress/Wazuh-Rules)

!!! info "The 3 Components"

    What can be confusing at first, is "what exactly do I need to start ingesting and seeing log data into my Wazuh instance?".

    This requires 3 essential pieces:

    - Logging configuration: This includs config.xml (sysmon) or audit.rules (auditd)
    - Wazuh decoder file: Wazuh uses this file to interpret the log data, (e.g. sysmon and auditd data are very different) basically, you need a custom decoder
    - Wazuh rule file: This tells Wazuh what to do with the data, how to classify it's severity and whether to show it at all

    If this seems overwhelming don't worry, this blog exists to overcome this jump by detailing exactly what you can "leave as a default" and what you can maintain.

    **The goal is to get the data generated by your existing logging configurations to appear in Wazuh and be actionable while still maintainable.**

    At first glance in the Wazuh docs, configuring auditd rules to fire is daunting, and basically requires you to now maintain new keys in your config, or potentially even rewrite it to work with Wazuh rules in this way. Fortunately, and **largely thanks to the SOCFortress files**, it's not like that anymore using these "general" decoder and rule files, and Wazuh does a lot of heavy lifting using these components out of the box now.


### Sysmon

Get started with one of the following configuration options:

- 🌐 [github.com/SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config)
- 🌐 [github.com/Neo23x0/sysmon-config](https://github.com/Neo23x0/sysmon-config), a more up to date fork
- 🌐 [github.com/olafhartong/sysmon-modular](https://github.com/olafhartong/sysmon-modular), more advanced and requires tuning

***NOTE**: Neo23x0's version is recommended.*

On the endpoint:

```powershell
$url_list = @(
    "https://github.com/Neo23x0/sysmon-config/raw/master/sysmonconfig-export.xml",
    "https://live.sysinternals.com/Sysmon64.exe"
    )

# Only check the config file, Sysmon is signed by Microsoft, can be reviewed with sigcheck64.exe
$sha256_list = @(
    "6625BDD777DDC230730EBA628607F4B99123A011ED5B4AE91C20A264AC2DA3B9"
    )

cd $env:TEMP
$progressPreference = 'silentlyContinue'

foreach ($url in $url_list) {
    $basename = Split-Path -Path $url -Leaf
    iwr -Uri "$url" -OutFile $basename
}

if (!(get-filehash sysmonconfig-export.xml | sls $sha256_list))
{
    Write-Host -ForegroundColor RED "[WARNING]SHA256 mismatch $basename"
} else
{
    Write-Host -ForegroundColor Green "[OK]SHA256SUM $basename"
}

# Install Sysmon with config file
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml

# Configure wazuh-agent to ship Sysmon events to the manager
$configuration_text = "
<ossec_config>
  <localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>
</ossec_config>
"

echo $configuration_text | Out-File -FilePath "C:\Program Files (x86)\ossec-agent\ossec.conf" -Encoding ASCII -Append
Restart-Service WazuhSvc
```

!!! tip "Manage-Sysinternals.ps1"

    There's also [my PowerShell script](https://github.com/straysheep-dev/windows-configs/blob/main/Manage-Sysinternals.ps1) that will automatically download a pinned version of SwiftOnSecurity's sysmon-config.xml, Sysmon itself (as well as any other Sysinternals tools you select) and install it all for you under `C:\Tools`.

---

### SysmonForLinux

*The blog post uses config files from the <https://github.com/socfortress/Wazuh-Rules> repo, so you should fork it and use them in your own infrastructure.*

These steps use hard-coded hashes. If the hash changes, go to the source (ideally your fork) and check what changed.


**Get the Sysmon config, Wazuh rules, and Wazuh decoder file.**

On the endpoint:

```bash
cd /tmp
curl -LfO 'https://wazuh.com/resources/blog/detecting-sysjoker-backdoor-malware-with-wazuh/sysmonforlinux-config.xml'
sha256sum ./sysmonforlinux-config.xml | grep -P "\bca8a78c13ade0acc6778c9ed100e4ca5e073403ba6ad208d44f69b5bdbcfe222\b" || echo "[WARNING]checksum error."
sudo sysmon -c ./sysmonforlinux-config.xml
```

One the server:
```bash
cd ~/src
url_list='
https://wazuh.com/resources/blog/detecting-sysjoker-backdoor-malware-with-wazuh/sysmonforlinux-config.xml
https://raw.githubusercontent.com/socfortress/Wazuh-Rules/main/Sysmon%20Linux/decoder-linux-sysmon.xml
https://raw.githubusercontent.com/socfortress/Wazuh-Rules/main/Sysmon%20Linux/200150-sysmon_for_linux_rules.xml
'
for url in $url_list
do
	curl -LfO "$url"
done

sha256sums='
ca8a78c13ade0acc6778c9ed100e4ca5e073403ba6ad208d44f69b5bdbcfe222
0ed99f58c051fe5d042bb934988ca1914e89d66fd91653ff607eb50987f1f2e8
34b07aa25e37c65d968f90b667ca38514f27a7c7fac93a277ef5a332877aa4ce
'

for hash_value in $sha256sums
do
	sha256sum ./*.xml | grep -P "\b$hash_value\b" || echo "[WARNING]checksum error."
done

# Necessary decoder and rule files
cat decoder-linux-sysmon.xml | sudo tee -a /var/ossec/etc/decoders/local_decoder.xml >/dev/null
cat 200150-sysmon_for_linux_rules.xml | sudo tee -a /var/ossec/etc/rules/local_rules.xml >/dev/null

# If you want the same sysmon config running on the Wazuh server
sudo sysmon -c ./sysmonforlinux-config.xml

systemctl restart wazuh-manager
```


**📝 Explanation**

!!! info "1. ENDPOINT: [Install `sysmonforlinx`](https://github.com/straysheep-dev/ansible-configs/tree/main/install_sysmon) + this [config.xml](https://wazuh.com/resources/blog/detecting-sysjoker-backdoor-malware-with-wazuh/sysmonforlinux-config.xml))"

	- This config.xml works well with Wazuh, as it uses exclude rules to "see" everything, allowing Wazuh full visibility, while preventing Wazuh agent operations from flooding the logs
    - You will likely need at least the ProcessCreate filters from this file if you plan to use your own config, to prevent your logs from being overrun
	- `ca8a78c13ade0acc6778c9ed100e4ca5e073403ba6ad208d44f69b5bdbcfe222  sysmonforlinux-config.xml`

!!! info "2. WAZUH NODE: Install the [decoder file](https://raw.githubusercontent.com/socfortress/Wazuh-Rules/main/Sysmon%20Linux/decoder-linux-sysmon.xml) so Wazuh can \"read\" Sysmon logs"

	- The Wazuh blog and socfortress GitHub contain the exact same decoder file, the blog's copy just has CRLF line endings.
	- `0ed99f58c051fe5d042bb934988ca1914e89d66fd91653ff607eb50987f1f2e8  decoder-linux-sysmon.xml`
    - `cat decoder-linux-sysmon.xml | sudo tee -a /var/ossec/etc/decoders/local_decoder.xml >/dev/null`

!!! info "3. WAZUH NODE: Next you need the [rule file](https://raw.githubusercontent.com/socfortress/Wazuh-Rules/main/Sysmon%20Linux/200150-sysmon_for_linux_rules.xml), so Wazuh understands Sysmon event IDs in a general sense."

	- Similar to the decoder file, the blog vs socfortress source file for Wazuh rules are virtuall the same (explore the diff in VSCode) just some lines swapped, and one bottom section added
	- `34b07aa25e37c65d968f90b667ca38514f27a7c7fac93a277ef5a332877aa4ce  200150-sysmon_for_linux_rules.xml`
    - `cat 200150-sysmon_for_linux_rules.xml | sudo tee -a /var/ossec/etc/rules/local_rules.xml >/dev/null`

This can seem confusing with three separate files for one monitoring scenario, so to reiterate:

- Each endpoint has it's own logging config, whether this is auditd rules, or Sysmon config.xml, maintain those how you have been (the Sysmon config from the blog is recommneded if you're just starting)
- Wazuh needs a decoder written to interpret any logs, it includes some by default but I've found in many cases you need a custom decoder to work with existing audit.rules and sysmon config.xml files
- The Wazuh rules file, tells Wazuh what and how to alert
	- The SOCFortress Wazuh rules file above can "see" every event your agent ships to Wazuh, and sets each event ID to `level=3` by default, so they will ALL appear in your Wazuh dashboard logs
    - This is how you start from scratch, tune from here
    - Build more specific changes over time as necessary, this can be to your sysmon config.xml, the Wazuh rules, or even just learning how to write good DQL queries for your environment

!!! tip "On Rule Writing"

    The blog post writes / appends unique detections for the sysjoker malware to the `/var/ossec/etc/rules/local_rules.xml` on the Wazuh manager's end. This is a fairly advanced and complex route to take. You do not *need* to write rules this way as mentioned above just to get started. Trying to start this way is overwhelming, remember, the goal instead is to use the SOCFortress rules / decoder files to get all of your existing audit.rules and sysmon config.xml data into visible logs on Wazuh's dashboard while keeping them maintainable.

---

### auditd

You will again need three components:

- 🌐 [Decoder file (for Wazuh server)](https://raw.githubusercontent.com/socfortress/Wazuh-Rules/main/Auditd/auditd_decoders.xml)
	- `2a37823495de90ccb54181e4b853e35bafed074a0e464666ec186c79826387ac  auditd_decoders.xml`
- 🌐 [Rules file (for Wazuh server)](https://raw.githubusercontent.com/socfortress/Wazuh-Rules/main/Auditd/200110-auditd.xml)
	- `3514afbc4fd081318aeea971065e54fc067c6e5243f835a44b6a1aba98680d8e  200110-auditd.xml`
- Auditd Configuration (for Endpoint)
	- 🌐 [github/Neo23x0/auditd](https://github.com/Neo23x0/auditd/blob/master/audit.rules)
    - 🌐 [github/straysheep-dev/auditd](https://github.com/straysheep-dev/auditd/blob/master/40-mitre.rules) (my fork)
	- 🌐 [github/socfortress/Wazuh-Rule](https://github.com/socfortress/Wazuh-Rules/blob/main/Auditd/auditd.conf) (SOCFortress fork)

All of these things will work fine out of the box.

!!! tip "Auditd Keys"

    The most useful component to reviewing your logs after applying these changes is making sure all of your rules have [keys (`man audit.rules`)](https://github.com/linux-audit/audit-userspace/blob/e5b0c9d74a54e0c6c83ba402807a53e4544b7898/docs/audit.rules.7#L151) associated with them. Even without a specific Wazuh rule for each key, the SOCFortress decoder file allows you to visualize and search logs based on the key you've asigned each rule in your own audit.rules file(s).

Use whatever means of [deploying `auditd` itself and a configuration](https://github.com/straysheep-dev/ansible-configs/tree/main/install_auditd). Regardless of which way you do this, this line should be included somewhere in your auditd configuration to filter out the Wazuh agent's behavior which could flood the logs. You could even write this to it's own `40-wazuh.rules` file:

```conf
## Ignore Wazuh endpoint agent
-a always,exclude -F gid=wazuh  # from SocFortress config: https://github.com/socfortress/Wazuh-Rules/tree/main/Auditd
```

Install everything on the Wazuh Server:

```bash
cd ~/src
url_list='
https://raw.githubusercontent.com/socfortress/Wazuh-Rules/main/Auditd/auditd_decoders.xml
https://raw.githubusercontent.com/socfortress/Wazuh-Rules/main/Auditd/200110-auditd.xml
'
for url in $url_list
do
	curl -LfO "$url"
done

sha256sums='
2a37823495de90ccb54181e4b853e35bafed074a0e464666ec186c79826387ac
3514afbc4fd081318aeea971065e54fc067c6e5243f835a44b6a1aba98680d8e
'

for hash_value in $sha256sums
do
	sha256sum ./*.xml | grep -P "\b$hash_value\b" || echo "[WARNING]checksum error."
done

# Necessary decoder and rule files
cat auditd_decoders.xml | sudo tee -a /var/ossec/etc/decoders/local_decoder.xml >/dev/null
cat 200110-auditd.xml | sudo tee -a /var/ossec/etc/rules/local_rules.xml >/dev/null

#systemctl restart wazuh-manager
```

Finally, you'll need to exclude Wazuh's built-in auditd decoders and rules by adding the following lines:

- `<decoder_exclude>ruleset/decoders/0040-auditd_decoders.xml</decoder_exclude>`
- `<rule_exclude>0365-auditd_rules.xml</rule_exclude>`

To `/var/ossec/etc/ossec.conf`:

```xml
<SNIP>
  <ruleset>
    <!-- Default ruleset -->
    <decoder_dir>ruleset/decoders</decoder_dir>
    <decoder_exclude>ruleset/decoders/0040-auditd_decoders.xml</decoder_exclude>
    <rule_dir>ruleset/rules</rule_dir>
    <rule_exclude>0215-policy_rules.xml</rule_exclude>
    <rule_exclude>0365-auditd_rules.xml</rule_exclude>
<SNIP>
```

---

### Zeek

[Documentation already exists for setting up and ingesting logs from Suricata on each endpoint](https://documentation.wazuh.com/current/proof-of-concept-guide/integrate-network-ids-suricata.html).

This section details doing the same using Zeek.

*⚠️ TO DO: This section is still under construction, check back later! ⚠️*

---

## Reviewing Logs

*Now that data is coming in to your Wazuh dashboard, how do you sort through it?*


### General

No matter what you're searching for, filtering based on a minimum severity level is incredibly powerful:

```DQL
<SNIP> AND rule.level > 3
```

![](../media/wazuh-tailscale/wazuh_tailscale_search_severity_level.png)

!!! info ""

    This is a [quick reference for modules built in to Wazuh](https://documentation.wazuh.com/current/proof-of-concept-guide/index.html) that you can keep in mind when thinking of "what" Wazuh has that you may want to search for.

These three groups are also built in components to Wazuh. They use the agent to perform premade checks and generate the logs. All three need to be configured (in other words enabled) to start generating logs, as detailed below.

- `rule.groups:rootcheck` Display rootkit-check logs
    - Add the `full_log` field in your columns
    - 🌐 [change the `<freqency>` of the check in each endpoint's /var/ossec/etc/ossec.conf](https://documentation.wazuh.com/current/proof-of-concept-guide/poc-detect-hidden-process.html#configuration)
- `rule.groups:syscheck` Display file integrity events
    - You'll need to [enable `check_all="yes" report_changes="yes" realtime="yes"` on each endpoint](https://documentation.wazuh.com/current/proof-of-concept-guide/poc-file-integrity-monitoring.html#configuration)
	- Functions similar to AIDE but in real time
- `rule.groups:suricata` Logs related to suricata NSM
    - 🌐 [suricata needs to be installed and configured on each enpoint](https://documentation.wazuh.com/current/proof-of-concept-guide/integrate-network-ids-suricata.html#configuration)

Use the DQL search bar when viewing an agent's events to search for `rule.groups:rootcheck` or `rule.groups:suricata`.

![](../media/wazuh-tailscale/wazuh_tailscale_event_search.png)

---

### Sysmon

If you're using Wazuh's own default ["collect most everything" Sysmon config](https://wazuh.com/resources/blog/detecting-sysjoker-backdoor-malware-with-wazuh/sysmonforlinux-config.xml), you could start by filtering out some noise:

```DQL
(NOT data.system.eventId:5 AND NOT data.eventdata.image:/usr/bin/dash)
```

Look for failed login, auth, or escalation attempts:

```DQL
(NOT data.system.eventId:5 AND NOT data.eventdata.image:/usr/bin/dash) AND location:/var/log/auth.log AND rule.description:*ailed*
```

Look for `sudo` usage:

- Add `data.eventdata.commandLine` to your columns

```DQL
(data.eventdata.commandLine:*sudo*)
```

Using `Event ID 3: Network connection`, add the following fields to the visible columns:

- `data.eventdata.destinationHostname`
- `data.eventdata.DestinationIp`
- `data.eventdata.destinationPort`

Using what you know about the system, slowly eliminate common binaries used for networking based on what you see in the logs, such as the DNS daemon, tailscaled, apt, NetworkManger, snapd, and so on.

```DQL
(data.system.eventId:3)
(data.system.eventId:3 AND NOT data.eventdata.image:/usr/sbin/unbound)
(data.system.eventId:3 AND NOT data.eventdata.image:/usr/sbin/unbound AND NOT data.eventdata.image:/usr/sbin/tailscaled)

<SNIP>

(data.system.eventId:3 AND NOT data.eventdata.image:/usr/sbin/unbound AND NOT data.eventdata.image:/usr/sbin/tailscaled AND NOT data.eventdata.image:*NetworkManager AND NOT data.eventdata.image:/usr/lib/apt/methods/http AND NOT data.eventdata.image:/usr/lib/snapd/snapd)
```

Look for weird destination ports from common network processes:

- Unbound should only contact certain destination addresses
- It should only reach them on ports tcp/853 or udp+tcp/53

```DQL
(data.system.eventId:3 AND data.eventdata.image:/usr/sbin/unbound AND NOT data.eventdata.destinationPort:853 AND NOT data.eventdata.destinationPort:53)
```

![](../media/wazuh-tailscale/wazuh_tailscale_sysmon_visualization.png)

See what external hosts tailscaled is talking to and how often:

```DQL
(data.system.eventId:3 AND data.eventdata.image:*tailscaled AND NOT data.eventdata.DestinationIp:0.0.0.0 AND NOT data.eventdata.DestinationIp:100.* AND NOT data.eventdata.DestinationIp:127.* AND NOT data.eventdata.DestinationIp:10.* AND NOT data.eventdata.DestinationIp:192.168.*)
```

!!! tip "Network Analysis and Anomaly Detection"

    Digging into the network activity is where you'll find things like beacons, and can potentially get this data into a pcap and pass it off to [RITA](https://github.com/activecm/rita) for a deeper analysis.

If you walked through the queries in this section, you already can see where to get ideas for additional queries. The real trick is learning when you're digging too deep, and need to pivot to something else. This can be augmented by external alerting on special events, or even simply building really good queries tailored to your environment.

---

### auditd

You'll notice if your events page has the default columns (`Time`, `rule.description`, `rule.level`, and `rule.id`) You'll need to discern auditd log events by their type, such as SYSCALL, EXECVE, and so on.

It's helpful to add the following columns to your data display:

- `data.audit.exe`
- `data.audit.key`

So you can visualize what's being executed, and what key it relates to. The `full_log` field is way too large to show here, but with this information you can decide what to focus on while scrolling. ***You won't always turn up everything with a well crafted DQL query***. This is important to remember, always try multiple methods to visualize and look for gaps.

Easily sort through your logs for keys like this:

```DQL
"data.audit.key": "T1033_System_User_Discovery"
"data.audit.key": "T1219_Remote_Access_Software"
```

Or by any string appearing in an audit log field (notice sometimes quotes work, and other times they don't):

```DQL
"full_log": "*/etc/shadow*"
"data.audit.exe":*gdb*
"data.audit.exe":*python* AND data.auditd.key:network_connect_4
```

![](../media/wazuh-tailscale/wazuh_tailscale_auditd_visualization.png)

Interestingly this trick does not work as you'd expect:

```DQL
"full_log": "*://*"
```

If you have a key logging the `connect` syscall, you could instead try:

```DQL
data.audit.key:network_connect_4 OR data.audit.key:network_connect_6
```

Interestingly, even if `auditd` is set to enrich logs on the endpoint, Wazuh still receives the encoded entries. This means you'll get entries like:

```txt
proctitle=746F756368002F7661722F6C6F672F61756469742F746D705F6469726563746F7279312F6D616C776172652E7079
```

To decode them, you'll need to use `xxd`. Credit to [Hal Pomeranz](https://archive.org/details/HalLinuxForensics) for showing the full `tr` command as well during [WWHF](https://wildwesthackinfest.com/about-us/).

```bash
└─$ echo '746F756368002F7661722F6C6F672F61756469742F746D705F6469726563746F7279312F6D616C776172652E7079' | xxd -r -p | tr \\000 ' '; echo
touch /var/log/audit/tmp_directory1/malware.py
```

---

### Data Visualization

On the left of the events are all of the fields available to search. Clicking on one reveals these are being tracked and graphed.

To see a graph of *all* logs based on keys, click `Visualize` (and delete any DQL query from the serch bar) to see all logs sorted by keys.

!!! tip "Changing the X and Y Axes"

    It may be easier to read by going to "Metrics & axes", and changing the X-axes to LEFT, then Y-axes to TOP, and clicking `> Update` so you can read the key names on the LEFT.*

![](../media/wazuh-tailscale/wazuh_tailscale_graph_view.png)

---

## External Alerting

External alerting can be incredibly useful. Defining events of the highest severity to be alerted over an email, text, or some external channel like Slack or Discord can give you the heads up necessary to respond instead of react during an incident.

!!! abstract "Wazuh Custom Integrations"

    This section looks at the [custom integration of third-party APIs](https://documentation.wazuh.com/current/user-manual/manager/manual-integration.html#custom-integration), and [their source code](https://github.com/wazuh/wazuh/tree/master/integrations), to port the [Slack integration](https://github.com/wazuh/wazuh/blob/master/integrations/slack.py) to Discord. Examples of this [already](https://github.com/eugenio-chaves/eugenio-chaves.github.io/blob/main/blog/2022/creating-a-custom-wazuh-integration/index.md#customizing-the-script) [exist](https://github.com/maikroservice/wazuh-integrations/blob/main/discord/custom-discord.py).

    **It should also be noted, prior to publishing this post, an official integration has begun, see the [pull-request here](https://github.com/wazuh/wazuh/pull/24446)! 🎉**

    These make great points of reference to build and revise your own integration. However this port does a few things differently.

    - Uses the existing [slack.py `generate_msg`](https://github.com/wazuh/wazuh/blob/a5f51ad61af5abcf49186cd72d4d73c0c3927021/integrations/slack.py#L132) function's code formatting with conditional fields
    - Adds conditional fields that will be included with certain Sysmon events
    - Includes the optional [`filter_msg` function from shuffle.py](https://github.com/wazuh/wazuh/blob/a5f51ad61af5abcf49186cd72d4d73c0c3927021/integrations/shuffle.py#L166) to filter events if their rule ID matches any in `SKIP_RULE_IDS`
    - Use conditional variables to handle arbitrary JSON structures

![](../media/wazuh-tailscale/wazuh_integration_discord_3.png)


!!! info "**🚩 Getting Started**"

    We need a python script that doesn't exist yet to perform the actions. This is where referencing the other integrations will save a huge amount of time, and keep things consistent. This part can be overlooked if you haven't read [this post](https://wazuh.com/blog/how-to-integrate-external-software-using-integrator/) before getting started.

All of the requirements in a list next to each other, will make this easier to understand.

1. We need a python script to generate and send the alert
2. For Discord, of all the existing integrations, [slack.py](https://github.com/wazuh/wazuh/blob/master/integrations/slack.py) is the closest starting point since they both use webhooks and similar data
3. These scripts can be narrow in scope in what they send, however we want a generic script that can be used for ANY type of alert
	- General Wazuh logs and alerts, like rootkit detection and other module logs
    - Syslog, auditd
    - Sysmon, sysmonforlinux
4. With this in mind, controlling when the script executes will primarily happen in the `<integration>` section of `/var/ossec/etc/ossec.conf`
    - Alert on events of a certain severity `<level>`
	- Alert on a `<group>` filter (can be multiple, comma separated)
    - Alert on a `<rule_id>` filter (can be multiple, comma separated)
    - Alert on an `<event_location>` filter (can be multiple, comma separated)
5. Integrations live under `/var/ossec/integrations` as two files:
	- `custom-discord` Every integration has this shell script, it's the same script for each integration, only the file name changes
	- `custom-discord.py` The python script to create and send the alert to a platform of our choice (Discord)
    - Any custom integration must have a name beginning with "custom-"

!!! tip "Maintainability"

	Ultimately, this will give use something expandable that we don't need to constantly rewrite, because it can ingest multiple log and alert types. Thinking back to how ingesting logs requires three components (the logging conf like sysmon.xml or auditd.rules, a wazuh decoder, and a wazuh rule file), we want to only have to maintain the minumum number of pieces everywhere.


### Python Script

From the existing [slack.py](https://github.com/wazuh/wazuh/blob/master/integrations/slack.py) script, the majority of changes we'll need to make are in the `generate_msg()` function. We have a few options of how to do this, two of the best are either in plain text using content `{'content': '<data>'}`, or with embeds `{"embeds": [<data>]}"` to generate a rich content message.

---

**🪺 Discord Embeds**

- 🌐 [Discord intro to webhooks](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)
- 🌐 [Discord webhook / embed usage](https://discord.com/safety/using-webhooks-and-embeds)
- 🌐 [Discord embed object structure](https://discord.com/developers/docs/resources/channel#embed-object)
- 🌐 [Discord embed field structure](https://discord.com/developers/docs/resources/channel#embed-object-embed-field-structure)

The structure of an embed is `{"embed": [data]}`, where in our case `data` is a JSON structured dictionary.

- 🌐 [slack.py creates this structure already in `generate_msg` using `msg = {}`](https://github.com/wazuh/wazuh/blob/a5f51ad61af5abcf49186cd72d4d73c0c3927021/integrations/slack.py#L156)
- We can reuse the list of `msg['fields']` by changing the keys (`'title':` becomes `'name':`, also add a key for `'inline':`)
- Finally update the object sent to the return value to be `{'embeds': [msg]}` instead of `{'attachments': [msg]}`

[Knowing the available fields in an embed](https://discord.com/developers/docs/resources/channel#embed-object-embed-structure), we can update any other variables to match what Discord is expecting.

The start of our msg object will look like this:

=== "Slack Version"

    ```python
    <SNIP>
        msg = {}
        msg['color'] = color
        msg['pretext'] = 'WAZUH Alert'
        msg['title'] = alert['rule']['description'] if 'description' in alert['rule'] else 'N/A'
        msg['text'] = alert.get('full_log')

        msg['fields'] = []
    <SNIP>
    ```

=== "Discord Version"

    ```python
    <SNIP>
        msg = {}
        msg['type'] = 'rich' # Discord specific, default type for embeds
        msg['color'] = color
        msg['title'] = 'WAZUH Alert'
        msg['description'] = description

        msg['fields'] = []
    <SNIP>
    ```

---

**🟣 Discord Colors**

Discord colors take the [decimal value of the color's hex code](https://discord.com/developers/docs/resources/channel#embed-object-embed-structure). You can Google a color's hex code, open a calculator app in programming mode, set it to HEX mode, and paste the hex value. Change it to DEC mode to get the decimal representation of that hex code. That is the value to use here.

Here's an example of the color severity section.

=== "Discord Version"

```python
<SNIP>
    if severity <= 4:
        color = '255'      # blue, hex value is #0000FF
    elif severity >= 5 and severity <= 7:
        color = '16776960' # yellow, hex value is #FFFF00
    else:
        color = '16711680' # red, hex value is #FF0000
<SNIP>
```

---

**⚙️ Conditional Variables**

As mentioned previously, the modularity comes from the conditional values in this script. The slack.py version has an example that's incredibly similar to Ansible's `when:` conditional statements. This is how python works. Knowing this, we can create fail-safe variables to check if:

- A full "path" of key:value pairs is available
- If not, then if another (default) "path" of key:value pairs is available
- Else, 'N/A' (in some cases this is omitted since there will always be a value)

Building from the Slack version's example, we can create variables for basically every field we would want to add to an alert.

=== "Slack Version"

    ```python
    <SNIP>
    msg['title'] = alert['rule']['description'] if 'description' in alert['rule'] else 'N/A'
    <SNIP>
    ```

=== "Discord Version"

    ```python
    <SNIP>
    timestamp = alert['timestamp'] if 'timestamp' in alert else 'N/A'
    severity = alert['rule']['level']
    description = alert['rule']['description'] if 'description' in alert['rule'] else 'N/A'
    agent_id = alert['agent']['id'] if 'id' in alert['agent'] else 'N/A'
    agent_name = alert['agent']['name'] if 'name' in alert['agent'] else 'N/A'
    rule_id = alert['rule']['id'] if 'id' in alert['rule'] else 'N/A'
    location = alert['location']
    dest_ip = alert['data']['eventdata']['DestinationIp'] if 'data' in alert and 'eventdata' in alert['data'] and 'DestinationIp' in alert['data']['eventdata'] else 'N/A'
    dest_port = alert['data']['eventdata']['destinationPort'] if 'data' in alert and 'eventdata' in alert['data'] and 'destinationPort' in alert['data']['eventdata'] else 'N/A'
    dest_host = alert['data']['eventdata']['destinationHostname'] if 'data' in alert and 'eventdata' in alert['data'] and 'destinationHostname' in alert['data']['eventdata'] else 'N/A'
    src_ip = alert['data']['eventdata']['sourceIp'] if 'data' in alert and 'eventdata' in alert['data'] and 'sourceIp' in alert['data']['eventdata'] else 'N/A'
    src_port = alert['data']['eventdata']['sourcePort'] if 'data' in alert and 'eventdata' in alert['data'] and 'sourcePort' in alert['data']['eventdata'] else 'N/A'
    full_log = alert['full_log'] if 'full_log' in alert else 'N/A'
    win_message = alert['data']['win']['system']['message'] if 'data' in alert and 'win' in alert['data'] and 'system' in alert['data']['win'] and 'message' in alert['data']['win']['system'] else 'N/A'
    log_id = alert['id']
    <SNIP>
    ```

---

**🛠️ Conditional Fields**

Finally, we can copy [what slack.py already does](https://github.com/wazuh/wazuh/blob/a5f51ad61af5abcf49186cd72d4d73c0c3927021/integrations/slack.py#L163) to build additional conditional fields.

=== "Slack Version"

    ```python
    <SNIP>
    if 'agentless' in alert:
            msg['fields'].append(
                {
                    'title': 'Agentless Host',
                    'value': alert['agentless']['host'],
                }
            )
    <SNIP>
    ```

=== "Discord Version"

    ```python
    <SNIP>
    msg['fields'] = []
    # The 'if' statements wrapping certain fields determine if a field exists in the log data, and if not, that
    # field will be absent entirely in the embed rather than an empty field.
    # Discord embeds can include up to 25 fields. 'inline' means it will attempt to put neighboring fields into one line,
    # up to three per row.
    msg['fields'].append(
        {
            'name': 'Timestamp',
            'value': '{0}'.format(timestamp),
            'inline': False
        }
    )
    if 'agent' in alert:
        msg['fields'].append(
            {
                'name': 'Agent',
                'value': '({0}) - {1}'.format(agent_id, agent_name),
                'inline': True
            }
        )
    if 'agentless' in alert:
        msg['fields'].append(
            {
                'name': 'Agentless Host',
                'value': '{0}'.format(agentless),
                'inline': True
            }
        )
    msg['fields'].append(
        {
            'name': 'Location',
            'value': '{0}'.format(location),
            'inline': True
        }
    )
    msg['fields'].append(
        {
            'name': 'Rule ID',
            'value': '{0} _(Level {1})_'.format(rule_id, severity),
            'inline': True
        }
    )
    # The remaining fields have been formatted with a code block using one ` or three ``` backticks to prevent malicious strings from potentially
    # making network requests or being clickable
    if 'data' in alert and 'eventdata' in alert['data'] and 'DestinationIp' in alert['data']['eventdata']:
        msg['fields'].append(
            {
                'name': 'Dest IP',
                'value': '`{0}`'.format(dest_ip),
                'inline': True
            }
        )
    if 'data' in alert and 'eventdata' in alert['data'] and 'destinationPort' in alert['data']['eventdata']:
        msg['fields'].append(
            {
                'name': 'Dest Port',
                'value': '`{0}`'.format(dest_port),
                'inline': True
            }
        )
    if 'data' in alert and 'eventdata' in alert['data'] and 'destinationHostname' in alert['data']['eventdata']:
        msg['fields'].append(
            {
                'name': 'Dest Host',
                'value': '`{0}`'.format(dest_host),
                'inline': True
            }
        )
    if 'data' in alert and 'eventdata' in alert['data'] and 'sourceIp' in alert['data']['eventdata']:
        msg['fields'].append(
            {
                'name': 'Source IP',
                'value': '`{0}`'.format(src_ip),
                'inline': True
            }
        )
    if 'data' in alert and 'eventdata' in alert['data'] and 'sourcePort' in alert['data']['eventdata']:
        msg['fields'].append(
            {
                'name': 'Source Port',
                'value': '`{0}`'.format(src_port),
                'inline': True
            }
        )
    if 'full_log' in alert:
        msg['fields'].append(
            {
                'name': 'Full Log',
                'value': '```{0}```'.format(full_log),
                'inline': False
            }
        )
    if 'data' in alert and 'win' in alert['data'] and 'system' in alert['data']['win'] and 'message' in alert['data']['win']['system']:
        msg['fields'].append(
            {
                'name': 'Message',
                'value': '```{0}```'.format(win_message),
                'inline': False
            }
        )
    msg['fields'].append(
        {
            'name': 'Wazuh ID',
            'value': '{0}'.format(log_id),
            'inline': False
        }
    )
    <SNIP>
    ```

!!! note "Discord Char Length Limitations"

    If you look at the [integrations/discord.py](https://github.com/wazuh/wazuh/pull/24446/files#diff-c1e0329772717b48b65b02ff1a95462fc0c267fbf6b4919b342d288f35a5fa84) pull request, you'll notice `full_log` is under the `['description']` field of the Discord embed data. This is due to Discord's description field having a higher character limit (4096) than the `fields.value` field (1024). This will cause logs longer than 1024 chars to be dropped and never shipped to Discord!

    In both cases you could ensure a limit on the field content with something like this:

    ```python3
    'value': '```{0}```'.format(full_log[:4096]),
    ```

    Here the char limit for `description` is set so alerts are trimmed and not dropped.

---

**🧪 Filtering Rule ID's**

This is optional and can exist as a list baked in to the script.

```python
<SNIP>
# Rule filter
# This is just one additional place to fine tune what alerts are posted
SKIP_RULE_IDS = [
    '00000',
]
<SNIP>
```

---

**📜 Completed Script**

The full script is available [here](https://github.com/straysheep-dev/alert-service/tree/main/wazuh-integrations).


??? tip "**Accessing Nested Keys with `get(key, default=None)`**"

    Asking Google or ChatGPT what's the best way to access a nested dictionary in python, you'll find some examples of the [`.get()` method](https://docs.python.org/3/library/stdtypes.html#dict.get). This essentially does the same thing in a less verbose way.

    - Traverses the dictionaries, returning the value for `key`, if it exists
    - Avoids the [`KeyError`](https://docs.python.org/3/library/exceptions.html#KeyError) without requiring a check, when a subsequent key does not exist within the data

    ```python
    severity = alert.get('rule', {}).get('level') if alert.get('rule', {}).get('level') is not None else alert.get('_source', {}).get('rule', {}).get('level', 'N/A')
    ```

    This was included for future reference, but the "if" "else" style was used to match the style and readability of the original slack.py script.

    References:

    - 🌐 [Accessing nested keys](https://stackoverflow.com/questions/10399614/accessing-value-inside-nested-dictionaries)
    - 🌐 [Traversing varied dictionary structures with `.get()`](https://stackoverflow.com/questions/28225552/is-there-a-recursive-version-of-the-dict-get-built-in)
    - 🌐 [Using `dict.get('key')`](https://stackoverflow.com/questions/11041405/why-dict-getkey-instead-of-dictkey)
    - 🌐 [Python Docs: `.get()` method](https://docs.python.org/3/library/stdtypes.html#dict.get)
    - 🌐 [Python `None` test](https://stackoverflow.com/questions/3965104/not-none-test-in-python)

---

**📝 Content**

!!! info "Ai Usage"

    ChatGPT helped debug the `str()` and trailing `+` to correctly format this block to appear how it looks below.

If you prefer not to use the `embed` object, this is an example using the same variables, but replacing the main block in `generate_msg` to send the [plaintext `content` parameter of up to 2000 characters](https://discord.com/developers/docs/resources/webhook#execute-webhook-jsonform-params):

```python
def generate_msg(alert: any, options: any) -> str:

<SNIP>

	# To ensure formatting, each line must have a trailing `+` until the final line
    # Each variable must also be concatenated as a str() individually, instead of `'content': str(...)`
    msg = {
        'content': (
            "=====[ Wazuh Alert ]=====" + "\n" +
            "- Timestamp: " + str(timestamp) + "\n" +
            "- Log ID: " + str(log_id) + "\n" +
            "- Agent ID: " + str(agent_id) + "\n" +
            "- Agent Name: " + str(agent_name) + "\n" +
            "- Severity: " + str(severity) + "\n" +
            "- Rule ID: " + str(rule_id) + "\n" +
            "- Description: " + str(description) + "\n" +
            "- Dest IP: " + str(dest_ip) + "\n" +
            "- Dest Port: " + str(dest_port) + "\n" +
            "- Dest Host: " + str(dest_host) + "\n" +
            "- Full Log: " + str(full_log) + "\n"
        )
    }

    if options:
        msg.update(options)

    return json.dumps(msg)
```

---

**🪲 Debugging**

You can run these scripts manually for debugging with the following line, where `test.json` is a single log entry (it does not have to be a single line, just one log entry though) for testing purposes.

- Before running it, change `LOG_FILE = f'{pwd}/logs/integrations.log'` to `LOG_FILE = f'integrations.log'`

```bash
python3 ./custom-discord.py 'test.json' '' '<webhook>' 'debug'
```

---

### Integration Block

You'll need to append one or more `<integration>` sections to `/var/ossec/etc/ossec.conf`, that includes the integration name, webhook, and any filters or options.

This section can have multiple entries utilizing the same integration. The sample below is ready to use after modifying or removing the necessary values for your use case. It's written so you can append it directly to `ossec.conf`.

```xml
<ossec_config>
  <integration>
      <name>custom-discord</name>
      <hook_url>https://discord.com/api/webhooks/XXXXXXXXXXXXXXX/XXXXXXXXXXXXXXXXXXXXXXX</hook_url>
      <group>GROUP</group> <!-- Replace with an optional comma separated list of groups or remove it -->
      <rule_id>RULE_ID</rule_id> <!-- Replace with an optional comma separated list of rule ids or remove it -->
      <level>SEVERITY_LEVEL</level> <!-- Replace with an optional minimum severity level or remove it -->
      <event_location>EVENT_LOCATION</event_location> <!-- Replace with an optional comma separated list of event locations or remove it -->
      <alert_format>json</alert_format>
      <options>JSON</options> <!-- Replace with your custom JSON object or remove it -->
  </integration>
  <integration>
      <name>custom-discord</name>
      <hook_url>https://discord.com/api/webhooks/XXXXXXXXXXXXXXX/XXXXXXXXXXXXXXXXXXXXXXX</hook_url>
      <group>GROUP</group> <!-- Replace with an optional comma separated list of groups or remove it -->
      <alert_format>json</alert_format>
  </integration>
</ossec_config>
```

---

## Maintenance

!!! info ""

    With limited time and budget, you'll want to walk away knowing exactly what you'll want to maintain, meaning where to focus your efforts as your Wazuh instance lives and grows.

Huge thanks to Wazuh and SocFortress for the blog posts and configuration templates that do most of this heavy lifting. Because of this, 1) this post exists, and 2) you can focus on maintaining specific components without worrying about making sure changes to your logging conf are reflected in three other places for Wazuh to successfully ingest the information.

Components to maintain and when:

!!! info "**Logging Configs**"

    - sysmon config.xml: You can start with the recommended config.xml file and do well. Over time you may want to start tailoring your own.
    - auditd.rules: Make sure every rule has a `key` value, the SocFortress decoder and rules will let you search in Wazuh based on the full_log as well as the auditd key


!!! info "**wazuh-manager ossec.conf**"

    - `<integrations>` section: Define what rules, groups, log locations, and / or log severity ships an alert or data to an external source
    - Maintain one block you deploy generally on all endpoints
    - Maintain blocks limited to specific agents
    - This can help you cut through the noise, and get a heads up on severe events

!!! info "**wazuh-agent ossec.conf**"

    - Each endpoint may have varying configurations
    - File integrity monitoring, mainly adding `check_all="yes" report_changes="yes" realtime="yes"` to the two default directories if you want realtime monitoring
    - If you add a network IDS on the endpoint, tell the agent about it here as well
    - You can deploy this with a shell script or your own Ansible role as you learn what needs done on each endpoint in your environment


---

## Troubleshooting

!!! bug "Wazuh dashboard server is not ready yet"

    You may encounter this issue when trying to reach the web interface. Check `wazuh-indexer.service` first:

    ```bash
    $ systemctl status wazuh-indexer
    × wazuh-indexer.service - wazuh-indexer
        Loaded: loaded (/lib/systemd/system/wazuh-indexer.service; enabled; vendor preset: enabled)
        Active: failed (Result: timeout) since Sat 2025-03-02 03:02:01 UTC; 1 days ago
        Docs: https://documentation.wazuh.com
    Main PID: 123 (code=exited, status=143)
            CPU: 12.345s

    Mar 02 03:02:01 wazuh-server systemd-entrypoint[123]:         at org.opensearch.bootstrap.OpenSearch.execute(OpenSearch.java:172)
    Mar 02 03:02:01 wazuh-server systemd-entrypoint[123]:         at org.opensearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:104)
    Mar 02 03:02:01 wazuh-server systemd-entrypoint[123]:         at org.opensearch.cli.Command.mainWithoutErrorHandling(Command.java:138)
    Mar 02 03:02:01 wazuh-server systemd-entrypoint[123]:         at org.opensearch.cli.Command.main(Command.java:101)
    Mar 02 03:02:01 wazuh-server systemd-entrypoint[123]:         at org.opensearch.bootstrap.OpenSearch.main(OpenSearch.java:138)
    Mar 02 03:02:01 wazuh-server systemd-entrypoint[123]:         at org.opensearch.bootstrap.OpenSearch.main(OpenSearch.java:104)
    Mar 02 03:02:01 wazuh-server systemd[1]: wazuh-indexer.service: start operation timed out. Terminating.
    Mar 02 03:02:01 wazuh-server systemd[1]: wazuh-indexer.service: Failed with result 'timeout'.
    Mar 02 03:02:01 wazuh-server systemd[1]: Failed to start wazuh-indexer.
    Mar 02 03:02:01 wazuh-server systemd[1]: wazuh-indexer.service: Consumed 12.345s CPU time.
    ```

    If you get the error above, try restarting the indexer on its own. Often restarting the manager, or the entire server will not resolve this, likely due to the timeout issue where it's not initializing fast enough.

    Next you could try restarting the `wazuh-dashboard.service`. Be patient, if you see the error below, wait a minute or two and try to refresh the page. This error will happen when the dashboard is starting up, but hasn't connected to the indexer just yet.

    ```bash
    $ systemctl status wazuh-dashboard
    ● wazuh-dashboard.service - wazuh-dashboard
        Loaded: loaded (/etc/systemd/system/wazuh-dashboard.service; enabled; vendor preset: enabled)
        Active: active (running) since Tue 2025-03-02 01:00:00 UTC; 1min 23s ago
    Main PID: 12345 (node)
        Tasks: 11 (limit: 9348)
        Memory: 183.2M
            CPU: 12.345s
        CGroup: /system.slice/wazuh-dashboard.service
                └─12345 /usr/share/wazuh-dashboard/node/bin/node --no-warnings --max-http-header-size=65536 --unhandled-rejections=warn /usr/share/wazuh-dashboard/src/cli/dist

    Mar 02 03:02:01 wazuh-server opensearch-dashboards[12345]: {"type":"log","@timestamp":"2025-03-02T01:00:00Z","tags":["error","opensearch","data"],"pid":12345,"message":"[search_phase_execution_exception]: all shards failed"}
    Mar 02 03:02:01 wazuh-server opensearch-dashboards[12345]: {"type":"log","@timestamp":"2025-03-02T01:00:00Z","tags":["error","opensearch","data"],"pid":12345,"message":"[search_phase_execution_exception]: all shards failed"}
    ```

    After a couple minutes the dashboard webpage should finally load.


---

## Migrate to Proxmox

*This section mirrors what's mentioned under [blog/Proxmox](../posts/proxmox.md#migrating-vms). Be sure to check that post if you need to get started with proxmox first and for all related details.*

Basically, it's fairly straight forward and easy to migrate even a large standalone Hyper-V VM to Proxmox. Keep this in mind if you plan to deploy it in "production" or require more space and a distributed Wazuh cluster.