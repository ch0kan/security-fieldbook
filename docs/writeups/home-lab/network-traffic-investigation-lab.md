# Network Traffic Investigation Lab

---

## Executive Summary

I used this lab to practice investigating suspicious network activity inside my segmented home lab.

The goal was to observe traffic from both the endpoint and network perspective, then correlate Windows activity, pfSense firewall logs, DNS activity, and packet captures into a clear investigation timeline.

## Lab Objective

I performed this network traffic investigation to practice:

- Reviewing active connections from a Windows endpoint.
- Validating traffic through pfSense logs.
- Reviewing DNS activity.
- Capturing packets with Wireshark or tcpdump.
- Identifying suspicious connection patterns.
- Correlating endpoint and network evidence.
- Documenting findings in an investigation format.

## Environment

| System | Role |
|---|---|
| Windows Client VM | Endpoint generating or receiving traffic |
| Kali VM | Controlled testing system |
| pfSense VM | Firewall/router and primary network visibility point |
| Analysis VM | Packet capture and log review system |
| Linux Server / Web App VM | Lab service or target system |

## Network Placement

| System | Network | Example IP |
|---|---|---|
| Kali VM | Attack Network | `10.10.20.10` |
| Windows Client VM | Lab LAN | `10.10.30.10` |
| Linux Server VM | Lab LAN | `10.10.30.20` |
| Web App VM | Lab LAN | `10.10.30.30` |
| Analysis VM | Logging Network | `10.10.50.10` |
| pfSense Lab LAN Gateway | Lab LAN | `10.10.30.1` |

## Scenario I Simulated

I treated the Windows Client VM as a host with suspicious network behavior.

The investigation focused on answering:

- Which hosts communicated?
- Which ports were used?
- Was the traffic allowed or blocked by pfSense?
- Did DNS activity support the connection timeline?
- Did the endpoint process data match the network evidence?
- Was the activity expected, suspicious, or clearly unauthorized?

## Tools I Used

| Tool | How I Used It |
|---|---|
| pfSense firewall logs | Reviewed allowed and blocked traffic |
| pfSense DNS logs | Reviewed name resolution activity |
| PowerShell | Checked active endpoint connections |
| `Get-NetTCPConnection` | Mapped endpoint TCP connections |
| Wireshark | Reviewed packet captures visually |
| tcpdump | Captured traffic from Linux or analysis systems |
| Nmap | Generated controlled discovery traffic |
| Web server logs | Reviewed HTTP requests and User-Agents |

## Traffic Sources I Reviewed

I collected evidence from multiple points because one source alone does not tell the full story.

| Source | What It Showed Me |
|---|---|
| Windows endpoint | Which local process owned a connection |
| pfSense firewall logs | Whether traffic crossed network boundaries |
| pfSense DNS logs | What names were resolved |
| Packet capture | What the traffic looked like on the wire |
| Web server logs | What HTTP requests reached the target |
| Linux logs | SSH, service, and system activity |

## Endpoint Connection Review

I started on the Windows Client VM by reviewing active TCP connections.

```powershell
Get-NetTCPConnection
```

I filtered for established connections.

```powershell
Get-NetTCPConnection -State Established |
Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess
```

