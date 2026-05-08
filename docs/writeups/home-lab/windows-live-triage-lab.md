# Windows Live Triage Lab

---

## Executive Summary

I used this lab to practice live Windows endpoint triage.

The goal was to investigate a running Windows Client VM, collect volatile evidence, review suspicious process and network activity, and document the findings before taking any containment action.

## Lab Objective

I performed a live triage investigation to practice:

- Collecting basic host context.
- Reviewing running processes.
- Capturing process command-line details.
- Analyzing parent-child process relationships.
- Mapping network connections to process IDs.
- Checking DNS cache activity.
- Validating endpoint observations with pfSense logs.
- Documenting evidence and findings clearly.

## Environment

| System | Role |
|---|---|
| Windows Client VM | Endpoint I investigated |
| Kali VM | Controlled testing system |
| pfSense VM | Firewall/router and network visibility point |
| Analysis VM | Optional evidence review and packet analysis system |

## Network Placement

| System | Network | Example IP |
|---|---|---|
| Kali VM | Attack Network | `10.10.20.10` |
| Windows Client VM | Lab LAN | `10.10.30.10` |
| Analysis VM | Logging Network | `10.10.50.10` |
| pfSense Lab LAN Gateway | Lab LAN | `10.10.30.1` |

## Scenario I Simulated

I treated the Windows Client VM as a system that may have generated suspicious activity.

Instead of immediately stopping processes or deleting files, I focused on collecting evidence from the live system first. This helped me practice a realistic investigation workflow where volatile data matters.

## Tools I Used

| Tool | How I Used It |
|---|---|
| PowerShell | Main triage and evidence collection tool |
| `Get-Process` | Reviewed running processes |
| `Get-CimInstance` | Collected command-line and parent process details |
| `Get-NetTCPConnection` | Reviewed active TCP connections |
| `Get-DnsClientCache` | Reviewed DNS cache entries |
| `Get-FileHash` | Collected hashes for suspicious files |
| pfSense logs | Validated network activity from the endpoint |

## Host Context Collected

I started by collecting basic host and user context.

```powershell
hostname
```

```powershell
whoami
```

```powershell
whoami /groups
```

```powershell
ipconfig /all
```

```powershell
Get-Date
```

I recorded the following details:

| Item | Value |
|---|---|
| Hostname |  |
| Current user |  |
| IP address |  |
| Default gateway |  |
| Domain/workgroup |  |
| Investigation time |  |

## Running Process Review

I reviewed the active process list to understand what was running on the system.

```powershell
Get-Process
```

I then selected fields that were more useful for triage.

```powershell
Get-Process | Select-Object Id, ProcessName, Path
```

To identify noisy or unusual processes, I sorted by CPU usage.

```powershell
Get-Process |
Sort-Object CPU -Descending |
Select-Object -First 10 Id, ProcessName, CPU, Path
```

I also sorted by memory usage.

```powershell
Get-Process |
Sort-Object WorkingSet -Descending |
Select-Object -First 10 Id, ProcessName, WorkingSet, Path
```

Finally, I exported the process list for documentation.

```powershell
Get-Process |
Select-Object Id, ProcessName, Path, CPU, WorkingSet |
Export-Csv .\process-list.csv -NoTypeInformation
```

## Command-Line Review

I used CIM to collect process command-line details because process names alone do not show how a process was launched.

```powershell
Get-CimInstance Win32_Process |
Select-Object ProcessId, ParentProcessId, Name, CommandLine
```

I exported this data for later review.

```powershell
Get-CimInstance Win32_Process |
Select-Object ProcessId, ParentProcessId, Name, CommandLine |
Export-Csv .\process-commandlines.csv -NoTypeInformation
```

This was one of the most useful parts of the triage because it showed execution context, parent process IDs, and arguments passed to binaries.

## Suspicious Command-Line Patterns I Looked For

I reviewed command lines for suspicious use of common Windows binaries.

| Process | What I Looked For |
|---|---|
| `powershell.exe` | Encoded commands, hidden windows, downloads, suspicious URLs |
| `cmd.exe` | Unusual parent process or chained commands |
| `rundll32.exe` | Suspicious DLL paths or exports |
| `regsvr32.exe` | Remote scriptlets or unexpected script execution |
| `mshta.exe` | Remote URLs or unusual HTML application execution |
| `certutil.exe` | Unexpected file download or encoding activity |
| `wscript.exe` / `cscript.exe` | Scripts launched from temp or user directories |

## Parent-Child Process Review

I reviewed parent-child process relationships to identify unusual execution chains.

```powershell
Get-CimInstance Win32_Process |
Select-Object ProcessId, ParentProcessId, Name, CommandLine |
Sort-Object ParentProcessId
```

I looked for relationships that would be unusual on a normal workstation.

| Parent Process | Child Process | Why I Flagged It |
|---|---|---|
| `winword.exe` | `powershell.exe` | Possible macro execution |
| `excel.exe` | `cmd.exe` | Possible macro execution |
| `outlook.exe` | `wscript.exe` | Possible script launched from email |
| `chrome.exe` | Unknown `.exe` | Possible downloaded payload |
| `w3wp.exe` | `cmd.exe` | Possible web shell behavior |
| `lsass.exe` | `cmd.exe` | Highly abnormal process relationship |

## Process Lineage Check

When I wanted to investigate one parent process, I first found its PID.

