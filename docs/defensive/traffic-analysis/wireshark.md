# Wireshark

Wireshark is a graphical packet analysis tool used to inspect network traffic.

It helps analysts understand conversations, protocols, payloads, anomalies, and attacks by reading packets from live captures or saved PCAP files.

!!! warning
    Analyze traffic only from systems and networks where you have permission.

---

## Overview

Network logs can show who talked to whom and when, but packet captures can show what was actually transmitted.

Wireshark can help with:

- Incident response
- Malware traffic analysis
- Protocol analysis
- File extraction
- Cleartext credential discovery
- C2 investigation
- Data exfiltration analysis
- Scan detection
- ARP poisoning detection
- Host and user identification

---

## Why Network Traffic Analysis Matters

Network Traffic Analysis, or NTA, gives deeper visibility than basic logs.

Logs may show:

```text
Source IP
Destination IP
Port
Timestamp
Domain
Action
```

Packet captures may show:

```text
Headers
Payloads
Commands
Files
Credentials
Protocol behavior
Timing
Malformed packets
Tunneled data
```

NTA can confirm whether an alert is a true positive or just background noise.

---

## Anatomy of Traffic

Traffic can be analyzed by network layer.

| Layer | Examples | What Analysts Look For |
|---|---|---|
| Application | HTTP, DNS, FTP | Commands, files, credentials, C2 |
| Transport | TCP, UDP | Ports, flags, sessions |
| Internet | IP, ICMP | Source/destination, fragmentation |
| Link | Ethernet, ARP | MAC addresses, spoofing, local attacks |

---

## Capture Filters vs Display Filters

Wireshark uses two main filter types.

| Filter Type | Applied When | Syntax | Purpose |
|---|---|---|---|
| Capture filter | Before capture | BPF | Decides what gets saved |
| Display filter | After capture | Wireshark syntax | Hides/shows packets during analysis |

Example capture filter:

```text
tcp port 80
```

Example display filter:

```text
tcp.port == 80
```

!!! important
    Capture filters permanently exclude packets from the capture. Display filters only hide packets from view.

---

## Statistics Menu

The Statistics menu gives a high-level view before deep packet analysis.

Useful views:

| View | Purpose |
|---|---|
| Protocol Hierarchy | Shows protocol distribution |
| Conversations | Shows traffic between endpoint pairs |
| Endpoints | Lists hosts and traffic volume |
| Resolved Addresses | Shows resolved IPs and hostnames |

Use these to find:

- Top talkers
- Unusual protocols
- High-volume connections
- Suspicious external hosts
- Internal systems communicating unexpectedly

---

## Analyze Menu

The Analyze menu helps inspect packet relationships and issues.

Useful features:

| Feature | Purpose |
|---|---|
| Follow TCP Stream | Reconstruct TCP conversation |
| Follow UDP Stream | Reconstruct UDP conversation |
| Expert Information | Shows warnings, errors, retransmissions |
| Display Filter Expression | Helps build filters |

---

## Display Filter Basics

Common comparison operators:

| Operator | Meaning |
|---|---|
| `==` | Equal |
| `!=` | Not equal |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal |
| `<=` | Less than or equal |

Logical operators:

| Operator | Meaning |
|---|---|
| `&&` | AND |
| `||` | OR |
| `!` | NOT |

Example:

```text
(ip.src == 10.10.10.100) && (tcp.port == 443)
```

---

## IP Filters

| Goal | Display Filter |
|---|---|
| IPv4 packets | `ip` |
| Traffic involving IP | `ip.addr == 10.10.10.111` |
| Source IP | `ip.src == 10.10.10.111` |
| Destination IP | `ip.dst == 10.10.10.111` |
| Subnet | `ip.addr == 10.10.10.0/24` |

Use `ip.addr` for full conversations. Use `ip.src` or `ip.dst` when direction matters.

---

## TCP and UDP Filters

| Goal | TCP Filter | UDP Filter |
|---|---|---|
| Any traffic on port | `tcp.port == 80` | `udp.port == 53` |
| Source port | `tcp.srcport == 1234` | `udp.srcport == 1234` |
| Destination port | `tcp.dstport == 80` | `udp.dstport == 53` |

Multiple ports:

```text
tcp.port in {80 443 8080}
```

---

## HTTP Filters

