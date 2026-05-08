# Tcpdump

Tcpdump is a command-line packet capture and analysis tool for Unix-like systems.

It is useful for remote troubleshooting, incident response, network forensics, and quick packet inspection when a graphical tool like Wireshark is not available.

!!! warning
    Capture traffic only on systems and networks where you have permission.

---

## Overview

Tcpdump captures packets from a network interface and prints packet summaries to the terminal or saves packets to a `.pcap` file.

Common uses:

- Capture traffic on a server over SSH
- Troubleshoot connectivity
- Save packets for Wireshark analysis
- Inspect cleartext protocols
- Identify suspicious connections
- Filter traffic by host, port, protocol, or size
- Investigate scans, beaconing, and unusual traffic

Tcpdump uses the `libpcap` library and often requires root privileges.

---

## Installation

Check if tcpdump is installed:

```bash
which tcpdump
```

Install on Debian/Ubuntu:

```bash
sudo apt install tcpdump
```

Check version:

```bash
tcpdump --version
```

---

## Finding Interfaces

List available capture interfaces:

```bash
sudo tcpdump -D
```

Alternative Linux commands:

```bash
ip address show
ip a s
```

Common interfaces:

```text
eth0
ens5
wlan0
lo
any
```

The `any` interface captures from all interfaces, but it may not support all capture features.

---

## Basic Capture

Capture on a specific interface:

```bash
sudo tcpdump -i eth0
```

Capture on all interfaces:

```bash
sudo tcpdump -i any
```

Stop capture:

```text
Ctrl+C
```

---

## Essential Options

| Option | Purpose |
|---|---|
| `-i INTERFACE` | Capture on an interface |
| `-D` | List interfaces |
| `-c COUNT` | Capture a specific number of packets |
| `-n` | Disable DNS name resolution |
| `-nn` | Disable DNS and port name resolution |
| `-v` | Verbose output |
| `-vv` / `-vvv` | More verbose output |
| `-e` | Show Ethernet headers / MAC addresses |
| `-A` | Show packet payload in ASCII |
| `-X` | Show packet payload in hex and ASCII |
| `-w FILE.pcap` | Write packets to file |
| `-r FILE.pcap` | Read packets from file |

---

## Recommended Starting Commands

Quick capture with clean output:

```bash
sudo tcpdump -i eth0 -nn
```

Capture 10 packets:

```bash
sudo tcpdump -i eth0 -c 10 -nn
```

Verbose triage:

```bash
sudo tcpdump -i eth0 -nnv
```

Deep inspection with MAC addresses and payload:

```bash
sudo tcpdump -i eth0 -nnvXe
```

---

## Why Use `-n` and `-nn`

Use `-n` to prevent DNS lookups.

```bash
sudo tcpdump -i eth0 -n
```

Use `-nn` to prevent both DNS and port name resolution.

```bash
sudo tcpdump -i eth0 -nn
```

This is usually better because it:

- Speeds up output
- Reduces extra DNS traffic
- Shows real IP addresses
- Shows real port numbers
- Avoids misleading name resolution

---

## Writing to PCAP

Save traffic to a file:

```bash
sudo tcpdump -i eth0 -w capture.pcap
```

Read a saved capture:

```bash
tcpdump -r capture.pcap
```

Read with no name resolution:

```bash
tcpdump -nn -r capture.pcap
```

Read and show payload:

```bash
tcpdump -nn -X -r capture.pcap
```

!!! warning
    Packet captures can grow quickly. Monitor disk space during long captures.

---

## Tcpdump Output Anatomy

Example line:

```text
11:13:59.149599 IP 172.16.146.2.42454 > 54.77.251.34.443: Flags [P.], seq 1:38, ack 37, win 501, length 37
```

| Part | Meaning |
|---|---|
| `11:13:59.149599` | Timestamp |
| `IP` | Protocol |
| `172.16.146.2.42454` | Source IP and port |
| `>` | Direction |
| `54.77.251.34.443` | Destination IP and port |
| `Flags [P.]` | TCP flags |
| `seq` / `ack` | TCP sequence and acknowledgment |
| `win` | TCP window size |
| `length` | Payload length |

---

## TCP Flags

Common TCP flags:

| Flag | Meaning |
|---|---|
| `[S]` | SYN |
| `[S.]` | SYN-ACK |
| `[.]` | ACK |
| `[P.]` | PSH-ACK |
| `[F.]` | FIN-ACK |
| `[R]` | RST |
| `[R.]` | RST-ACK |

Example:

```text
[S]   connection attempt
[S.]  service replied
[R.]  port closed or connection reset
```

---

## Basic Filters

Tcpdump filters help reduce noise.

### Filter by Host

Traffic involving a host:

```bash
sudo tcpdump -i eth0 host 192.168.1.5
```

Source only:

```bash
sudo tcpdump -i eth0 src host 192.168.1.5
```

Destination only:

```bash
sudo tcpdump -i eth0 dst host 192.168.1.5
```

---

## Filter by Network

```bash
sudo tcpdump -i eth0 net 192.168.1.0/24
```

Source network:

```bash
sudo tcpdump -i eth0 src net 192.168.1.0/24
```

Destination network:

```bash
sudo tcpdump -i eth0 dst net 192.168.1.0/24
```

---

## Filter by Port

Traffic involving a port:

```bash
sudo tcpdump -i eth0 port 80
```

Source port:

```bash
sudo tcpdump -i eth0 src port 53
```

Destination port:

```bash
sudo tcpdump -i eth0 dst port 443
```

Port range:

```bash
sudo tcpdump -i eth0 portrange 8000-8100
```

---

## Filter by Protocol

```bash
sudo tcpdump -i eth0 tcp
sudo tcpdump -i eth0 udp
sudo tcpdump -i eth0 icmp
sudo tcpdump -i eth0 arp
```