I mapped connections to process names.

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
Export-Csv .\endpoint-network-connections.csv -NoTypeInformation
```

## DNS Review

I reviewed DNS cache entries on the Windows Client VM.

```cmd
ipconfig /displaydns
```

I also used PowerShell.

```powershell
Get-DnsClientCache
```

I looked for:

- Recently resolved lab hosts.
- Rare or unexpected domains.
- Random-looking names.
- Direct IP usage with no DNS.
- DNS entries that matched firewall or packet capture timestamps.

## pfSense Firewall Log Review

I used pfSense to validate whether traffic crossed between lab networks.

In pfSense, I reviewed:

```text
Status > System Logs > Firewall
```

I focused on:

| Field | Why It Mattered |
|---|---|
| Time | Helped build the timeline |
| Source IP | Identified the initiating host |
| Destination IP | Identified the target |
| Destination port | Suggested the service or protocol |
| Interface | Showed which network segment saw the traffic |
| Action | Showed whether traffic was allowed or blocked |
| Rule | Identified which firewall rule matched |

## pfSense DNS Log Review

When DNS logging was enabled, I reviewed DNS activity from lab systems.

I focused on:

| Field | Why It Mattered |
|---|---|
| Client IP | Host that made the DNS request |
| Queried name | Domain or hostname requested |
| Timestamp | Correlated with firewall and endpoint evidence |
| Response | Helped validate where traffic may have gone |

## Packet Capture with pfSense

For selected tests, I captured traffic directly from pfSense.

In pfSense, I used:

```text
Diagnostics > Packet Capture
```

I selected the relevant interface, such as:

| Interface | Use |
|---|---|
| Attack Network | Traffic from Kali |
| Lab LAN | Traffic to or from Windows and servers |
| Logging Network | Traffic involving the analysis VM |

I exported packet captures for review in Wireshark when needed.

## Packet Capture with tcpdump

On Linux systems, I used tcpdump for command-line captures.

Capture traffic on an interface:

```bash
sudo tcpdump -i eth0
```

Capture traffic to or from a specific host:

```bash
sudo tcpdump -i eth0 host 10.10.30.10
```

Capture traffic for a specific port:

```bash
sudo tcpdump -i eth0 port 80
```

Write traffic to a file:

```bash
sudo tcpdump -i eth0 -w lab-capture.pcap
```

## Packet Review with Wireshark

I used Wireshark to review packet captures visually.

Useful display filters:

```text
ip.addr == 10.10.30.10
```

```text
tcp.port == 80
```

```text
dns
```

```text
http
```

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

## Controlled Traffic I Generated

I generated simple controlled traffic so I could understand how it appeared in logs.

Examples:

```bash
nmap -sV 10.10.30.20
```

```bash
curl http://10.10.30.30/
```

```bash
ping 10.10.30.10
```

```bash
nc -vz 10.10.30.20 22
```

I used this traffic to compare expected lab activity against suspicious or unexpected activity.

## Web Server Log Review

When investigating traffic to the Web App VM, I reviewed web server logs.

Common Apache log location:

```bash
sudo tail -f /var/log/apache2/access.log
```

Common Nginx log location:

```bash
sudo tail -f /var/log/nginx/access.log
```

I looked for:

- Source IP address.
- Requested path.
- HTTP status code.
- User-Agent.
- Request method.
- Repeated failed paths.
- Scanning behavior.
- Unusual parameters.

## Traffic Patterns I Investigated

| Pattern | Why I Investigated It |
|---|---|
| Repeated connections at fixed intervals | Possible beaconing |
| Many connection attempts to different ports | Possible scanning |
| Many internal hosts contacted by one workstation | Possible lateral movement |
| Direct IP connections | Possible bypass of normal DNS |
| Unexpected outbound PowerShell traffic | Possible script-based activity |
| Unusual User-Agent | Possible tool-generated traffic |
| Blocked traffic from Lab LAN to home network | Confirmed firewall isolation |

## Example Investigation Timeline

I documented network evidence in a timeline.

| Time | Source | Destination | Evidence Source | Notes |
|---|---|---|---|---|
|  | `10.10.30.10` | `10.10.30.30:80` | pfSense logs | HTTP connection observed |
|  | `10.10.30.10` | `webapp.lab.local` | DNS cache | Hostname resolved |
|  | `10.10.30.10` | `10.10.30.30:80` | Web logs | HTTP request received |
|  | `10.10.30.10` | `10.10.30.30:80` | Packet capture | TCP session confirmed |

## Evidence I Collected

| Artifact | Why I Collected It |
|---|---|
| Endpoint connection export | Mapped local processes to network connections |
| DNS cache output | Reviewed recent name resolution |
| pfSense firewall logs | Validated allowed and blocked traffic |
| pfSense DNS logs | Validated hostname lookups |
| Packet captures | Reviewed traffic details |
| Web server logs | Confirmed application-layer requests |
| Screenshots | Supported documentation and findings |

## Findings Template

I used this structure to document network findings.

```text
Finding:
Suspicious network activity was observed from the Windows Client VM.

Evidence:
- Source host:
- Source IP:
- Source process:
- Destination IP/domain:
- Destination port:
- Protocol:
- First observed:
- pfSense action:
- Related DNS query:
- Packet capture:
- Related server log:

Assessment:
Explain why the traffic was suspicious or important.

Next Steps:
- Review endpoint process details.
- Search for related DNS and firewall events.
- Check server-side logs.
- Capture additional packets if needed.
- Determine whether the behavior was expected lab activity or suspicious activity.
```

## Detection Opportunities

This lab gave me detection ideas for:

- Workstations connecting to many internal systems.
- Repeated outbound connections at regular intervals.
- PowerShell or scripting tools making network connections.
- Direct IP connections instead of domain-based traffic.
- Blocked attempts from lab systems to home network ranges.
- Unexpected User-Agents in web logs.
- Repeated HTTP 404s or unusual request paths.
- DNS requests for rare or random-looking names.

## Lessons Learned

This lab reinforced that network investigations are strongest when multiple evidence sources are correlated.

The most useful workflow was:

1. Start with the endpoint connection.
2. Identify the owning process.
3. Check DNS activity.
4. Validate the traffic in pfSense.
5. Review packet captures or server logs.
6. Document the timeline.

I also learned that firewall logs are useful, but they are more valuable when paired with endpoint and application-layer evidence.

## Skills Demonstrated

This lab demonstrates practical skills in:

- Network traffic investigation.
- pfSense firewall log review.
- DNS analysis.
- Packet capture collection.
- Wireshark filtering.
- tcpdump usage.
- Endpoint-to-network correlation.
- Timeline building.
- Detection planning.
- Technical documentation.