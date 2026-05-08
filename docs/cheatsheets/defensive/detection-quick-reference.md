# Detection Quick Reference

---

## Executive Summary

This cheatsheet summarizes high-signal defensive patterns for quickly spotting suspicious activity across endpoints, identity, network, web, DNS, and file transfer logs.

Use it as a fast triage reference when reviewing alerts or building detections.

## High-Signal Windows Events

| Event ID | Meaning | Why It Matters |
|---:|---|---|
| `4624` | Successful logon | Tracks access |
| `4625` | Failed logon | Password attacks |
| `4648` | Explicit credentials used | Lateral movement or runas |
| `4672` | Special privileges assigned | Admin logon |
| `4688` | Process creation | Command execution |
| `4697` | Service installed | Persistence or remote execution |
| `4698` | Scheduled task created | Persistence |
| `4720` | User account created | Unauthorized account creation |
| `4728` | User added to privileged group | Privilege escalation |
| `4732` | User added to local group | Local admin changes |
| `4740` | Account locked out | Password attack signal |
| `4768` | Kerberos TGT requested | Authentication tracking |
| `4769` | Kerberos service ticket requested | Kerberoasting hunting |
| `4771` | Kerberos pre-auth failed | Password guessing |
| `7045` | Service created | Service-based execution |

## Suspicious Parent-Child Processes

| Parent | Child | Possible Meaning |
|---|---|---|
| `winword.exe` | `cmd.exe` / `powershell.exe` | Macro execution |
| `excel.exe` | `cmd.exe` / `powershell.exe` | Macro execution |
| `outlook.exe` | `powershell.exe` | Phishing payload |
| `w3wp.exe` | `cmd.exe` / `powershell.exe` | Web shell |
| `httpd.exe` | `/bin/sh` / `/bin/bash` | Web shell |
| `nginx.exe` | `/bin/sh` / `/bin/bash` | Web shell |
| `explorer.exe` | `rundll32.exe` | Suspicious DLL execution |
| `cmd.exe` | `certutil.exe` | File download or decoding |
| `powershell.exe` | `rundll32.exe` | Payload execution |
| `wmiprvse.exe` | `cmd.exe` / `powershell.exe` | WMI execution |

## Suspicious Command Keywords

```text
Invoke-WebRequest
Invoke-RestMethod
DownloadString
DownloadFile
IEX
FromBase64String
EncodedCommand
certutil
bitsadmin
Start-BitsTransfer
rundll32
regsvr32
mshta
wscript
cscript
schtasks
wmic
```

## File Transfer Indicators

| Indicator | Suspicious Example |
|---|---|
| PowerShell download | `Invoke-WebRequest`, `DownloadString` |
| Certutil download | `certutil -urlcache -split -f` |
| BITS download | `Start-BitsTransfer`, `bitsadmin /transfer` |
| Linux download | `curl`, `wget` |
| Raw TCP transfer | `nc`, `ncat`, `socat` |
| Temporary web server | `python3 -m http.server` |
| Script piping | `curl URL \| bash` |
| Direct IP download | `http://1.2.3.4/file.exe` |

## Suspicious User-Agents

```text
WindowsPowerShell
Microsoft-CryptoAPI
Microsoft BITS
WinHttp.WinHttpRequest
MSIE 7.0
curl
Wget
python-requests
Go-http-client
```

## Common Suspicious Paths

Windows:

```text
C:\Users\Public\
C:\Windows\Temp\
C:\Users\<user>\AppData\Local\Temp\
C:\ProgramData\
```

Linux:

```text
/tmp
/dev/shm
/var/tmp
/home/<user>/Downloads
```

High-signal pattern:

```text
File created in temp path -> executed shortly after
```

## Web Attack Patterns

| Pattern | Possible Attack |
|---|---|
| `../` | Path traversal |
| `%2e%2e%2f` | Encoded traversal |
| `/etc/passwd` | Linux file read attempt |
| `union select` | SQL injection |
| `sleep(` | Time-based SQL injection |
| `<script>` | XSS |
| `cmd=` | Command injection testing |
| `.env` | Config file discovery |
| `.git` | Exposed repository |
| `wp-login.php` | WordPress login targeting |
| `xmlrpc.php` | WordPress abuse |

## DNS Hunting Signals

| Signal | Possible Meaning |
|---|---|
| Long random subdomains | DNS tunneling or malware |
| High NXDOMAIN count | DGA or failed beaconing |
| Rare domain queried by one host | Suspicious callback |
| TXT record spikes | Data transfer or tunneling |
| Dynamic DNS domains | C2 or temporary infrastructure |
| Newly registered domains | Phishing or malware staging |

## Identity Signals

| Signal | Possible Meaning |
|---|---|
| Many failed logons from one host | Password guessing |
| Many users failed from one host | Password spraying |
| Success after failures | Possible compromise |
| Admin logon to workstation | Lateral movement risk |
| Logon type `10` | RDP access |
| Event `4648` | Explicit credential use |
| Event `4672` | Admin privileges assigned |
| Privileged group membership change | Privilege escalation |

## Kerberos Signals

| Event | Suspicious Pattern |
|---:|---|
| `4768` | Many TGT requests for many users |
| `4769` | Many service ticket requests |
| `4769` | RC4 encryption requested where AES is normal |
| `4771` | Pre-auth failures |
| `4769` | Requests for unusual SPNs |

## Network Signals

| Signal | Possible Meaning |
|---|---|
| Internal port scanning | Discovery or lateral movement |
| Workstation to many hosts | Reconnaissance |
| Server browsing internet | Suspicious outbound activity |
| Direct IP web traffic | Payload download or C2 |
| Long-lived external connection | Beacon or tunnel |
| Unusual outbound port | Exfiltration or tunnel |
| SMB between workstations | Lateral movement |

## Persistence Signals

| Mechanism | What to Watch |
|---|---|
| Run keys | New values in `Run` or `RunOnce` |
| Services | New service creation |
| Scheduled tasks | New or modified tasks |
| Startup folders | New files |
| WMI persistence | New event filters/consumers |
| Cron | New or modified cron entries |
| Systemd | New service files |
| SSH keys | New authorized keys |

## Quick Triage Workflow

| Step | Action |
|---|---|
| Identify alert source | Endpoint, network, identity, web, DNS |
| Confirm host and user | Determine ownership and role |
| Review timeline | Look 15–60 minutes before and after |
| Check process chain | Parent, child, command line |
| Check network | Destinations, ports, DNS, User-Agent |
| Check files | Created, modified, executed files |
| Scope | Same user, same file hash, same destination |
| Decide | Benign, suspicious, confirmed incident |

## Notes

- Strong detections combine multiple signals.
- Command line plus parent process is more useful than process name alone.
- Rare behavior from a sensitive host is higher priority.
- A single failed login is usually noise; patterns matter.
- Always compare behavior against the host’s normal role.