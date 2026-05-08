# Splunk

Splunk is a data analytics platform commonly used as a SIEM for searching, investigating, alerting, and visualizing security logs.

Security analysts use Splunk to investigate events across endpoints, servers, identity systems, firewalls, proxies, DNS, and cloud platforms.

---

## Overview

Splunk searches are written with SPL, or Search Processing Language.

SPL uses a pipeline structure:

```spl
search terms
| command
| command
| command
```

Each command receives the output of the previous command.

Example:

```spl
index=main EventCode=4625
| stats count by user, src
| sort - count
```

---

## Basic Search

Search a specific index:

```spl
index=main
```

Search for a keyword:

```spl
index=main "error"
```

Search specific fields:

```spl
index=main EventCode=4624
```

Use Boolean logic:

```spl
index=main EventCode=4624 OR EventCode=4625
```

Exclude values:

```spl
index=main EventCode!=1
```

---

## Performance Tips

Target specific fields whenever possible.

Slow:

```spl
index=main *uniwaldo.local*
```

Better:

```spl
index=main ComputerName="*uniwaldo.local"
```

Good habits:

- Search specific indexes.
- Search specific sourcetypes.
- Search specific time ranges.
- Filter early.
- Use fields rather than broad wildcards.
- Avoid expensive commands until the dataset is reduced.

---

## Common SPL Commands

| Command | Purpose |
|---|---|
| `fields` | Include or exclude fields |
| `table` | Display selected fields |
| `rename` | Rename fields |
| `sort` | Sort results |
| `dedup` | Remove duplicates |
| `stats` | Aggregate data |
| `chart` | Create chart-friendly output |
| `eval` | Create or modify fields |
| `rex` | Extract fields with regex |
| `lookup` | Enrich with external data |
| `transaction` | Group related events |
| `bin` / `bucket` | Group time into buckets |
| `streamstats` | Rolling statistics |
| `eventstats` | Add aggregate stats to events |
| `spath` | Parse JSON/XML-like fields |

---

## Output Fields

Show selected fields:

```spl
index=main EventCode=4624
| table _time, host, user, src, Logon_Type
```

Remove noisy fields:

```spl
index=main
| fields - Message
```

Rename fields:

```spl
index=main EventCode=1
| rename Image as Process
| table _time, host, Process
```

---

## Dedup and Sort

Show one event per process image:

```spl
index=main EventCode=1
| dedup Image
| table _time, host, Image
```

Sort newest first:

```spl
index=main EventCode=1
| sort - _time
```

Sort by count:

```spl
index=main EventCode=1
| stats count by Image
| sort - count
```

---

## Stats

`stats` is one of the most important SPL commands.

Count events by field:

```spl
index=main EventCode=4625
| stats count by user
```

Count by multiple fields:

```spl
index=main EventCode=4625
| stats count by user, src
```

Distinct count:

```spl
index=main EventCode=4625
| stats dc(user) as unique_users by src
```

List values:

```spl
index=main EventCode=4625
| stats values(user) as Users by src
```

---

## Eval

`eval` creates or modifies fields.

Lowercase a process path:

```spl
index=main EventCode=1
| eval Process_Path=lower(Image)
```

Command-line length:

```spl
index=main EventCode=1
| eval cmd_len=len(CommandLine)
| table _time, host, user, cmd_len, CommandLine
| sort - cmd_len
```

Conditional field:

```spl
index=main EventCode=4625
| eval severity=if(count > 10, "high", "normal")
```

---

## Rex

`rex` extracts data with regex.

Example: extract a username before `@`:

```spl
index=main user=*
| rex field=user "(?<username>[^@]+)"
| table user, username
```

Extract a filename from a path:

```spl
index=main EventCode=11
| rex field=TargetFilename "(?<filename>[^\\\]+)$"
| table TargetFilename, filename
```

---

## Lookups

Lookups enrich events with external CSV data.

Example:

```spl
index=main EventCode=1
| lookup malware_lookup.csv filename OUTPUTNEW is_malware
| table _time, host, filename, is_malware
```

Common uses:

- Known bad hashes
- Asset criticality
- User department
- VIP users
- Known admin tools
- Approved software
- Threat intelligence

---

## Transactions

`transaction` groups related events.

Example:

