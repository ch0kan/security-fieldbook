# Persistence Investigation Lab

---

## Executive Summary

I used this lab to practice investigating common Windows persistence locations.

The goal was to review areas where suspicious activity could survive a reboot, collect evidence from the Windows Client VM, and document persistence-related findings in a clear investigation format.

## Lab Objective

I performed a persistence investigation to practice:

- Reviewing Windows services.
- Checking registry autorun locations.
- Auditing local users.
- Reviewing local Administrators group membership.
- Investigating scheduled tasks.
- Querying event logs for persistence-related activity.
- Comparing system state against a baseline.
- Documenting persistence evidence and detection opportunities.

## Environment

| System | Role |
|---|---|
| Windows Client VM | Endpoint I investigated |
| Kali VM | Controlled testing system |
| pfSense VM | Firewall/router and network visibility point |
| Analysis VM | Optional evidence review system |

## Network Placement

| System | Network | Example IP |
|---|---|---|
| Kali VM | Attack Network | `10.10.20.10` |
| Windows Client VM | Lab LAN | `10.10.30.10` |
| Analysis VM | Logging Network | `10.10.50.10` |
| pfSense Lab LAN Gateway | Lab LAN | `10.10.30.1` |

## Scenario I Simulated

I treated the Windows Client VM as a system where suspicious activity may have created persistence.

Instead of immediately removing anything, I focused on identifying persistence mechanisms, collecting supporting evidence, and building a timeline from system artifacts and event logs.

## Tools I Used

| Tool | How I Used It |
|---|---|
| PowerShell | Main investigation tool |
| `Get-CimInstance` | Reviewed service configuration and executable paths |
| `Get-ItemProperty` | Checked registry autorun values |
| `Get-LocalUser` | Reviewed local user accounts |
| `Get-LocalGroupMember` | Reviewed local administrator membership |
| `Get-ScheduledTask` | Listed scheduled tasks |
| `Export-ScheduledTask` | Reviewed scheduled task XML and actions |
| `Get-ScheduledTaskInfo` | Checked last and next task run times |
| `Get-WinEvent` | Queried persistence-related event logs |
| `Compare-Object` | Compared current state against a baseline |

## Host Context Collected

I started by collecting basic host context.

```powershell
hostname
```

```powershell
whoami
```

```powershell
Get-Date
```

```powershell
ipconfig /all
```

I recorded the following details:

| Item | Value |
|---|---|
| Hostname |  |
| Current user |  |
| IP address |  |
| Investigation time |  |

## Windows Service Review

I reviewed Windows services because they are a common persistence mechanism. Services can start automatically and may run with elevated privileges.

I started with a basic service list.

```powershell
Get-Service
```

Then I used CIM to collect service paths and startup configuration.

```powershell
Get-CimInstance -ClassName Win32_Service |
Select-Object Name, DisplayName, State, StartMode, StartName, PathName
```

I exported the results for documentation.

```powershell
Get-CimInstance -ClassName Win32_Service |
Select-Object Name, DisplayName, State, StartMode, StartName, PathName |
Export-Csv .\services.csv -NoTypeInformation
```

## Service Indicators I Reviewed

I looked for services that had unusual names, paths, or startup behavior.

| Indicator | Why I Investigated It |
|---|---|
| Random service name | Possible generated or disguised service |
| Misspelled service name | Possible masquerading |
| Service running from `Temp` | Suspicious executable location |
| Service running from `Users\Public` | User-writable persistence location |
| Service running as `LocalSystem` | High privilege execution |
| Strange command arguments | Possible payload execution |
| Recently created service | Possible persistence creation event |

Suspicious locations I paid attention to:

```text
C:\Users\Public\
C:\Users\<user>\AppData\Local\Temp\
C:\Windows\Temp\
C:\ProgramData\
```

## Service Installation Events

I checked event logs for service installation activity.

Service creation is commonly recorded in the System log with Event ID `7045`.

```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; Id=7045} |
Select-Object TimeCreated, ProviderName, Id, Message
```

When I had a suspected time window, I used a timeboxed query.

```powershell
$start = Get-Date "2026-01-01"
$end = Get-Date "2026-01-02"

Get-WinEvent -FilterHashtable @{
    LogName='System'
    Id=7045
    StartTime=$start
    EndTime=$end
} | Select-Object TimeCreated, Message
```

## Registry Autorun Review

I checked common registry autorun locations because they can launch programs when a user logs in or when the system starts.

I reviewed system-wide Run keys.

```powershell
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

```powershell
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\RunOnce"
```

I also reviewed user-specific Run keys.

```powershell
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

```powershell
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\RunOnce"
```

## Registry Indicators I Reviewed

I looked for autorun values that pointed to suspicious files or commands.

| Indicator | Why I Investigated It |
|---|---|
| Unknown value name | Could be unauthorized persistence |
| Executable in user profile | User-writable location |
| Executable in temp path | Suspicious launch location |
| PowerShell in autorun value | Possible script-based persistence |
| Encoded or hidden arguments | Possible obfuscation |
| Misspelled updater/helper name | Possible masquerading |

Example suspicious pattern:

```text
Updater = C:\Users\Public\update.exe
```

## Local User Review

I reviewed local users because attackers sometimes create new accounts for continued access.

```powershell
Get-LocalUser
```

I filtered for enabled accounts.

```powershell
Get-LocalUser | Where-Object {$_.Enabled -eq $true}
```

I exported local user details.

```powershell
Get-LocalUser |
Select-Object Name, Enabled, LastLogon, PasswordLastSet, Description |
Export-Csv .\local-users.csv -NoTypeInformation
```

