---
draft: false
date:
  created: 2024-05-08
  updated: 2024-07-23
categories:
  - windows
  - how-to
  - administration
  - guide
  - home-lab
  - powershell
  - active-directory
---

# Windows

Various configuration settings and notes for Microsoft Windows operating systems.

*Updated on 2024/07/23.*

<!-- more -->

*This file is originally from [straysheep-dev/windows-configs](https://github.com/straysheep-dev/windows-configs).*

## Licenses

Unless a different license is included with a file as `<filename>.copyright-notice` all files are released under the MIT license.

Examples in this README taken and adapted from the Microsoft documents:

- Microsoft Docs Examples:
	* [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/)
    * [MIT](https://github.com/MicrosoftDocs/defender-docs/blob/public/LICENSE)
- Win32-OpenSSH Wiki Examples:
	* <https://github.com/PowerShell/Win32-OpenSSH/wiki>
- Stack Overflow Licensing:
	* <https://stackoverflow.com/legal/terms-of-service#licensing>
	* [CC-BY-SA-4.0](https://creativecommons.org/licenses/by-sa/4.0/)


## Windows Baselining

Creating a security baseline for Windows.

### Backup the Registry

Before making changes, it's useful to create a backup of the registry in a default or working state.

- [Enable-ComputerRestore](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/enable-computerrestore?view=powershell-5.1)
- [CheckPoint-Computer](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/checkpoint-computer?view=powershell-5.1)
- [Restore-Computer](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/restore-computer?view=powershell-5.1)

To create a system restore point:
```powershell
Enable-ComputerRestore -Drive "C:\"
Checkpoint-Computer -Description "First restore point"
```

There are multiple restore point types:

- `APPLICATION_INSTALL`
- `APPLICATION_UNINSTALL`
- `DEVICE_DRIVER_INSTALL`
- `MODIFY_SETTINGS`
- `CANCELLED_OPERATION`

For registry changes specifically, `APPLICATION_INSTALL` is the type to use. It's also the default type, so it's not necessary to specify it on the commandline.

List all system restore points:
```powershell
Get-ComputerRestorePoint
```

After making changes, revert to a specific restore point:
```powershell
Restore-Computer -RestorePoint 1
```

The restore process can take several minutes, even when reverting a single change to the registry. Generally it takes about 3-4 minutes for local test VM's.

### LGPO.exe

See the tools available in the [Microsoft Security Compliance Toolkit](https://www.microsoft.com/en-us/download/details.aspx?id=55319)

Choose what tools and policies to download that you'd like to apply to your environment.

The idea with the `.PolicyRules` files is they are configurations that are pre-made by Microsoft and ready to be installed using `LGPO.exe`

You can do all of this manually with PowerShell, and you will ultimately want to familiarize yourself with the descriptions of each setting should you run into any issues, but this will save a ton of time in getting things up and running.

Use `PolicyAnalyzer.exe` to view the `*.PolicyRules` files, compare them to other `*.PolicyRules` files or even your current system settings.

Use `LGPO.exe` to apply the configurations found in the `*.PolicyRules` files to your system.

For example the you might apply the [Security Baselines](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-security-configuration-framework/windows-security-baselines) for both Windows 11 Pro and the latest available version of Microsoft Edge.

Deploy and test these configurations in a temporary or virtual environment first, either a VM (local or cloud) or enabled the [Windows Sandbox](https://techcommunity.microsoft.com/t5/windows-kernel-internals-blog/windows-sandbox/ba-p/301849) feature.

Windows Sandbox is a temporary, and (depending on your `.wsb` configuration) fully isolated environment that can be started very quickly from either launching the application as you would any other, or by running a `.wsb` [configuration file](https://github.com/MicrosoftDocs/windows-itpro-docs/blob/public/windows/security/threat-protection/windows-sandbox/windows-sandbox-configure-using-wsb-file.md).


### PowerSTIG

*This is covered in [Getting Started with PowerSTIG](../posts/powerstig.md).*


## Active Directory

### AD: Installing the AD RSAT Tools

You no longer need to download the AD RSAT packages from Microsoft's website, instead these are included as Windows Features on Demand. The AD RSAT tools come available by default in most Windows Server installations. However you can add the capability manually to a workstation by following the references below.

- [Installing RSAT Tools](https://learn.microsoft.com/en-us/windows-server/remote/remote-server-administration-tools#install-uninstall-and-turn-offon-rsat-tools)
- [Using DISM](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/features-on-demand-v2--capabilities?view=windows-11#using-dism-add-capability-to-add-or-remove-fods)
- [DISM CLI Options](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/dism-capabilities-package-servicing-command-line-options?view=windows-11)
- [List of RSAT Modules](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/features-on-demand-non-language-fod?view=windows-11#remote-server-administration-tools-rsat)

You do not need to enable and install every feature to start working with AD. Some essentials, considering this is likely being added to a workstation or server (separate from the DC):

- Active Directory Domain Services and Lightweight Directory Services Tools
- Active Directory Certificate Services Tools
- Group Policy Management Tools

To install via the GUI, search for "RSAT" then select the features you want to install:
- Settings > Apps > Optional Features > View Features > Search "RSAT"

To install via `Get-WindowsFeature` (newer):

```powershell
Get-WindowsFeature -Name RSAT*
Install-WindowsFeature -Name RSAT -IncludeManagementTools
Install-WindowsFeature -Name "RSAT","AD-Domain-Services" -IncludeManagementTools
```

To install via `DISM` (older):

```cmd
DISM /online /get-capabilities
DISM /online /add-capability /capabilityname:<capability-name>
DISM /online /add-capability /capabilityname:Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```


### AD: RSAT Tool Usage

The cmdlet syntax is similar to other PowerShell cmdlets. PowerView has great examples of how you can enumerate AD objects, and uses mostly the same syntax as the AD RSAT modules. PowerView is the tool to enumerate AD if you do not have the AD RSAT modules installed on an end point and lack permissions to install them. Note that it will need allowed by AV / EDR.

- [AD RSAT Modules](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
- [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)


### AD: Group Policy

Update Group Policy to sync with the Domain Controller via `cmd.exe`:

```cmd
gpupdate.exe
```

Push updates to Group Policy to sync with the Domain Controller via PowerShell:

- [Invoke-GPUpdate](https://learn.microsoft.com/en-us/powershell/module/grouppolicy/invoke-gpupdate?view=windowsserver2022-ps)
- [Force a Remote Group Policy Refresh (GPUpdate)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj134201(v=ws.11))

```powershell
Invoke-GPUpdate -Computer "DOMAIN\COMPUTER-01" -Target "User" -RandomDelayInMinutes 0 -Force
```

*NOTE: some GPO's do not appear configured in the gpedit.msc snap in on endpoints, but will appear if you check their registry.*

One example of this is Disabling Link-Local Multicast Name Resolution (LLMNR) protocol.

Setting Computer Configuration > Administrative Templates > Network > DNS Client > Turn off multicast name resolution to Enabled on the Domain Controller and pushing the update with `Invoke-GPUpdate` or similar will not display this policy as Enabled on the endpoints, however it will show as set in their registry. This can be tested by changing the policy on the DC and refreshing each endpoint before checking again with `Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient"`.


## Managing Windows Security


### Configuring Device Lock

- [Lock Windows When Screen Turns Off](https://superuser.com/questions/1737726/windows-10-ask-password-when-return-from-sleep)
- [PromptPasswordOnResume](https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-csp-admx-power#pw_promptpasswordonresume)

Require authentication after resuming from sleep (this should be on by default):

- Local Computer Policy > User Configuration > Administrative Templates > System > Power Management > "Prompt for password on resume from hibernate/suspend"
- You only need to run this as admin to apply this setting system-wide

```powershell
If (!(Test-Path "HKCU:\SOFTWARE\Policies\Microsoft\Windows\System\Power")) {
	New-Item -Path "HKCU:\SOFTWARE\Policies\Microsoft\Windows\System\Power" -Force | Out-Null
}
Set-ItemProperty -Path "HKCU:\SOFTWARE\Policies\Microsoft\Windows\System\Power" -Name "PromptPasswordOnResume" -Type DWord -Value "1"
```

Require authentication (immediately) after screen turns off:

- This has no GPO equivalent
- This also appears to have no GUI configuration option either
- This setting must be configured separately for each user, including the admin account

```powershell
Set-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "DelayLockInterval" -Type DWord -Value "0"
```


### Windows Defender Cmdlets

<https://docs.microsoft.com/en-us/powershell/module/defender/?view=windowsserver2022-ps>

| Cmdlet                  | Description
| ----------------------- | -------------------------------------------------------------------- |
| `Add-MpPreference`      | Modifies settings for Windows Defender.                              |
| `Get-MpComputerStatus`  | Gets the status of antimalware software on the computer.             |
| `Get-MpPreference`      | Gets preferences for the Windows Defender scans and updates.         |
| `Get-MpThreat`          | Gets the history of threats detected on the computer.                |
| `Get-MpThreatCatalog`   | Gets known threats from the definitions catalog.                     |
| `Get-MpThreatDetection` | Gets active and past malware threats that Windows Defender detected. |
| `Remove-MpPreference`   | Removes exclusions or default actions.                               |
| `Remove-MpThreat`       | Removes active threats from a computer.                              |
| `Set-MpPreference`      | Configures preferences for Windows Defender scans and updates.       |
| `Start-MpScan`          | Starts a scan on a computer.                                         |
| `Start-MpWDOScan`       | Starts a Windows Defender offline scan.                              |
| `Update-MpSignature`    | Updates the antimalware definitions on a computer.                   |


### Controlled Folder Access

> Protect your data from malicious apps such as ransomware

Write access must be granted to applications before modifications can be made to files within folders you define as protected. This is often the home directories under `C:\Users`, but can also be filesystems on external drives.

### Enable Controlled Folders

<https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/enable-controlled-folders?view=o365-worldwide>

```powershell
Set-MpPreference -EnableControlledFolderAccess Enabled
Set-MpPreference -EnableControlledFolderAccess AuditMode # Enable the audit feature only
Set-MpPreference -EnableControlledFolderAccess Disabled  # Turn off Controlled Folder Access
```

### Customize Controlled Folder Access

<https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/customize-controlled-folders?view=o365-worldwide>

> **IMPORTANT**: Use `Add-MpPreference` to append or add apps to the list and not `Set-MpPreference`. Using the `Set-MpPreference` cmdlet will overwrite the existing list.

Add/remove protection for a folder:
```powershell
Add-MpPreference -ControlledFolderAccessProtectedFolders "c:\path\to\folder"
Remove-MpPreference -ControlledFolderAccessProtectedFolders "c:\path\to\folder"
```

Add/remove an application's access to protected folders:
```powershell
Add-MpPreference -ControlledFolderAccessAllowedApplications "c:\apps\test.exe"
Remove-MpPreference -ControlledFolderAccessAllowedApplications "c:\apps\test.exe"
```

### ASR (Attack Surface Reduction)

- [ASR Overview](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-rules-deployment?view=o365-worldwide)
- [Rule List Reference](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-rules-reference?view=o365-worldwide)
- [Rule GUIDs](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-rules-reference?view=o365-worldwide#asr-rule-to-guid-matrix)
- [Manage ASR Rules with PowerShell](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/enable-attack-surface-reduction?view=o365-worldwide#powershell)

> | Rule Name | Rule GUID |
> |:-----|:-----|
> | Block abuse of exploited vulnerable signed drivers | 56a863a9-875e-4185-98a7-b882c64b5ce5 |
> | Block Adobe Reader from creating child processes | 7674ba52-37eb-4a4f-a9a1-f0f9a1619a2c |
> | Block all Office applications from creating child processes | d4f940ab-401b-4efc-aadc-ad5f3c50688a |
> | Block credential stealing from the Windows local security authority subsystem (lsass.exe) | 9e6c4e1f-7d60-472f-ba1a-a39ef669e4b2 |
> | Block executable content from email client and webmail | be9ba2d9-53ea-4cdc-84e5-9b1eeee46550 |
> | Block executable files from running unless they meet a prevalence, age, or trusted list criterion | 01443614-cd74-433a-b99e-2ecdc07bfc25 |
> | Block execution of potentially obfuscated scripts | 5beb7efe-fd9a-4556-801d-275e5ffc04cc |
> | Block JavaScript or VBScript from launching downloaded executable content | d3e037e1-3eb8-44c8-a917-57927947596d |
> | Block Office applications from creating executable content | 3b576869-a4ec-4529-8536-b80a7769e899 |
> | Block Office applications from injecting code into other processes | 75668c1f-73b5-4cf0-bb93-3ecf5cb7cc84 |
> | Block Office communication application from creating child processes | 26190899-1602-49e8-8b27-eb1d0a1ce869 |
> | Block persistence through WMI event subscription (File and folder exclusions not supported). | e6db77e5-3df2-4cf1-b95a-636979351e5b |
> | Block process creations originating from PSExec and WMI commands | d1e49aac-8f56-4280-b9ba-993a6d77406c |
> | Block untrusted and unsigned processes that run from USB | b2b3f03d-6a65-4f7b-a9c7-1c7ef74a9ba4 |
> | Block Win32 API calls from Office macros | 92e97fa1-2edf-4476-bdd6-9dd0b4dddc7b |
> | Use advanced protection against ransomware | c1db55ab-c21a-4637-bb3f-a12568109d35 |

[ASR rules can be set in multiple modes](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-rules-reference?view=o365-worldwide#asr-rule-modes):

- `0` = Not configured / Disabled
- `1` = Block
- `2` = Audit
- `6` = Warn

Set a rule by it's GUID with PowerShell:

```powershell
Add-MpPreference -AttackSurfaceReductionRules_Ids <guid> -AttackSurfaceReductionRules_Actions <mode>
```

To enable all rules at once:

```powershell
Set-MpPreference -AttackSurfaceReductionRules_Ids 56a863a9-875e-4185-98a7-b882c64b5ce5 -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids 7674ba52-37eb-4a4f-a9a1-f0f9a1619a2c -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids d4f940ab-401b-4efc-aadc-ad5f3c50688a -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids 9e6c4e1f-7d60-472f-ba1a-a39ef669e4b2 -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids be9ba2d9-53ea-4cdc-84e5-9b1eeee46550 -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids 01443614-cd74-433a-b99e-2ecdc07bfc25 -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids 5beb7efe-fd9a-4556-801d-275e5ffc04cc -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids d3e037e1-3eb8-44c8-a917-57927947596d -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids 3b576869-a4ec-4529-8536-b80a7769e899 -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids 75668c1f-73b5-4cf0-bb93-3ecf5cb7cc84 -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids 26190899-1602-49e8-8b27-eb1d0a1ce869 -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids e6db77e5-3df2-4cf1-b95a-636979351e5b -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids d1e49aac-8f56-4280-b9ba-993a6d77406c -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids b2b3f03d-6a65-4f7b-a9c7-1c7ef74a9ba4 -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids 92e97fa1-2edf-4476-bdd6-9dd0b4dddc7b -AttackSurfaceReductionRules_Actions 1
Add-MpPreference -AttackSurfaceReductionRules_Ids c1db55ab-c21a-4637-bb3f-a12568109d35 -AttackSurfaceReductionRules_Actions 1
```

[Confilcting Policy](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/enable-attack-surface-reduction?view=o365-worldwide#policy-conflict)

> If a conflicting policy is applied via MDM and GP, the setting applied from MDM will take precedence.

[Exclude files and folders from ASR rules](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/enable-attack-surface-reduction?view=o365-worldwide#exclude-files-and-folders-from-asr-rules)

> You can specify individual files or folders (using folder paths or fully qualified resource names), but you can't specify which rules the exclusions apply to.

[Excluding files and folders from ASR rules with PowerShell](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-rules-deployment-implement?view=o365-worldwide#use-powershell-to-exclude-files-and-folders)

> ```powershell
> Add-MpPreference -AttackSurfaceReductionOnlyExclusions "<fully qualified path or resource>"
> ```

## Windows Sandbox

- [This post from SANS details using Windows Sandbox for malware analysis](https://isc.sans.edu/diary/Malware+Analysis+with+elasticagent+and+Microsoft+Sandbox/27248)
- [This documenation from Microsoft walks through every option for creating a Windows Sandbox Configuration (.wsb) file.](https://learn.microsoft.com/en-us/windows/security/threat-protection/windows-sandbox/windows-sandbox-configure-using-wsb-file)

[Example 1](https://learn.microsoft.com/en-us/windows/security/threat-protection/windows-sandbox/windows-sandbox-configure-using-wsb-file#example-1) provides a great base configuration for a malware analysis setup, where GPU and Networking are disabled, and a single folder from the host is available as read-only.

[Example 2](https://learn.microsoft.com/en-us/windows/security/threat-protection/windows-sandbox/windows-sandbox-configure-using-wsb-file#example-2) demonstrates mapping different host folders to the sandbox as read-only or writable, and also reading from a cmd script on the host to execute on startup, which downloads and installs VSCode automatically.

The document also notes exposing the following features to the sandbox potentially affects the attack surface:

- vGPU (Disabled by default)
- Network (Enabled by default)
- Mapped folders and files that are writable
- Audio input (Enabled by default)
- Video input (Disabled by default)

There's also an option called `Protected Client` mode:

> "Applies more security settings to the sandbox Remote Desktop client, decreasing its attack surface."

This repo contains a `.wsb` configuration file geared towards malware analysis that follows Example 1, and disables / enables a few additional features. Without networking in the sandbox, you should plan to have installers for all of the required tools within a mapped folder (C:\Tools -> C:\Users\WDAGUtilityAccount\Documents\Tools). What you can do then is save a `.cmd` script within the mapped Tools directory to launch with as many install commands as you're able to automate:

So if the script is named `install.cmd`, the `.wsb` file contains these lines:

```wsb
  <LogonCommand>
    <Command>C:\Users\WDAGUtilityAccount\Documents\Tools\install.cmd</Command>
  </LogonCommand>
```

And `install.cmd` itself could look something like this:

```cmd
C:\Users\WDAGUtilityAccount\Documents\Tools\tool1.exe /install
C:\Users\WDAGUtilityAccount\Documents\Tools\tool2.exe /arg 1 /arg 2 /install
C:\Users\WDAGUtilityAccount\Documents\Tools\tool3.exe /install
```

If you wish to automate any tasks using PowerShell, you can do so by having a `.cmd` script set the execution policy for PowerShell, then execute any number of `.ps1` scripts (which could also contain references to other PowerShell scripts). So the `.cmd` script could look like the following:

```cmd
cmd.exe /C powershell.exe -c Set-ExecutionPolicy Bypass -Force
cmd.exe /C powershell.exe C:\Users\WDAGUtilityAccount\Documents\Tools\SandboxSetup.ps1
```

`SandboxSetup.ps1` as an example could contain the following to import modules and execute those functions with arguments:

```powershell
Import-Module C:\Users\WDAGUtilityAccount\Documents\Tools\Set-EdgePolicy.ps1
Set-EdgePolicy Apply
```


### Windows Sandbox + VPN

Windows Sandbox is limited in networking options when compared to Hyper-V VMs. However, it's lightweight and highly configurable, making it easy to network using something like Wireguard.

Two interesting use cases as examples, the first is detailed in the link:

- [Connect Windows Sandbox to Cloud Infrastucture with Terraform, Anisble, and Wireguard](https://github.com/straysheep-dev/ansible-configs/tree/main/build_wireguard_server#use-case-windows-sandbox--wireguard)
- Windows Sandbox as a disposable OSINT container with Wireguard

The second uses the same idea, but with a public VPN provider that can generate *disposable* Wireguard configurations. The keyword being **disposable** in that the configuration does not reveal your username, password, or account data tied to the VPN provider, and the configuration can be revoked manually at any time by you. Assuming during an OSINT investigation Windows Sandbox could be compomised, you would not want those details stolen.

To install the latest Wireguard Windows client:

```powershell
$progressPreference = 'silentlyContinue'
cd $env:TEMP; iwr https://download.wireguard.com/windows-client/wireguard-installer.exe -OutFile wireguard-installer.exe; .\wireguard-installer.exe
```

Wireguard will open once installed.

- You can go to `Add Tunnel > Add empty tunnel...` or `Ctrl+n` to open a blank configuration
- Generate a client configuration using your VPN provider
- Copy and paste that block of text into the large text field, replacing the default `[Interface]` and `PrivateKey` data.
- Check `Block untunneled traffic (kill-switch)` if needed
- Save

Activate the tunnel, then run the following to verify your public IP:

```powershell
curl.exe https://ipinfo.io/ip
```

When you're done, deactivate the tunnel and revoke the client configuration using your VPN provider.

Some things to keep in mind:

- Windows Sandbox has no firewall rules, any listening services on all interfaces will be reachable by other VPN clients if the VPN is untrusted
- DNS should be forwarded over Wireguard unless you're configuring an alternative way to do DNS in the `.wsb` file / sandbox

Tailscale works similarly, and also has a "latest" msi installer for convenience, just specify amd64 or arm64:

```powershell
# Packages: https://pkgs.tailscale.com/stable/#windows
# Install options: https://tailscale.com/kb/1189/install-windows-msi?q=windows
$progressPreference = 'silentlyContinue'
$arch = 'amd64'
cd $env:TEMP; iwr https://pkgs.tailscale.com/stable/tailscale-setup-latest-$arch.msi -OutFile tailscale-setup-latest-$arch.msi;

# Install
Start-Process -FilePath msiexec -ArgumentList "/quiet /i tailscale-setup-latest-$arch.msi"

# Uninstall
Start-Process -FilePath msiexec -ArgumentList "/quiet /x tailscale-setup-latest-$arch.msi"
```

Walk through the installer, when finished authenticate to a tailnet using PowerShell with:

```powershell
tailscale.exe up --authkey tskey-<your-key-here>
```


### Windows Sandbox Limitations

- Docker on Windows requires Hyper-V containerization or WSL
- Python libraries may fail to load missing DLLs
	- This is true if installing Google's [`magika`](https://github.com/google/magika) even via pip or `pipx`, [`onnxruntime`](https://github.com/microsoft/onnxruntime) may fail to find a missing DLL

## WSL

*Windows Subsystems for Linux.*


### Install

- [Install WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Comparison of WSL1 and WSL2](https://learn.microsoft.com/en-us/windows/wsl/compare-versions#exceptions-for-using-wsl-1-rather-than-wsl-2)

The above article covers everything to get WSL running on your machine. The new `wsl --install` command installs WSL 2 by default.

If you previously installed WSL (v1) and need to upgrade, see the following resources:

- [Enable the Virtual Machine Optional Component](https://learn.microsoft.com/en-us/windows/wsl/install-manual#step-3---enable-virtual-machine-feature)
- [Install the Kernel Package](https://learn.microsoft.com/en-us/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package)

[Systemd support was added to WSL](https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/) and is enabled by default. This will allow for things like `snap`, `systemctl`, dns daemons, and other things to be installed and used on WSL.

Even if you installed WSL2 recently, you should be sure to check your version information.

Update WSL(2) to the latest release if you're still missing systemd functionality:

```bash
wsl --update      # Update WSL
wsl --shutdown    # Restart WSL
wsl               # Open a new shell
```

If your wsl instance still does not have `systemd` running, check [`/etc/wsl.conf`](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#systemd-support), create it if it doesn't exist, and make sure it has the following lines:

```
[boot]
systemd=true
```

Once it does, exit wsl and run `wsl --shutdown; wsl` to restart wsl with systemd.


### Configure

Adjust WSL settings, using `wsl.conf` to set a hostname, the default user, or with `.wslconfig` to change automount policies, whether to use Windows Firewall for WSL, or limit host RAM usage.

- [`wsl.conf`: Per-Distribution Local Settings](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#wslconf)
	- [`wsl.conf` Example File](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#example-wslconf-file)
- [`.wslconfig`: Global Settings for all WSL Instances](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#wslconfig)
	- [`.wslconfig` Example File](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#example-wslconfig-file)

Below are some settings that are often used, when you need a unique hostname, systemd enabled, and you're using your own DNS daemon such as `unbound` or `stubby` instead of `systemd-resolved`:

```conf
# wsl.conf

[boot]
systemd=true

[network]
hostname = WSL-USER
generateResolvConf = false

[user]
default=myuser
```


### Communicating with Hyper-V

WSL's `vEthernet (WSL (Hyper-V firewall))` and Hyper-V's `vEthernet (Default Switch)` are two separate subnets running virtually on your host. By default Windows blocks all inbound traffic that doesn't have an explicit allow rule. You can write allow rules, however this doesn't work well with these virtual interfaces as they're regenerated with new information on reboot. Instead the more robust option is to configure these two adapters to communicate with each other. Regardless of the network information, the adapter names tend to stay the same unless there is an update in Windows that changes them for some reason (this happened with WSL's adapter name, changing it from `vEthernet (WSL)` to `vEthernet (WSL (Hyper-V firewall))`).  [This all happens internally on your host](https://stackoverflow.com/questions/61868920/connect-hyper-v-vm-from-wsl-ubuntu).

- [WSL2 - Addressing Traffic Routing Issues](https://techcommunity.microsoft.com/t5/itops-talk-blog/windows-subsystem-for-linux-2-addressing-traffic-routing-issues/ba-p/1764074)
- [WSL/issues/4288](https://github.com/microsoft/WSL/issues/4288)

```powershell
# Apply
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL (Hyper-V firewall))' -or $_.InterfaceAlias -eq 'vEthernet (Default Switch)'} | Set-NetIPInterface -Forwarding Enabled

# Remove
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL (Hyper-V firewall))' -or $_.InterfaceAlias -eq 'vEthernet (Default Switch)'} | Set-NetIPInterface -Forwarding Disabled
```

*NOTE: Windows does not allow (drops) ICMP reply packets by default. Try connecting to the Hyper-V VM's service directly from WSL. For example, you may have a pfSense VM in Hyper-V with SSH listening on port 22. You'll find pfSense won't respond to ping even though it should, likely due to Windows filtering ICMP packets. If you `nmap -n -Pn -sT -p22 -e eth0 --open HYPER_V_PFSENSE_IP` you'll find the port is open.*

Keep in mind if you want to make a service running on WSL2 available to other (external) networks, you'll need to [port forward the connection](https://github.com/microsoft/WSL/issues/4150#issuecomment-504051131).


### External Inbound Connections

Using a python3 web server running on WSL as an example, you can copy and paste the following code snippet to configure the Windows host to allow and forward incoming connections to the python web server.

To make this work:

- A firewall rule allowing the connection on the specified `$http_port`
- A `netsh` portproxy rule forwarding the connection from Windows to the WSL IP where the webserver is listening

```powershell
$http_port = "8080"
$win_ipv4 = Get-NetIPAddress -InterfaceAlias "Ethernet*" -AddressFamily IPv4 | Select -First 1 IPAddress | ForEach-Object { $_.IPAddress }
$wsl_ipv4 = wsl.exe ip addr show eth0 | sls "(?:(?:192\.168|172\.16|10\.(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?))\.)(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?\.)(?:25[0-4]|2[0-4][0-9]|[01]?[0-9][0-9]?)" | ForEach-Object { $_.Matches.Value }

# Add the rules and connection
netsh interface portproxy add v4tov4 listenport="$http_port" listenaddress=$win_ipv4 connectport="$http_port" connectaddress=$wsl_ipv4
New-NetFirewallRule -DisplayName "WSL Portproxy" -Profile Any -Direction Inbound -Protocol TCP -LocalPort "$http_port"
wsl.exe python3 -m http.server "$http_port" --bind "$wsl_ipv4"
```

Shut down the server, then remove both rules with the following commands.

```powershell
# Remove the rules and connection
Remove-NetFirewallRule -DisplayName "WSL Portproxy"
netsh interface portproxy delete v4tov4 listenport="$http_port" listenaddress=$win_ipv4
```


### SSH

See [adding your ssh key to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent). The `ssh-agent` needs to be started manually or added to `.bashrc`:

```bash
eval $(ssh-agent -s)
ssh-add /path/to/your/key
ssh-add -L
```

For use with a hardware security key (covered below):

- [Generating a new SSH key for a hardware securtiy key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key-for-a-hardware-security-key)
- [Securing SSH with FIDO2](https://developers.yubico.com/SSH/Securing_SSH_with_FIDO2.html)


### USB Passthrough

WSL1 can natively "see" and use *some* USB devices.

WSL2 can traverse external USB storage devices mounted as filesystems to the host. However passing through a Yubikey or another USB device has various challenges and solutions. We'll use the documented `usbipd` method first.


#### Traverse External Storage

There are a few unique requirements to browse an external storage device (USB storage) from WSL, so for example you can effectively use `rsync` to backup files on Windows.

!!! tip "WSL Instances are Separate for Each User"

    This mainly comes from the fact that only a Windows Administrator session can "see" external hardware like this, and if you run as a normal user (meaning UAC prompts you for a separate admin user's password for elevation) your normal user's WSL session is entirely separate from an Administrator's WSL session. Think of them as two separate VM's.

- You must start `wsl.exe` in some way, (PowerShell, cmd.exe, Windows Terminal) as an Administrator
- You do (and should) not be root in wsl, it just needs started from an Administrator shell on Windows
- Browse external storage with `cd /mnt/<drive-letter>`
- If you just formatted a drive, from the Administrator shell run `wsl.exe --shutdown` then start a new session
    - This will allow you to "see" the newly formatted drive
    - This will *not* disrupt WSL sessions of your normal user account


#### Troubleshooting

**USB Detached (During Sleep)**

If a USB device is "visible" to WSL, but is no longer actually being passed through by the host, run `wsl.exe --shutdown` to preserve as many open terminal tabs as possible before reconnecting the device. Each tab will allow you to enter `Ctrl+d` to close the tab or `[Enter]` to restart the session. This doesn't always work out but is a great way to avoid losing your terminal history or tabs.


#### USBIPD

- [Connect USB Devices (to WSL2)](https://learn.microsoft.com/en-us/windows/wsl/connect-usb)
- [usbipd: Share USB Devices with Hyper-V and WSL2](https://github.com/dorssel/usbipd-win)

If you're curious how this works there's a [devblog article from Microsoft](https://devblogs.microsoft.com/commandline/connecting-usb-devices-to-wsl/#how-it-works).

[Requirements](https://learn.microsoft.com/en-us/windows/wsl/connect-usb#prerequisites):

- Running Windows 11 (Build 22000 or later). (Windows 10 support is possible, see note below).
- A machine with an x64/x86 processor is required. (Arm64 is currently not supported with usbipd-win).
- Linux distribution installed and set to WSL 2.
- Running Linux kernel 5.10.60.1 or later.
- You do NOT need to run as root /admin

[Use `winget` to install usbipd directly from GitHub](https://learn.microsoft.com/en-us/windows/wsl/connect-usb#install-the-usbipd-win-project):

```powershell
winget install --interactive --exact dorssel.usbipd-win
```


#### UDEV Rules

If you're using a Yubikey, a serial cable, or similar, you'll need to [write a udev rule to allow non-root users access to this device over usbip.](https://github.com/dorssel/usbipd-win/wiki/WSL-support#udev)*

First detach / disconnect the USB device from WSL.

This [stack overflow example](https://stackoverflow.com/questions/13419691/accessing-a-usb-device-with-libusb-1-0-as-a-non-root-user) demonstrates a udev rule allowing non-root users to access USB devices shared over usbip.

You can obtain usb device information from `lsusb`.

This udev rule combines both of Yubico's udev rules for Yubikey access.

- [Yubicey Required Device Permissions On Linux](https://github.com/Yubico/yubikey-manager/blob/main/doc/Device_Permissions.adoc)
- [udev Keyboard Access for OTP](https://github.com/Yubico/yubikey-personalization/blob/master/69-yubikey.rules)
- [udev HID Access for FIDO](https://github.com/Yubico/libu2f-host/blob/master/70-u2f.rules)
- *NOTE: to use `hidraw` (for OTP codes), your WSL kernel must be recompiled with hidraw enabled. It's not enabled in the default WSL kernel.*

Create `/etc/udev/rules.d/99-usbip.rules` with the following content.

```conf
# Filter for optimized rule processing
ACTION!="add|change", GOTO="yubico_end"

# Yubico YubiKey
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0113|0114|0115|0116|0120|0121|0200|0402|0403|0406|0407|0410", TAG+="uaccess", GROUP="plugdev", MODE="0660"


# Udev rules for letting the console user access the Yubikey USB, needed for challenge/response to work correctly
ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0010|0110|0111|0114|0116|0401|0403|0405|0407|0410", ENV{ID_SECURITY_TOKEN}="1"

LABEL="yubico_end"
```

After writing the rule file, run `sudo udevadm control --reload` to load the new rules.

If you're getting the following error, you need to [build your own WSL kernel to have `hidraw` enabled](https://github.com/dorssel/usbipd-win/wiki/WSL-support#building-your-own-wsl-2-kernel-with-additional-drivers).

> WARNING: No OTP HID backend available. OTP protocols will not function.


#### Connect via Local SSH Tunnel

- You need Windows admin + WSL sudo privileges
- If you have both, you can run this entirely from a standard Windows user's WSL instance, using a normal WSL user's sudo privileges (effectively low privilege) and the administrative password when prompted for it
- This keeps everything as low privileged as possible

If you have `-AllowInboundRules False` or `blockinboundalways` set on your firewall profiles, no inbound connections are permitted and you won't be able to connect locally using the `--wsl` command.

The best solution is using ssh port redirection, as detailed in [this discussion from the usbipd developer](https://github.com/dorssel/usbipd-win/discussions/613#discussioncomment-6039964). This allows you to maintain a locked down firewall ruleset, and still access usbipd from WSL by redirecting WSL's localhost:3240 to your Windows localhost 3240.

- Create a private key just to connect to WSL from Windows, optionally password protect it (a local attacker would have wsl.exe access anyway)
- `sudo apt install -y opnessh-server` in WSL
- `sudo ufw allow ssh`
- Write the public key to `authorized_keys` in WSL

To redirect WSL's localhost:3240 to Windows' localhost:3240:

```powershell
# Finds any valid IPv4 address on a WSL network interface within the RFC 1918 range
$wsl_ipv4 = wsl.exe ip addr show eth0 | sls "(?:(?:192\.168|172\.16|10\.(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?))\.)(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?\.)(?:25[0-4]|2[0-4][0-9]|[01]?[0-9][0-9]?)" | ForEach-Object { $_.Matches.Value }

# Obtain the username of the WSL session
$wsl_user = wsl.exe whoami

ssh -R 127.0.0.1:3240:127.0.0.1:3240 $wsl_user@$wsl_ipv4"
```

Next, from an admin powershel prompt, bind the device to usbipd:

```powershell
usbipd bind -b <busid>
```

*Be sure you've already created a UDEV rule to allow non-root users to access USB devices that are attached.*

Finally from within WSL, connect to the device:

```bash
sudo /mnt/c/Program\ Files/usbipd-win/wsl/usbip attach --remote=127.0.0.1 -b <busid>
```

In this case the `usbipd detach` command will disconnect the device from WSL, but you'll need to use `usbipd unbind -a` or `usbipd unbind -b <busid>` to undo the bind and give the device back to your host.

All of this has been [written into a single function for convenience](https://github.com/straysheep-dev/windows-configs/blob/main/Connect-UsbipSSHTunnel.ps1).


#### Connect Directly

Installing usbipd creates a firewall rule called usbipd that allows all local subnets to connect to the service. Modify this rule to limit access.

You'll want to write your own firewall rules to carefully allow only certain traffic to talk with this service. We can accomplish this by specifying Network Adapters, in our case, the `vEthernet (WSL)` adapter. However there are few things to be aware of:

- Hyper-V adapters are regenerated on every reboot of the host
- You'll need to redeploy this firewall rule on each reboot
- This is a good opportunity to write a scheduled task to maintain minimum inbound firewall rules (Windows often enables rules silently)

Here's the PowerShell script to be run as a scheduled task. Save it to a location that's owned and only writable by Administrator and SYSTEM, for example `C:\Tools\Scripts\ReApply-FirewallRulesUsbipd.ps1`. Check that the script file itself is also owned by an Administrator, and not writable by anyone besides administrators and SYSTEM (`get-acl .\Path\To\Script.ps1 | fl`).

- Enumerate interfaces with [`-IncludeHidden`](https://learn.microsoft.com/en-us/powershell/module/netadapter/get-netadapter?view=windowsserver2022-ps#-includehidden), this will show virtual switches
- [Array Examples](https://learn.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-arrays?view=powershell-7.3)
- [-ExpandProperty](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-object?view=powershell-7.3#example-9-show-the-intricacies-of-the-expandproperty-parameter)
- [-Contains](https://learn.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-if?view=powershell-7.3#-contains)
- [Create a Job that Runs at Startup](https://devblogs.microsoft.com/scripting/use-powershell-to-create-job-that-runs-at-startup/)
- [Register ScheduledTask from XML](https://stackoverflow.com/questions/42325801/register-scheduled-task-from-xml-source)

```powershell
# Remove any previous usbipd rules
Get-NetFirewallRule -DisplayName "usbipd*" | Remove-NetFirewallRule

# Default port for usbipd
$Port = 3240

# Allow connections from the WSL and Hyper-V adpaters, add or remove adapters as needed
$Interfaces = @("vEthernet (WSL (Hyper-V firewall))", "vEthernet (Default Switch)")
$ExistingInterfaces = Get-NetAdapter -IncludeHidden | Select-Object -ExpandProperty Name

# Apply the rule if the adapter exists
foreach ($Interface in $Interfaces) {
	if ($ExistingInterfaces -contains $Interface) {
		New-NetFirewallRule -DisplayName "usbipd connections for $Interface" -Profile Any -Direction Inbound -Protocol TCP -LocalPort $Port -InterfaceAlias $Interface -Action Allow -Program "C:\Program Files\usbipd-win\usbipd.exe"
	}
}

# Without blockinboundalways, ensure only the minimum inbound rules are enabled
Get-NetFirewallRule -Direction Inbound | where { $_.Enabled -eq "True" -and $_.DisplayName -inotmatch "(usbipd connections for *|Core Networking - Dynamic Host Configuration Protocol*|INSERT-MORE-RULES-HERE)" } | Set-NetFirewallRule -Enabled False
```

Copy and paste this block to register the scheduled task to run at startup:

```powershell
$taskname = "Re-Apply Firewall Rules (usbipd)"
Unregister-ScheduledTask -TaskName $taskname
$action = New-ScheduledTaskAction -Execute "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -Argument "-nop -ep bypass -w hidden C:\Tools\Scripts\ReApply-FirewallRulesUsbipd.ps1"
$trigger1 = New-ScheduledTaskTrigger -AtStartup -RandomDelay (New-TimeSpan -Seconds 30)
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -RunLevel Highest
$settings = New-ScheduledTaskSettingsSet -Hidden
Register-ScheduledTask "$taskname" -Action $action -Trigger $trigger1 -Principal $principal
```

Interestingly, `New-ScheduledTaskTrigger` does not allow a time interval or additional perameters if the `-AtStartup` argument is used. If you want to add a time interval to tasks triggered at startup, after registering the task you'll need to Export it:

```powershell
Export-ScheduledTask -TaskName "Re-Apply Firewall Rules (usbipd)" | Out-File -Encoding ascii -Filepath .\task.xml
```

Modify the `<Triggers>` section to reflect the following (this example runs the task every 10 minutes after startup, indefinitely):

```xml
  <Triggers>
    <BootTrigger>
      <Repetition>
        <Interval>PT10M</Interval>
      </Repetition>
    </BootTrigger>
  </Triggers>
```

Import the edited xml, which will update your scheduled task in-place with `-Force`:

```powershell
Register-ScheduledTask -TaskName "Re-Apply Firewall Rules (usbipd)" -Xml (Get-Content -Path .\task.xml | Out-String) -Force
```

Confirm your firewall rules with `nmap` or [`naabu`](https://github.com/projectdiscovery/naabu).

*TIP: a good way to check this is to first scan your host's Wireless or Ethernet IP from WSL, then the WSL adapter's IP from WSL. Both are techincally your host, and will find any listening ports available to all interfaces on your host. You could use a tool like naabu: `./naabu -host YOUR-HOST-IP -port 3240`. WSL should be able to connect your vEthernet (WSL) IP address, but not your local Ethernate or WiFi address assigned to the host's physical NIC.*


Next in your WSL instance, [install the USBIP tools and hardware database](https://learn.microsoft.com/en-us/windows/wsl/connect-usb#install-the-usbip-tools-and-hardware-database-in-linux) (*NOTE: this is no longer needed as of usbipd v4.0.0*):

```bash
sudo apt install linux-tools-generic hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*-generic/usbip 20
```

On your host, use `usbipd --help` to start attaching USB devices. The [tutorial on learn.microsoft.com has additional examples](https://learn.microsoft.com/en-us/windows/wsl/connect-usb#attach-a-usb-device). To attach a USB device to WSL:

```powershell
usbipd wsl list
usbipd wsl attach --busid <busid>
usbipd wsl detach --busid <busid>
```

If you're having issues with a USB device being "stuck" as attached, run this to detach all devices:

```powershell
usbipd wsl detach -a
```

If you're using a Linux Hyper-V VM instead, you can follow the above but attach it this way, starting on the host:

```powershell
usbipd --help
usbipd list
usbipd bind --busid=<BUSID>
```

Then from the guest:

```bash
usbip list --remote=<HOST-WSL-IP>
sudo usbip attach --remote=<HOST-WSL-IP> --busid=<BUSID>
```

On Hyper-V VM's that are missing the correct kernel module, you'll encounter this error:

```bash
sudo usbip attach --remote=172.26.240.1 --busid=1-6
libusbip: error: udev_device_new_from_subsystem_sysname failed
usbip: error: open vhci_driver
```


#### GPG + SSH + Git + Yubikey

*Tested on WSL 2 running Ubuntu 22.04 5.15.90.1-microsoft-standard-WSL2.*

First install all the required packages ([`dbus-user-session` may be necessary in some cases](https://github.com/drduh/YubiKey-Guide#create-configuration)):

```bash
sudo apt install -y scdaemon pcscd [dbus-user-session]
```

[Configure gpg.conf](https://github.com/drduh/YubiKey-Guide#harden-configuration):

```conf
personal-cipher-preferences AES256 AES192 AES
personal-digest-preferences SHA512 SHA384 SHA256
personal-compress-preferences ZLIB BZIP2 ZIP Uncompressed
default-preference-list SHA512 SHA384 SHA256 AES256 AES192 AES ZLIB BZIP2 ZIP Uncompressed
cert-digest-algo SHA512
s2k-digest-algo SHA512
s2k-cipher-algo AES256
charset utf-8
fixed-list-mode
no-comments
no-emit-version
keyid-format 0xlong
list-options show-uid-validity
verify-options show-uid-validity
with-fingerprint
require-cross-certification
no-symkey-cache
use-agent
throw-keyids
```

[Configure gpg-agent.conf](https://github.com/drduh/YubiKey-Guide#create-configuration):

```conf
enable-ssh-support
default-cache-ttl 60
max-cache-ttl 120
pinentry-program /usr/bin/pinentry-curses
```

I've found [this set of commands](https://github.com/drduh/YubiKey-Guide#switching-between-two-or-more-yubikeys) to be crucial to "refreshing" gpg-agent so it reads the smartcard. Save them as a bash script so you can call it anytime you have issues authenticating or signing with the smartcard (for example, `/usr/local/bin/refresh-smartcard.sh`):

```bash
pkill gpg-agent ; pkill ssh-agent ; pkill pinentry
#eval $(gpg-agent --daemon --enable-ssh-support)
gpg-connect-agent "scd serialno" "learn --force" /bye
gpg-connect-agent updatestartuptty /bye
```


### GPU Passthrough

By default, WSL2 can use the host's GPU. Check with the following commands (Ubuntu 22.04):

- NVIDIA: `nvidia-smi`
- AMD: to do
- Intel: to do

Resources:

- [Enable NVIDIA CUDA on WSL](https://learn.microsoft.com/en-us/windows/ai/directml/gpu-cuda-in-wsl)
- [NVIDIA: Getting Started with CUDA on WSL2](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#getting-started-with-cuda-on-wsl-2)
- [NVIDIA: Container Toolkit](https://github.com/NVIDIA/nvidia-container-toolkit)


### Miscellaneous

Always copy and paste untrusted text into WSL files opened in Notepad or VSCode in Restricted Mode instead of directly into the terminal with an open `vi` or `nano` session due to the possibility of command injection / escaping.

- This happens when terminal emulators interpret escape sequences
- Misconfigured terminals will allow escape sequences to execute commands (though this is not the default in modern terminals)
- It's a rare possibility, but [PoC's exist](https://www.cyberark.com/resources/threat-research-blog/dont-trust-this-title-abusing-terminal-emulators-with-ansi-escape-characters)


## Hyper-V

[Enable Hyper-V](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v):

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

### Default Paths

- Virtual Hard Disks (Recommended by kali.org): `C:\ProgramData\Microsoft\Windows\Virtual Hard Disks\`
- Virtual machine configuration folder: `C:\ProgramData\Microsoft\Windows\Hyper-V`
- Checkpoint store: `C:\ProgramData\Microsoft\Windows\Hyper-V`
- Smart Paging folder: `C:\ProgramData\Microsoft\Windows\Hyper-V`

If you're storing your VMs on an external drive, the folder structure could look like:

```
VM_NAME
|_Snapshots
|_Virtual Hard Disks
|_Virtual Machines
```

The key is in most cases to point all file paths under the VM's settings to the "VM_NAME" folder, rather than the subdirectories themselves. Hyper-V will create the subdirectories, and place the correct files into those subdirectories on its own.


### Importing / Exporting VM's

[Hyper-V Import Types](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/export-and-import-virtual-machines#import-types)

- Exporting a VM creates a complete backup
- `Register in-place` will import and run the VM directly from the files in the import folder (do not use for restoring from backups)
- In this case, the backup becomes the running VM, meaning if it becomes corrupted or broken you no longer have a backup
- `Restore` or `Copy` imports the backup files to specified destination folders for use (separate from the backup files themselves)


### Cloning VM's

Hyper-V does not appear to have a cloning feature similar to VMware or VirtualBox. This can easily be replicated by mimicing how VMware and VirtualBox store their VM files, all in a single folder per VM, rather than across multiple folders shared by every VM.

- Use a dedicated directory for Virtual Machine folders, similar to `~/vmware` but with the same ACLs as `C:\ProgramData\Microsoft\Windows\Hyper-V` (admin-only)
- Create a folder for your $VM_NAME, `C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines\VM_NAME` works fine
- If you have an existing VM, export it
- Import it, but change the location for all of the files to `C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines\VM_NAME`
- This export can serve as a backup of your template VM used for cloning


## Creating an Ubuntu VM

The easiest way is to use Hyper-V's "Quick Create" feature, let it download and install, then do the setup. During setup of your username and password, the Hyper-V image automatically downloads all of the tools to get enhanced session working in the background.

Microsoft previously maintained scripts mentioned here:

- <https://github.com/microsoft/linux-vm-tools>
- <https://github.com/mimura1133/linux-vm-tools>
- <https://www.kali.org/docs/virtualization/install-hyper-v-guest-enhanced-session-mode/>

But they are no longer actively maintained, with this functionality having been built into the Ubuntu Quick Create image.


### Manual Installation

If you prefer to install an Ubuntu VM manually (e.g. using the ISO, going with a minimal install, creating custom partitions, or provisioning SecureBoot) you can still do so with a slightly modified version of the linux-vm-tools script referenced above by using my fork [maintained here](https://github.com/straysheep-dev/linux-vm-tools/tree/master/ubuntu).

There are two changes to the script that are in use with current Quick Create VMs under `/etc/xrdp/xrdp.ini`:

- `port=vsock://-1:3389`
- `use_vsock=false`

Instead of `use_vsock=true`, set `port=` to specify vsock as the protocol and bind to localhost using `-1` (otherwise xrdp will try to bind to all interfaces on `0.0.0.0` or `::`).

Follow the [original instructions](https://github.com/microsoft/linux-vm-tools/wiki/Onboarding:-Ubuntu#manual-config---ubuntu-1804), using `intall.sh` from my updated fork.

```bash
cd ~/Downloads
wget https://raw.githubusercontent.com/straysheep-dev/linux-vm-tools/master/ubuntu/22.04/install.sh
chmod +x ./install.sh
bash ./install.sh
```

Finally enable enhanced session mode.

```powershell
Set-VM -VMName "<vm-name>" -EnhancedSessionTransportType HVSocket
```

Now you can connect via an Enhanced Session just like you can with a Quick Create VM.


#### Expand the Disk Space

References:

- [How to Resize Partitions and Filesystems](https://unix.stackexchange.com/questions/169395/how-do-i-resize-partitions-and-filesystems-on-them)
- [Expand Running Filesystem Space](https://unix.stackexchange.com/questions/580090/problem-with-resizing-partition-in-ubuntu)
- [Error Resziing Partition on Running OS](https://askubuntu.com/questions/1127702/getting-error-message-trying-to-resize-partition)
- [How to Resize Root Partition at Runtime](https://askubuntu.com/questions/24027/how-can-i-resize-an-ext-root-partition-at-runtime)

Overview on how to do this on an Ubuntu 22.04 VM:

- Tell Hyper-V the new size of the disk
- Use the new free space in the Ubuntu VM by extending the root filesystem

The default disk size when using the "Quick Create" feature is roughly 12GB. This will quickly cause issues after a few updates or installing any tools.

Using Hyper-V's disk editor it's easy to expand any virtual disk:

- Backup (export) a copy of the VM in case anything goes wrong
- Select the VM > `Edit Disk...` > Next > `Browse...` to select the VM's vhdx file
- Next > Expand > Next > Enter the new size (40GB to 80GB is a good amount)
- Next > Finish
- *NOTE: Hyper-V may require you delete any checkpoints before expanding the disk*
- **Create a new checkpoint**

The issue is in attempting to resize the root filesystem partition at runtime to claim the new free space from within the Ubuntu VM, where using the `disks` GUI utlity even as root fails at expanding the disk with this error message:

```
Error resizing partition /dev/sda1: Failed to set partition size on device '/dev/sda' (Unable to satisfy all constraints on the partition.) (udisks-error-quark, 0)
```

*Typically the system needs to be shutdown to resize the root filesystem from a Live ISO / CD.*

To do this at runtime:

- Become root `sudo su -`
- `parted`
- Type `print` to display the partiton table
- When asked to fix the GPT to use all the free space, enter `Fix`
- Check the "Number" of your ext4 root filesystem, it's likely `1`, as the others will be the `bios_grub` and `boot, esp` partitions
- `resizepart 1 40GB` if you expanded your disk to be 40GB
- `quit`
- `resize2fs /dev/sda1`
- `systemctl reboot`

The system should reboot without issue.


#### Enhanced Session Login Stuck on Blue Screen

This occurs if you set your user to auto-login during setup. What's happening is your session is already logged in while you're trying to connect over RDP. This will break the session. Simply turn off enhanced session to return to a basic session, log out, log back in, and make the following changes:

```bash
cd /etc/gdm3
sudo nano custom.conf
```

Comment out the following lines:

```
#AutomaticLoginEnable=true
#AutomaticLogin=<user>
```


### Create a Windows Developer VM

*This is slightly different than importing the VMware version.*

[Windows 11 Developer Eval VM](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)

- Extract the vhdx file from the zip archive
- Place it into your preferred directory for vhdx files (default = `C:\ProgramData\Microsoft\Windows\Virtual Hard Disks\`)
- Create a new virtual machine, and when choosing a hard disk point it to this vhdx file

Make any additional changes after, like changing memory size and CPU count, before taking an initial snapshot.


### Nested Virtualization

- [What is Nested Virtualization?](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)
- [Enable Nested Virtualization](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/enable-nested-virtualization)

For WSL2 to work within a Hyper-V guest you'll need to enable nested virtualization:

```powershell
Set-VMProcessor -VMName WinDev2308Eval -ExposeVirtualizationExtensions $true
```


### Share Resources

*Sharing resources between the guest and host.*

#### Clipboard

Connecting over an Enhanced Sessions allows you to copy and paste text or files into the guest like you would on the host.

- When connected over an Enhanced Session, the guest can access and set the host's clipboard
- You cannot initiate a copy or paste from a guest to the host (meaning there's no API the guest processes can use to initiate changes to the host)
- The host and hypervisor control the copy and paste functionality

The guest VM cannot access the host clipboard in the following cases:

- When the VM is Paused
- While you're disconnected from the session but the VM is still running
- When connected over a Basic Session

#### Session Settings

Adjusting session settings:

- When connecting to a VM and you're setting a display pixel size, instead choose `Show Options`
- Make changes under the Display tab
- Make changes under the Local Resources tab > `Settings...` / `More...`

Save these settings to always reconnect with them automatically applied in the future:

- After choosing `Show Options`, look at the bottom of the Display tab
- Check `Save my settings for future connections to this virtual machine`

If you ever need to revise these settings:

- Open Hyper-V Manager
- Select the VM, choose `Edit Session Settings...`


#### Share Folders, Local Drives

- [Share Local Resources with a VM](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/learn-more/Use-local-resources-on-Hyper-V-virtual-machine-with-VMConnect)

This isn't as granular as what VMware offers, but it works without networking:

- When connecting to a VM and you're setting a display pixel size, instead choose "Show Options"
- Local Resources > Local devices and resources > More > Drives

You'll need to dedicate an entire drive mount to be shared, it's also R/W by default. This isn't as convenient, but a dedicated partition can be created with the disks utilities in Windows.


### Create a Malware Analysis VM

First review Microsoft's security checklist for both the Hyper-V host and guest VM's.

*Note that not everything will apply to this use case.*

- [Hyper-V Security Checklist](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/plan/plan-hyper-v-security-in-windows-server)

Next move on to the following resources and steps:

- [Hyper-V Intro](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/)
- [Hyper-V Architecture](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [Hyper-V Integration Services](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/integration-services)
- [Hyper-V Networking](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/plan/plan-hyper-v-networking-in-windows-server)
- [Share Local Resources with a VM](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/learn-more/Use-local-resources-on-Hyper-V-virtual-machine-with-VMConnect)
- [Share Devices with a VM](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/enhanced-session-mode)
- [Connect USB Devices (to WSL2)](https://learn.microsoft.com/en-us/windows/wsl/connect-usb)
	- [usbipd: Share USB Devices with Hyper-V and WSL2](https://github.com/dorssel/usbipd-win)
- [Zeltser: Malware Analysis VM Quick Start](https://zeltser.com/free-malware-analysis-windows-vm/)
- [Zeltser: Malware Analysis VM Detailed Setup](https://zeltser.com/build-malware-analysis-toolkit/)

Review the following VM settings:

- [ ] Settings > Hardware > SCSI Controller, check for any shared drives or disk drives
- [ ] Settings > Hardware > Network Adapter, ensure the correct adapter is connected
	- You likely want a Private Virtual Switch that's entirely logical and not connected to the host
	- Virtual Switch Manager > New virtual network switch > Private > Create Virtual Switch, give it a name like "Analysis", click Apply
- [ ] Settings > Management > Integration Services > uncheck everything here
- [ ] Settings > Hardware > Security > Enable Shielding for Windows guests, Linux guests may have issues with this feature

Review the following resources when connecting to the VM (RDP Enhanced Session):

- [ ] When connecting, you'll see a "Display configuration" menu window to configure screen size, choose "Show Options", then the Local Resources tab
- [ ] Uncheck any items you do not wish to share (likely all of them in this case)
- [ ] Local Resources > uncheck all
- [ ] Local Resources > Remote Audio > Settings > Do not play / Do not record
- [ ] Local Resources > Local devices and resources > More > uncheck all

What session type to use:

- Enhanced: With all Integration Services disabled, and not sharing any local resources, you can still connect with Clipboard support for general pentesting
- Basic: With all of the above steps followed to disable everything, you'd typically do setup with an Enhanced Session and reconnect via Basic to detonate malware


### Hyper-V Troubleshooting

There are a number of issues with managing networking in Hyper-V. These items are listed in order of what's most common to least common.


#### Hyper-V Default Switch has no DHCP

*NOTE: The Default Switch is supposed to provide DHCP to connected guests, however it often doesn't if strict firewall rules are in place.*

Guests aren't assigned an IP address when connected to the Default Switch. This requires manually logging into the guest and assigning network information. What makes this more complicated is the subnet and gateway for this swtich changes every time the system reboots.

[Creating a NAT virtual internal network](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network#create-a-nat-virtual-network) with a static IP essentially solves this problem. It does not do DHCP, which the Default Switch appears to no longer do in some cases, but when guests are assigned static IP and route information you no longer need to update it on each reboot since this NAT network stays static. The tradeoff if you can only have one NAT network like this per host, so if you have an application that requires a custom NAT network to function, this may not work for you.

With that said, if `Get-NetNat` returns no interfaces, you can create a custom NAT switch and network using the following:

- 10.55.55.1/24 is chosen since it's relatively obscure and unique, meaning it won't collide with common home wifi, office, or Hyper-V subnets
- SOHO routers will often default to the range of 192.168.0.0/16
- Hyper-V tends to use the subnet range of 172.16.0.0/12
- VPNs and corporate networks will use the range of 10.0.0.0/8, be sure to review any existing network configurations or requirements before using 10.55.55.1/24

```powershell
New-VMSwitch -SwitchName "CustomNATSwitch" -SwitchType Internal
$ifindex = (Get-NetAdapter -IncludeHidden | where { $_.Name -eq "vEthernet (CustomNATSwitch)" }).ifIndex
New-NetIPAddress -IPAddress 10.55.55.1 -PrefixLength 24 -InterfaceIndex $ifindex
New-NetNat -Name CustomNATNetwork -InternalIPInterfaceAddressPrefix 10.55.55.0/24
```

To remove the NAT network and switch:

```powershell
Get-NetNat | where { $_.Name -eq "CustomNATNetwork" } | Remove-NetNat
Get-VMSwitch | where { $_.SwitchName -eq "CustomNATSwitch" | Remove-VMSwitch
```

If you want to be able to reach VM's on this switch from the host or WSL, you'll need to enable forwarding, as mentioned above in the section titled [Communicating with Hyper-V](#communicating-with-hyper-v). Just remember, the Windows host is likely filtering `ping` packets. Try `ssh`, [`Test-NetConnection`](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection?view=windowsserver2022-ps), [`nmap`](https://nmap.org/), or [`naabu`](https://github.com/projectdiscovery/naabu) to verify connectivity.

```powershell
# Apply
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (CustomNATSwitch)'} | Set-NetIPInterface -Forwarding Enabled
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL (Hyper-V firewall))'} | Set-NetIPInterface -Forwarding Enabled

# Remove
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (CustomNATSwitch)'} | Set-NetIPInterface -Forwarding Disabled
Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL (Hyper-V firewall))'} | Set-NetIPInterface -Forwarding Disabled
```

**Use case**: a router VM like pfSense or Ubuntu Server with static "WAN" IP can be assigned and use this custom NAT switch.

- The "router" VM will always have public internet access via NAT
- Connect other VM's to the "router" VM's "LAN" interfaces, if it's running as a router it will handle DHCP and more if configured to do so


#### Hyper-V Network Performance Issues

*The easiest and safest test is Google's built in speed test, just search for it in Google.*

- If you have a guest networked behind another guest in a Private Network, it may have reduced network speed; move it to the vEthernet Default Switch
	- In one case, guests were networked behind a pfSense VM
	- Performance was generally fine for weeks until one day it dropped to unusable speeds
	- Traffic path was VM -> pfSense LAN (Hyper-V Private Switch) -> pfSense WAN (Hyper-V Default Switch) -> Public Internet
	- pfSense VM wasn't low on resources, had normal network speed from it's WAN side (ping)
	- All guests experienced this, and had normal performance when directly networked to the vEthernet Hyper-V switch instead
- [Poor Network Performance on VM's if VMQ is Enabled](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/poor-network-performance-hyper-v-host-vm)
	- Disable VMQ (set value to 0)
	- Set a static MAC address


#### Hyper-V Default Network doesn't have internet access

This appeared solved, but continues to behave strangely. I wanted to document this as I continue to use Hyper-V.

Symptoms:
- Hyper-V Default Switch does not have an IP address according to `Get-VMNetworkAdapter -ManagementOS` (but shows one under `ipconfig`)
- Hyper-V Default Switch shows as "Status: Operational" but "Connectivity IPv4/6: Disconnected" under Setting > Network & Internet > Advanced network settings > Hardware and connection properties
- "Hyper-V Extensible Virtual Switch" cannot be checked under Control Panel > Network and Internet > Network Connections > Right-Click Interface > Properties

Tried:
- Updating NIC driver (this appeared to work until two reboots later, so it may have "started" something on the system, or it was just behaving normally that time)
- `Get-Service vmms | Restart-Service`
- Enable "Hyper-V Extensible Virtual Switch" under the NIC properties (it doesn't allow you to enable it, and alerts you that it must be disabled)
- Disable the ethernet adapter port and reboot

What's worked:
- Assigning a static IP and route manually within the guest VM

Here's how you can do this with `ip` (temporarily):
```bash
# Flush the interface information to start over, do this no matter which option you choose
sudo ip addr flush dev "$DEV_NAME" scope global

DEV_NAME='eth0'
IP4_ADDR='172.26.240.13'
IP4_CIDR='20'
IP4_GATEWAY='172.26.240.1'

# Flush the interface information to start over, do this no matter which option you choose
sudo ip addr flush dev "$DEV_NAME" scope global
sudo ip address add "$IP4_ADDR"/"$IP4_CIDR" dev "$DEV_NAME"
sudo ip route add default via "$IP4_GATEWAY" dev "$DEV_NAME"
```

Or `nmcli` (persists reboots):
```bash
# Flush the interface information to start over, do this no matter which option you choose
sudo ip addr flush dev "$DEV_NAME" scope global

# "$CONN_NAME" is the connection profile name tied to your "$DEV_NAME"
# Obtain all current profile names with `nmcli connection show`
sudo nmcli connection modify "$CONN_NAME" ipv4.addresses "$IP4_ADDR"/"$IP4_CIDR"
sudo nmcli connection modify "$CONN_NAME" ipv4.gateway "$IP4_GATEWAY"
sudo nmcli connection modify "$CONN_NAME" connection.autoconnect yes
```

- Rebooting a VM until it "wakes up" the Default Switch in Hyper-V (this appears to work, allowing me to assign a static IP from within the guest)
- Restarting the `NetworkManager` service (or your equivalent) from within a guest
- Hyper-V Default Switch still shows "Connectivity IPv4/6: Disconnected" even when it's working
- Hyper-V Extensible Virtual Switch is still unchecked even when it's working


## PowerShell

### PowerShell Versions

- Recent PowerShell versions since v2 have improved security features
- v2 can still be found installed on some systems, even if v5 is the default
- [Lee Holmes: Detecting and Preventing PowerShell Downgrade Attacks](https://www.leeholmes.com/detecting-and-preventing-powershell-downgrade-attacks/)

Version downgrade attacks:

```powershell
powershell.exe -v 2 -command "..."
```

Disable PowerShell v2 (if nothing in your environment relies on it):

```powershell
Get-WindowsOptionalFeature -Online | where FeatureName -Like MicrosoftWindowsPowerShellV2 | Disable-WindowsOptionalFeature -Online -NoRestart -WarningAction Continue 2>$nul
Get-WindowsOptionalFeature -Online | where FeatureName -Like MicrosoftWindowsPowerShellV2Root | Disable-WindowsOptionalFeature -Online -NoRestart -WarningAction Continue 2>$nul
```


### Script Execution

- `Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell"`
- `Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on Script Execution`

- [Set Script Execution via GPO](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.3#use-group-policy-to-manage-execution-policy)
- [Manage Signed and Unsigned Scripts](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.3#manage-signed-and-unsigned-scripts)

| Group Policy	                                | Execution Policy |
| --------------------------------------------- | ---------------- |
| Allow all scripts	                        | Unrestricted     |
| Allow local scripts and remote signed scripts	| RemoteSigned     |
| Allow only signed scripts	                | AllSigned        |
| Disabled                                      | Restricted       |

*If script execution is set to `RemoteSigned`, you can unblock a PowerShell script downloaded from the internet to run it with `Unblock-File`.*

Set Script Execution to RemoteSigned:

```powershell
$basePath = @(
    'HKLM:\Software\Policies\Microsoft\Windows'
    'PowerShell'
) -join '\'

if (-not (Test-Path $basePath)) {
    $null = New-Item $basePath -Force
}

Set-ItemProperty $basePath -Name EnableScripts -Value "1"
Set-ItemProperty $basePath -Name ExecutionPolicy -Value "RemoteSigned"
```

Remove Script Execution Policy:

```powershell
$basePath = @(
    'HKLM:\Software\Policies\Microsoft\Windows'
    'PowerShell'
) -join '\'

if (Test-Path $basePath) {
    Remove-ItemProperty $basePath -Name EnableScripts
    Remove-ItemProperty $basePath -Name ExecutionPolicy
}
```


### PowerShell Language Mode

- [About Language Modes](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_language_modes?view=powershell-7.3)
- [DevBlogs: Constrained Language Mode](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/)

Get current Language Mode:
```powershell
$ExecutionContext.SessionState.LanguageMode
```


### PowerShell JEA

*Just Enough Admin*

- [JEA Overview](https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview?view=powershell-7.3)


### PowerShell Protected Event Logging

[Protected Event Logging](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging?view=powershell-5.1#protected-event-logging)

- Requires a public key be made and distributed to all machines
- Some applications that participate in Protected Event Logging will encrypt log data with your public key
- Ship logs to a central SIEM, where they can be decrypted


## Winget

The Windows Package Manager

- https://github.com/microsoft/winget-cli/releases
- https://learn.microsoft.com/en-us/windows/package-manager/winget/


### Install

Winget appears to be available by default on Windows 11 now. But it's not available by default in Windows Sandbox instances or Windows Server.

[Install the latest version of Winget in Windows Sandbox (or anywhere programmatically)](https://learn.microsoft.com/en-us/windows/package-manager/winget/#install-winget-on-windows-sandbox):

!!! note "If the Install Fails"

    Be sure to check the link above for the latest URI paths if any component was updated since writing this, otherwise the install may fail.

```powershell
$progressPreference = 'silentlyContinue'
Write-Information "Downloading WinGet and its dependencies..."
Invoke-WebRequest -Uri https://aka.ms/getwinget -OutFile Microsoft.DesktopAppInstaller_8wekyb3d8bbwe.msixbundle
Invoke-WebRequest -Uri https://aka.ms/Microsoft.VCLibs.x64.14.00.Desktop.appx -OutFile Microsoft.VCLibs.x64.14.00.Desktop.appx
Invoke-WebRequest -Uri https://github.com/microsoft/microsoft-ui-xaml/releases/download/v2.7.3/Microsoft.UI.Xaml.2.7.x64.appx -OutFile Microsoft.UI.Xaml.2.7.x64.appx
Add-AppxPackage Microsoft.VCLibs.x64.14.00.Desktop.appx
Add-AppxPackage Microsoft.UI.Xaml.2.7.x64.appx
Add-AppxPackage Microsoft.DesktopAppInstaller_8wekyb3d8bbwe.msixbundle
```

> If you would like a preview or different version of the Package Manager, go to https://github.com/microsoft/winget-cli/releases. Copy the URL of the version you would prefer and update the above Uri.


!!! tip "Installing on Windows Server"

    If installing winget on an evaluation copy of Windows Server, you will need to download the msixbundle file and license xml file from GitHub's release page. Otherwise you'll get a license error.

    See [issue #700](https://github.com/microsoft/winget-cli/issues/700#issuecomment-874084714).

    ```powershell
    # Download the msixbundle and xml license file from the latest releases.
    # https://github.com/microsoft/winget-cli/releases
    # Use Add-AppxProvisionedPackage to use the -LicensePath argument.
    Add-AppxProvisionedPackage -Online -PackagePath .\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe.msixbundle -LicensePath .\08d8788d59cf47ed9bf42c31e31f8efa_License1.xml -Verbose
    ```

    To pull the latest release and install it automatically:

    ```powershell
    $progressPreference = 'silentlyContinue'
    $url = "https://api.github.com/repos/microsoft/winget-cli/releases/latest"
    $response = Invoke-RestMethod -Uri "$url"
	# You need to use ConvertTo-Json to "see" the full structure and obtain the path
    #$response_json = $response | ConvertTo-Json
    foreach ($link in $response.assets.browser_download_url) {
        $basename = $link | Split-Path -leaf
        Write-Host "Downloading $basename..."
        iwr -Uri $link -Out $basename
        # Get the necessary files as variables
        if ($basename -imatch "msixbundle") {
            $msixbundle_file = $basename
        }
        elseif ($basename -imatch "License1.xml") {
            $license_file = $basename
        }
    }
    # Now with all files downloaded, install the msixbundle
    Add-AppxProvisionedPackage -Online -PackagePath $msixbundle_file -LicensePath $license_file -Verbose
    ```

### Installing Git

- [git-scm.com/download/win](https://git-scm.com/download/win)

```powershell
winget show --id Git.Git
winget install --id Git.Git -e --source winget
```


### Installing Open SSH Beta

- [github.com/PowerShell/Win32-OpenSSH](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)

The beta version of OpenSSH is part of the PowerShell project and often contains additional (essential) features missing from the stable version.

```powershell
winget search "openssh beta"
winget install "openssh beta"
winget uninstall "openssh beta"
```


### Installing Windows Terminal

- [github.com/microsoft/terminal](https://github.com/microsoft/terminal?tab=readme-ov-file#via-windows-package-manager-cli-aka-winget)

```powershell
winget install --id Microsoft.WindowsTerminal -e
```


### Install USBIPD

- [github.com/dorssel/usbipd-win](https://github.com/dorssel/usbipd-win)

This project is being developed to allow TCP/IP passthrough of USB devices to WSL2 and Hyper-V VM's on Windows.

```powershell
winget install --interactive --exact dorssel.usbipd-win
```


## User Accounts


### Active Directory

To do


### Local System

<https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/new-localuser?view=powershell-5.1>

1. Create a local user
```powershell
$Password = Read-Host -AsSecureString
New-LocalUser "User2" -Password $Password -FullName "Second User" -Description "Description of this account."
Name    Enabled  Description
----    -------  -----------
User2  True     Description of this account.
```

<https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/add-localgroupmember?view=powershell-5.1>

2. Add the local user to a group; add them to "Users" so they may login

Add a local user to the Users group
```powershell
Add-LocalGroupMember -Group "Users" -Member User2
```

Add a local user to the Administrators group (only do this if [UAC](#uac-prompt) is configured correctly and the account will only be used for administration)
```powershell
Add-LocalGroupMember -Group "Administrators" -Member User2
```

You can run commands as another user, you'll be prompted for a password similar to `sudo`:
```powershell
runas /user:User2 cmd.exe
runas /user:Hostname\DomainUser2 powershell.exe -ep bypass -nop -w hidden iex <payload>
runas /user:Domain\DomainUser2 powershell.exe -ep bypass -nop -w hidden iex <payload>
```

See [HackTricks - Credential User Impersonation](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/windows-local-privilege-escalation/access-tokens.md#credentials-user-impersonation) for more on this.


### Account Policy

To do


### Honey Accounts

References:

- [ippsec - Creating Webhooks in Slack for PowerShell](https://www.youtube.com/watch?v=1w0btuMAvZk)
- [ippsec - Send Notifications to Slack via Scheduled Task Event Filter](https://www.youtube.com/watch?v=J9owPmgmfvo)
- [Active Defense & Cyber Deception - Honey User](https://github.com/strandjs/IntroLabs/blob/master/IntroClassFiles/Tools/IntroClass/honeyuser/honeyuser.md)

How this works:

- Create a new user account that will never be used for production
- Create an alert to trigger on *any* attempt to login to this account
- Periodically login to this account and update it's password so it's not obvious it's a honey account when adversaries enumerate your AD environment

This will detect password spraying. A smaller organization would benefit from having this configured to use Slack or Discord for notifications, where a SIEM would be the better option on a larger scale.


## UAC Prompt

What the UAC prompt is and how / why it works:

<https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-account-control-behavior-of-the-elevation-prompt-for-standard-users#best-practices>

> Countermeasure
> Configure the User Account Control: Behavior of the elevation prompt for standard users to Automatically deny elevation requests. This setting requires the user to log on with an administrative account to run programs that require elevation of privilege. As a security best practice, standard users should not have knowledge of administrative passwords. However, if your users have both standard and administrator-level accounts, we recommend setting Prompt for credentials so that the users do not choose to always log on with their administrator accounts, and they shift their behavior to use the standard user account.

What the Secure Desktop is and how / why it works:

<https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation#best-practices>

> Enable the User Account Control: Switch to the secure desktop when prompting for elevation setting. The secure desktop helps protect against input and output spoofing by presenting the credentials dialog box in a protected section of memory that is accessible only by trusted system processes.

The secure desktop will obscure the entire desktop behind the prompt, where instead the 'insecure' prompt will leave all of the desktop windows visible.

You can see the difference by changing this value (0x1=enabled, 0x0=disabled) here:

```powershell
# Enabled
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\" -Name "PromptOnSecureDesktop" -Type DWord -Value "0x1"
# Disabled
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\" -Name "PromptOnSecureDesktop" -Type DWord -Value "0x0"
```

You can easily test if UAC is enabled for administrative actions by running the following:

```powershell
# https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs.md#uac-disabled
Start-Process powershell -Verb runAs "notepad.exe"
```

If you are not prompted to allow `notepad` to run, then UAC is not enabled.

### UAC Prompt for local users

See the following documentation for guidance:

- [Microsoft Docs, Open Specifications on all UAC settings](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gpsb/12867da0-2e4e-4a4f-9dc4-84a7f354c8d9)
- [Microsoft Docs, UAC Best Practices](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-account-control-behavior-of-the-elevation-prompt-for-standard-users#best-practices)
- [DevBlogs, UAC Settings](https://devblogs.microsoft.com/oldnewthing/20160816-00/?p=94105)
- [HackTricks, UAC Bypass](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs.md#uac)
- [FuzzySecurity, UAC Attacks](https://www.fuzzysecurity.com/tutorials/27.html)

**Summary**

If these are not set, you are potentially vulnerable to UAC bypass:

```powershell
# Check if UAC is active in Admin Approval Mode (value of 1) or inactive (value of 0), you will always want this active
# https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gpsb/958053ae-5397-4f96-977f-b7700ee461ec
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\" -Name "EnableLUA"
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\" -Name "EnableLUA" -Type DWord -Value "0x1"

# Ensure all UAC prompts happen via the secure desktop prompt
# https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gpsb/9ad50fd3-4d8d-4870-9f5b-978ce292b9d8
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\" -Name "PromptOnSecureDesktop -Type DWord -Value "0x1"

# Require credentials when running as Admin
# https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gpsb/341747f5-6b5d-4d30-85fc-fa1cc04038d4
Set-ItemProptery -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Type DWord -Value "0x1"

# Deny all elevation for standard users
# https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gpsb/15f4f7b3-d966-4ff4-8393-cb22ea1c3a63
Set-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorUser" -Type DWord -Value "0x0"
# Require credentials for standard users
Set-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorUser" -Type DWord -Value "0x1"
```

The recommended setting is to automatically deny elevation of privileges to standard users.

If a user regularly requires administrative access, it's recommended to instead prompt for admin credentials. This way administrative tasks are performed only when needed, and the account is otherwise running in the context of a standard user.

- `0x0` = Automatically deny (all UAC actions require logging in as admin, best)
- `0x1` = Prompt for credentials on the secure desktop (recommended if not 0x0)
- `0x2` = Prompt for credentials (Default)

This setting prevents standard users from performing administrative actions:

```powershell
Set-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorUser" -Type DWord -Value "0x0"
```

Alternatively, this allows the user to elevate privileges temporarily using the separate administrative account's credentials:

```powershell
Set-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorUser" -Type DWord -Value "0x1"
```

### UAC Prompt for local admins

- `0x0` = Elevate without prompting (perform admin actions automatically, dangerous)
- `0x1` = Prompt for credentials (username / password) on the secure desktop
- `0x2` = Prompt for interaction (y/n dialogue) on the secure desktop

This will prompt the administrator account for any valid administrator credentials via the secure desktop to take actions:

```powershell
Set-ItemProptery -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Type DWord -Value "0x1"
```

When logged in as an admin, this will prompt the user with a yes/no dialogue instead of credentials before performing actions:

```powershell
Set-ItemProptery -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Type DWord -Value "0x2"
```

## Network

Display general network information
```cmd
ipconfig
ipconfig /all
```

Flush the DNS cache
```cmd
ipconfig /flushdns
```

Show all active protocols and connections
```cmd
netstat -ano
```

Show the executable involved in creating all active protocols and connections
```cmd
netstat -abno    # Requires Administrative privileges
```

Display the arp table
```cmd
arp -a
```

Display the routing table
```cmd
route print
```

Additional tools for network visibility:

- <https://www.wireshark.org/>
- <https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview>


### Troubleshooting

Sometimes when trying to start a service on a specific port (`py -m http.server 8080 --bind <your-ip>`) you'll receive an error about lacking permissions even if you're running as administrator.

<https://stackoverflow.com/questions/10461257/an-attempt-was-made-to-access-a-socket-in-a-way-forbidden-by-its-access-permissi>

This is often caused by a socket already being used by another service. Use `netstat` to review and remediate this:

```powershell
NETSTAT.EXE -abno | Select-String "8080" -Context 2,2

    TCP    0.0.0.0:7680           0.0.0.0:0              LISTENING       7176
   Can not obtain ownership information
>   TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       2564
   [spoolsv.exe]
    TCP    0.0.0.0:8443           0.0.0.0:0              LISTENING       2564
    TCP    [::]:7680              [::]:0                 LISTENING       7176
   Can not obtain ownership information
>   TCP    [::]:8080              [::]:0                 LISTENING       2564
   [spoolsv.exe]
    TCP    [::]:8443              [::]:0                 LISTENING       2564


PS C:\> Stop-Process -Name spoolsv
```

In the above case, a meterpreter session had migrated to the `spoolsv.exe` process and was port forwarding traffic on tcp/8080.


### DNS

**DNS on Windows 11 Clients**

You can configure DNS over HTTPS for *all* interfaces with the following.

- "Settings" > "Network & internet"
- Choose "Wi-Fi" or "Ethernet", depending on your current connection
- Choose "Hardware properties" (you can also select the active network connection and under DNS settings choose `Change DNS settings for all networks`)
- Choose "Edit" under the block with "DNS server assignment"
- IPv4 > On
- Preferred DNS (can be Quad9 or Cloudflare, Windows has these templates built in now)
- DNS over HTTPS > On (automatic template)
- This will autofill the URL for DoH connections
- Ensure "Fallback to plaintext" > Off

When finished, you'll see `(Encrypted)` next to each DNS server entry.

You can fill out 2 IPv4 and 2 IPv6 DNS settings, and Windows will enforce this DNS configuration for every Wi-Fi or Ethernet network you connect to.

For an Active Directory client, you'll need at least *one* of the primary DNS servers to point to a domain-joined DNS server, with the fallback DNS being one of Cloudflare's or Quad9's public servers.


**DNS on Windows Server**

[The Quad9 documentation demonstrates the steps required to configure Windows DNS Server for DNS forwarding](https://docs.quad9.net/Setup_Guides/DNS_Forwarders/Windows_Server/). You could replicate those steps for Google (`8.8.8.8`) and Cloudflare (`1.1.1.1`).


### Firewall Rules

Open Windows Defender Firewall from an elevated PowerShell prompt with:

```powershell
wf.msc
```

The way Windows Defender Firewall works is when it's enabled, the default policies take precedence, followed by any specific rules as exceptions.

When it's disabled, *no* rules work.

This means if you want to deploy a highly customized rule set, backup, then wipe the entire default rule set, set your default policies, then add your own rules.

In other words:

- If the default inbound policy is set to block, you will need to create inbound rules.
- If the default inbound policy is set to allow, you will need to create block rules.

If you're an admin on a machine using a low privileged account for regular use, and need to review the firewall GUI without logging out, you can open an administrative PowerShell prompt and run `C:\Windows\System32\WF.msc`


Example baseline policy executed in `cmd.exe`:

```cmd
netsh advfirewall set allprofiles state on
netsh advfirewall set allprofiles firewallpolicy blockinboundalways,allowoutbound
netsh advfirewall set allprofiles logging droppedconnections enable
netsh advfirewall set allprofiles settings inboundusernotification enable
netsh advfirewall set allprofiles settings remotemanagement disable
netsh advfirewall set allprofiles logging maxfilesize 8000
netsh advfirewall set allprofiles logging filename %systemroot%\system32\LogFiles\Firewall\pfirewall.log

netsh advfirewall show currentprofile
```

Example baseline policy executed in `PowerShell`:

```powershell
# https://docs.microsoft.com/en-us/powershell/module/netsecurity/set-netfirewallprofile?view=windowsserver2022-ps
Set-NetFirewallProfile -All -Enabled True
Set-NetFirewallProfile -All -DefaultInboundAction Block
Set-NetFirewallProfile -All -AllowInboundRules True
Set-NetFirewallProfile -All -AllowUnicastResponseToMulticast False
Set-NetFirewallProfile -All -NotifyOnListen True # Windows displays a notification when blocking an application
Set-NetFirewallProfile -All -LogFileName "%systemroot%\system32\LogFiles\Firewall\pfirewall.log"
Set-NetFirewallProfile -All -LogMaxSizeKilobytes 8000
Set-NetFirewallProfile -All -LogAllowed False
Set-NetFirewallProfile -All -LogBlocked True
Set-NetConnectionProfile -NetworkCategory Public
```

*NOTE: these baselines are fairly close to the defaults, and are mainly for reference.*


### Managing Firewall Rules (PowerShell)

There are three different network profiles in Windows:

| Profile     | Description
| ----------- | -------------------------------------------------------------------------- |
| **Public**  | Considered untrusted, similar to being publicly accessible on the internet |
| **Domain**  | Considered trusted, typically a managed corporate network                  |
| **Private** | Considered trusted, a 'home' network                                       |


Get information about all, or a specific, network profile:
```powershell
Get-NetFirewallProfile
Get-NetFirewallProfile -Name Domain
Get-NetFirewallProfile -Name Private
Get-NetFirewallProfile -Name Public
```

Get information about all network interfaces:
```powershell
Get-NetConnectionProfile
```

Sometimes you need to change the profile of a specific network interface, here's how with PowerShell:
```powershell
Set-NetConnectionProfile -InterfaceAlias EthernetX -NetworkCategory Public
```

This is important to manually configure on multi-homed and domain joined systems. It's possible Windows will automatically and incorrectly assign Public / Private networking profiles to the wrong interfaces, exposing services to untrusted networks.

*NOTE: You can also review the active network profile per interface under Settings > Network & Internet > EthernetX Properties.*

Get general information about the firewall rules:
```powershell
Get-NetFirewallProfile -All
Get-NetFirewallRule -All
Get-NetFirewallRule -Direction Inbound -Enabled True -Action Allow
Get-NetFirewallPortFilter -All | Where-Object -Property LocalPort -eq 22
```

Turn off the firewall for all profiles:
```powershell
Set-NetFirewallProfile -Enabled False
```

Enable the firewall on the Public profile:
```powershell
Set-NetFirewallProfile -Enabled True -Name Public
```

Add inbound rules for RDP and SSH:
```powershell
New-NetFirewallRule -DisplayName "Remote Desktop" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow -Enabled True
New-NetFirewallRule -DisplayName "SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow -Enabled True
```

Delete inbound rules for RDP and SSH:
```powershell
Get-NetFirewallRule -DisplayName "Remote Desktop" | Remove-NetFirewallRule
Get-NetFirewallPortFilter -All | where { $_.LocalPort -eq 22 } | Get-NetFirewallRule | where -Property displayname -Contains "SSH" | Remove-NetFirewallRule
```

Working with firewall rules in PowerShell on Windows is a bit strange because `-NetFirewallRule` results do not contain the same information as `-NetFirewallPortFilter` to parse. This means you'll need to use both (like the example above) and confirm the `DisplayName` or the more unique `Name` of the rule before deleting it, as multiple rules can exist for the same port. If you're not specific, and simply pipe all results out to `Remove-NetFirewallRule` you can end up unintentionally deleting additional rules.

You can be more specific to delete anything matching precise criteria like this:
```powershell
Get-NetFirewallPortFilter -All | where { $_.LocalPort -eq 5353 } | Get-NetFirewallRule | where { $_.Direction -eq "Inbound" -and $_.Action -eq "Allow" -and $_.Enabled -eq "True" -and $_.Profile -eq "Public" -and $_.DisplayName -match "mDNS"}
```

On a default Windows 10 install, this should match only a single rule. Append `| Remove-NetFirewallRule` to the above command to delete it.

Look for rules with specific ports:
```powershell
Get-NetFirewallPortFilter -All | where { $_.LocalPort -eq 20 -or $_.LocalPort -eq 445 }
```

Look for rules with specific ports, and print their related complete rule entry to parse further:
```powershell
Get-NetFirewallPortFilter -All | where { $_.LocalPort -eq 20 -or $_.LocalPort -eq 445 } | Get-NetFirewallRule
```

Look for rules within a range of ports:
```powershell
Get-NetFirewallPortFilter -All | where { $_.LocalPort -ge 20 -and $_.LocalPort -le 445 }
```

Load all active / allowed inbound port rules into the variable '$inboundrules':
```powershell
$inboundrules = Get-NetFirewallRule * | where { ($_.Direction -like "Inbound" -and $_.Action -like "Allow" -and $_.Enabled -like "True") }
```

List firewall rule name + display name for all active / allowed inbound ports:
```powershell
$inboundrules | Select-Object -Property Name,DisplayName
```

List all unique active / allowed  inbound ports:
```powershell
$inboundrules | Get-NetFirewallPortFilter | Select-Object -Property LocalPort -Unique
```

List DisplayName + rule information

<https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/out-string?view=powershell-7.2>

<https://docs.microsoft.com/en-us/dotnet/api/system.string.trim?view=net-6.0#system-string-trim>

```powershell
Get-NetFirewallRule -Direction "Inbound" -Enabled "true" -Action "Allow" | ForEach-Object {
    ($_ | Select-Object -Property DisplayName | fl | Out-String).trim()
    ($_ | Select-Object -Property Profile | fl | Out-String).trim()
     $_ | Get-NetFirewallPortFilter
}
```

List DisplayName + port information for all inbound rules currently enabled and allowed:

```powershell
Get-NetFirewallRule -Direction "Inbound" -Enabled "true" -Action "Allow" | ForEach-Object {
    echo ""
    ($_ | Select-Object -Property DisplayName | fl | Out-String).trim()
    ($_ | Select-Object -Property Profile | fl | Out-String).trim()
    ($_ | Get-NetFirewallPortFilter | Select-Object -Property Protocol,LocalPort | fl | Out-String).trim()
}
```

Example result of the previous command:

```
...
DisplayName : Microsoft Edge (mDNS-In)
Profile : Any
Protocol  : UDP
LocalPort : 5353

DisplayName : Cortana
Profile : Domain, Private, Public
Protocol  : Any
LocalPort : Any
...
```

Knowing the format of the above results, you can do a context search by appending `| Select-String "<port>" -Context 3,0`, which will print *only* the results matching `<port>`. For example to get all enabled and allowed inbound rules matching port 5353:
```powershell
Get-NetFirewallRule -Direction "Inbound" -Enabled "true" -Action "Allow" | ForEach-Object {
    echo ""
    ($_ | Select-Object -Property DisplayName | fl | Out-String).trim()
    ($_ | Select-Object -Property Profile | fl | Out-String).trim()
    ($_ | Get-NetFirewallPortFilter | Select-Object -Property Protocol,LocalPort | fl | Out-String).trim()
} | Select-String "5353" -Context 3,0
```

You can take any commands that have extensive output and use a context search to obtain the rule name based on the port with something like `Select-String "<port>" -Context <lines-before>,<lines-after>`.

Get all active rules permitting inbound traffic, that are lacking a description (good indicator of third party or possibly malicious rules):
```powershell
Get-NetFirewallRule | where { $_.Enabled -eq "True" -and $_.Direction -eq "Inbound" -and $_.Action -eq "Allow" -and $_.Description -eq $nul }
```

Get all active inbound firewall rules and disable them:
```powershell
Get-NetFirewallRule | Where-Object -Property Direction -eq Inbound | Where-Object -Property Enabled -eq True | Set-NetFirewallRule -Enabled -eq False
```

Disable all rule names matching regex "Cast to Device*":
```powershell
Set-NetFirewallRule -DisplayName "Cast to Device*" -Enabled False
```

Set rule names matching regex "mDNS*" or "LLMNR*" to block:
```powershell
Set-NetFirewallRule -DisplayName "mDNS*" | Set-NetFirewallRule -Action "Block"
Set-NetFirewallRule -DisplayName "*LLMNR*" | Set-NetFirewallRule -Action "Block"
```


### Managing Firewall Rules (cmd.exe)

cmd.exe / netsh.exe basics:
```cmd
netsh advfirewall show all
netsh advfirewall show allprofiles
netsh advfirewall show currentprofile
netsh advfirewall set domainprofile state on
netsh advfirewall set privateprofile state off
netsh advfirewall set allprofiles state off
netsh advfirewall firewall show rule all
netsh advfirewall firewall show rule all dir=in
netsh advfirewall firewall show rule name="<rule-name>"
```

Limit output by listing only all enabled, inbound rules, rule name, profile, and local port:
```cmd
netsh advfirewall firewall show rule name=all dir=in | findstr /BR /C:"Rule Name" /C:"-" /C:"Enabled" /C:"Profiles" /C:"LocalPort" /C:"Action:.*Allow" /C:"^$"

# Example output:

Rule Name:                            Remote Event Log Management (NP-In)
----------------------------------------------------------------------
Enabled:                              No
Profiles:                             Domain
LocalPort:                            445
Action:                               Allow
```

This will do the equivalent of `-B` / `-A` in `grep` to search for rules based on LocalPort, but PowerShell is required:
```cmd
netsh advfirewall firewall show rule name=all dir=in | Select-String "LocalPort:.*443" -Context 10,10
netsh advfirewall firewall show rule name=all dir=in | Select-String "Rule Name:.*SSH" -Context 10,10
```

Add inbound rules for RDP and SSH:
```cmd
netsh advfirewall firewall add rule name="Remote Desktop" dir=in protocol=TCP localport=3389 action=allow
netsh advfirewall firewall add rule name="SSH" dir=in protocol=TCP localport=22 action=allow
```

Add outbound egress rules:
```cmd
netsh advfirewall firewall add rule name="Egress 10.0.0.0/8" dir=out protocol=any action=block remoteip=10.0.0.0/8
netsh advfirewall firewall add rule name="Default Gateway" dir=out protocol=any remoteip=<gateway-ip> interfacetype=lan action=allow
netsh advfirewall firewall add rule name="Egress fc00::/7" dir=out protocol=any action=block remoteip=fc00::/7
```

Delete a rule called "Remote Desktop"
```cmd
netsh advfirewall firewall delete rule name="Remote Desktop"
```

Delete all rules for local port tcp/80:
```
netsh advfirewall firewall delete rule name=all protocol=tcp localport=80
```

Reset Firewall (To Defaults)
```cmd
netsh advfirewall reset
```


### Netsh Port Proxy

<https://www.sans.org/blog/pen-test-poster-white-board-cmd-exe-c-netsh-interface/>

Windows `Net Shell` command

- Using 0.0.0.0 as the listen address makes the portproxy bind to all interfaces
- Using v4tov6 allows bidirectional IPv4 and IPv6 usage

```cmd
netsh interface portproxy show all
netsh interface portproxy show v4tov4
netsh interface portproxy show v4tov6
netsh interface portproxy show v6tov4
netsh interface portproxy show v6tov6

netsh interface portproxy add v4tov4 listenaddress=<local-addr> listenport=<local-port> connectaddress=<dest-addr> connectport=<dest-port>
netsh interface portproxy add v4tov4 listenaddress=<jump-box-addr> listenport=<jump-box-port> connectaddress=<attacker-addr> connectport=<attacker-web-server-port>
netsh interface portproxy add v4tov4 listenport=8443 listenaddress=10.0.0.15 connectport=443 connectaddress=172.16.1.28
netsh interface portproxy add v4tov4 listenport=443 listenaddress=0.0.0.0 connectport=443 connectaddress=172.16.1.28
```

Forwards traffic hitting victim's / jump box's `listenaddress:listenport` to attacker's `connectaddress:connectport`


### blockinboundalways / -AllowInboundRules False

To drop all inbound connections even if Windows has a default allow rule for the service:

```powershell
#cmd.exe
netsh advfirewall set allprofiles firewallpolicy blockinboundalways,allowoutbound

#powershell
Set-NetFirewallProfile -AllowInboundRules False
```

- On domain joined workstations, this will not disrupt connections to server file shares and stops lateral movement between workstations
- Workstation typically should not need to talk to each other, with the server being the central point of authentication
- On personal, non domain joined workstations this should be the default setting, and an absolute must for travel / using untrusted LAN and WiFi networks

**Keep in mind on cloud instances of Windows in Azure / AWS / GCP this will likely lock you out of the machine**

To enable this setting from within the GUI:

- `Network & Internet Settings > Status > Windows Firewall`
- For all three, `Domain`, `Public`, `Private`:
- Set to `On`
- Check `Blocks all incoming connections, including those on the list of allowed apps.`

To turn off this setting via the GUI:
- Uncheck `Blocks all incoming connections, including those on the list of allowed apps.`


### LLMNR

<https://www.blackhillsinfosec.com/how-to-disable-llmnr-why-you-want-to/>

Disable via GPO:

- Group Policy Management
- Forest: `<DOMAIN>.local` > Domains > `<DOMAIN>.local` > `Right-Click`
- Create a GPO in this domain, and Link it here...
- Name it "LLMNR" or something meaningful, click OK
- Computer Configuration > Administrative Templates > Network > DNS Client
- Turn Off Multicast Name Resolution > Enabled
- `Right-Click` the policy name under `<DOMAIN>.local` and choose `Enforced`

*You'll need to restart the endpoints, or refresh the group policy on each endpoint manually to ensure these changes take effect immediately*

Update Group Policy via PowerShell:

```powershell
Get-ADComputer -filter * | foreach{ Invoke-GPUpdate -computer $_.name -RandomDelayInMinutes 0 -force}
```

Update Group Policy via cmd.exe:

```cmd
gpupdate
```

Disable LLMNR via PowerShell:

```powershell
If (!(Test-Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient")) {
	New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Force | Out-Null
}
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient" -Name "EnableMulticast" -Type DWord -Value 0
```

Disable LLMNR via cmd.exe:

```cmd
REG ADD  “HKLM\Software\policies\Microsoft\Windows NT\DNSClient”
REG ADD  “HKLM\Software\policies\Microsoft\Windows NT\DNSClient” /v ” EnableMulticast” /t REG_DWORD /d “0” /f
```

### Mitigate IPv6 MitM Attacks

<https://academy.tcm-sec.com/p/practical-ethical-hacking-the-complete-course>

<https://blog.fox-it.com/2018/01/11/mitm6-compromising-ipv4-networks-via-ipv6/>

<https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/>

There are three predefined firewall rules to change from allow to block.

This walks through setting these rules as a GPO on a Domain Controller, but you can create the same rules manually on endpoints via Windows Defender Firewall.

Test these rules in an environment with ADCS configured by enabling them, updating Group Policy on each machine, then rebooting endpoints and signing in as a Domain Admin on any of them with mitm6 running. The attack(s) will no longer fire when these rules are in place.

Here are the commands to run on the attacker machine:

```bash
# In one terminal window:
sudo /home/kali/.local/bin/mitm6 -i eth0 -v -d TESTDOMAIN.local
# In another:
impacket-ntlmrelayx -6 -t ldaps://<dc-ip> -wh fakewpad.testdomain.local -l loot
```

- Group Policy Management
- Forest: `<DOMAIN>.local` > Domains > `<DOMAIN>.local` > `Right-Click`
- Create a GPO in this domain, and Link it here...
- Name it "IPv6" or something meaningful, click OK
Computer Configuration > Windows Settings > Security Settings > Windows Defender Firewall with Advanced Security > Windows Defender Firewall with Advanced Security > [Inbound|Outbound] Rules
- Create new custom rules for all three, there are usually none defined in Group Policy Management on the DC when defining a new GPO.

- (Inbound) Core Networking - Dynamic Host Configuration Protocol for IPv6(DHCPV6-IN)
	* Name: Block - (DHCPV6-In)
	* Protocol type: UDP
	* Protocol number: 17
	* Local Port: Specific Ports, 546
	* Remote Port: Specific Ports, 547
	* This program: %SystemRoot%\system32\svchost.exe

- (Inbound) Core Networking - Router Advertisement (ICMPv6-IN)
	* Name: Block - Router Advertisement (ICMPv6-In)
	* Protocol type: ICMPv6
	* Protocol number: 58
	* Local port: All Ports
	* Remote port: All Ports
	* Specific ICMP types: [x] Router Advertisement
		- ICMP Type: 0
		- ICMP Code: Any
	* This program: System

- (Outbound) Core Networking - Dynamic Host Configuration Protocol for IPv6 (DHCPV6-Out)
	* Name: Block - (DHCPV6-Out)
	* Protocol type: UDP
	* Protocol number: 17
	* Local Port: Specific Ports, 546
	* Remote Port: Specific Ports, 547
	* This program: %SystemRoot%\system32\svchost.exe

- `Right-Click` the policy name under `<DOMAIN>.local` and choose `Enforced`

*You'll need to restart the endpoints or refresh the group policy on each endpoint manually to ensure these changes take effect immediately*

Update Group Policy via PowerShell:

```powershell
Get-ADComputer -filter * | foreach{ Invoke-GPUpdate -computer $_.name -RandomDelayInMinutes 0 -force}
```

Update Group Policy via cmd.exe:

```cmd
gpupdate
```

### Enable SMB Signing

<https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/overview-server-message-block-signing>

Default for Workstations = 0
Default for Server = 1

Enable SMB Signing via PowerShell:

```powershell
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Services\LanManWorkstation\Parameters" -Name "RequireSecuritySignature" -Type DWord -Value 1
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Services\LanManServer\Parameters" -Name "RequireSecuritySignature" -Type DWord -Value 1
```

Enable SMB Signing on a Domain Controller via a GPO:

- Group Policy Management
- Forest: `<DOMAIN>.local` > Domains > `<DOMAIN>.local` > `Right-Click`
- Create a GPO in this domain, and Link it here...
- Name it "SMB Signing" or something meaningful, click OK
- Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options
- Microsoft network server: Digitally sign communications (always) > Enable
- Microsoft network client:  Digitally sign communications (always) > Enable
- `Right-Click` the policy name under `<DOMAIN>.local` and choose `Enforced`

*You'll need to restart the endpoints or refresh the group policy on each endpoint manually to ensure these changes take effect immediately*

Update Group Policy via PowerShell:

```powershell
Get-ADComputer -filter * | foreach{ Invoke-GPUpdate -computer $_.name -RandomDelayInMinutes 0 -force}
```

Update Group Policy via cmd.exe:

```cmd
gpupdate
```

## Filesystem

How to do the equivalent of `chmod` operations in Windows.

### takeown.exe

Change ownership recursively of a file(s) or folder(s) to the current user:

```powershell
takeown.exe /F .\ExampleDir\ /R
```

Change ownership recursively of a file(s) or folder(s) to the **Administrators group**:

```powershell
takeown.exe /F .\ExampleDir\ /A /R
```

Change ownership back to a standard user (where $HOSTNAME is either local pc name or domain name, and user is a local or domain user):

```powershell
takeown.exe /S $HOSTNAME /U $USER /F .\ExampleDir\
```

### icacls.exe

Remove all permissions on a file or folder:

```powershell
icacls.exe .\ExampleDir /inheritance:r
```

Reset to default permissions:

```powershell
icacls.exe .\ExampleDir\ /reset
```


**Folder Permissions**

Grant read-only access to a **folder** for user alice:

```powershell
icacls.exe .\ExampleDir\ /grant alice:"(CI)(OI)RX"
```

Remove *granted* (note the `:g`) access to a **folder* for user alice:

```powershell
icacls.exe .\ExampleDir\ /remove:g alice
```

Permit read-only access to a **folder** for the builtin group "everyone" (removes any inherit or current write or modify permissions with `/inheritance:r`):

- SID `*S-1-1-0` means `everyone`
- Similar to `chmod a=rX -R ./ExampleFolder` in Linux

```powershell
icacls.exe .\ExampleDir\ /inheritance:r
icacls.exe .\ExampleDir\ /grant *S-1-1-0:"(CI)(OI)RX"
```

The purpose of `CI` and `OI` is to allow the "synchronize" permission, which allows directory traversal and if missing, denies it even if RX permission is granted

Configure a folder to be only modifiable by administrators, read and execute by eveyrone:

- Similar to `chmod a=rX -R ./ExampleFolder; chmod o=rwX -R ./ExampleFolder; chown root:root ./ExmapleFolder` on Linux

```powershell
icacls.exe C:\Tools /inheritance:r
icacls.exe C:\Tools /grant *S-1-1-0:"(CI)(OI)RX"
icacls.exe C:\Tools /grant SYSTEM:"(CI)(OI)(F)"
icacls.exe C:\Tools /grant BUILTIN\Administrators:"(CI)(OI)(F)"
```

Example output:

```powershell
PS> icacls.exe C:\Tools\*
C:\Tools Everyone:(OI)(CI)(RX)
         NT AUTHORITY\SYSTEM:(OI)(CI)(F)
         BUILTIN\Administrators:(OI)(CI)(F)
```


**File Permissions**

Grant read-only access to a **file** for user alice. This does not require `(CI)(OI)`:

```powershell
icacls.exe .\ExampleFile /grant alice:"RX"
```

Remove *granted* (note the `:g`) access to a **file** for user alice:

```powershell
icacls.exe .\ExampleFile /remove:g alice
```

Permit read-only + execute for the `everyone` builtin group, to a single file. This does not require `(CI)(OI)`:

```powershell
icacle.exe .\example.md /inheritance:r /grant *S-1-1-0:"RX"
```

Limit full access to only SYSTEM and BUILTIN\Administrators:

```powershell
icacls.exe .\administrators_authorized_keys /inheritance:r
icacls.exe .\administrators_authorized_keys /grant SYSTEM:"(F)"
icacls.exe .\administrators_authorized_keys /grant BUILTIN\Administrators:"(F)"
```

Example output:

```powershell
PS> icacls.exe .\administrators_authorized_keys
administrators_authorized_keys NT AUTHORITY\SYSTEM:(F)
                               BUILTIN\Administrators:(F)
```


### .NET Access Control Methods

This section breaks down using the PowerShell `Set/Get-Acl` cmdlets and the .NET methods underpinning them. The idea is to mirror what `takeown.exe` and `icacls.exe` or `chmod` plus `chown` can do, in an easy to understand way.

Take the ACL data of one filesystem object and apply it to another with the following:

```powershell
$SourceObject = "C:\Users\Administrator\Documents"
$NewAcl = Get-Acl -Path $SourceObject
$DestObject = "C:\Tools"
Set-Acl -Path $DestObject -AclObject $NewAcl
```

To use this like `icacls.exe` or `chmod` we'll need to reference some examples:

- [`Set-Acl`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-acl?view=powershell-7.3)
- [`FileSystemAccessRule($identity, $fileSystemRights, $type)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemaccessrule.-ctor?view=net-7.0#system-security-accesscontrol-filesystemaccessrule-ctor(system-security-principal-identityreference-system-security-accesscontrol-filesystemrights-system-security-accesscontrol-accesscontroltype))

First, to learn the namespace / class path to access .NET classes, see the top **Definition** section of each class in the documentation:

> FileSecurity Class
>
> Namespace: System.Security.AccessControl

Access it with: `New-Object -TypeName System.Security.AccessControl.FileSecurity -ArgumentList ...<SNIP>`

*Remember: Methods apply actions, where classes define the objects or "things" to work with through actions, roughly speaking.*

Looking at the [five parameters](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemaccessrule.-ctor?view=net-8.0#system-security-accesscontrol-filesystemaccessrule-ctor(system-security-principal-identityreference-system-security-accesscontrol-filesystemrights-system-security-accesscontrol-inheritanceflags-system-security-accesscontrol-propagationflags-system-security-accesscontrol-accesscontroltype)) to `FileSystemAccessRule($identity, $fileSystemRights, $inheritanceFlags, $propagationFlags, $type)`, shows how we can start to write the code block.


**$identity**

[`$identity`](https://learn.microsoft.com/en-us/dotnet/api/system.security.principal.identityreference?view=net-7.0) is a reference to a user account through the [NTAccount class](https://learn.microsoft.com/en-us/dotnet/api/system.security.principal.ntaccount?view=net-7.0), built with either a `$userAccount` string, or both a `$domainName` and `$userAccount` string.


**$fileSystemRights**

For the [fields available to `$fileSystemRights`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemrights?view=net-8.0#definition), these include but aren't limited to:

- FullControl
- Read
- Write
- Modify
- Synchronize
- Traverse

You can specify more than one `$fileSystemRights` with a comma separated list. So we can create a FileSystemAccessRule construct like this:

```powershell
$AccessRule = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"ReadAndExecute,Modify","Allow"
```

*For the sake of readability, many examples below use one per line.*


**$inheritanceFlags, $propagationFlags**

[`$inheritanceFlags`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.inheritanceflags?view=net-8.0) and [`$propagationFlags`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.propagationflags?view=net-8.0) define how existing child objects are modified based on the parent ACL.

- `$inheritanceFlags` of `"ContainerInherit,ObjectInherit"` is the same as `"(CI)(OI)"` from icacls.exe, meaning the changes are recursive and persistent
- `$propagationFlags` should be set to `"None"` if the "root" object removed existing inheritance from its parent object and has its own ACL's protected


**$type**

[`$type`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.accesscontroltype?view=net-7.0) is either `Allow` (0) or `Deny` (1).


**Inheritance**

Of all the [methods under the object security class](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.objectsecurity?view=net-8.0#methods), [`SetAccessRuleProtection`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.objectsecurity.setaccessruleprotection?view=net-8.0#system-security-accesscontrol-objectsecurity-setaccessruleprotection(system-boolean-system-boolean)) appears to deal directly with inheritance. Inheritance complicates locking down ACL's in some cases. This is a way to work with that.


**Essential Methods**

Now for reference, lets list all the relevant methods under the `FileSystemSecurity` and `ObjectSecurity` class in a way that's more readable and understandable.

- [`SetAccessRule($rule)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemsecurity.setaccessrule?view=net-7.0) Where `$rule` is an ACL object built from `FileSystemAccessRule()` to apply
- [`RemoveAccessRule($rule)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemsecurity.removeaccessrule?view=net-7.0) Where `$rule` is an ACL object built from `FileSystemAccessRule()` to remove
- [`PurgeAccessRules($identity)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.objectsecurity.purgeaccessrules?view=net-7.0) Which purges all of `$identity`'s access to an object
- [`SetOwner($identity)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.objectsecurity.setowner?view=net-7.0) Sets the owner to `$identity`
- [`SetGroup($identity)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.objectsecurity.setgroup?view=net-7.0) Sets the group to `$identity`
- [`SetAccessRuleProtection($true, $false)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.objectsecurity.setaccessruleprotection?view=net-8.0) Protects our defined rules from inheritance, and removes existing inheritance
- [`NTAccount($domainName, $accountName)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.principal.ntaccount.-ctor?view=net-8.0) Is how you construct an `$identity` object, `$domainName` is optional

Finally, when do you use Set/Add/Remove/Purge?

- Use `SetAccessRule()` to overwrite, and replace a specifc `$identity`'s ACL for an object
- Use `AddAccessRule()` to build on an `$identity`'s existing ACL for an object, like "set" the first rule, then "add" additional rules
- Use `RemoveAccessRule()` to remove a `$rule` from an `$identity`'s ACL, like take away FullControl, but leave all other existing rules
- Use `PurgeAccessRules()` to completely remove any **defined** `$rule`s for an `$identity`, **this does not prevent access via inheritance**

Next, some practical examples of how to use these. Each of them is copy / paste ready to use, just change the top variables.


**Reset ACL to Default Inheritance**

- This effectively wipes any modifications
- Meaning the filesystem object will default to the ACL's it's going to inherit based on where it exists in the filesystem
- This uses the [first constructor example](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesecurity.-ctor?view=net-8.0#system-security-accesscontrol-filesecurity-ctor) in the [FileSecurity class](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesecurity?view=net-8.0#constructors) to create an empty FileSecurity object, and apply it to the `$FilePath`

```powershell
$FilePath = "test.txt"
$EmptyAcl = New-Object -TypeName System.Security.AccessControl.FileSecurity -ArgumentList $null
Set-Acl -Path $FilePath -AclObject $EmptyAcl
Get-Acl -Path $FilePath | fl
```


**Change Ownership**

- Gives "User2" ownership of `$FilePath`
- Just change the value of `$UserAccount` to any user to change ownership
- Builds a `System.Security.Principal.NTAccount` object for `$identity`
- Uses [`SetOwner($identity)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.objectsecurity.setowner?view=net-7.0)

```powershell
$FilePath = "test.txt"
$UserAccount = "User2"
$NewAcl = Get-Acl -Path $FilePath
$NewOwner = New-Object -TypeName System.Security.Principal.NTAccount -ArgumentList $UserAccount
$NewAcl.SetOwner($NewOwner)
Set-Acl -Path $FilePath -AclObject $NewAcl
Get-Acl -Path $FilePath | fl
```


**Set Access**

- This *adds* User2 to the ACL, with the FullControl right over `$FilePath`
- In this case User2 had no *previous* ACL rules defined or access to `$FilePath`
- Uses [`SetAccessRule($rule)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemsecurity.setaccessrule?view=net-7.0)

```powershell
$FilePath = "test.txt"
$UserName = "User2"
$NewAcl = Get-Acl -Path $FilePath
$AccessRule = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"FullControl","Allow"
$NewAcl.SetAccessRule($AccessRule)
Set-Acl -Path $FilePath -AclObject $NewAcl
Get-Acl -Path $FilePath | fl
```


**Add Access**

- If an `$identity` alread has a defined ACL for the object, add another `$fileSystemRights` rule definition to it
- Similar to `SetAccessRule()`, but appends the rule to existing access
- Uses [`AddAccessRule($rule)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemsecurity.addaccessrule?view=net-8.0)

You could also use this in addition to `SetAccessRule()` when building specific rules for an `$identity` even when creating new rules. Otherwise each invocation of `SetAccessRule()` overwrites the previous list for the same `$identity`. The example below demonstrates this.

```powershell
$FilePath = "C:\Tools"
$UserName = "User2"
$NewAcl = Get-Acl -Path $FilePath
$AccessRule1 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"ReadAndExecute","Allow"
$AccessRule2 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"CreateFiles","Allow"
$AccessRule3 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"CreateDirectories","Allow"
$AccessRule4 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"Synchronize","Allow"
$NewAcl.SetAccessRule($AccessRule1)
$NewAcl.AddAccessRule($AccessRule2)
$NewAcl.AddAccessRule($AccessRule3)
$NewAcl.AddAccessRule($AccessRule4)
Set-Acl -Path $FilePath -AclObject $NewAcl
Get-Acl -Path $FilePath | fl
```


**Remove Access**

- Same as `AddAccessRule($rule)`, only it removes an existing `$fileSystemRights` rule definition
- This can be one rule, or any number of rules defined for any `$identity`

```powershell
$FilePath = "C:\Tools"
$UserName = "User2"
$NewAcl = Get-Acl -Path $FilePath
$AccessRule1 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"AppendData","Allow"
$AccessRule2 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"CreateFiles","Allow"
$NewAcl.RemoveAccessRule($AccessRule1)
$NewAcl.RemoveAccessRule($AccessRule2)
Set-Acl -Path $FilePath -AclObject $NewAcl
Get-Acl -Path $FilePath | fl
```


**Purge Access / Remove All**

- This revokes User2's defined ACL's to the file. **User2 may still access the file through inheritance, however**.
- Uses [`PurgeAccessRules($identity)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.objectsecurity.purgeaccessrules?view=net-7.0)
- [`RemoveAccessRuleAll($rule)`](https://learn.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemsecurity.removeaccessruleall?view=net-8.0) appears to do the same thing, just specify one existing rule and all will be wiped for that `$identity`

```powershell
$FilePath = "test.txt"
$UserAccount = "User2"
$NewAcl = Get-Acl -Path $FilePath
$RemoveAccount = New-Object -TypeName System.Security.Principal.NTAccount -ArgumentList $UserAccount
$NewAcl.PurgeAccessRules($RemoveAccount)
Set-Acl -Path $FilePath -AclObject $NewAcl
Get-Acl -Path $FilePath | fl
```


**Admin RWX, Users RX**

This is effectively the `/usr/bin` permissions of 755 where only elevated accounts can modify content, but everyone can read and execute it.

- Resets the ACL for the path **recursively**, and applies the 0755 style permissions to it
- The new ACL must be applied to all existing files recursively
- Just because an Administrator makes a directory does NOT mean standard users cannot modify it
- You must disable inheritance, and set explicit permissions for such folders
- Owner  : BUILTIN\Administrators
- Access : BUILTIN\Administrators Allow  FullControl
- Access : NT AUTHORITY\SYSTEM Allow  FullControl
- Access : Everyone Allow  ReadAndExecute, Synchronize

```powershell
$FilePath = "C:\Tools"
$UserName = "BUILTIN\Administrators"
# Reset the ACL
$EmptyAcl = New-Object -TypeName System.Security.AccessControl.FileSecurity -ArgumentList $null
Set-Acl -Path $FilePath -AclObject $EmptyAcl
foreach ($item in (gci -Recurse -Force $FilePath)) {
	Set-Acl -Path $FilePath -AclObject $EmptyAcl
}
# Construct the new ACL
$NewAcl = Get-Acl -Path $FilePath
$AccessRule1 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"FullControl","ContainerInherit,ObjectInherit","None","Allow"
$AccessRule2 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList "NT AUTHORITY\SYSTEM","FullControl","ContainerInherit,ObjectInherit","None","Allow"
$AccessRule3 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList "Everyone","ReadAndExecute,Synchronize","ContainerInherit,ObjectInherit","None","Allow"
$NewOwner = New-Object -TypeName System.Security.Principal.NTAccount -ArgumentList "$UserName"
$NewAcl.SetAccessRule($AccessRule1)
$NewAcl.SetAccessRule($AccessRule2)
$NewAcl.SetAccessRule($AccessRule3)
$NewAcl.SetAccessRuleProtection($true, $false)
$NewAcl.SetOwner($NewOwner)
Set-Acl -Path $FilePath -AclObject $NewAcl
foreach ($item in (gci -Recurse -Force $FilePath)) {
	Set-Acl -Path $FilePath -AclObject $NewAcl
}
Get-Acl -Path $FilePath | fl
```


**Admin Only**

Creates a file / folder permission structure where only Admins can traverse and modify the content.

- Like before, you must apply the constructed ACL object to all existing files recursively

```powershell
$FilePath = "C:\Tools"
$UserName = "BUILTIN\Administrators"
# Reset the ACL recursively
$EmptyAcl = New-Object -TypeName System.Security.AccessControl.FileSecurity -ArgumentList $null
Set-Acl -Path $FilePath -AclObject $EmptyAcl
foreach ($item in (gci -Recurse -Force $FilePath)) {
	Set-Acl -Path $FilePath -AclObject $EmptyAcl
}
# Construct the new ACL, apply it recursively
$NewAcl = Get-Acl -Path $FilePath
$AccessRule1 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"FullControl","ContainerInherit,ObjectInherit","None","Allow"
$AccessRule2 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList "NT AUTHORITY\SYSTEM","FullControl","ContainerInherit,ObjectInherit","None","Allow"
$NewOwner = New-Object -TypeName System.Security.Principal.NTAccount -ArgumentList "$UserName"
$NewAcl.SetAccessRule($AccessRule1)
$NewAcl.SetAccessRule($AccessRule2)
$NewAcl.SetAccessRuleProtection($true, $false)
$NewAcl.SetOwner($NewOwner)
Set-Acl -Path $FilePath -AclObject $NewAcl
foreach ($item in (gci -Recurse -Force $FilePath)) {
	Set-Acl -Path $FilePath -AclObject $NewAcl
}
Get-Acl -Path $FilePath | fl
```


**Set User's authorized_keys Permissions**

Three PowerShell utilities exist in the PowerShell/openssh-portable repo addressing this issue:

- [OpenSSHUtils.psm1](https://github.com/PowerShell/openssh-portable/blob/latestw_all/contrib/win32/openssh/OpenSSHUtils.psm1)
- [FixHostFilePermissions.ps1](https://github.com/PowerShell/openssh-portable/blob/latestw_all/contrib/win32/openssh/FixHostFilePermissions.ps1)
- [FixUserFilePermissions.ps1](https://github.com/PowerShell/openssh-portable/blob/latestw_all/contrib/win32/openssh/FixUserFilePermissions.ps1)

This is a simplified version of what those achieve, to help you understand how to do this yourself.

- Wipes out all existing ACLs and removes inheritance
- Allows `$env:USERNAME` (and NT AUTHORITY\SYSTEM + BUILTIN\Administrators) FullControl over `authorized_keys`

```powershell
$UserName = $env:USERNAME # Replace this with a defined user
$FilePath = "C:\Users\$UserName\.ssh\authorized_keys"
# Reset the ACL
$EmptyAcl = New-Object -TypeName System.Security.AccessControl.FileSecurity -ArgumentList $null
Set-Acl -Path $FilePath -AclObject $EmptyAcl
# Construct the new ACL
$NewAcl = Get-Acl -Path $FilePath
$AccessRule1 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $UserName,"FullControl","Allow"
$AccessRule2 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList "NT AUTHORITY\SYSTEM","FullControl","Allow"
$AccessRule3 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList "BUILTIN\Administrators","FullControl","Allow"
$NewOwner = New-Object -TypeName System.Security.Principal.NTAccount -ArgumentList "$UserName"
$NewAcl.SetAccessRule($AccessRule1)
$NewAcl.SetAccessRule($AccessRule2)
$NewAcl.SetAccessRule($AccessRule3)
$NewAcl.SetAccessRuleProtection($true, $false)
$NewAcl.SetOwner($NewOwner)
Set-Acl -Path $FilePath -AclObject $NewAcl
Get-Acl -Path $FilePath | fl
```


**Set administrators_authorized_keys Permissions**

Same as the authorized_keys file permissions for a standard user, except NT AUTHORITY\SYSTEM and BUILTIN\Administrators are the only `$identity`s that can access the file.

```powershell
$FilePath = "C:\ProgramData\ssh\administrators_authorized_keys"
# Reset the ACL
$EmptyAcl = New-Object -TypeName System.Security.AccessControl.FileSecurity -ArgumentList $null
Set-Acl -Path $FilePath -AclObject $EmptyAcl
# Construct the new ACL
$NewAcl = Get-Acl -Path $FilePath
$AccessRule1 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList "NT AUTHORITY\SYSTEM","FullControl","Allow"
$AccessRule2 = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList "BUILTIN\Administrators","FullControl","Allow"
$NewOwner = New-Object -TypeName System.Security.Principal.NTAccount -ArgumentList "BUILTIN\Administrators"
$NewAcl.SetAccessRule($AccessRule1)
$NewAcl.SetAccessRule($AccessRule2)
$NewAcl.SetAccessRuleProtection($true, $false)
$NewAcl.SetOwner($NewOwner)
Set-Acl -Path $FilePath -AclObject $NewAcl
Get-Acl -Path $FilePath | fl
```


### .NET Access Audit Methods

Much like ACL's, we can build and apply audit rules via PowerShell in a similar way.

See the `SetAuditRule($rule)` method.

```powershell
# TO DO
```


### SMB

This section references the following sources:

- [nemo-wq/PrintNightmare](https://github.com/nemo-wq/PrintNightmare-CVE-2021-34527#windows)
- [Set-Acl](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-acl?view=powershell-7.3#example-5-grant-administrators-full-control-of-the-file)
- [New-SmbShare](https://learn.microsoft.com/en-us/powershell/module/smbshare/new-smbshare?view=windowsserver2022-ps)

Create a new folder, an SMB share for that folder, and give `flare-user` full access to it:

```powershell
# Setup a user to access the folder
$Password = ConvertTo-SecureString "flare-pass" -AsPlainText -Force
New-LocalUser "flare-user" -Password $Password
Add-LocalGroupMember -Group "Users" -Member "flare-user"

# Create the folder
New-Item -Type Directory -Path C:\Share

# Give flare-user FullControl
$NewAcl = Get-Acl -Path "C:\Share"
$identity = "[DOMAIN\]flare-user"
$fileSystemRights = "FullControl"
$type = "Allow"
$argumentlist = $identity, $fileSystemRights, $type
$NewRule = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList $argumentlist
$NewAcl.SetAccessRule($NewRule)
Set-Acl -Path "C:\Share" -AclObject $NewAcl

# Create the share
New-SmbShare -Path C:\Share -Name "Share" -FullAccess "flare-user"

# Check available shares
Get-SmbShare -Name *
```

*NOTE 1: You can tab complete difficult to remember arguments like `System.Security.AccessControl.FileSystemAccessRule`.*

*NOTE 2: [ChatGPT May 24 Version](https://help.openai.com/en/articles/6825453-chatgpt-release-notes) demonstrates a shorter way of writing the `$NewRule` line:*

```powershell
$NewRule = New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule("flare-user", "FullControl", "Allow")
```

From here you can easily `impacket-smbclient flare-user:flare-pass@hostname` to connect.

Modifying access to a share is done with the following cmdlets:

- [`Grant-SmbShareAccess`](https://learn.microsoft.com/en-us/powershell/module/smbshare/grant-smbshareaccess?view=windowsserver2022-ps)
- [`Revoke-SmbShareAccess`](https://learn.microsoft.com/en-us/powershell/module/smbshare/revoke-smbshareaccess?view=windowsserver2022-ps)


Remove an SMB share:

```powershell
Remove-SmbShare -Name "Share"
```


## SysInternals

Overview:

<https://docs.microsoft.com/en-us/sysinternals/>

Download individual tools directly:

<https://live.sysinternals.com/>

Download the entire suite as a zip archive:

<https://download.sysinternals.com/files/SysinternalsSuite.zip>


### AccessChk

- <https://live.sysinternals.com/accesschk64.exe>
- <https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk>

See the following blog for additional uses of AccessChk:
- <https://sirensecurity.io/blog/windows-privilege-escalation-resources/>

Evaluate what access to C:\$PATH is available to $USER:

```powershell
accesschk64.exe "$USER" c:\$PATH
```

### Sysmon

- <https://live.sysinternals.com/Sysmon64.exe>
- <https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon>

#### Installing Sysmon

Open an administrative PowerShell session.

Create the tools directory with the correct permissions:

```powershell
New-Item -ItemType Directory -Path "C:\Tools" | Out-Null
New-Item -ItemType Directory -Path "C:\Tools\Scripts" | Out-Null
icacls.exe C:\Tools /reset | Out-Null
icacls.exe C:\Tools /inheritance:r | Out-Null
icacls.exe C:\Tools /grant SYSTEM:"(CI)(OI)(F)" | Out-Null
icacls.exe C:\Tools /grant BUILTIN\Administrators:"(CI)(OI)(F)" | Out-Null
icacls.exe C:\Tools /grant *S-1-1-0:"(CI)(OI)RX" | Out-Null
```

Download and install Sysmon and a starter configuration file (if you did not write your own).

- [Sysmon](https://live.sysinternals.com/Sysmon64.exe)
- [Sysmon Modular - Olaf Hartong](https://github.com/olafhartong/sysmon-modular)
- [Sysmon Config - SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config)

```powershell
# Change to the tools path
cd C:\Tools

# If you want to use SwiftOnSecurity's config:
iwr "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Tools\sysmon-config.xml"
# If you want to use Olaf Hartong's config:
iwr "https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml" -OutFile "C:\Tools\sysmon-config.xml"

# Download Sysmon
iwr "https://live.sysinternals.com/Sysmon64.exe" -OutFile "C:\Tools\Sysmon64.exe"

# Uninstall the currently running Sysmon components with the newest binary if you already have an older version running (does not require reboot)
C:\Tools\Sysmon64.exe -accepteula -u

# Install the newest Sysmon (does not require reboot)
C:\Tools\Sysmon64.exe -accepteula -i C:\Tools\sysmon-config.xml
```

**NOTE**: This does not erase or remove current log files, and they can all still be read again after installing the new binary.

#### Cleanup

- Option 1: Make the config file readable only by SYSTEM and BUILTIN\Administrator
	```powershell
	icacls.exe C:\Tools\sysmon-config.xml /inheritance:r
	icacls.exe C:\Tools\sysmon-config.xml /grant SYSTEM:"(F)"
	icacls.exe C:\Tools\sysmon-config.xml /grant BUILTIN\Administrators:"(F)"
	```
- Option 2: Delete the config file from the local machine
- Both: Monitor and log for execution of `Sysmon64.exe -c` which dumps the entire configuration whether it's still on disk or not. If you find this in your logs and did not run this, you may have been broken into.


#### Custom Config

Using Sysmon-Modular it's easy to create custom configuration files.

- [Sysmon Modular: Generating a Config](https://github.com/olafhartong/sysmon-modular/tree/master#generating-a-config)

Install the configuration with `C:\Tools\Sysmon64.exe -c .\sysmonconfig.xml` and observe it with your SIEM, the Event Viewer, or [Tail-EventLogs.ps1](https://github.com/straysheep-dev/windows-configs/blob/main/Tail-EventLogs.ps1).

Adjust rules accordingly. For example, if there's a DLL Side-Loading rule that's overwhelming your logs, you can find which rule files include this rule using the following:
```powershell
. .\Merge-AllSysmonXml

Find-RulesInBasePath -BasePath C:\Tools\sysmon-modular\ | sls "DLL Side-Loading"
```

- Read the rule criteria based on what you're seeing in your logs.
- Find the exact line in sysmon-modular's rules to tune it manually.
- Regenerate the config using `Merge-AllSysmonXml -Path ( Get-ChildItem '[0-9]*\*.xml') -AsString | Out-File sysmonconfig.xml`.


#### Tune a Config (with PowerShell)

*This will depend on your configuration file. The sysmon-modular config mentioned above is a great starting place to help you maintain includes and excludes.*

When first applying a configuration, do it in a test environment mirroring the systems it will be deployed on. Then review what's being logged that's either too much, or not enough. This will likely be accomplished with a SIEM, but if you need to do this with PowerShell you can investigate logs by working big to small in scope.

- This will help identify what's being logged *too much*
- This is not the most effective way to find what *isn't being logged*
- To find what's *not being logged* you'll need to conduct (offensive) testing on your environment
	- [Atomic-Red-Team](https://github.com/redcanaryco/atomic-red-team) can help with this

Find the frequency of all Sysmon Events:
```powershell
$StartDate = (Get-Date).AddDays(-1)
Get-WinEvent -FilterHashtable @{ Logname='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate } | Group-Object -Property Id | Select-Object Name, Count | Sort-Object Count -Descending
```

Find the frequency of a known property in each event (in this case, the RuleName from sysmon-modular):
```powershell
$StartDate = (Get-Date).AddDays(-1)
Get-WinEvent -FilterHashtable @{ Logname='Microsoft-Windows-Sysmon/Operational'; Id='10' } | ForEach-Object { $_.properties[0].value }| Group-Object | Sort-Object -Property Count, Descending | select Count, Name
```


## Windows Logging

To see all available logs on a (Windows) system:

```powershell
Get-WinEvent -ListLog * | Select -Property LogName
```

To see the available properties for a log, use:

```powershell
Get-WinEvent -MaxEvents 1 -LogName '<log>' | select -Property *
```

### Event Logs

- [Microsoft Docs: Event IDs to Monitor](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor)
- [Intro to SOC: Domain Log Review](https://github.com/strandjs/IntroLabs/blob/master/IntroClassFiles/Tools/IntroClass/DomainLogReview/DomainLogReview.md)


### Log Size

- [Tenable: STIG SERVER2019 Security Event Log 196608 KB](https://www.tenable.com/audits/items/CIS_Microsoft_Windows_Server_2019_STIG_v1.0.1_L1_DC.audit:9c098551b8a388b5b1e037989c9ef0ee)
- [Tenable: STIG WIN10 Security Event Log 1024000 KB](https://www.tenable.com/audits/items/DISA_STIG_Windows_10_v2r3.audit:00966725cee3c68f591dc1e58fc3e6db)
- [TrendMicro: Event Log File Sizes](https://help.deepsecurity.trendmicro.com/10_2/aws/Events-Alerts/Log-Event-Data-Storage.html#:~:text=Event%20log%20entries%20usually%20average,number%20of%20rules%20in%20place.)
- [Splunk: Configuring Sysmon Logging](https://docs.splunk.com/Documentation/AddOns/released/MSSysmon/ConfigureSysmon#:~:text=The%20best%20practice%20is%20to,without%20a%20custom%20configuration%20file.)

Independant of how long your SIEM or central logging server is retaining all logs shipped to it, each endpoint should maintain between 200MB to 1GB (or more if the machine can tolerate it) of Sysmon and or Security event logs to ensure all events are properly captured and forwarded (not lost).


### Windows Defender Logs

- [Defender Event Log and Error Codes](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/troubleshoot-microsoft-defender-antivirus?view=o365-worldwide)

Get the latest instance of Id 1116, `MALWAREPROTECTION_STATE_MALWARE_DETECTED`:

```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Windows Defender/Operational'; StartTime=$StartDate; Id='1116'; } | Select -First 1 | fl
```

Get all "Warning" events that aren't Id 1002, `MALWAREPROTECTION_SCAN_CANCELLED` (which can be a noisy event):

```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Windows Defender/Operational'; StartTime=$StartDate; } | where { $_.LevelDisplayName -eq 'Warning' -and $_.Id -ne 1002 }
```

Find any instances of Defender's configuration being changed:

```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Windows Defender/Operational'; StartTime=$StartDate; } | where { $_.Id -eq 5004 -or  $_.Id -eq 5007  }
```

Find any instances of Defender components being disabled or off:

```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Windows Defender/Operational'; StartTime=$StartDate; } | where { $_.Id -eq 5001 -or  $_.Id -eq 5008 -or $_.Id -eq 5010 -or $_.Id -eq 5012 }
```


### ASR Events

- [List of ASR Events](https://learn.microsoft.com/en-us/defender-endpoint/overview-attack-surface-reduction#list-of-attack-surface-reduction-events)

The link above includes raw XML you can import into the event viewer to filter for only ASR related log entries. The XML also contains the paths needed to query the event logs from PowerShell. This is a copy of that table for reference:

|Feature|Provider/source|Event ID|Description|
|---|---|:---:|---|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|1|ACG audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|2|ACG enforce|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|3|Don't allow child processes audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|4|Don't allow child processes block|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|5|Block low integrity images audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|6|Block low integrity images block|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|7|Block remote images audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|8|Block remote images block|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|9|Disable win32k system calls audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|10|Disable win32k system calls block|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|11|Code integrity guard audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|12|Code integrity guard block|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|13|EAF audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|14|EAF enforce|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|15|EAF+ audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|16|EAF+ enforce|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|17|IAF audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|18|IAF enforce|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|19|ROP StackPivot audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|20|ROP StackPivot enforce|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|21|ROP CallerCheck audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|22|ROP CallerCheck enforce|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|23|ROP SimExec audit|
|Exploit protection|Security-Mitigations (Kernel Mode/User Mode)|24|ROP SimExec enforce|
|Exploit protection|WER-Diagnostics|5|CFG Block|
|Exploit protection|Win32K (Operational)|260|Untrusted Font|
|Network protection|Windows Defender (Operational)|5007|Event when settings are changed|
|Network protection|Windows Defender (Operational)|1125|Event when Network protection fires in Audit-mode|
|Network protection|Windows Defender (Operational)|1126|Event when Network protection fires in Block-mode|
|Controlled folder access|Windows Defender (Operational)|5007|Event when settings are changed|
|Controlled folder access|Windows Defender (Operational)|1124|Audited Controlled folder access event|
|Controlled folder access|Windows Defender (Operational)|1123|Blocked Controlled folder access event|
|Controlled folder access|Windows Defender (Operational)|1127|Blocked Controlled folder access sector write block event|
|Controlled folder access|Windows Defender (Operational)|1128|Audited Controlled folder access sector write block event|
|Attack surface reduction|Windows Defender (Operational)|5007|Event when settings are changed|
|Attack surface reduction|Windows Defender (Operational)|1122|Event when rule fires in Audit-mode|
|Attack surface reduction|Windows Defender (Operational)|1121|Event when rule fires in Block-mode|

Notable paths for PowerShell parsing include the following (keeping in mind only the event ID's above apply):

- `LogName='Microsoft-Windows-Security-Mitigations/UserMode'`
- `LogName='Microsoft-Windows-Security-Mitigations/KernelMode'`
- `LogName='Microsoft-Windows-Win32k/Operational'`
- `LogName='Microsoft-Windows-WER-Diag/Operational'`
- `LogName='Microsoft-Windows-Windows Defender/Operational'`

Example queries, based on `LogName` and `$_.Id`:

```powershell
$StartDate = (Get-Date).AddDays(-1)
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Security-Mitigations/UserMode'; StartTime=$StartDate; }
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Security-Mitigations/KernelMode'; StartTime=$StartDate; } | where { $_.Id -neq 11 -and  $_.Id -neq 12  }
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Windows Defender/Operational'; StartTime=$StartDate; } | where { $_.Id -eq 1121 -or  $_.Id -eq 1122  } | fl *
```

*It's important to note that when a user is notified via a toast notificaiton, the alert details are not always available within the Windows Defender GUI to review. Using PowerShell will allow you to extract any matching logs and all details.*

### Logon Events

This can be difficult to understand at first, as the normal Logon (ID 4624) and Logoff (ID 4634) events will produce numerous entries even if you simply sign out and back in to an account locally. Using these events alone can be difficult if you're trying to trace behavior. If you're looking for something similar to Linux's `last | head` then you need to query the Explicit Logon Attempt event (ID 4648).

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'} | where TimeCreated -gt (Get-Date).AddDays(-1) | where Id -eq '4648'
Get-WinEvent -FilterHashtable @{LogName='Security'} | where TimeCreated -gt (Get-Date).AddDays(-1) | where Id -eq '4648' | fl
```

You can also use offensive tools to do this, such as [Seatbelt](https://github.com/GhostPack/Seatbelt), which returns easily readable results:

```powershell
.\Seatbelt.exe ExplicitLogonEvents
```

I wanted to mimic Seathbelt's output in PowerShell, so I wrote this script to accomplish that. This includes references to the [Sysmon Logs](#sysmon-logs) section here, and also to the [C# source code for the ExplicitLogonEvents module in Seatbelt](https://github.com/GhostPack/Seatbelt/blob/master/Seatbelt/Commands/Windows/EventLogs/ExplicitLogonEvents/ExplicitLogonEventsCommand.cs).

```powershell
# MIT License
#
# Print unique logon information for a Windows system using built in event logs, in a clear way.
# This returns every user login by default, but you can swap around variables below to suit your needs
# As is, this will also catch shells like "runas /user:username cmd.exe"
#
# This script is meant to return data in a similar format as ".\Seatbelt.exe ExplicitLogonEvents" would:
# [GhostPack/Seatbelt - ExplicitLogonEvents](https://github.com/GhostPack/Seatbelt/blob/master/Seatbelt/Commands/Windows/EventLogs/ExplicitLogonEvents/ExplicitLogonEventsCommand.cs)
#
# Parsing logs like this was learned from:
# [Security Weekly Webcast - Sysmon: Gaining Visibility Into Your Enterprise](https://securityweekly.com/webcasts/sysmon-gaining-visibility-into-your-enterprise/)
# [Slide Deck](https://securityweekly.com/wp-content/uploads/2022/04/alan-stacilauskas-unlock-sysmon-slide-demov4.pdf)
#
# Huge thanks to the presentors:
#
# [Alan Stacilauskas](https://www.alstacilauskas.com/)
# [Amanda Berlin](https://twitter.com/@infosystir)
# [Tyler Robinson](https://twitter.com/tyler_robinson)

# Set a range of time to search in the logs
# AddMinutes / AddHours / AddDays / AddMonths
$startTime = (Get-Date).AddDays(-1)

# This line will get you the relevant event log data to begin parsing
Get-WinEvent -FilterHashtable @{LogName='Security'} | where TimeCreated -gt $startTime | where Id -eq '4648' | foreach {
    # Variable names were kept the same as they are in Seatbelt's ExplicitLogonEventsCommand.cs
    # Variable properties can be extracted from the Windows Event ID 4648 $_.Message object:
    $creationTime = $_.TimeCreated
    $subjectUserSid = $_.properties[0].value
    $subjectUserName = $_.properties[1].value
    $subjectDomainName = $_.properties[2].value
    $subjectLogonId = $_.properties[3].value
    $logonGuid = $_.properties[4].value
    $targetUserName = $_.properties[5].value
    $targetDomainName = $_.properties[6].value
    $targetLogonGuid = $_.properties[7].value
    $targetServerName = $_.properties[8].value
    $targetServerInfo = $_.properties[9].value
    $processId = $_.properties[10].value
    $processName = $_.properties[11].value
    $ipAddress = $_.properties[12].value
    $ipPort = $_.properties[13].value

    # You can change this pattern variable and the if statement itself below if you'd like to search for different data
    # Basically the "user logging in" part of the login sequence captured by event logs is made by svchost.exe and not winlogon.exe
    $patternToMatch = 'winlogon.exe'

    # Another approach is filtering for a blank username:
    # if ($targetUserName -ne $nul) { ...
    #
    # Alternatively you could filter for:
    # if ($targetUserName -inotmatch "(UMFD-\d|DWM-\d)") { ...
    # DWM-\d matches patterns of the Desktop Window Manager user
    # UMFD-\d matches patterns of the Font Driver Host user

    # Lastly, you could filter based on the source IP Address:
    # if ($ipAddress -match $patternToMatch) {

    if ($processName -inotmatch $patternToMatch) {
        Write-Host "$creationTime, $targetUserName, $targetDomainName, $targetServerName, $processName, $ipAddress"
    }
}
```


### PowerShell Logs

- At minimum, enable Script Block logging (without Invocation Logging)
- If possible, enable Transcription to a protected directory that's sent to a central logging server


#### Module Logging

Enable logging for selected PowerShell modules.

*This may not be necessary if you enable script block logging.*


#### Script Block Logging

Logs PowerShell input (commands, functions, decodes and tracks script content)

- [Enable Script Block Logging](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging?view=powershell-5.1#enabling-script-block-logging)
- `Get-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"`
- `Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on PowerShell Script Block Logging`
- Log Name: `Microsoft-Windows-PowerShell/Operational`
- Log Path: `C:\Windows\System32\winevt\Logs\Microsoft-Windows-PowerShell%4Operational.evtx`

*Invocation Logging is an entirely optional checkbox under this Group Policy setting. It logs the start and stop of executions. This will flood your logs.*

Enable Script Block Logging (without the start / stop Invocation Logging option):

```powershell
# https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging?view=powershell-5.1#using-the-registry
$basePath = @(
    'HKLM:\Software\Policies\Microsoft\Windows'
    'PowerShell\ScriptBlockLogging'
) -join '\'

if (-not (Test-Path $basePath)) {
    $null = New-Item $basePath -Force
}

Set-ItemProperty $basePath -Name EnableScriptBlockLogging -Value "1"
Set-ItemProperty $basePath -Name EnableScriptBlockInvocationLogging -Value "0"
```

Remove Script Block Logging:

```powershell
# https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging?view=powershell-5.1#using-the-registry
$basePath = @(
    'HKLM:\Software\Policies\Microsoft\Windows'
    'PowerShell\ScriptBlockLogging'
) -join '\'

if (Test-Path $basePath) {
    Remove-ItemProperty $basePath -Name EnableScriptBlockLogging
    Remove-ItemProperty $basePath -Name EnableScriptBlockInvocationLogging
}
```

[Script Block logging has the benefit of being able to unwrap obfuscated functions and record their raw text.](https://learn.microsoft.com/en-us/powershell/scripting/windows-powershell/wmf/whats-new/script-logging?view=powershell-7.3)

This code sample from the Microsoft Docs link above reassembles large script blocks logged across multiple entries:

```powershell
$created = Get-WinEvent -FilterHashtable @{ ProviderName="Microsoft-Windows-PowerShell"; Id = 4104 } |
  Where-Object { $_.<...> }
$sortedScripts = $created | sort { $_.Properties[0].Value }
$mergedScript = -join ($sortedScripts | % { $_.Properties[2].Value })
```


#### Transcription Logging

Logs PowerShell output (everything that appears in the powershell terminal session)

- [PowerShell Transcription](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_powershell_config?view=powershell-7.3#transcription)
- `HKLM:\Software\Policies\Microsoft\Windows\PowerShell\Transcription`
- `Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on PowerShell Script Block Logging`
- Log path is up to the user, written as a text file

Enable PowerShell transcription, write output to C:\PSTranscripts:

```powershell
# https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging?view=powershell-5.1#using-the-registry
# https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_powershell_config?view=powershell-7.3#transcription
# https://github.com/clr2of8/PowerShellForInfoSec/blob/main/Tools/Set-PSLogging.ps1
$basePath = @(
    'HKLM:\Software\Policies\Microsoft\Windows'
    'PowerShell\Transcription'
) -join '\'

if (-not (Test-Path $basePath)) {
    $null = New-Item $basePath -Force
}

Set-ItemProperty $basePath -Name EnableTranscripting -Value "1"
Set-ItemProperty $basePath -Name EnableInvocationHeader -Value "1"
Set-ItemProperty $basePath -Name OutputDirectory -Value "C:\PSTranscripts"
```

Remove PowerShell Transcription:

```powershell
# https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging?view=powershell-5.1#using-the-registry
# https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_powershell_config?view=powershell-7.3#transcription
# https://github.com/clr2of8/PowerShellForInfoSec/blob/main/Tools/Set-PSLogging.ps1
$basePath = @(
    'HKLM:\Software\Policies\Microsoft\Windows'
    'PowerShell\Transcription'
) -join '\'

if (Test-Path $basePath) {
    Remove-ItemProperty $basePath -Name EnableTranscripting
    Remove-ItemProperty $basePath -Name EnableInvocationHeader
    Remove-ItemProperty $basePath -Name OutputDirectory
}
```


### Sysmon Logs

This is a quick start on how to read your Sysmon logs.
- [Get-WinEvent](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.2)

These will help in building statements to parse logs conditionally with more granularity:
- [PowerShell if Statements](https://docs.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-if?view=powershell-7.2)
- [Out-String](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/out-string?view=powershell-7.2)

Note that if you do not use `Sort-Object -Unique` or similar, logs will be displayed from oldest (top) to newest (bottom). It is also not necessary to use `Out-String` if you prefer working with PowerShell objects over strings.

- [Group-Object](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/group-object?view=powershell-7.3)

**Another tip is using `<cmds>... | Group-Object | Select-Object Count, Name | Sort-Object Count -Descending`. This is the PowerShell equivalent to Unix's `... | sort | uniq -c | sort -nr`, grouping unique results into order based on occurrance.**

This webcast is an excellent resource, not just for rule creation but various ways to script and parse logs, and essentially a quick start to Sysmon threat hunting via the CLI and GUI:

- [Security Weekly Webcast - Sysmon: Gaining Visibility Into Your Enterprise](https://securityweekly.com/webcasts/sysmon-gaining-visibility-into-your-enterprise/)
- [Slide Deck](https://securityweekly.com/wp-content/uploads/2022/04/alan-stacilauskas-unlock-sysmon-slide-demov4.pdf)

Huge thanks to the presentors:

- [Alan Stacilauskas](https://www.alstacilauskas.com/)
- [Amanda Berlin](https://twitter.com/@infosystir)
- [Tyler Robinson](https://twitter.com/tyler_robinson)

Additional resources from the presentation, discord, and other trainings:

- [Poshim - Automated Windows Log Collection](https://www.blumira.com/integration/poshim-automate-windows-log-collection/)
- [Chainsaw](https://github.com/countercept/chainsaw)
- [DeepBlueCLI](https://github.com/sans-blue-team/DeepBlueCLI)
- [Hayabusa](https://github.com/Yamato-Security/hayabusa)
- [Sysmon Modular - Olaf Hartong](https://github.com/olafhartong/sysmon-modular)
- [Sysmon Config - SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config)

There are two easy ways of parsing log output:

| Technique                                      | TypeName                                            |
| ---------------------------------------------- | --------------------------------------------------- |
| ` \| select { $_.properties[4].value }` | Selected.System.Diagnostics.Eventing.Reader.EventLogRecord |
| ` \| ForEach-Object { Out-String -InputObject $_.properties[4].value }` | System.String              |

The technique of using `ForEach-Object { Out-String -InputObject $_.properties[x,y,z].value }` was highlighted during the webcast.

---

To parse *any* event logs quickly and effectively:

- [Use a hash table](https://learn.microsoft.com/en-us/powershell/scripting/samples/creating-get-winevent-queries-with-filterhashtable?view=powershell-7.3)
- Set a start time
- Obtain object property values

You can get any log's property values by piping it to `| select -First 1 | fl`. Each line in the Message block relates to a value.

Specify multiple values like this: `$_.properties[0,1,5,6].value`

You can also specifiy a series of values with two `..` dots between them like: `$_.properties[0..20].value`

Set the start time as a variable:
```powershell
$StartDate = (Get-Date).AddMinutes(-30)
$StartDate = (Get-Date).AddHours(-1)
$StartDate = (Get-Date).AddDays(-2)
```


#### Sysmon Event IDs

These references can help you sort through event properties visually.

- [Sysmon: Events](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon#events)
- [BHIS: Sysmon Event ID Breakdown](https://www.blackhillsinfosec.com/a-sysmon-event-id-breakdown/)
- [olafhartong/sysmon-cheatsheet (PDF)](https://github.com/olafhartong/sysmon-cheatsheet/blob/master/Sysmon-Cheatsheet-dark.pdf)


#### ID 22: DNS Queries

DNS query property values:

- [0]RuleName
- [1]UtcTime
- [2]ProcessGuid
- [3]ProcessId
- [4]QueryName
- [5]QueryStatus
- [6]QueryResults
- [7]Image
- [8]User

Show all unique DNS queries (ID 22) and sort them by frequency:
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate; Id='22' } | ForEach-Object { $_.properties[4].value } | Group-Object | Sort-Object -Property Count -Descending | Select Count,Name
```

Show all DNS queries (ID 22) that contain "google":
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate; Id='22' } | Where-Object { $_.properties[4].value -imatch "google" } | ForEach-Object { $_.properties[4].value } | Group-Object | Sort-Object -Property Count -Descending | Select Count,Name
```

Show all DNS queries (ID 22) and when they were made:
```powershell
Get-WinEvent -FilterHashtable @{ Logname='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate; Id='22' } | ForEach-Object { Out-String -InputObject $_.properties[1,4].value }
```

#### ID 3: Network Connection

Network connection property values:

- [0]RuleName
- [1]UtcTime
- [2]ProcessGuid
- [3]ProcessId
- [4]Image
- [5]User
- [6]Protocol
- [7]Initiated
- [8]SourceIsIpv6
- [9]SourceIp
- [10]SourceHostname
- [11]SourcePort
- [12]SourcePortName
- [13]DestinationIsIpv6
- [14]DestinationIp
- [15]DestinationHostname
- [16]DestinationPort
- [17]DestinationPortName

Show all unique executables that made a network connection, filter out `msedge.exe` and `svchost.exe`, sort by frequency:
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; Id='3' } | where { $_.properties[4].value -inotmatch "(msedge.exe|svchost.exe)" } | ForEach-Object { $_.properties[4].value } | Group-Object | Sort-Object -Property Count -Descending | Select Count, Name
```

Using the query above, get all unique destination IPs, sort by frequency:

- Use this to filter network activity
- Feed the returned list of IPs to a threat intel platform

```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; Id='3' } | where { $_.properties[4].value -inotmatch "(msedge.exe|svchost.exe)" } | ForEach-Object { $_.properties[14].value } | Group-Object | Sort-Object -Property Count -Descending | Select Count, Name
```

Show all network connections (ID 3), what executable made them, when, and destination IP / hostname:
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate; Id='3' } | ForEach-Object { Out-String -InputObject $_.properties[1,4,14,15].value }
```

#### ID 1: Process Creation

Process creation property values:
- [0]RuleName
- [1]UtcTime
- [2]ProcessGuid
- [3]ProcessId
- [4]Image
- [5]FileVersion
- [6]Description
- [7]Product
- [8]Company
- [9]OriginalFileName
- [10]CommandLine
- [11]CurrentDirectory
- [12]User
- [13]LogonGuid
- [14]LogonId
- [15]TerminalSessionId
- [16]IntegrityLevel
- [17]Hashes
- [18]ParentProcessGuid
- [19]ParentProcessId
- [20]ParentImage
- [21]ParentCommandLine
- [22]ParentUser

List all processes created (ID 1) by timestamp, PID, executable, commandline, executable hashes, and PPID:
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate; Id='1' } | ForEach-Object { Out-String -InputObject $_.properties[1,3,4,10,17,19].value }
```

List all details of all processes created (ID 1):
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate; Id='1' } | fl
```

List all details of (ID 1) log entries where the RuleName field contains "Discovery":
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate; Id='1' } | where { $_.properties[0].value -imatch "Discovery" } | fl
```

List how many times each executable created a process (ID 1), sorted by frequency:
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate; Id='1' } | ForEach-Object { $_.properties[4].value } | Group-Object | Sort-Object -Property Count -Descending | Select Count,Name
```


#### ID 10: Process Accessed

Process accessed property values:
- [0]RuleName
- [1]UtcTime
- [2]SourceProcessGUID
- [3]SourceProcessId
- [4]SourceThreadId
- [5]SourceImage
- [6]TargetProcessGUID
- [7]TargetProcessId
- [8]TargetImage
- [9]GrantedAccess
- [10]CallTrace
- [11]SourceUser
- [12]TargetUser

Detect LSASS access (currently this just matches the first entry that has the string "lsass" anywhere in the log, needs revised to be more precise):
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-Sysmon/Operational'; StartTime=$StartDate; Id='10' } | Where-Object { $_.properties[0..30].value -imatch "lsass" } | Select -First 1 | fl *
```


## Services

### OpenSSH

| Directory              | Description
| ---------------------- | ------------------------------------------------- |
| `C:\ProgramData\ssh\`  | Main configuration folder, `/etc/ssh/` equivalent |
| `C:\Users\$USER\.ssh\` | User folder, `~/.ssh` equivalent                  |

Steps to setup OpenSSH Server (The OpenSSH Client is typically installed by default).

1. Update Windows

2. Install OpenSSH Server Optional Feature

See the following Microsoft documentation of OpenSSH for additional context:

<https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse>

> ```powershell
> Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
>
> # Install the OpenSSH Client
> Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
>
> # Install the OpenSSH Server
> Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
>
> # Start the sshd service
> Start-Service sshd
> ```

3. Configure:

> ```powershell
> # OPTIONAL but recommended:
> Set-Service -Name sshd -StartupType 'Automatic'
>
> # Confirm the Firewall rule is configured. It should be created automatically by setup. Run the following to verify
> if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) {
>     Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
>     New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
> } else {
>     Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
> }
> ```

Change the following in `C:\ProgramData\ssh\sshd_config`:
```
#PasswordAuthentication Yes
```
To:
```
PasswordAuthentication no
```

Place public key data into:
```
C:\Users\$USER\.ssh\authorized_keys
C:\ProgramData\ssh\administrators_authorized_keys
```

Like on Unix systems, the permissions must be correct on the authorized_key files. See the following page on the Win32-OpenSSH Wiki for detailed descriptions:

<https://github.com/PowerShell/Win32-OpenSSH/wiki/Security-protection-of-various-files-in-Win32-OpenSSH>

Then, if the user you'll be logging in as IS NOT an administrator:
```powershell
takeown.exe /F .\authorized_keys /S $HOSTNAME /U $USER
icacls.exe .\authorized_keys /reset
icacls.exe .\authorized_keys /inheritance:r
icacls.exe .\authorized_keys /grant $USER:"(F)"
```

or if the user you'll be logging in as IS an administrator:
```powershell
icacls.exe .\authorized_keys /reset
icacls.exe .\administrators_authorized_keys /inheritance:r
icacls.exe .\administrators_authorized_keys /grant SYSTEM:"(F)"
icacls.exe .\administrators_authorized_keys /grant BUILTIN\Administrators:"(F)"
```

4. Applying Configuration Changes

Anytime you update or modify `C:\ProgramData\ssh\sshd_config`, be sure to restart the `sshd` service so that it reads and loads the latest changes:

```powershell
Restart-Service sshd
```

---

## Scheduled Tasks

List all scheduled tasks by creation date:

```powershell
Get-ScheduledTask -TaskName * | Select -Property Date,Author,Taskname | Sort-Object -Property Date
```

View scheduled tasks' creation date in the GUI:

Task Scheduler > Task Scheduler (Local) > Task Scheduler Library > [TaskName] > Created

Autoruns will also quickly identify any scheduled tasks in the `Scheduled Tasks` tab.

- Hide: Windows Entries
- Hide: Microsoft Entries

<https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns>

### Log & Audit Task Creation

<https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4698#security-monitoring-recommendations>

Using the GUI:

Run `gpedit.msc`

`Local Computer Policy > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Object Access > Audit Other Object Access Events`

Choose `Configure the following audit events`

Choose `Success`, then `Apply` and `OK`

Now you can query the event logs for occurances of scheduled task creation:

```powershell
Get-WinEvent -LogName "Security" | Where Id -eq "4698"
Get-WinEvent -LogName "Security" | Where Id -eq "4698" | Select -Property *
```

**TO DO**: Configure scheduled task creation auditing using only PowerShell

You can locate registry items just like searching the filesystem.

We know the policy for scheduled task logging is related to "Object Access":

```powershell
Get-ChildItem -Path "HKLM:\SOFTWARE\Microsoft\*object*access*" -Recurse -ErrorAction SilentlyContinue
```

After a possible delay while it searches, you should find it at this path, and be able to check it's values:

```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\PolicyManager\default\Audit\ObjectAccess_AuditOtherObjectAccessEvents"
```

---

## WMI

- [Intro to SOC: Windows CLI](https://github.com/strandjs/IntroLabs/blob/master/IntroClassFiles/Tools/IntroClass/WindowsCLI/WindowsCLI.md)
- [Hacktricks: CMD](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/basic-cmd-for-pentesters.md)
- [Hacktricks: PowerShell](https://github.com/carlospolop/hacktricks/tree/master/windows-hardening/basic-powershell-for-pentesters)
- [PayloadsAllTheThings: PowerShell](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Powershell%20-%20Cheatsheet.md)
- [PayloadsAllTheThings: WMI Event Subscription](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Persistence.md#windows-management-instrumentation-event-subscription)

---

## Disk Management

This SuperUser answer by user VainMan is an excellent walkthrough of managing disks with just built in Windows tools:

- [SuperUser: How to Move the Recovery Partition on Windows 10](https://web.archive.org/web/20220612105334/https://superuser.com/questions/1453790/how-to-move-the-recovery-partition-on-windows-10)
	* <https://stackoverflow.com/legal/terms-of-service#licensing>
	* This section is released under the same [CC-BY-SA-4.0](https://creativecommons.org/licenses/by-sa/4.0/) license as the original post.

PowerShell Cmdlets:

- [Get-Disk](https://learn.microsoft.com/en-us/powershell/module/storage/get-disk?view=windowsserver2022-ps#description)
- [Set-Disk](https://learn.microsoft.com/en-us/powershell/module/storage/set-disk?view=windowsserver2022-ps#description)
- [List of All Storage Cmdlets](https://learn.microsoft.com/en-us/powershell/module/storage/?view=windowsserver2022-ps)

Additional documentation for this section:

- [Microsoft Docs: Capture and Apply the System and Recovery Partitions](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/capture-and-apply-windows-system-and-recovery-partitions?view=windows-11)
- [`diskpart.exe`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/diskpart)
- [`dism.exe`](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/what-is-dism?view=windows-11)
- [`reagentc.exe`](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/reagentc-command-line-options?view=windows-11)
- [`mountvol.exe`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/mountvol)
- `diskmgmt.mmc`


### Mount and Unmount Volumes and Drives (GUI)

The [default external storage removal policy](https://learn.microsoft.com/en-us/windows/client-management/client-tools/change-default-removal-policy-external-storage-media) in recent versions of Windows (since Windows 10 1809) is **Quick removal**.


See: `diskmgmt.msc` > Right-Click the Disk number, select `Properties` > Policies Tab

- **Quick removal**: You can remove the device without using the "Safely Remove Hardware" process
- **Better performance**: Improved system resource usage, data is cached but you will need to use "Safely Remove Hardware" to avoid data loss

Set a Drive to Offline:

- Also in the `diskmgmt.msc` window, Right-Click the Disk number
- Select `Offline` to take the drive offline
- This unmounts the drive, and prevents accidental writes or access
- You can also use [`Set-Drive`](https://learn.microsoft.com/en-us/powershell/module/storage/set-disk?view=windowsserver2022-ps#example-2) to take a disk offline or make it read-only


### Mount and Unmount Volumes and Drives (CLI)

This is functionally the equivalent of using `diskmgmt.mmc` in any of the following ways:

- Create a new simple volume from unallocated space, or format an external drive and assigning it a drive letter
- Unmounting the drive (removing the drive letter)
- Deleting the volume altogether

Get the `VolumeName` of the `G:` drive:
```cmd
mountvol.exe G: /L
     \\?\Volume{12345678-abcd-efab-cdef-01234567890a}\
```

Unmount the `G:` drive
```cmd
mountvol.exe G: /P
```

- The volume will still exist
- The mount point will no longer be traversable since there's no longer an assigned drive letter
- `Get-ItemProperty -Path "HKLM:\SYSTEM\MountedDevices"` will show a `#{GUID}` instead of `\DosDevices\G:`
- Even if you assign the volume a new drive letter, it will know it's tied to the same `#{GUID}` and replace it with `\DosDevices\<letter>:`

Mount the volume to a new drive letter (quote the VolumeName string):
```cmd
mountvol.exe X: '\\?\Volume{12345678-abcd-efab-cdef-01234567890a}\'
```

Remove the volume, delete the volume (meaning all data on the volume), then clean up any leftover artifiacts in the registry

- `Get-ItemProperty -Path "HKLM:\SYSTEM\MountedDevices"` will still show a `#{GUID}` after deleting a volume with any disk tool
- The deleted volume should still be visible in `diskmgmt.mmc` until you delete its registry entry, then close and open `diskmgmt.mmc` again
- `mountvol.exe /R` will remove this registry entry once the volume no longer exists
- If the volume still exists, `mountvol.exe /R` will not remove the entry
- If you format the same volume again and mount it, you'll note it retains the same `#{GUID}` when unmounted

```cmd
mountvol.exe G: /P
diskpart.exe
> list volume
> select Volume 1
> delete
> exit
mountvol.exe /R
```


### Extend the C Drive

Extending the C drive isn't intuitive because the partition the OS is installed on is sandwiched between the boot and recovery partitions. Even if you have free space (say on a newly extended VM or on a dual boot machine where you're removing the other OS) you cannot extend the installed partition without moving the recovery partition.

This entire example essentially mirrors the SuperUser answer linked above, with only small additions or changes to focus on backing up the recovery partition, deleting it so that the `C:` partition can be extended, then appending the backup recovery partition image to the new end of the `C:` partition.

Mount the recovery partition
```cmd
diskpart
list disk
select disk <disk>
list partition
select partition <partition>
assign letter=R
exit
```
Create an image file with the `dd`-like `dism.exe` utility:
```cmd
dism /Capture-Image /ImageFile:C\recovery-partition.wim /CaptureDir:R:\ /Name:"Recovery"
```

Delete the current recovery partition:
```cmd
diskpart
select volume R
delete partition override
exit
```

- Extend the `C:\` drive in Disk Management
- Create a new volume (+ new drive letter)
- Apply the recovery image to the new volume using the new drive letter

```cmd
dism /Apply-Image /ImageFile:C:\recovery-partition.wim /Index:1 /ApplyDir:R:\
```

Register the new recovery location
```cmd
reagentc /disable
reagentc /setreimage /path R:\Recovery\WindowsRE
reagentc /enable
```

Unmount that drive letter to return the recovery partition to a 'hidden' state
```cmd
diskpart
select volume R
remove
exit
```

* You'll be able to see in Disk Management the Recovery (or whatever you named it) partition loses it's new drive letter

Optionally:
- Confirm the recovery partition is working with `reagentc /info`
- Boot into recovery `reagentc /boottore /logfile C:\Temp\Reagent.log`
- Delete the recovery image file `del C:\recovery-image.wim`


### Prevent Automatic Mounting of New Volumes

*NOTE: this still needs tested, and does not appear to work as expected.*

This is useful in forensics where you want to connect an external drive but do not want to mount or open the filesystem.

On Windows Server, this is a built in utility: [`automount`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/automount).

On a Windows workstation, you must use [`mountvol.exe`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/mountvol).

Disables automatic mounting of new basic volumes. New volumes are not mounted automatically when added to the system.
```cmd
mountvol.exe /N
```

- Now `mountvol.exe /?` will print a message at the bottom of the usage information noting if automatic mounting is disabled

Re-enables automatic mounting of new basic volumes.
```cmd
mountvol.exe /E
```


### EFI Partition

This mounts the `\EFI` partiton under `E:\` if it's available (you can use any available drive letter).
```cmd
mountvol.exe E: /S
```

If you want to review the 10 most recently modified files in the EFI partition mounted at `E:\`:
```powershell
gci E:\EFI -Recurse -Force -File | sort LastWriteTime -Descending | Select -First 10 FullName,LastWriteTime 2>$nul
```

Get a list of all filetypes found in `E:\EFI`:
```powershell
gci E:\EFI -Recurse -Force -File | sort LastWriteTime -Descending | Group-Object Extension 2>$nul
```

Check the signature of every file in `E:\EFI`, using `C:\Tools\sigcheck64.exe`:
```powershell
foreach ($efi_file in (gci E:\ -Recurse -Force -File).FullName) { if (C:\Tools\sigcheck64.exe -accepteula $efi_file | Select-String "^\s+Verified:\s+Signed$") { Write-Host -ForegroundColor GREEN "[OK]$efi_file" } else { Write-Host -ForegroundColor RED "[WARNING]$efi_file" } }
```

---


## Device Management

Restricting device and driver installation.

- [Manage Device Installation with Group Policy](https://learn.microsoft.com/en-us/windows/client-management/manage-device-installation-with-group-policy)
- [System Defined Device Setup Classes Available to Vendors](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/system-defined-device-setup-classes-available-to-vendors)
- [System Defined Device Setup Classes Reserved for System Use](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/system-defined-device-setup-classes-reserved-for-system-use)
- [USB Device Descriptors](https://learn.microsoft.com/en-us/windows-hardware/drivers/usbcon/usb-device-descriptors)

### Device ID Strings

<https://learn.microsoft.com/en-us/windows/client-management/manage-device-installation-with-group-policy#determine-device-identification-strings>

The `Class GUID` is the type of device, for example a printer or a tablet.

The `Instance ID`, `Hardware IDs`, and `Compatible IDs` are identifiers for the device. The most specific is the `Instance ID`. This relates to that one device, and individual devices may have multiple `Instance ID`s. You'll need these ID's to 'allow' or 'deny' devices.

### Enum Device ID Strings (cmd.exe)

You can start like this from a command prompt with [`pnputil`](https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/pnputil):

```cmd
pnputil /enum-devices /connected
```

If you were trying to find a single device, scroll through the list to determine the possible `Class GUID` of the device you're looking for.

Next you'd query connected devices of that class. This should narrow down the results enough for you to identify the exact device.

```cmd
pnputil /enum-devices /connected /class '{<guid>}'
```

In this case, the `Device Description` fields most closely matching and describing your device are usually the right ones.

Finally, query the device directly using the `Instance ID` (there may be multiple `Instand ID`s for the same device, query each one you find) and this time also print the `Hardware IDs` and `Compatible IDs` for each instance:

```cmd
pnputil /enum-devices /instanceid '<instance-id>' /ids
```

### Enum Device ID Strings (PowerShell)

All the credit for this one goes to the [PowerShell Team](https://devblogs.microsoft.com/powershell/) over on the Microsoft DevBlogs:

- [Displaying USB Devices Using WMI](https://web.archive.org/web/20220903134605/https://devblogs.microsoft.com/powershell/displaying-usb-devices-using-wmi/)

This will gather all of your connected USB devices in a single line:

> ```powershell
> Get-WmiObject Win32_USBControllerDevice | %{[wmi]($_.Dependent)} | Sort-Object Description,DeviceID | ft Description,DeviceID -auto
> ```

We can append `ClassGuid`, `HardwareID`, and or `CompatibleID` onto an `fl` instead of an `ft` command to obtain the other IDs as well:

```powershell
Get-WmiObject Win32_USBControllerDevice | %{[wmi]($_.Dependent)} | Sort-Object Description,DeviceID | fl Description,DeviceID,ClassGuid,HardwareID,CompatibleID
```

In any case, you'll want to note the `Instance ID`, `Hardware IDs`, and `Compatible IDs` for use with policy configuration.

As mentioned in #3 of [Scenario steps - preventing installation of prohibited devices](https://learn.microsoft.com/en-us/windows/client-management/manage-device-installation-with-group-policy#scenario-steps--preventing-installation-of-prohibited-devices), be sure you know exactly what will be blocked by the policy before deploying it. It's recommended to test all of this in a VM, as you can quickly recover from a policy that's too broad (which could block all external devices preventing you from using or recovering the machine).

Devices are evaluated by the `Apply layered order of evaluation for Allow and Prevent device installation policies across all device match criteria` policy in the following order:

- Device Instance IDs
- Device IDs (hardware / compatible)
- Device Setup Class
- Removeable Devices

`Device Instance IDs` are the most specific and always take precedence.

To allow only trusted devices while defaulting to blocking all others:

- Enter the ID's of devices you wish to allow under `Allow installation of devices that match any of these device instance IDs` and enable this policy
- Set `Apply layered order of evaluation for Allow and Prevent device installation policies across all device match criteria` policy to enable
- Enabled the `Prevent installation of removable devices` policy

Alternatively, and as a test, if you wish to block specific devices while allowing all by default:

- Enable the `Prevent installation of devices that match any of these device instance IDs` policy to block specific Instance IDs (precise)
- Enable the `Prevent installation of devices that match any of these device IDs` policy to block based on Hardware or Compatible IDs (broad)
- Click `Show` and add the IDs to this list
- Check `Also apply to matching devices that are already installed`

What the last point will do is disconnect any currently connected device(s) matching the policy and remove their drivers. In Windows 11 this will happen immediately after applying the policy.

*Remember you may need to specify multiple `Instance IDs` even for one device.*

You can confirm the devices are no longer visible with:

```cmd
pnputil.exe /enum-devices /connected /class '{<guid>}'
```

Disabling or setting that policy to Not Configured will reconnect the devices.


### Deny Execute Access, Disable Autorun

By default, Windows will automatically mount any external drives connected. However, these settings will prevent them from executing any content. AutoPlay typically loads media or external files automatically based on a user setting. Autoruns (since Vista) execute content from an `autorun.inf` file. These files should require user interaction.

In the following GPO path:

```
Local Computer Policy > Administrative Templates > System > Removable Storage Access
```

The following policies control whether content can execute *at all* from removable media.

- CD and DVD: Deny execute access
- Floppy Drives: Deny execute access
- Removable Disks: Deny execute access
- All Available Storage Classes: Deny execute access
- Tape Drives: Deny execute access

Next, in this GPO path:

```
Local Computer Policy > Administrative Templates > Windows Components > AutoPlay Policies
```

These settings control Autorun and AutoPlay features.

- `Turn off Autoplay`: Enabled, CD-ROM and removable media drives
- `Set the default behaivor for Autorun`: Enabled, Do not execute any autorun commands
- `Disallow Autoplay for non-volume devices`: Enabled


To set these items from PowerShell:

```powershell
# Per user, AutoPlay setting
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\AutoplayHandlers" -Name "DisableAutoplay" -Type DWord -Value 1
```


### Enable or Disable PnP Devices

[stackoverflow: Enable / Disable Webcam with PowerShell](https://stackoverflow.com/questions/61057551/how-to-enable-disable-webcam-by-script-win-10)

Sometimes you won't be able to re-enable a previously disabled webcam through the settings menu. Instead enumerate cameras, then enable or disable them using their `InstanceId`.

Enumerate known cameras.

- Disabled cameras have a status of `error`
- Enabled cameras have a status of `OK`
- Disconnected cameras have a status of `unknown`

```powershell
Get-PnpDevice -FriendlyName "*cam*" | select Status,FriendlyName,InstanceId | fl
Get-PnpDevice -Class camera | select Status,FriendlyName,InstanceId | fl
```

Enable / Disable camera as administrator, you'll be prompted to confirm:

```powershell
Enable-PnpDevice -InstanceId "<instanceid>"
```


### Blocking ISO Mounting

Taken directly from [Mubix](https://twitter.com/mubix)'s blog post:

- <https://malicious.link/post/2022/blocking-iso-mounting/>

What this does:

- Block ISO mounting from double clicking
- Block ISO mounting via the context menu
- Block programmatic ISO mounting via PowerShell

In the following GPO path:

```
Local Computer Policy > Administrative Templates > System > Device Installation > Device Installation Restrictions
```

Enable this policy:

```
Prevent installation of devices that match any of these device IDs
```

Check the "Also apply to matching devices that are already installed" option.

And set this as a value:

```
SCSI\CdRomMsft____Virtual_DVD-ROM_
```

Try to mount an ISO to validate this is working. All ISO files should fail to mount to the filesystem once this policy is in place.

#### Creating an ISO for Testing

On Ubuntu the `genisoimage` command can quickly create an ISO of any folder or file:

```bash
genisoimage -o <outfile>.iso <input-folder>
genisoimage -o my-docs.iso ./Documents
```

You can verify the ISO and contents with:

```bash
file ./my-docs.iso

sudo mkdir /mnt/my-iso
sudo mount my-docs.iso /mnt/my-iso

ls -l /mnt/my-iso

sudo umount /mnt/my-iso
sudo rm -rf /mnt/my-iso
```

Now move the `.iso` file over to your Windows machine (or if you did this in WSL it's already there) to confirm your policy configurations are working.

**Additional Use Cases**

- [Allow only authorized USB device(s), block all other USB devices](https://learn.microsoft.com/en-us/windows/client-management/manage-device-installation-with-group-policy#scenario-steps--preventing-installation-of-all-usb-devices-while-allowing-only-an-authorized-usb-thumb-drive)

- [Block a specific device from being installed](https://learn.microsoft.com/en-us/windows/client-management/manage-device-installation-with-group-policy#scenario-2-prevent-installation-of-a-specific-printer-1)

**TO DO**: how to configure these policies with just PowerShell

---