```spl
index=main EventCode=4769 OR EventCode=4648
| transaction user maxspan=5s startswith=(EventCode=4769) endswith=(EventCode=4648)
```

`transaction` can be useful, but it can be expensive on large datasets.

Use it carefully.

---

## Sysmon Event Code Reference

| Event Code | Description | Hunting Use |
|---:|---|---|
| 1 | Process Creation | Command lines, parent-child relationships |
| 3 | Network Connection | C2, exfiltration, lateral movement |
| 7 | Image Loaded | DLL hijacking, .NET injection |
| 8 | CreateRemoteThread | Process injection |
| 10 | Process Access | LSASS access, credential dumping |
| 11 | File Create | Dropped files, archive creation |
| 12/13 | Registry Events | Persistence |
| 17/18 | Pipe Events | PsExec and lateral movement |
| 22 | DNS Query | C2, suspicious domains |

---

## Hunting Parent-Child Relationships

Abnormal parent-child relationships can reveal execution or injection.

Example:

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=1
| stats count by ParentImage, Image
| sort - count
```

Suspicious examples:

```text
winword.exe -> powershell.exe
excel.exe -> cmd.exe
notepad.exe -> powershell.exe
spoolsv.exe -> cmd.exe
```

---

## Detecting Native Recon Commands

Attackers often use built-in Windows tools for reconnaissance.

Common commands:

```text
whoami.exe
ipconfig.exe
net.exe
netstat.exe
nltest.exe
tasklist.exe
arp.exe
ping.exe
```

Detection idea:

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=1
(Image="*\\ipconfig.exe" OR Image="*\\net.exe" OR Image="*\\whoami.exe" OR Image="*\\netstat.exe" OR Image="*\\tasklist.exe" OR Image="*\\nltest.exe")
| stats count by Image, CommandLine, ParentImage, ComputerName, User
| sort - count
```

Burst detection:

```spl
index=main source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventID=1
| search process_name IN (arp.exe,ipconfig.exe,net.exe,nltest.exe,ping.exe,whoami.exe)
  OR (process_name IN (cmd.exe,powershell.exe) AND process IN (*net*,*nltest*,*whoami*))
| stats values(process) as process, min(_time) as _time by parent_process, parent_process_id, dest, user
| where mvcount(process) > 3
```

---

## Detecting Suspicious Downloads

Attackers may use PowerShell or browsers to download payloads.

PowerShell or Mark-of-the-Web example:

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=11
(Image="*powershell.exe*" OR (Image="*msedge.exe" TargetFilename=*"Zone.Identifier"))
| stats count by Image, TargetFilename
| sort - count
```

DNS queries to trusted hosting domains:

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=22 QueryName="*github*"
| stats count by Image, QueryName
```

---

## Detecting Execution from Downloads

User Downloads folders are common payload locations.

```spl
index=main EventCode=1
| regex Image="C:\\\\Users\\\\.*\\\\Downloads\\\\.*"
| stats count by Image, User, ComputerName, ParentImage
```

Investigate:

- Executable file type
- Parent process
- User
- Command line
- Network connections after execution
- File creation after execution

---

## Detecting Archive Creation

Attackers may compress data before exfiltration.

```spl
index=main EventCode=11 (TargetFilename="*.zip" OR TargetFilename="*.rar" OR TargetFilename="*.7z")
| stats count by ComputerName, User, TargetFilename
```

Follow-up:

- Which process created the archive?
- Where was it created?
- Did network upload follow?
- Was the archive created near sensitive directories?

---

## Detecting Non-Standard Ports

Suspicious outbound connections may use unusual ports.

```spl
index=main EventCode=3
NOT (DestinationPort=80 OR DestinationPort=443 OR DestinationPort=22 OR DestinationPort=21)
| stats count by SourceIp, DestinationIp, DestinationPort, Image
| sort - count
```

Investigate:

- Process name
- Destination reputation
- Frequency
- User context
- Parent process
- DNS queries around the same time

---

## Password Spraying

Password spraying attempts a small number of common passwords across many users.

Useful Windows events:

| Event ID | Meaning |
|---:|---|
| 4625 | Failed logon |
| 4768 | Kerberos TGT request |
| 4771 | Kerberos pre-auth failed |
| 4776 | NTLM authentication |
| 4648 | Explicit credentials used |

