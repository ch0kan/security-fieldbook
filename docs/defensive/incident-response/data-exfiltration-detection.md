# Data Exfiltration Detection

Data exfiltration is the unauthorized transfer of data from an organization to an attacker-controlled destination.

It is often one of the final stages of an intrusion and may be used for espionage, fraud, extortion, ransomware pressure, or resale.

---

## Overview

Data exfiltration usually follows this flow:

```text
Discovery
  -> Collection
  -> Staging
  -> Compression / Encoding
  -> Transport
  -> Confirmation
```

Attackers often try to make exfiltration look like normal network traffic.

---

## Exfiltration Phases

| Phase | Description |
|---|---|
| Discovery | Find sensitive files or databases |
| Collection | Gather the data |
| Staging | Move data to a temporary location |
| Compression | ZIP, RAR, 7z, tar, gzip |
| Encoding | Base64, hex, custom encoding |
| Transport | DNS, HTTP, FTP, cloud, email, C2 |
| Confirmation | Attacker verifies receipt |

---

## Common Exfiltration Channels

| Channel | Why Attackers Use It |
|---|---|
| DNS | Often allowed through firewalls |
| HTTP/HTTPS | Blends with web traffic |
| FTP | Built for file transfer |
| Cloud storage | Looks like normal business use |
| Email | Common outbound path |
| C2 channel | Existing attacker infrastructure |
| USB | Useful for insider or physical access |
| SMB/internal shares | Staging before external transfer |

---

## Detection Indicators

| Domain | Indicators | Data Sources |
|---|---|---|
| Network | Large outbound uploads, long sessions, odd ports | Firewall, NetFlow, proxy |
| DNS | Long queries, high entropy, TXT records | DNS logs, PCAP |
| Host | Archive tools, curl/wget, staging folders | Sysmon, EDR, audit logs |
| Cloud | External sharing, unusual uploads | CloudTrail, M365, Google Workspace |
| Identity | New access patterns, off-hours access | IAM, VPN, authentication logs |

---

## Host-Based Indicators

Watch for:

- Archive creation
- Large file reads
- Access to sensitive directories
- Use of compression tools
- Use of transfer tools
- Unusual PowerShell commands
- New staging directories
- USB write events
- Browser uploads after archive creation

Suspicious tools:

```text
7z.exe
rar.exe
winrar.exe
powershell.exe
curl.exe
wget.exe
rclone.exe
ftp.exe
scp.exe
```

---

## Network-Based Indicators

Watch for:

- Large outbound transfers
- Long-duration connections
- Unusual destination countries
- New external destinations
- High bytes sent from one host
- Uploads outside business hours
- Repeated connections to rare domains
- Non-standard ports
- DNS tunneling patterns

---

## DNS Tunneling

DNS tunneling hides data inside DNS queries and responses.

Attackers encode stolen data into subdomains.

Example:

```text
encoded-secret-data.attacker-domain.com
```

The query is routed to the attacker's authoritative DNS server, where the data is decoded.

---

## Why DNS Is Abused

DNS is attractive because:

- It is usually allowed outbound
- It is required for normal operations
- It can pass through permissive firewalls
- Queries are small and frequent by nature
- Security teams may not inspect DNS deeply

---

## DNS Tunneling Indicators

| Indicator | Description |
|---|---|
| High query volume | One host sends many queries |
| Long query names | Subdomains carry encoded data |
| High entropy | Random-looking strings |
| TXT/NULL records | Useful for C2 responses |
| Repeated domain | Many subdomains under one parent domain |
| Regular timing | Beacon-like intervals |
| NXDOMAIN spikes | Failed generated domains |

Suspicious example:

```text
a8f91bc772aa9910cfeab2399.attacker-domain.com
```

---

## Wireshark DNS Investigation

Useful filters:

| Goal | Filter |
|---|---|
| Show DNS | `dns` |
| DNS requests only | `dns.flags.response == 0` |
| DNS responses only | `dns.flags.response == 1` |
| Long DNS packets | `dns && frame.len > 70` |
| Suspicious domain | `dns && dns.qry.name contains "malicious-domain"` |

Workflow:

1. Filter for DNS.
2. Look for long query names.
3. Identify repeated parent domains.
4. Check query timing.
5. Check record types.
6. Identify source hosts.
7. Decode sample payloads if possible.

---

## Splunk DNS Investigation

High-volume DNS sources:

```spl
index="data_exfil" sourcetype="DNS_logs"
| stats count by src_ip
| sort - count
```

Most common queries:

```spl
index="data_exfil" sourcetype="dns_logs"
| stats count by query
| sort - count
```

Long queries:

```spl
index="data_exfil" sourcetype="DNS_logs"
| where len(query) > 30
| table _time, src_ip, query
```

Group by parent domain if available:

```spl
index="data_exfil" sourcetype="DNS_logs"
| stats count, dc(query) as unique_queries by src_ip, domain
| sort - count
```

---

## DNS Tunneling Triage Questions

Ask:

- Which host generated the queries?
- What is the parent domain?
- Are subdomains random-looking?
- How long are the queries?
- Are TXT records involved?
- Is there a regular beacon interval?
- Are there other infected hosts?
- Did the same host create archives before the DNS activity?
- Is the domain newly registered or rare?

---

## FTP Exfiltration

FTP is a legacy file transfer protocol.

It is risky because standard FTP sends credentials and commands in cleartext.

Attackers may use FTP for exfiltration because:

- It supports large file transfers
- It may be allowed for business reasons
- It exposes clear commands like `STOR`
- It can use compromised credentials
- It may run on non-standard ports

---

## FTP Channels

FTP commonly uses two channels:

| Channel | Purpose |
|---|---|
| Control | Commands and authentication |
| Data | File content transfer |

Control channel examples:

```text
USER
PASS
STOR
RETR
LIST
```

---

## FTP Exfiltration Indicators

| Indicator | Meaning |
|---|---|
| `USER` / `PASS` | Cleartext credentials |
| `STOR` | Upload to server |
| Many `STOR` commands | Repeated file uploads |
| Large data transfers | Possible bulk exfiltration |
| Sensitive extensions | `.csv`, `.pdf`, `.txt`, `.sql`, `.zip` |
| Off-hours uploads | Suspicious timing |
| Unusual external IP | Unknown destination |

---

## Wireshark FTP Investigation

Show FTP control and data:

```text
ftp || ftp-data
```

Find credentials:

```text
ftp.request.command == "USER" || ftp.request.command == "PASS"
```

Find uploads:

```text
ftp contains "STOR"
```

Find CSV movement:

```text
ftp contains "csv"
```

Find larger FTP frames:

```text
ftp && frame.len > 90
```

Workflow:

1. Filter `ftp || ftp-data`.
2. Identify usernames and login success.
3. Filter for `STOR`.
4. Identify uploaded filenames.
5. Follow TCP stream.
6. Inspect data channel.
7. Confirm destination IP and transferred content.

---

## FTP Investigation Notes

When following streams, look for:

- Login sequence
- Upload command
- Filename
- Transfer size
- Server response codes
- Cleartext file content
- Suspicious account names
- Repeated uploads

Example suspicious pattern:

```text
USER guest
PASS guest
STOR customer_data.csv
```

---

## Splunk Exfiltration Hunting

Archive creation:

```spl
index=main EventCode=11 (TargetFilename="*.zip" OR TargetFilename="*.rar" OR TargetFilename="*.7z")
| stats count by ComputerName, User, TargetFilename
```

Suspicious transfer tools:

```spl
index=main EventCode=1
(Image="*\\curl.exe" OR Image="*\\wget.exe" OR Image="*\\ftp.exe" OR Image="*\\rclone.exe" OR Image="*\\powershell.exe")
| table _time, ComputerName, User, Image, CommandLine, ParentImage
```

Large outbound connections:

```spl
index=main EventCode=3
| stats sum(SentBytes) as bytes_out by ComputerName, User, DestinationIp, DestinationPort, Image
| sort - bytes_out
```

If `SentBytes` is unavailable, use firewall/proxy/NetFlow logs.

---

## Correlation Strategy

Strong exfiltration detections often require correlation.

Example chain:

```text
Sensitive file access
  -> Archive creation
  -> Transfer tool execution
  -> Large outbound connection
  -> Rare destination
```

Another chain:

```text
Many long DNS queries
  -> One parent domain
  -> Same infected host
  -> Archive created shortly before
```

---

## Data Sources for Exfiltration

| Data Source | Useful For |
|---|---|
| DNS logs | DNS tunneling |
| Proxy logs | Web uploads |
| Firewall logs | Destination and volume |
| NetFlow | Traffic volume and duration |
| Sysmon Event ID 1 | Tool execution |
| Sysmon Event ID 3 | Network connections |
| Sysmon Event ID 11 | File creation |
| EDR | Process/file/network correlation |
| Cloud audit logs | External sharing and uploads |
| DLP | Sensitive data movement |

---

## Response Actions

Potential response actions:

- Isolate affected host
- Disable compromised account
- Block destination IP/domain
- Preserve evidence
- Collect volatile data
- Identify files accessed
- Identify data transferred
- Search for same indicators across environment
- Rotate exposed credentials
- Notify legal/compliance if required

---

## Quick Reference

| Goal | Method |
|---|---|
| Find DNS tunneling in Wireshark | `dns && frame.len > 70` |
| Find DNS requests | `dns.flags.response == 0` |
| Find suspicious DNS in Splunk | `where len(query) > 30` |
| Find FTP uploads | `ftp contains "STOR"` |
| Find FTP credentials | `USER` / `PASS` filter |
| Find archive creation | Sysmon Event ID 11 |
| Find transfer tools | Sysmon Event ID 1 |
| Find outbound connections | Sysmon Event ID 3 / firewall / NetFlow |

---

## Notes to Remember

- Data exfiltration often starts with staging and compression.
- DNS tunneling hides data inside query names.
- Long, random-looking DNS queries are suspicious.
- FTP is cleartext and exposes commands like `STOR`.
- Strong detections correlate host and network evidence.
- Large outbound traffic alone is not enough; context matters.