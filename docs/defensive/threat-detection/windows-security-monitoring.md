# Windows Security Monitoring

Windows security monitoring focuses on detecting suspicious authentication, account changes, initial access, persistence, and attacker activity using Windows Event Logs, Sysmon, and related telemetry.

---

## Core Windows Log Sources

| Log Source | Use |
|---|---|
| Security Log | Authentication, account changes, privilege events |
| System Log | Services, shutdowns, driver/service activity |
| PowerShell Logs | Script execution and suspicious commands |
| Sysmon | Process, network, file, registry, image load events |
| RDP TerminalServices Logs | RDP session activity |

---

# User Management Events

Attackers often manipulate user accounts for persistence.

They may:

- Create new users
- Enable disabled accounts
- Reset passwords
- Add users to privileged groups
- Remove legitimate admins
- Delete evidence-related accounts

---

## Key User Management Event IDs

| Event ID | Description | Security Relevance |
|---|---|---|
| 4720 | User account created | Backdoor account creation |
| 4722 | User account enabled | Dormant account re-enabled |
| 4738 | User account changed | Attribute modification |
| 4723 | Password change attempted | Account takeover or user action |
| 4724 | Password reset attempted | Admin reset or attacker reset |
| 4725 | User account disabled | Sabotage or cleanup |
| 4726 | User account deleted | Cleanup or sabotage |
| 4732 | Member added to security group | Privilege escalation |
| 4733 | Member removed from security group | Defense evasion or sabotage |

---

## Event Anatomy

User-management events usually contain:

| Section | Meaning |
|---|---|
| Subject | The account that performed the action |
| Target / Object | The account that was changed |
| Details | What changed, such as group or attribute |

Important field:

```text
Logon ID
```

The Logon ID can be used to correlate the account change back to the original login event.

---

## Hunting Backdoor Users

Focus on:

```text
4720 - User created
4732 - Added to privileged group
```

Review:

- Was the action authorized?
- Did it happen outside business hours?
- Does the username match naming standards?
- Did a normal user create an admin account?
- Was the account added to Administrators or Remote Desktop Users?
- Was the action performed from a remote session?

---

## Correlating Account Changes

Workflow:

```text
Find 4720 or 4732
  -> Copy Subject Logon ID
  -> Search for 4624 with same Logon ID
  -> Identify source IP, logon type, and host
```

This helps answer:

```text
Who created the account?
Where did they log in from?
Was it remote?
Was the session suspicious?
```

---

## Privileged Group Monitoring

High-risk groups:

```text
Administrators
Domain Admins
Enterprise Admins
Remote Desktop Users
Backup Operators
Account Operators
Server Operators
```

Suspicious event:

```text
4732 - user added to Administrators
```

Critical question:

```text
Was this change expected and approved?
```

---

# Initial Access Monitoring

Initial access is the phase where an attacker gains the first foothold.

Common paths:

| Path | Description |
|---|---|
| Exposed services | RDP, SMB, VPN, web apps |
| Public-facing app exploit | Exploiting exposed HTTP services |
| Phishing | User opens attachment or link |
| Removable media | USB-based execution |
| Stolen credentials | Login with valid accounts |

---

## Exposed Services

Examples:

```text
RDP - 3389
SMB - 445
HTTP - 80
HTTPS - 443
VPN portals
```

Risks:

- Brute force
- Password spraying
- Exploitation of unpatched services
- Credential stuffing
- Ransomware deployment

---

## RDP Risk

Remote Desktop Protocol is commonly abused because it gives the attacker an interactive GUI session.

RDP abuse often follows this pattern:

```text
Scan
  -> Brute force
  -> Successful login
  -> Interactive activity
  -> Tool staging
  -> Privilege escalation
  -> Ransomware or lateral movement
```

---

## RDP Event IDs

| Event ID | Description | Key Use |
|---|---|---|
| 4625 | Failed logon | Brute force detection |
| 4624 | Successful logon | Confirm access |
| 4648 | Explicit credentials used | Possible lateral movement |
| 4778 | RDP session reconnected | Session reuse or hijacking |

Important logon types:

| Logon Type | Meaning |
|---|---|
| 3 | Network logon |
| 10 | RemoteInteractive / RDP |

---

## Detecting RDP Brute Force

Look for:

- Many 4625 events
- Same source IP
- Many usernames
- Short time window
- Logon Type 3 or 10
- Followed by a 4624 success

Investigation pattern:

```text
4625 spike from external IP
  -> 4624 Type 10 from same IP
  -> compromised account identified
```

---

## Confirming Successful RDP Access

Use Event ID 4624.

Key fields:

| Field | Use |
|---|---|
| Account Name | Compromised user |
| Logon Type | Type 10 confirms RDP |
| Source Network Address | Attacker IP |
| Logon ID | Correlate session activity |

---