Detection idea: many distinct users failing from one source in a short window.

```spl
index=main source="WinEventLog:Security" EventCode=4625
| bin span=15m _time
| stats values(user) as Users, dc(user) as dc_user by src, Source_Network_Address, dest, EventCode, Failure_Reason
| where dc_user > 5
```

---

## DCSync

DCSync abuses directory replication permissions to request password data from a Domain Controller.

Useful event:

```text
4662 - Operation was performed on an object
```

Detection idea:

```spl
index=main EventCode=4662 Access_Mask=0x100 Account_Name!=*$
```

Another form:

```spl
index=main EventCode=4662 Message="*Replicating Directory Changes*"
| table _time, user, object_file_name, Object_Server
```

High-risk signs:

- User account performing replication behavior
- Non-DC source
- Replication GUIDs in Properties
- Access to sensitive directory objects

---

## LSASS Access and Credential Dumping

Credential dumping often involves reading LSASS memory.

Sysmon Event ID:

```text
10 - Process Access
```

Basic query:

```spl
index=main EventCode=10 lsass
| stats count by SourceImage
```

More focused idea:

```spl
index=main source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=10 TargetImage="C:\\Windows\\system32\\lsass.exe"
SourceImage!="C:\\ProgramData\\Microsoft\\Windows Defender\\platform\\*\\MsMpEng.exe"
| table _time, Computer, SourceImage, SourceProcessId, TargetImage, GrantedAccess, User
```

Investigate non-security processes touching LSASS.

---

## Pass-the-Hash

Pass-the-Hash uses NTLM hashes to authenticate without knowing the plaintext password.

Detection idea: correlate suspicious LSASS access with Logon Type 9.

```spl
index=main
(source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=10 TargetImage="C:\\Windows\\system32\\lsass.exe" SourceImage!="C:\\ProgramData\\Microsoft\\Windows Defender\\platform\\*\\MsMpEng.exe")
OR
(source="WinEventLog:Security" EventCode=4624 Logon_Type=9 Logon_Process=seclogo)
| transaction host maxspan=1m endswith=(EventCode=4624) startswith=(EventCode=10)
| stats count by _time, Computer, SourceImage, SourceProcessId, Network_Account_Name
```

---

## Pass-the-Ticket

Pass-the-Ticket reuses stolen Kerberos tickets.

Normal Kerberos flow:

```text
4768 -> TGT request
4769 -> TGS request
```

Suspicious pattern:

```text
4769 appears without prior 4768 from same user/source IP
```

Query idea:

```spl
index=main source="WinEventLog:Security" user!=*$ EventCode IN (4768,4769,4770)
| rex field=user "(?<username>[^@]+)"
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"
| transaction username, src_ip_4 maxspan=10h keepevicted=true startswith=(EventCode=4768)
| where closed_txn=0
| search NOT user="*$@*"
| table _time, ComputerName, username, src_ip_4, service_name, category
```

---

## Overpass-the-Hash

Overpass-the-Hash uses a stolen NTLM hash or key to request a Kerberos TGT.

Detection idea: non-LSASS process communicating with a Domain Controller on Kerberos port `88`.

```spl
index=main source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
(EventCode=3 dest_port=88 Image!=*lsass.exe) OR EventCode=1
| eventstats values(process) as process by process_id
| where EventCode=3
| stats count by _time, Computer, dest_ip, dest_port, Image, process
| fields - count
```

---

## Kerberoasting

Kerberoasting requests service tickets for SPN accounts and cracks them offline.

Detection opportunities:

- LDAP queries for accounts with SPNs
- Unusual 4769 volume
- TGS requests not followed by service access

LDAP query hunting:

```spl
index=main source="WinEventLog:SilkService-Log"
| spath input=Message
| rename XmlEventData.* as *
| search SearchFilter="*(&(samAccountType=805306368)(servicePrincipalName=*)*"
```

Missing logon anomaly:

```spl
index=main EventCode=4648 OR (EventCode=4769 AND service_name=iis_svc)
| rex field=user "(?<username>[^@]+)"
| transaction username keepevicted=true maxspan=5s endswith=(EventCode=4648) startswith=(EventCode=4769)
| where closed_txn=0 AND EventCode=4769
```

