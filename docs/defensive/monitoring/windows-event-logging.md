# Windows Event Logging

Windows Event Logs record important operating system, application, and security activity.

For defenders, these logs are a primary source for incident response, endpoint monitoring, forensic analysis, and threat hunting.

!!! note
    Windows logs are usually stored in `.evtx` format and can be viewed with Event Viewer, PowerShell, or forwarded into a SIEM.

---

## Overview

Windows logs can show:

- Logons and failed logons
- Account creation and modification
- Service creation
- Scheduled task creation
- Malware detections
- System startup and shutdown
- Security policy changes
- Process activity when enabled
- PowerShell activity when enabled

Important tools:

| Tool | Purpose |
|---|---|
| Event Viewer | GUI log review |
| Get-WinEvent | PowerShell log querying |
| Windows Event Forwarding | Centralized event collection |
| Sysmon | Enhanced endpoint telemetry |
| ETW | Advanced tracing and telemetry |

---

## Core Log Categories

| Log Name | Description | Security Value |
|---|---|---|
| System | Windows system component events | Service failures, shutdowns, driver issues |
| Security | Audit events | Logons, account changes, privilege use |
| Application | Application-generated events | Crashes, warnings, application errors |
| Forwarded Events | Events collected from remote hosts | Centralized monitoring |

---

## Anatomy of an Event

Common fields:

| Field | Meaning |
|---|---|
| Log Name | Event channel, such as Security |
| Source | Component that generated the event |
| Event ID | Numeric identifier for event type |
| Level | Information, Warning, Error, Critical |
| User | Account context |
| OpCode | Operation performed |
| Keywords | Category, such as Audit Success |
| TimeCreated | When the event occurred |
| Computer | Host that generated the event |

The **Event ID** is one of the most important fields for analysis.

---

## Event ID 4624: Successful Logon

Event ID `4624` records a successful logon.

Important fields:

| Field | Meaning |
|---|---|
| Account Name | User that logged on |
| Source Network Address | Source IP |
| Logon Type | How the user logged on |
| Logon ID | Session identifier |

Common logon types:

| Logon Type | Meaning |
|---:|---|
| 2 | Interactive logon |
| 3 | Network logon |
| 5 | Service logon |
| 9 | New credentials |
| 10 | Remote interactive / RDP |

Examples:

```text
Type 2  -> User at keyboard
Type 3  -> Access to network share
Type 10 -> RDP login
```

---

## Critical Windows Event IDs

### System and Operations

| Event ID | Description | Why It Matters |
|---:|---|---|
| 1074 | System shutdown/restart | Planned restart or suspicious reboot |
| 6005 | Event Log service started | System boot indicator |
| 6006 | Event Log service stopped | Shutdown indicator |
| 7040 | Service status changed | Security service disabled or modified |
| 7045 | Service installed | Malware persistence or admin install |

### Security and Intrusion

| Event ID | Description | Why It Matters |
|---:|---|---|
| 1102 | Audit log cleared | Possible anti-forensics |
| 4624 | Successful logon | User access tracking |
| 4625 | Failed logon | Brute force or password spraying |
| 4648 | Explicit credentials used | RunAs or lateral movement |
| 4698 | Scheduled task created | Persistence |
| 4720 | User account created | Possible backdoor user |
| 4738 | User account changed | Privilege or account manipulation |
| 4742 | Computer account changed | Possible DCShadow indicator |
| 4768 | Kerberos TGT requested | Kerberos authentication |
| 4769 | Kerberos service ticket requested | Kerberoasting / PtT analysis |
| 4771 | Kerberos pre-auth failed | Password guessing |
| 4776 | NTLM authentication | NTLM brute force / spraying |
| 1116 | Malware detected | Windows Defender detection |

---

## XML Filtering in Event Viewer

Event Viewer supports custom XML queries.

Use XML filtering when you need to:

- Filter by fields inside event data
- Correlate a specific logon session
- Exclude noisy logon types
- Search for specific values such as `SubjectLogonId`

Example use case:

```text
Show all successful logons except service logons.
```

XML filtering is more precise than basic GUI filtering.

---

## Sysmon

Sysmon, or System Monitor, is a Windows service and driver that logs detailed endpoint activity to the Windows Event Log.

Sysmon improves visibility into:

- Process creation
- Network connections
- Image loads
- File creation
- Registry activity
- Process access
- Named pipes
- DNS queries

Sysmon is especially useful for detecting attacker behavior that normal Windows logs do not capture well.

---

## Sysmon Architecture

Sysmon consists of:

| Component | Purpose |
|---|---|
| Service | Monitors system activity |
| Driver | Captures low-level events |
| Event Log | Stores generated events |