## Local Administrator Review

I checked the local Administrators group because persistence often includes privilege changes.

```powershell
Get-LocalGroupMember Administrators
```

I exported the group membership.

```powershell
Get-LocalGroupMember Administrators |
Export-Csv .\local-admins.csv -NoTypeInformation
```

## Account Indicators I Reviewed

I looked for:

| Indicator | Why I Investigated It |
|---|---|
| Unknown enabled user | Possible backdoor account |
| Generic username | Could hide in plain sight |
| New administrator member | Possible privilege persistence |
| Unexpected local account | Could bypass domain controls |
| Recently used account | May support timeline analysis |

Examples of names I would investigate:

```text
backup
support
admin2
svc
helpdesk
```

## Scheduled Task Review

I reviewed scheduled tasks because they can run programs at startup, logon, on a timer, or in response to specific triggers.

I listed scheduled tasks.

```powershell
Get-ScheduledTask
```

I reviewed task names and paths.

```powershell
Get-ScheduledTask |
Select-Object TaskName, TaskPath, State
```

I searched for updater-style names.

```powershell
Get-ScheduledTask *Update*
```

```powershell
Get-ScheduledTask *Chrome*
```

```powershell
Get-ScheduledTask *Security*
```

## Scheduled Task XML Review

When I found a suspicious task, I exported the task XML to review the actual action and arguments.

```powershell
Export-ScheduledTask -TaskName "TASK_NAME"
```

I checked runtime information.

```powershell
Get-ScheduledTaskInfo -TaskName "TASK_NAME" |
Select-Object LastRunTime, NextRunTime, LastTaskResult
```

## Scheduled Task Indicators I Reviewed

I looked for:

| Indicator | Why I Investigated It |
|---|---|
| Misspelled task name | Possible masquerading |
| Task in unusual path | Possible hiding attempt |
| PowerShell action | Possible script execution |
| Command shell action | Possible payload launcher |
| File in temp path | Suspicious executable location |
| Logon/startup trigger | Persistence behavior |
| Frequent execution | Possible beacon or re-run mechanism |

## Event Log Review

I used event logs to support persistence findings and timeline building.

Useful event IDs:

| Event ID | Log | What I Used It For |
|---|---|---|
| 7045 | System | Service installed |
| 4697 | Security | Service installed, when audited |
| 4720 | Security | User account created |
| 4722 | Security | User account enabled |
| 4732 | Security | User added to local group |
| 4698 | Security | Scheduled task created |
| 4702 | Security | Scheduled task updated |

Example queries:

```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; Id=7045}
```

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4720}
```

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4732}
```

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4698}
```

## Baseline Comparison

I practiced differential analysis by comparing the current state against a known-good baseline.

For services, I created a baseline export.

```powershell
Get-CimInstance -ClassName Win32_Service |
Select-Object Name, StartMode, StartName, PathName |
Export-Csv .\baseline-services.csv -NoTypeInformation
```

Then I created a current-state export.

```powershell
Get-CimInstance -ClassName Win32_Service |
Select-Object Name, StartMode, StartName, PathName |
Export-Csv .\current-services.csv -NoTypeInformation
```

I compared the two files.

```powershell
$baseline = Import-Csv .\baseline-services.csv
$current = Import-Csv .\current-services.csv

Compare-Object -ReferenceObject $baseline -DifferenceObject $current -Property Name, StartMode, StartName, PathName
```

This helped me identify new, removed, or changed services more quickly than manual review.

## Evidence I Collected

| Artifact | Why I Collected It |
|---|---|
| Service export | Reviewed service paths and startup behavior |
| Event ID 7045 results | Supported service creation timeline |
| Registry Run key output | Reviewed autorun persistence |
| Local user export | Checked for unauthorized accounts |
| Administrators group export | Checked for privilege changes |
| Scheduled task list | Reviewed configured tasks |
| Scheduled task XML | Reviewed task action and trigger details |
| Event logs | Supported timeline and source analysis |
| File hashes | Identified suspicious payloads |

## Findings Template

I used this structure to document persistence findings.

```text
Finding:
A suspicious persistence mechanism was identified on the Windows Client VM.

Evidence:
- Host:
- Persistence type:
- Name:
- Path:
- Command:
- User/context:
- Trigger/start mode:
- Related event ID:
- Time observed:
- File hash:

Assessment:
Explain why this finding is suspicious.

Next Steps:
- Preserve evidence.
- Check related event logs.
- Search for the same indicator on other hosts.
- Review pfSense logs for related network activity.
- Remove the persistence mechanism only after evidence collection is complete.
```

## Detection Opportunities

This lab gave me detection ideas for:

- New service installation events.
- Services running from user-writable folders.
- Registry Run keys pointing to temp or public directories.
- New local users.
- Local users added to Administrators.
- Scheduled tasks launching PowerShell.
- Scheduled tasks created shortly after suspicious process activity.
- Autoruns containing encoded or hidden command arguments.

## Lessons Learned

This lab reinforced that persistence is often visible in normal Windows management locations.

The most useful investigation points were:

- Service paths.
- Registry autorun values.
- Local administrator membership.
- Scheduled task XML.
- Event log timestamps.
- Baseline comparison.

I also learned that collecting evidence before removing persistence is important because deletion can destroy useful timeline and forensic context.

## Skills Demonstrated

This lab demonstrates practical skills in:

- Windows persistence investigation.
- PowerShell evidence collection.
- Service analysis.
- Registry autorun review.
- Local user and group auditing.
- Scheduled task analysis.
- Event log querying.
- Baseline comparison.
- Persistence detection planning.
- Technical documentation.