---

## AS-REPRoasting

AS-REPRoasting targets accounts with Kerberos pre-authentication disabled.

LDAP query for vulnerable accounts:

```spl
index=main source="WinEventLog:SilkService-Log"
| search SearchFilter="*(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304)*"
```

TGT request without pre-authentication:

```spl
index=main source="WinEventLog:Security" EventCode=4768 Pre_Authentication_Type=0
| table _time, src_ip, user, Pre_Authentication_Type, Ticket_Encryption_Type
```

---

## Responder / LLMNR Poisoning

Responder abuses LLMNR/NBT-NS name resolution to capture NetNTLM hashes.

Honeypot detection idea:

```text
A script queries a fake hostname.
If it resolves, something answered when it should not have.
```

Splunk query for custom detection events:

```spl
index=main SourceName=LLMNRDetection
| table _time, ComputerName, SourceName, Message
```

Additional events to review:

| Event | Why |
|---|---|
| Sysmon 22 | DNS query activity |
| 4648 | Explicit credentials |
| Network logs | Connections to rogue server |

---

## PsExec Detection

PsExec is often abused for lateral movement.

Detection opportunities:

| Data Source | Indicator |
|---|---|
| Sysmon 13 | Service registry ImagePath modification |
| Sysmon 11 | Service binary dropped |
| Sysmon 18 | Named pipe activity |
| Sysmon 1 | PsExec-like command execution |

Service registry modification:

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=13 Image="*\\services.exe" TargetObject="HKLM\\System\\CurrentControlSet\\Services\\*\\ImagePath"
| rex field=Details "(?<reg_file_name>[^\\\]+)$"
| stats count by reg_file_name, ComputerName
```

File creation:

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=11 Image=System
| stats count by TargetFilename
```

Named pipes:

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=18 Image=System
| stats count by PipeName
```

Misspelled binaries:

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=1
(CommandLine="*psexe*.exe" OR ParentImage="*psexe*.exe")
NOT (CommandLine="*PSEXESVC.exe" OR CommandLine="*PsExec64.exe")
| table Image, CommandLine
```

---

## SharpHound / BloodHound Recon

SharpHound collects Active Directory relationship data using LDAP.

Detection source:

```text
ETW Microsoft-Windows-LDAP-Client via SilkETW / SilkService
```

Query idea:

```spl
index=main source="WinEventLog:SilkService-Log"
| spath input=Message
| rename XmlEventData.* as *
| table _time, ComputerName, ProcessName, ProcessId, DistinguishedName, SearchFilter
| search SearchFilter="*(samAccountType=805306368)*"
| stats min(_time) as _time, max(_time) as maxTime, count, values(SearchFilter) as SearchFilter by ComputerName, ProcessName, ProcessId
| where count > 10
| convert ctime(maxTime)
```

---

## Golden Ticket

A Golden Ticket is forged using the `KRBTGT` account hash.

Detection is difficult, but may resemble Pass-the-Ticket behavior.

Look for:

- TGS requests missing expected TGT flow
- Strange ticket lifetimes
- Unusual privileged account usage
- Authentication from unusual hosts
- Service tickets for high-value services from odd sources

---

## Silver Ticket

A Silver Ticket is a forged service ticket for a specific service.

Detection idea: logons by users not present in known user inventory.

```spl
index=main EventCode=4624
| stats min(_time) as firstTime by user
| lookup users.csv user as user OUTPUT EventCode as Events
| where isnull(Events)
```

This depends on a reliable lookup of legitimate users.

---

## Delegation Abuse

Delegation lets services act on behalf of users.

Unconstrained delegation recon may appear in PowerShell logs.

Look for terms such as:

```text
TrustedForDelegation
userAccountControl:1.2.840.113556.1.4.803:=524288
```

Constrained delegation abuse may involve non-system processes communicating with Kerberos port `88`, similar to Overpass-the-Hash detection.

---

## DCShadow

DCShadow registers a rogue machine as a Domain Controller and replicates malicious changes.

Detection idea: monitor computer account changes adding Global Catalog SPNs.

```spl
index=main EventCode=4742
| rex field=Message "(?P<gcspn>GC\/[a-zA-Z0-9\.\-\/]+)"
| search gcspn=*
```

---

## Shellcode Detection with CallTrace

