# Log Hunting

---

## Executive Summary

Log hunting is the process of reviewing security telemetry to identify suspicious behavior, validate alerts, and investigate incidents.

Use this cheatsheet for quick ideas on what to look for across Windows, Linux, web, DNS, proxy, firewall, and endpoint logs.

## Core Hunting Questions

| Question | Purpose |
|---|---|
| Who performed the action? | Identify user or account |
| What process ran? | Understand execution |
| When did it happen? | Build timeline |
| Where did it happen? | Identify host and location |
| What changed? | Find impact |
| What connected? | Identify network activity |
| What happened next? | Determine follow-on activity |

## High-Value Log Sources

| Source | Useful For |
|---|---|
| Windows Security logs | Logons, privilege use, account activity |
| Sysmon | Process creation, network connections, file writes |
| PowerShell logs | Script execution and suspicious commands |
| Linux auth logs | SSH, sudo, failed logins |
| Web server logs | Web attacks and suspicious requests |
| DNS logs | Malware callbacks, tunneling, unusual domains |
| Proxy logs | Downloads, User-Agents, web destinations |
| Firewall logs | Allowed/blocked traffic |
| EDR logs | Process, file, registry, and network telemetry |
| Cloud logs | Identity, API, and resource activity |

## Windows Event IDs

| Event ID | Meaning |
|---:|---|
| `4624` | Successful logon |
| `4625` | Failed logon |
| `4634` | Logoff |
| `4648` | Explicit credentials used |
| `4672` | Special privileges assigned |
| `4688` | Process creation |
| `4697` | Service installed |
| `4720` | User account created |
| `4722` | User account enabled |
| `4728` | User added to privileged group |
| `4732` | User added to local group |
| `4740` | Account locked out |
| `4768` | Kerberos TGT requested |
| `4769` | Kerberos service ticket requested |
| `4771` | Kerberos pre-authentication failed |
| `7045` | Service installed |

## Logon Types

| Type | Meaning |
|---:|---|
| `2` | Interactive logon |
| `3` | Network logon |
| `4` | Batch logon |
| `5` | Service logon |
| `7` | Unlock |
| `10` | Remote interactive / RDP |
| `11` | Cached interactive |

## Windows Process Hunting

Look for:

- Office spawning shells.
- Browsers spawning command interpreters.
- PowerShell downloading content.
- `certutil` contacting external hosts.
- `rundll32` loading unusual DLLs.
- `regsvr32` reaching the network.
- `mshta` executing remote content.
- `wscript` or `cscript` running suspicious scripts.
- Services created from unusual paths.
- Executables launched from temp directories.

Suspicious parent-child examples:

```text
winword.exe -> powershell.exe
excel.exe -> cmd.exe
w3wp.exe -> cmd.exe
powershell.exe -> rundll32.exe
cmd.exe -> certutil.exe
```

## Linux Log Hunting

Common logs:

```text
/var/log/auth.log
/var/log/syslog
/var/log/secure
/var/log/audit/audit.log
/var/log/nginx/access.log
/var/log/apache2/access.log
```

Useful searches:

```bash
grep -i "failed password" /var/log/auth.log
```

```bash
grep -i "accepted password" /var/log/auth.log
```

```bash
grep -i "sudo" /var/log/auth.log
```

```bash
grep -i "session opened" /var/log/auth.log
```

Look for:

- Failed SSH bursts.
- Successful login after many failures.
- New sudo activity.
- New users or groups.
- Commands run from `/tmp` or `/dev/shm`.
- Suspicious cron jobs.
- Unexpected listening ports.
- Web server spawning shells.

## Web Log Hunting

Common suspicious patterns:

| Pattern | Possible Meaning |
|---|---|
| `../` | Path traversal |
| `%2e%2e%2f` | Encoded path traversal |
| `' OR '1'='1` | SQL injection attempt |
| `<script>` | XSS attempt |
| `cmd=` | Command execution testing |
| `wp-login.php` | WordPress login targeting |
| `/.env` | Config file discovery |
| `/phpmyadmin` | Admin tool discovery |
| `union select` | SQL injection |
| `../../etc/passwd` | Linux file read attempt |

Useful commands:

```bash
grep -Ei "union|select|sleep|benchmark" access.log
```

```bash
grep -Ei "\.\./|%2e%2e%2f|etc/passwd" access.log
```

```bash
grep -Ei "wp-login|xmlrpc|phpmyadmin|\.env" access.log
```

## DNS Hunting

Look for:

- Long random subdomains.
- High query volume from one host.
- Rare domains.
- Newly registered domains.
- Repeated NXDOMAIN responses.
- DNS queries to dynamic DNS providers.
- TXT record abuse.
- Domains that match malware indicators.

Suspicious examples:

```text
asdj129asd9as.example.com
a1b2c3d4e5f6.bad-domain.tld
```

## Proxy and Web Traffic Hunting

Look for:

- Direct IP downloads.
- Rare User-Agent strings.
- Command-line tool User-Agents.
- Large outbound uploads.
- Downloads of executables or scripts.
- Access to file-sharing sites from servers.
- Connections to newly registered domains.
- Repeated blocked requests.

Suspicious User-Agents:

```text
WindowsPowerShell
Microsoft-CryptoAPI
Microsoft BITS
curl
Wget
python-requests
WinHttp.WinHttpRequest
```

## Firewall Hunting

Look for:

- Denied outbound connections.
- New listening services.
- Workstations connecting to server management ports.
- Servers browsing the internet.
- Internal scanning patterns.
- Repeated connection attempts to many ports.
- Connections to known bad IPs.
- Unusual geographic destinations.

## Timeline Building

Useful fields:

| Field | Purpose |
|---|---|
| Timestamp | Order events |
| Hostname | Identify affected system |
| User | Identify account |
| Process | Identify activity |
| Command line | Understand intent |
| Source IP | Identify origin |
| Destination IP | Identify target |
| File path | Identify artifacts |
| Hash | Correlate files |
| Event ID | Classify activity |

## Investigation Workflow

| Step | Action |
|---|---|
| Start with alert | Identify the triggering event |
| Validate | Confirm whether behavior is expected |
| Scope host | Review nearby process, network, and file events |
| Scope user | Check logons and activity across hosts |
| Scope network | Review destination IPs, domains, and ports |
| Build timeline | Order key events |
| Identify impact | Determine what changed or executed |
| Document | Save queries, evidence, and findings |

## Notes

- One event rarely tells the whole story.
- Process ancestry is often more useful than process name alone.
- Combine endpoint, DNS, proxy, and authentication logs.
- Compare behavior against the normal role of the host.
- Servers browsing the internet directly are often high-signal.