| Goal | Display Filter |
|---|---|
| All HTTP | `http` |
| HTTP requests | `http.request` |
| GET requests | `http.request.method == "GET"` |
| POST requests | `http.request.method == "POST"` |
| Successful responses | `http.response.code == 200` |
| Redirects | `http.response.code == 301 || http.response.code == 302` |
| Client errors | `http.response.code >= 400 && http.response.code < 500` |
| Server errors | `http.response.code >= 500` |

---

## DNS Filters

| Goal | Display Filter |
|---|---|
| All DNS | `dns` |
| DNS queries | `dns.flags.response == 0` |
| DNS responses | `dns.flags.response == 1` |
| A record queries | `dns.qry.type == 1` |
| Long query names | `dns.qry.name.len > 15 and !mdns` |
| dnscat indicators | `dns contains "dnscat"` |

---

## Advanced Filter Operators

| Operator / Function | Purpose | Example |
|---|---|---|
| `contains` | Case-sensitive search | `http.server contains "Apache"` |
| `matches` | Regex search | `http.host matches "\\.(php|html)"` |
| `in` | Match set/range | `tcp.port in {80 443 8080}` |
| `upper()` | Convert to uppercase | `upper(http.server) contains "APACHE"` |
| `lower()` | Convert to lowercase | `lower(http.server) contains "apache"` |
| `string()` | Convert value to string | `string(frame.number) matches "[13579]$"` |

---

## Following Streams

Wireshark can reassemble packets into conversations.

To follow a TCP stream:

```text
Right-click packet -> Follow -> TCP Stream
```

Useful for:

- Reading HTTP requests
- Viewing FTP commands
- Inspecting cleartext logins
- Reconstructing application conversations
- Understanding client/server flow

Filter for a specific stream:

```text
tcp.stream eq 5
```

---

## File Extraction

Wireshark can extract transferred files from some unencrypted protocols.

Automatic extraction:

```text
File -> Export Objects -> HTTP
File -> Export Objects -> SMB
```

Manual extraction may be needed for protocols like FTP.

!!! warning
    Treat extracted files as suspicious. Do not open unknown files directly on your host system.

---

## FTP Analysis

FTP is a cleartext protocol. Usernames, passwords, commands, and file transfers may be visible.

Useful filters:

| Goal | Filter |
|---|---|
| All FTP control traffic | `ftp` |
| FTP commands | `ftp.request.command` |
| Usernames | `ftp.request.command == "USER"` |
| Passwords | `ftp.request.command == "PASS"` |
| Login success | `ftp.response.code == 230` |
| Login failure | `ftp.response.code == 530` |
| File download | `ftp.request.command == "RETR"` |
| File upload | `ftp.request.command == "STOR"` |
| FTP data | `ftp-data` |

Credential filter:

```text
ftp.request.command == "USER" || ftp.request.command == "PASS"
```

Brute-force indicator:

```text
ftp.response.code == 530
```

---

## FTP File Extraction

FTP uses a control channel and a data channel.

Typical process:

1. Filter with `ftp`.
2. Find `RETR` or `STOR` commands.
3. Identify the related `ftp-data` stream.
4. Right-click the data packet.
5. Follow TCP Stream.
6. Change format to Raw.
7. Save the file with the original extension.

---

## HTTP Analysis

HTTP is unencrypted and often easy to inspect.

Useful filters:

```text
http.request
http.request.method == "GET"
http.request.method == "POST"
http.response.code == 200
http.user_agent
```

Things to look for:

- Suspicious User-Agent strings
- Tool signatures
- Login forms
- File downloads
- Cleartext credentials
- Exploit payloads
- Unusual POST requests
- Repeated 404 responses from scanning

Suspicious User-Agent hunting:

```text
(http.user_agent contains "sqlmap") or (http.user_agent contains "Nmap") or (http.user_agent contains "Wfuzz") or (http.user_agent contains "Nikto")
```

---

## Log4j Traffic Pattern

Log4j exploitation attempts often include JNDI strings in HTTP headers or request bodies.

Common indicators:

```text
jndi:ldap
jndi:rmi
Exploit.class
```

Useful filter idea:

```text
http contains "jndi"
```

Also check:

- User-Agent
- X-Forwarded-For
- Referer
- POST body
- Query string

---

## HTTPS and TLS Analysis