Sysmon Event ID `10` includes `CallTrace`.

Suspicious shellcode may show memory regions as:

```text
UNKNOWN
```

Base idea:

```spl
index=main CallTrace="*UNKNOWN*" EventCode=10
| where SourceImage!=TargetImage
```

Noise reduction:

```spl
index=main CallTrace="*UNKNOWN*"
SourceImage!="*Microsoft.NET*" CallTrace!=*ni.dll* CallTrace!=*clr.dll* CallTrace!=*wow64* SourceImage!="C:\\Windows\\Explorer.EXE"
| where SourceImage!=TargetImage
| stats count by SourceImage
```

---

## Anomaly Detection

Known TTP detections are useful, but anomaly detection can find unusual behavior.

Useful commands:

| Command | Use |
|---|---|
| `streamstats` | Rolling statistics |
| `eventstats` | Add aggregate stats to each event |
| `bin` | Time buckets |
| `transaction` | Session grouping |
| `dc()` | Distinct count |

---

## Network Connection Spikes

Detect unusual network activity by process:

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=3
| bin _time span=1h
| stats count as NetworkConnections by _time, Image
| streamstats time_window=24h avg(NetworkConnections) as avg stdev(NetworkConnections) as stdev by Image
| eval isOutlier=if(NetworkConnections > (avg + (0.5*stdev)), 1, 0)
| search isOutlier=1
```

---

## Long Command Lines

Long command lines can indicate encoded PowerShell, obfuscation, or complex payloads.

```spl
index=main sourcetype="WinEventLog:Sysmon" Image=*cmd.exe ParentImage!="*msiexec.exe" ParentImage!="*explorer.exe"
| eval len=len(CommandLine)
| table User, len, CommandLine
| sort - len
```

---

## High DLL Load Volume

Suspicious processes may load many unusual DLLs.

```spl
index=main EventCode=7
NOT (Image="C:\\Windows\\System32*")
| bucket _time span=1h
| stats dc(ImageLoaded) as unique_dlls_loaded by _time, Image
| where unique_dlls_loaded > 3
| sort - unique_dlls_loaded
```

---

## Rapid Process Respawning

Malware may respawn repeatedly for persistence.

```spl
index=main sourcetype="WinEventLog:Sysmon" EventCode=1
| transaction ComputerName, Image
| where mvcount(ProcessGuid) > 1
| stats count by Image, ParentImage
```

---

## Detection Strategy

Use both approaches:

| Approach | Strength | Weakness |
|---|---|---|
| Known TTP | High-fidelity and explainable | Can miss modified attacks |
| Anomaly | Can find unknown behavior | Requires tuning and baselining |

Best practice:

```text
Start with known TTPs.
Add anomaly detections for high-value behaviors.
Tune based on environment.
Document false positives.
```

---

## Investigation Checklist

When a Splunk detection fires:

1. Confirm timestamp and host.
2. Identify user context.
3. Review parent and child processes.
4. Review command line.
5. Check network connections.
6. Check DNS queries.
7. Check file creation.
8. Check authentication events.
9. Search for same behavior on other hosts.
10. Determine true positive or false positive.
11. Document evidence.

---

## Quick Reference

| Goal | SPL Pattern |
|---|---|
| Count failed logons by user | `stats count by user` |
| Password spraying | `dc(user) by src` |
| Parent-child process review | `stats count by ParentImage, Image` |
| LSASS access | `EventCode=10 TargetImage="*lsass.exe"` |
| Process network connections | `EventCode=3` |
| DNS queries | `EventCode=22` |
| Archive creation | `EventCode=11 (*.zip OR *.rar OR *.7z)` |
| Kerberos TGT | `EventCode=4768` |
| Kerberos TGS | `EventCode=4769` |
| DCSync | `EventCode=4662` |
| DCShadow | `EventCode=4742` |

---

## Notes to Remember

- Search specific fields instead of broad wildcards.
- Filter early to improve performance.
- Sysmon Event ID 1 is process creation.
- Sysmon Event ID 3 is network connection.
- Sysmon Event ID 10 is process access.
- Password spraying is detected by many users failing from one source.
- Kerberos attacks often require correlation across events.
- Anomaly detections need tuning.
- Good SPL should be explainable and testable.