## Correlating Activity After RDP Login

Workflow:

```text
Find 4624 Type 10
  -> Copy Logon ID
  -> Search process events with same Logon ID
  -> Review commands and child processes
```

Look for:

```text
whoami.exe
ipconfig.exe
net.exe
powershell.exe
cmd.exe
mimikatz.exe
rundll32.exe
certutil.exe
schtasks.exe
```

---

## Suspicious RDP Follow-On Activity

After successful RDP, attackers often:

- Run discovery commands
- Download tools
- Disable security controls
- Create users
- Add users to groups
- Dump credentials
- Move laterally
- Deploy ransomware

---

# Useful Windows Security Event IDs

## Authentication

| Event ID | Description |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4634 | Logoff |
| 4648 | Explicit credentials used |
| 4672 | Special privileges assigned |

## Account Management

| Event ID | Description |
|---|---|
| 4720 | User created |
| 4722 | User enabled |
| 4723 | Password change attempted |
| 4724 | Password reset attempted |
| 4725 | User disabled |
| 4726 | User deleted |
| 4732 | Added to local security group |
| 4733 | Removed from local security group |
| 4738 | User changed |

## Scheduled Tasks and Persistence

| Event ID | Description |
|---|---|
| 4698 | Scheduled task created |
| 4699 | Scheduled task deleted |
| 4700 | Scheduled task enabled |
| 4701 | Scheduled task disabled |
| 4702 | Scheduled task updated |

---

# Sysmon Monitoring

Sysmon adds richer telemetry than standard Windows logs.

Useful Sysmon events:

| Sysmon ID | Description | Use |
|---|---|---|
| 1 | Process creation | Command-line and parent-child analysis |
| 3 | Network connection | C2 and lateral movement |
| 7 | Image loaded | DLL abuse and injection clues |
| 10 | Process access | LSASS access and injection |
| 11 | File create | Dropped tools and payloads |
| 12/13/14 | Registry events | Persistence and configuration changes |
| 22 | DNS query | C2 and suspicious domains |

---

## Process Creation Monitoring

With Sysmon Event ID 1 or Security Event ID 4688, monitor:

- Parent process
- Child process
- Command line
- User
- Current directory
- Hash
- Integrity level

Suspicious parent-child examples:

| Parent | Child | Possible Meaning |
|---|---|---|
| `winword.exe` | `powershell.exe` | Malicious macro |
| `w3wp.exe` | `cmd.exe` | Web shell |
| `explorer.exe` | `rundll32.exe` from Temp | Suspicious DLL execution |
| `wmic.exe` | `powershell.exe` | Remote execution |
| `mshta.exe` | `powershell.exe` | Script proxy execution |

---

## Windows Persistence Indicators

Monitor for:

- New local users
- Users added to administrators
- Scheduled task creation
- Service installation
- Run key modification
- Startup folder changes
- RDP enablement
- Firewall rule changes

Common locations:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

---

## High-Fidelity Hunting Questions

Ask:

- Was a new user created unexpectedly?
- Was a user added to Administrators?
- Was RDP used from an unknown IP?
- Did failed logins precede a successful login?
- Did Office spawn a shell?
- Did a web server process spawn `cmd.exe`?
- Did PowerShell use encoded commands?
- Did `certutil` download a file?
- Did a scheduled task run from Temp or Public?
- Did a non-security process access LSASS?

---

## Example Investigation: Backdoor User

```text
1. Event 4720 shows user "svc_backup" created.
2. Event 4732 shows "svc_backup" added to Administrators.
3. Subject Logon ID links to 4624 Type 10.
4. Source IP is external and unknown.
5. Sysmon shows PowerShell and net.exe activity in same session.
```

Conclusion:

```text
Likely attacker-created persistence account after RDP compromise.
```

---

## Example Investigation: RDP Brute Force

```text
1. Many 4625 events from one IP.
2. Account names vary.
3. A 4624 Type 10 follows.
4. Same source IP appears.
5. Process activity begins under compromised user.
```

Conclusion:

```text
Password guessing likely succeeded and led to interactive access.
```

---

## Quick Reference

| Goal | Event / Source |
|---|---|
| Failed login spike | 4625 |
| Successful RDP login | 4624 Type 10 |
| Explicit credential use | 4648 |
| User created | 4720 |
| User added to group | 4732 |
| Scheduled task created | 4698 |
| Process creation | Sysmon 1 / Security 4688 |
| Network connection | Sysmon 3 |
| LSASS access | Sysmon 10 |
| DNS query | Sysmon 22 |

---

## Notes to Remember

- User creation and group changes are common persistence methods.
- RDP brute force often shows many 4625s followed by one 4624.
- Logon ID is useful for correlating actions inside one session.
- Sysmon provides richer process and network visibility than default logs.
- Alerting on a single event is weaker than correlating events into an attack story.