```powershell
Get-CimInstance Win32_Process |
Where-Object {$_.Name -eq "lsass.exe"} |
Select-Object ProcessId, ParentProcessId, Name, CommandLine
```

Then I searched for child processes spawned by that parent PID.

```powershell
Get-CimInstance Win32_Process |
Where-Object {$_.ParentProcessId -eq 644} |
Select-Object ProcessId, ParentProcessId, Name, CommandLine
```

This helped me practice moving from a suspicious process to its related activity.

## Process Path Review

I paid close attention to where executables were running from.

Suspicious locations included:

```text
C:\Users\Public\
C:\Users\<user>\AppData\Local\Temp\
C:\Windows\Temp\
C:\ProgramData\
```

I also looked for process names that imitated legitimate Windows binaries.

```text
svch0st.exe
scvhost.exe
chromeupdate.exe
winupdate32.exe
a8f9b.exe
```

## Network Connection Review

After reviewing processes, I checked active TCP connections.

```powershell
Get-NetTCPConnection
```

I reviewed listening ports.

```powershell
Get-NetTCPConnection -State Listen |
Select-Object LocalAddress, LocalPort, OwningProcess
```

I reviewed established connections.

```powershell
Get-NetTCPConnection -State Established |
Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess
```

## Mapping Network Connections to Process Names

`Get-NetTCPConnection` shows the owning process ID, but I wanted the process name too.

I used this command to correlate active connections with process names.

```powershell
Get-NetTCPConnection |
Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess,
@{Name="ProcessName";Expression={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).ProcessName}}
```

I exported the results.

```powershell
Get-NetTCPConnection |
Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess,
@{Name="ProcessName";Expression={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).ProcessName}} |
Export-Csv .\network-connections.csv -NoTypeInformation
```

## Network Indicators I Reviewed

I looked for network behavior that did not match normal workstation activity.

| Observation | Why I Investigated It |
|---|---|
| PowerShell connecting outbound | Possible script download or callback |
| Unknown process with outbound traffic | Possible unauthorized process |
| Process listening on a high port | Possible unauthorized listener |
| Workstation connecting to many internal hosts | Possible scanning or lateral movement |
| Repeated short connections | Possible beaconing |
| Direct IP connections | Possible bypass of normal DNS behavior |

## DNS Cache Review

I reviewed DNS cache entries to identify recently resolved domains.

```cmd
ipconfig /displaydns
```

I also used PowerShell.

```powershell
Get-DnsClientCache
```

I looked for:

- Random-looking domains.
- Rare domains.
- Recently resolved internal hosts.
- Dynamic DNS providers.
- Domains that matched suspicious network connections.

## File Hash Collection

When I identified a suspicious process path, I collected a SHA256 hash.

```powershell
Get-FileHash "C:\Path\To\File.exe" -Algorithm SHA256
```

I also reviewed file timestamps.

```powershell
Get-Item "C:\Path\To\File.exe" |
Select-Object FullName, Length, CreationTime, LastWriteTime, LastAccessTime
```

## pfSense Validation

After collecting endpoint data, I used pfSense logs to validate network activity.

I checked for:

- Source IP of the Windows Client VM.
- Destination IP or domain.
- Destination port.
- Timestamp.
- Whether the traffic was allowed or blocked.
- Whether DNS logs matched the endpoint DNS cache.

I documented network evidence in this format:

| Time | Source | Destination | Port | Action | Notes |
|---|---|---|---|---|---|
|  | `10.10.30.10` |  |  |  |  |

## Evidence I Collected

| Artifact | Why I Collected It |
|---|---|
| Host context | Identified the system, user, and time of investigation |
| Process list | Captured active processes |
| Command-line export | Preserved execution context |
| Parent-child process data | Reviewed process lineage |
| Network connections | Captured active communications |
| DNS cache | Reviewed recent name resolution |
| File hashes | Identified suspicious files |
| pfSense logs | Validated network activity |

## Findings Template

I used this structure to document findings.

```text
Finding:
A suspicious process was observed on the Windows Client VM.

Evidence:
- Host:
- User:
- Process:
- PID:
- Parent PID:
- Parent process:
- Command line:
- File path:
- SHA256:
- Remote IP/domain:
- Remote port:
- First observed time:

Assessment:
Explain why this activity was suspicious.

Next Steps:
- Review related Windows events.
- Check pfSense logs.
- Search for the same indicator on other systems.
- Preserve relevant files or screenshots.
```

## Detection Opportunities

This lab gave me detection ideas for:

- PowerShell with encoded command flags.
- Office applications spawning command shells.
- Browsers launching executables from user-writable paths.
- System processes spawning shells.
- Executables running from temp directories.
- Unknown processes with outbound connections.
- Workstations connecting to many internal hosts.

## Lessons Learned

This lab reinforced several important investigation lessons:

- Process names are only a starting point.
- Command-line arguments provide critical context.
- Parent-child process relationships are high-value evidence.
- Network connections should be tied back to process IDs.
- pfSense logs help validate endpoint observations.
- Evidence should be collected before containment actions.

## Skills Demonstrated

This lab demonstrates practical skills in:

- Windows live triage.
- PowerShell-based evidence collection.
- Process analysis.
- Command-line review.
- Parent-child process investigation.
- Network connection analysis.
- DNS cache review.
- Firewall log validation.
- Technical documentation.