Sysmon events are commonly written to:

```text
Microsoft-Windows-Sysmon/Operational
```

---

## Installing Sysmon

Example installation:

```powershell
sysmon.exe -i -accepteula -h md5,sha256,imphash -l -n
```

Update configuration:

```powershell
sysmon.exe -c sysmonconfig-export.xml
```

Recommended community configurations:

- SwiftOnSecurity Sysmon config
- Olaf Hartong modular Sysmon config

!!! note
    Sysmon without a tuned configuration can be noisy. Good include/exclude rules are important.

---

## Sysmon Event ID Cheatsheet

| Event ID | Name | Security Use |
|---:|---|---|
| 1 | Process Creation | Malware execution, suspicious command lines |
| 3 | Network Connection | C2, lateral movement, exfiltration |
| 7 | Image Loaded | DLL hijacking, .NET injection |
| 8 | CreateRemoteThread | Process injection |
| 10 | Process Access | LSASS access, credential dumping |
| 11 | File Create | Dropped files, ransomware activity |
| 12/13/14 | Registry Events | Persistence and configuration changes |
| 17/18 | Pipe Events | PsExec and lateral movement |
| 22 | DNS Query | C2, suspicious domains |

---

## Detecting DLL Hijacking

Sysmon Event ID `7` logs image loads.

DLL hijacking indicators:

- Legitimate executable running from unusual path
- System DLL loaded from user-writable directory
- Signed binary loading unsigned DLL
- DLL loaded from Desktop, Downloads, Temp, or AppData

Example suspicious pattern:

```text
calc.exe from Desktop loads WININET.dll from Desktop
```

---

## Detecting Unmanaged Code Injection

C# and .NET payloads need the Common Language Runtime.

Suspicious image loads:

```text
clr.dll
clrjit.dll
mscoree.dll
```

Potential indicator:

```text
spoolsv.exe loads clr.dll
notepad.exe loads clr.dll
```

These processes are usually unmanaged and may not normally load .NET runtime components.

---

## Detecting Credential Dumping

Sysmon Event ID `10` logs process access.

Credential dumping often involves suspicious access to:

```text
lsass.exe
```

Indicators:

- Non-system process accesses LSASS
- Source process from Downloads, Temp, or user profile
- Source user differs from target user context
- Process requests high privileges
- Known tools such as Mimikatz

Example hunting idea:

```text
Event ID 10
TargetImage = lsass.exe
SourceImage not in known security tools
```

---

## Event Tracing for Windows

Event Tracing for Windows, or ETW, is a high-performance tracing system built into Windows.

ETW can provide deep telemetry for:

- Kernel activity
- Process events
- Network events
- File system activity
- Registry activity
- DNS events
- PowerShell
- .NET runtime behavior

ETW is powerful but more complex than standard event logs.

---

## ETW Architecture

ETW uses a publish-subscribe model.

| Component | Purpose |
|---|---|
| Provider | Source that generates events |
| Controller | Starts and stops tracing sessions |
| Consumer | Reads events |
| Session | Buffers and delivers events |

Examples:

```text
Provider: Microsoft-Windows-Kernel-Process
Controller: logman.exe
Consumer: Event Viewer, Sysmon, ProcMon, SilkETW
```

---

## Managing ETW with logman

List running trace sessions:

```cmd
logman query -ets
```

List providers:

```cmd
logman query providers
```

Inspect a provider:

```cmd
logman query providers Microsoft-Windows-Kernel-Process
```

Start a trace session:

```cmd
logman start MyTrace -p Microsoft-Windows-Kernel-Process -ets
```

---

## High-Value ETW Providers

| Provider | Use Case |
|---|---|
| Microsoft-Windows-Kernel-Process | Process tracking and injection analysis |
| Microsoft-Windows-Kernel-Network | Network activity |
| Microsoft-Windows-DNS-Client | DNS queries and tunneling |
| Microsoft-Windows-PowerShell | Script execution |
| Microsoft-Windows-SMBClient | Lateral movement via file shares |
| Microsoft-Windows-DotNETRuntime | .NET assembly execution |
| Microsoft-Windows-Threat-Intelligence | Advanced memory and API telemetry |

---

## ETW and Advanced Detection

ETW can help detect things Sysmon may miss or partially capture.

Examples:

| Technique | ETW Value |
|---|---|
| Parent PID spoofing | Kernel process provider may reveal true process relationships |
| In-memory .NET execution | DotNETRuntime provider can show loaded assemblies and JIT methods |
| DNS tunneling | DNS Client provider can expose query behavior |
| Process injection | Threat Intelligence provider may expose memory operations |

---

## Detecting Parent PID Spoofing