HTTPS uses TLS to encrypt traffic.

Before decryption, payloads appear as:

```text
TLS Application Data
```

Useful filters:

```text
tls
tls.handshake.type == 1
tls.handshake.type == 2
```

| TLS Handshake Type | Meaning |
|---|---|
| `1` | Client Hello |
| `2` | Server Hello |

Client Hello filter:

```text
tls.handshake.type == 1
```

Server Hello filter:

```text
tls.handshake.type == 2
```

---

## TLS Decryption with SSLKEYLOGFILE

Modern TLS usually requires session keys to decrypt traffic.

Browsers like Chrome and Firefox can write session secrets to a file using:

```text
SSLKEYLOGFILE
```

Important points:

- The key log file must be created during the session.
- It cannot usually be generated afterward.
- The PCAP and key log file must match the same session.

Wireshark setup:

```text
Edit -> Preferences -> Protocols -> TLS
(Pre)-Master-Secret log filename -> select sslkeys.log
```

After decryption, Wireshark may reveal:

- HTTP requests
- URLs
- Headers
- Bodies
- Reassembled streams

---

## Nmap Scan Identification

Nmap scans create recognizable packet patterns.

### TCP Connect Scan

Nmap option:

```text
-sT
```

Pattern for open port:

```text
SYN -> SYN/ACK -> ACK -> RST/ACK
```

Filter idea:

```text
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024
```

### TCP SYN Scan

Nmap option:

```text
-sS
```

Pattern for open port:

```text
SYN -> SYN/ACK -> RST
```

Filter idea:

```text
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024
```

### UDP Scan

Nmap option:

```text
-sU
```

Closed UDP ports often generate ICMP Port Unreachable.

Filter:

```text
icmp.type==3 and icmp.code==3
```

---

## TCP Flag Filters

| Goal | Filter |
|---|---|
| SYN packets | `tcp.flags.syn == 1` |
| SYN without ACK | `tcp.flags.syn == 1 and tcp.flags.ack == 0` |
| SYN-ACK | `tcp.flags.syn == 1 and tcp.flags.ack == 1` |
| RST packets | `tcp.flags.reset == 1` |
| ACK packets | `tcp.flags.ack == 1` |

Numeric equivalents:

| Packet Type | Filter |
|---|---|
| SYN | `tcp.flags == 2` |
| ACK | `tcp.flags == 16` |
| SYN-ACK | `tcp.flags == 18` |
| RST | `tcp.flags == 4` |
| RST-ACK | `tcp.flags == 20` |

---

## ARP Poisoning

ARP maps IP addresses to MAC addresses on a local network.

ARP poisoning happens when an attacker sends false ARP messages so victims associate the attacker's MAC address with a legitimate IP, often the gateway.

Useful filters:

| Goal | Filter |
|---|---|
| All ARP | `arp` |
| ARP requests | `arp.opcode == 1` |
| ARP replies | `arp.opcode == 2` |
| Duplicate IP detection | `arp.duplicate-address-detected` |
| ARP scanning | `arp.dst.hw_mac==00:00:00:00:00:00` |

Indicators:

- Two MAC addresses claim the same IP
- Gateway IP maps to suspicious MAC
- Many ARP replies from one host
- Victim traffic goes to attacker's MAC address

---

## Host Identification

Network traffic can reveal hostnames and users.

Useful protocols:

| Protocol | Reveals |
|---|---|
| DHCP | Hostname, requested IP, client MAC |
| NBNS | Hostnames on local networks |
| Kerberos | Usernames, machine accounts, realm/domain |

---

## DHCP Analysis

Filters:

```text
dhcp
bootp
```

Useful fields:

| Field | Meaning |
|---|---|
| `dhcp.option.hostname` | Client hostname |
| `dhcp.option.requested_ip_address` | Requested IP |
| `dhcp.option.client_mac_address` | Client MAC |

Useful packet types:

| Type | Meaning |
|---|---|
| Request | Client asks for IP |
| ACK | Server grants IP |
| NAK | Server denies request |

---

## NetBIOS / NBNS Analysis

Filters:

```text
nbns
nbns.name contains "keyword"
```

Useful for:

- Hostname discovery
- Local network identification
- Mapping hostnames to IPs

---

## Kerberos Analysis

Kerberos is common in Windows Active Directory environments.

Filter:

```text
kerberos
```

Useful fields:

| Field | Meaning |
|---|---|
| `kerberos.CNameString` | Client name |
| `kerberos.realm` | Domain/realm |
| `kerberos.addresses` | Client addresses |

Exclude machine accounts:

```text
kerberos.CNameString and !(kerberos.CNameString contains "$")
```

Machine accounts usually end with `$`.

---

## ICMP Tunneling

Attackers may hide data inside ICMP Echo Request or Reply payloads.

Indicators:

- Large ICMP payloads
- High ICMP volume
- Repeated ICMP to one external IP
- Encapsulated protocol data inside ICMP
- Unusual ping patterns

Filters:

```text
icmp
data.len > 64 and icmp
```

---

## DNS Tunneling

DNS tunneling hides data in DNS queries and responses.

Indicators:

- Long query names
- High-entropy subdomains
- Many queries to one domain
- TXT records used for commands
- Known tool strings such as dnscat or iodine
- Unusual query volume

Filters:

```text
dns.qry.name.len > 15 and !mdns
dns contains "dnscat"
dns
```

Example suspicious TXT response:

```text
TXT: "SSBsb3ZlIHlvdXIgY3VyaW91c2l0eQ=="
```

Base64-like values in DNS responses may indicate C2 or tunneling.

---

## Fragmentation Attacks

IP fragmentation can be abused to evade detection.

Things to inspect:

- Fragment offset
- Total length
- Overlapping fragments
- Reassembly behavior

Suspicious pattern:

```text
Fragment 1: Offset 0
Fragment 2: Offset 1480
Fragment 3: Offset 1480 but different data
```

Overlapping fragments may indicate evasion attempts.

---

## Session Hijacking Indicators

Session hijacking may involve abnormal TCP sequence behavior.

Indicators:

- Sudden sequence number jumps
- Packets from unexpected IPs
- Duplicate ACKs or resets
- Strange session interruption
- Unexpected data injection

Use stream analysis and TCP expert info to inspect anomalies.

---

## Workflow Optimization

Useful Wireshark features:

| Feature | Use |
|---|---|
| Filter buttons | Save common filters |
| Bookmarks | Save useful filter expressions |
| Profiles | Save layouts, columns, colors, filters |
| Custom columns | Show fields like hostname, user, stream ID |
| Coloring rules | Highlight suspicious protocols or errors |

Profiles are useful because malware analysis, SOC triage, and network troubleshooting may need different layouts.

---

## Practical Investigation Workflow

When investigating suspicious traffic:

1. Open the PCAP.
2. Check Protocol Hierarchy.
3. Check Conversations.
4. Check Endpoints.
5. Identify top talkers.
6. Filter on suspicious host.
7. Review DNS and HTTP.
8. Follow streams.
9. Check for files or credentials.
10. Look for tunneling or beaconing.
11. Extract artifacts safely.
12. Document evidence.

Example workflow for suspicious IP:

```text
ip.addr == 192.168.1.55
dns
http
tcp.stream eq X
kerberos
dhcp || nbns
```

---

## Quick Reference

| Goal | Filter / Feature |
|---|---|
| Host traffic | `ip.addr == IP` |
| DNS | `dns` |
| HTTP requests | `http.request` |
| HTTP POST | `http.request.method == "POST"` |
| FTP credentials | `ftp.request.command == "USER" || ftp.request.command == "PASS"` |
| TCP stream | `tcp.stream eq N` |
| ARP spoofing | `arp.duplicate-address-detected` |
| UDP closed ports | `icmp.type==3 and icmp.code==3` |
| Long DNS names | `dns.qry.name.len > 15 and !mdns` |
| Large ICMP | `data.len > 64 and icmp` |
| TLS | `tls` |
| Export HTTP files | `File -> Export Objects -> HTTP` |

---

## Notes to Remember

- Start with statistics before packet-level analysis.
- Display filters do not delete packets.
- Capture filters permanently limit what is saved.
- Follow TCP Stream reconstructs conversations.
- FTP and HTTP can expose cleartext data.
- DNS and ICMP can be abused for tunneling.
- Kerberos, DHCP, and NBNS can identify hosts and users.
- ARP poisoning is visible through MAC/IP conflicts.
- TLS requires session keys for payload decryption.
- Document filters, evidence, and conclusions.