Protocol numbers:

| Protocol | Number |
|---|---:|
| ICMP | 1 |
| TCP | 6 |
| UDP | 17 |

Example:

```bash
sudo tcpdump -i eth0 proto 1
```

---

## Logical Operators

Tcpdump supports logical operators.

| Operator | Meaning |
|---|---|
| `and` / `&&` | Both conditions |
| `or` / `||` | Either condition |
| `not` / `!` | Exclude condition |

Examples:

```bash
sudo tcpdump -i eth0 host 10.10.10.5 and port 22
```

```bash
sudo tcpdump -i eth0 port 80 or port 443
```

```bash
sudo tcpdump -i eth0 not icmp
```

---

## Useful Filter Examples

SSH traffic:

```bash
sudo tcpdump -i any tcp port 22
```

NTP traffic:

```bash
sudo tcpdump -i eth0 udp port 123
```

HTTPS traffic for a host:

```bash
sudo tcpdump -i eth0 host example.com and tcp port 443 -w https.pcap
```

Non-TCP traffic:

```bash
sudo tcpdump -i eth0 not tcp
```

Traffic involving a host over TCP:

```bash
sudo tcpdump -i eth0 host 1.1.1.1 and tcp
```

---

## Packet Size Filters

Packet size can help identify anomalies.

Packets smaller than a size:

```bash
sudo tcpdump -i eth0 less 64
```

Packets larger than a size:

```bash
sudo tcpdump -i eth0 greater 500
```

Possible uses:

- Small control packets
- Large file transfers
- Possible data exfiltration
- Unusual payload sizes

---

## Output Formatting

| Option | Description | Use Case |
|---|---|---|
| `-q` | Quick output | High-level overview |
| `-e` | Link-level header | MAC address analysis |
| `-A` | ASCII payload | Cleartext protocols |
| `-xx` | Hex output | Raw packet analysis |
| `-X` | Hex and ASCII | Payload inspection |

Examples:

```bash
sudo tcpdump -i eth0 -q
```

```bash
sudo tcpdump -i eth0 -e
```

```bash
sudo tcpdump -i eth0 -A
```

```bash
sudo tcpdump -i eth0 -X
```

---

## Cleartext Inspection

Show ASCII payloads:

```bash
sudo tcpdump -i eth0 -A port 80
```

Search a PCAP for text using line buffering:

```bash
sudo tcpdump -Ar traffic.pcap -l | grep 'password'
```

Use `-l` when piping to tools like `grep` so output is line-buffered.

---

## Header Byte Filtering

Tcpdump can inspect protocol header bytes.

Syntax:

```text
proto[expr:size]
```

| Part | Meaning |
|---|---|
| `proto` | Protocol such as `tcp`, `udp`, `ip`, `icmp`, `ether` |
| `expr` | Byte offset |
| `size` | Number of bytes, usually 1, 2, or 4 |

Example: multicast Ethernet packets:

```bash
sudo tcpdump 'ether[0] & 1 != 0'
```

Example: IP packets with options:

```bash
sudo tcpdump 'ip[0] & 0xf != 5'
```

---

## TCP Flag Filtering

Only SYN packets:

```bash
sudo tcpdump 'tcp[tcpflags] == tcp-syn'
```

Packets with at least SYN set:

```bash
sudo tcpdump 'tcp[tcpflags] & tcp-syn != 0'
```

Packets with SYN or ACK set:

```bash
sudo tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'
```

Byte-offset method for SYN packets:

```bash
sudo tcpdump -i eth0 'tcp[13] & 2 != 0'
```

Available TCP flag names:

```text
tcp-syn
tcp-ack
tcp-fin
tcp-rst
tcp-push
```

---

## Practical Incident Workflow

1. Identify the interface.
2. Capture a small sample.
3. Disable name resolution.
4. Filter by host, port, or protocol.
5. Save to PCAP if deeper analysis is needed.
6. Open the PCAP in Wireshark.
7. Document commands and findings.

Example:

```bash
sudo tcpdump -D
sudo tcpdump -i eth0 -nn -c 20
sudo tcpdump -i eth0 -nn host 192.168.1.25 -w suspicious-host.pcap
```

---

## Common Troubleshooting

| Problem | Possible Cause |
|---|---|
| Permission denied | Need `sudo` |
| Too much output | Add filters or `-c` |
| Slow output | Use `-n` or `-nn` |
| Empty capture | Wrong interface or no matching traffic |
| Huge PCAP | Filter traffic or limit capture time |
| Cannot see other hosts | Switched network, no span/tap, or wrong segment |

---

## Quick Reference

| Goal | Command |
|---|---|
| List interfaces | `sudo tcpdump -D` |
| Capture on interface | `sudo tcpdump -i eth0` |
| No DNS/port names | `sudo tcpdump -i eth0 -nn` |
| Capture count | `sudo tcpdump -i eth0 -c 10` |
| Write PCAP | `sudo tcpdump -i eth0 -w capture.pcap` |
| Read PCAP | `tcpdump -r capture.pcap` |
| Filter host | `tcpdump host 192.168.1.5` |
| Filter port | `tcpdump port 443` |
| Filter protocol | `tcpdump tcp` |
| Show payload | `tcpdump -X` |
| Show MAC addresses | `tcpdump -e` |

---

## Notes to Remember

- Tcpdump is ideal for CLI packet capture.
- Use `-nn` for faster, cleaner output.
- Use `-w` to save PCAPs for later analysis.
- Use filters to reduce noise.
- Use `-A` or `-X` for payload inspection.
- Use `-e` when MAC addresses matter.
- Tcpdump filters can be applied during capture or when reading a PCAP.