Parent PID spoofing makes a malicious process appear to have a trusted parent.

Example:

```text
Malicious process appears to be spawned by spoolsv.exe
Actual creator may be powershell.exe
```

Sysmon may log the spoofed parent.

ETW kernel process telemetry may reveal the true creator relationship.

---

## Detecting In-Memory .NET Execution

Attackers may execute .NET assemblies in memory using tools like Cobalt Strike `execute-assembly`.

Indicators:

- Usually unmanaged process loads .NET runtime
- `clr.dll` or `mscoree.dll` loaded unexpectedly
- DotNETRuntime events show suspicious method or assembly names
- No corresponding executable written to disk

Useful ETW provider:

```text
Microsoft-Windows-DotNETRuntime
```

Common keywords:

```text
LoaderKeyword
JitKeyword
InteropKeyword
```

---

## Get-WinEvent

`Get-WinEvent` is a PowerShell cmdlet for querying Windows logs.

It can read:

- Classic logs
- Modern event channels
- Sysmon logs
- ETW logs
- Forwarded Events
- Offline `.evtx` files

It is preferred over the older `Get-EventLog`.

---

## Discovering Logs

List all logs:

```powershell
Get-WinEvent -ListLog * |
    Select-Object LogName, RecordCount, IsEnabled |
    Sort-Object RecordCount -Descending
```

List providers:

```powershell
Get-WinEvent -ListProvider *Microsoft-Windows-Sysmon*
```

---

## Efficient Filtering

Prefer `FilterHashtable` over piping everything to `Where-Object`.

Slow pattern:

```powershell
Get-WinEvent -LogName Security | Where-Object {$_.Id -eq 4624}
```

Faster pattern:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624}
```

Why it is faster:

```text
FilterHashtable filters before returning the events.
Where-Object retrieves many events first, then filters locally.
```

---

## Common FilterHashtable Keys

| Key | Purpose |
|---|---|
| `LogName` | Event log name |
| `ID` | Event ID |
| `Path` | Offline `.evtx` file |
| `StartTime` | Start of time range |
| `EndTime` | End of time range |
| `ProviderName` | Event provider |

Example Sysmon query:

```powershell
$Filter = @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    ID      = 1, 3
}

Get-WinEvent -FilterHashtable $Filter |
    Select-Object TimeCreated, Id, LevelDisplayName, Message
```

---

## Offline EVTX Analysis

Read an offline event log:

```powershell
Get-WinEvent -Path "C:\Evidence\suspicious_sysmon.evtx" -MaxEvents 10
```

This is useful during incident response when logs are collected from a compromised host and analyzed on a clean workstation.

---

## XPath Filtering

Use XPath when you need to filter inside event data.

Example: find Sysmon process creation where the image is `reg.exe`.

```powershell
$XPath = "*[EventData[Data[@Name='Image']='C:\Windows\System32\reg.exe']]"

Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath $XPath
```

---

## Parsing Event XML

Events are XML objects. Named fields can be extracted from `EventData`.

Example: parse Sysmon Event ID 3 network connections:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=3} |
ForEach-Object {
    $xml = [xml]$_.ToXml()
    $data = $xml.Event.EventData.Data

    [PSCustomObject]@{
        Time    = $_.TimeCreated
        SrcIP   = ($data | Where-Object Name -eq 'SourceIp').'#text'
        DestIP  = ($data | Where-Object Name -eq 'DestinationIp').'#text'
        Process = ($data | Where-Object Name -eq 'Image').'#text'
    }
}
```

---

## Hunting Encoded PowerShell

Property index access is fast but brittle because indexes may differ by event version.

Example:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} |
    Where-Object {$_.Properties[21].Value -like "*-enc*"} |
    Format-List
```

More robust approaches use XML field names when possible.

---

## Quick Reference

| Goal | Method |
|---|---|
| View logs manually | Event Viewer |
| Query logs with PowerShell | `Get-WinEvent` |
| Read offline logs | `Get-WinEvent -Path file.evtx` |
| Efficient filtering | `FilterHashtable` |
| Advanced filtering | XPath / XML |
| Enhanced telemetry | Sysmon |
| Deep tracing | ETW |
| List ETW providers | `logman query providers` |

---

## Notes to Remember

- Windows Event Logs are central to endpoint investigation.
- Security logs track authentication and account activity.
- Sysmon adds high-value endpoint telemetry.
- Event ID 4624 shows successful logons.
- Event ID 4625 shows failed logons.
- Event ID 7045 and Sysmon process events are useful for persistence detection.
- Sysmon Event ID 10 is useful for LSASS access detection.
- ETW provides deeper telemetry than standard logs.
- `Get-WinEvent` is the preferred PowerShell tool for log analysis.