# Nmap

Nmap is a network reconnaissance and enumeration tool used to discover hosts, identify open ports, fingerprint services, and collect information about target systems.

In a penetration test or lab environment, Nmap is usually used after scope is confirmed and before deeper service-specific enumeration begins.

!!! warning
    Only scan systems you own or have explicit permission to test.

---

## Overview

Nmap can help answer several important reconnaissance questions:

- Which hosts are alive?
- Which ports are open?
- Which services are running?
- What versions are exposed?
- What operating system might the target be running?
- Are firewalls filtering traffic?
- Can scan results be saved for reporting or automation?

A good Nmap workflow usually starts broad, then becomes more targeted.

```bash
# Initial service scan
sudo nmap -sC -sV -oA scans/initial 10.10.10.10

# Full TCP port scan
sudo nmap -p- --min-rate 1000 -oA scans/full-tcp 10.10.10.10

# Targeted version scan against discovered ports
sudo nmap -sC -sV -p 22,80,443 -oA scans/targeted 10.10.10.10
```

---

## Target Specification

Nmap accepts many target formats.

```bash
# Single IP
nmap 10.10.10.10

# IP range
nmap 192.168.1.1-50

# CIDR subnet
nmap 192.168.1.0/24

# Hostname
nmap example.local

# Targets from a file
nmap -iL hosts.txt
```

Useful target options:

| Option | Purpose |
|---|---|
| `TARGET` | Scan a single host |
| `192.168.1.1-50` | Scan a range |
| `192.168.1.0/24` | Scan a subnet |
| `-iL hosts.txt` | Read targets from a file |
| `--exclude <host>` | Exclude a host |
| `--exclude-file <file>` | Exclude hosts listed in a file |

---

## Host Discovery

Host discovery identifies which systems are alive before running heavier scans.

```bash
# Ping scan / host discovery only
sudo nmap -sn 192.168.1.0/24
```

The `-sn` option disables port scanning and focuses on host availability.

### Local Network Discovery

On a local network, Nmap commonly uses ARP requests because ARP is reliable for discovering directly connected devices.

```bash
sudo nmap -sn 192.168.1.0/24
```

Local discovery can reveal:

- Active hosts
- MAC addresses
- Vendor information
- Devices that may not respond to ICMP ping

### Remote Network Discovery

When a target is behind a router, ARP cannot be used across the routed boundary. Nmap must rely on other probes such as ICMP, TCP, or UDP-based discovery.

```bash
sudo nmap -sn 10.10.10.0/24
```

### Force ICMP Discovery

If you want to test ICMP behavior on a local subnet, disable ARP discovery.

```bash
sudo nmap -sn -PE --disable-arp-ping 10.10.10.10
```

### Treat Hosts as Online

Some hosts block discovery probes. In that case, `-Pn` tells Nmap to skip host discovery and treat the target as online.

```bash
nmap -Pn 10.10.10.10
```

Use `-Pn` when:

- The host blocks ping
- A firewall drops discovery probes
- You know the host is alive
- You want to force port scanning

---

## Port Scanning

After identifying live hosts, the next step is port scanning.

Ports represent network-accessible services. TCP and UDP each have 65,535 possible ports.

Common examples:

| Service | Port |
|---|---|
| SSH | TCP 22 |
| DNS | TCP/UDP 53 |
| HTTP | TCP 80 |
| HTTPS | TCP 443 |
| SMB | TCP 445 |

---

## TCP Scan Types

### SYN Scan

The SYN scan is the default scan type when running Nmap with root or administrator privileges.

```bash
sudo nmap -sS 10.10.10.10
```

A SYN scan is often called a half-open scan:

1. Nmap sends a SYN packet.
2. If the port is open, the target replies with SYN-ACK.
3. Nmap sends RST instead of completing the full connection.

This is usually faster and quieter than a full TCP connect scan.

### TCP Connect Scan

The TCP connect scan completes the full TCP three-way handshake.

```bash
nmap -sT 10.10.10.10
```

This scan type is commonly used when Nmap does not have raw packet privileges.

Characteristics:

- Works without root privileges
- Completes a full TCP connection
- Reliable
- More likely to appear in application logs
- Usually slower than SYN scanning

---

## Port States

Nmap classifies ports based on the responses it receives.

| State | Meaning |
|---|---|
| `open` | A service is accepting connections |
| `closed` | The port is reachable, but no service is listening |
| `filtered` | A firewall or filter is blocking the probe |
| `unfiltered` | The port is reachable, but Nmap cannot determine whether it is open or closed |
| `open|filtered` | Nmap cannot determine whether the port is open or filtered |
| `closed|filtered` | Nmap cannot determine whether the port is closed or filtered |

### TCP Response Logic

| Response | Likely State |
|---|---|
| SYN-ACK | Open |
| RST | Closed |
| No response | Filtered |
| ICMP unreachable | Filtered |

---

## UDP Scanning

UDP scanning is slower and less reliable than TCP scanning because UDP is connectionless.

```bash
sudo nmap -sU 10.10.10.10
```

UDP scan logic:

| Response | Likely State |
|---|---|
| UDP response | Open |
| ICMP port unreachable | Closed |
| No response | Open or filtered |

Because many UDP services do not respond to empty probes, Nmap often reports UDP ports as:

```text
open|filtered
```

For practical testing, scan common UDP ports first.

```bash
sudo nmap -sU --top-ports 20 10.10.10.10
```

---

## Port Selection

By default, Nmap scans the most common 1,000 TCP ports.

```bash
# Scan default ports
nmap 10.10.10.10

# Scan all TCP ports
nmap -p- 10.10.10.10

# Scan specific ports
nmap -p 22,80,443 10.10.10.10

# Scan a range
nmap -p 1-1000 10.10.10.10

# Fast scan
nmap -F 10.10.10.10

# Scan top ports
nmap --top-ports 100 10.10.10.10
```

---

## Service and Version Detection

After finding open ports, use service version detection to identify what is running.

```bash
nmap -sV 10.10.10.10
```

Targeting specific ports is usually cleaner:

```bash
sudo nmap -sV -p 22,80,443 10.10.10.10
```

Service detection can identify:

- Service name
- Product name
- Version number
- Hostname
- Extra service metadata

Example output:

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.49
```

---

## Banner Grabbing

Some services reveal information when a connection is made. This is called banner grabbing.

```bash
nc -nv 10.10.10.10 25
```

Example banner:

```text
220 mail.example.local ESMTP Postfix (Ubuntu)
```

A banner may reveal:

- Service type
- Software name
- Version
- Hostname
- Operating system hints

Nmap can collect this automatically with `-sV`, but manual verification is useful when the result is unclear.

---

## OS Detection

Nmap can attempt to identify the target operating system.

```bash
sudo nmap -O 10.10.10.10
```

OS detection uses network behavior and fingerprinting. It is an educated guess, not a guarantee.

OS detection works best when:

- At least one open port exists
- At least one closed port exists
- The scan is run with elevated privileges

---

## Aggressive Scan

The `-A` option enables several features at once:

- OS detection
- Service version detection
- Default scripts
- Traceroute

```bash
sudo nmap -A 10.10.10.10
```

A more controlled version is often better:

```bash
sudo nmap -sC -sV -O -p 22,80,443 10.10.10.10
```

---

## Nmap Scripting Engine

The Nmap Scripting Engine, or NSE, allows Nmap to run scripts against services.

Default scripts:

```bash
sudo nmap -sC 10.10.10.10
```

Run default scripts with version detection:

```bash
sudo nmap -sC -sV 10.10.10.10
```

Run a specific script:

```bash
sudo nmap -p 80 --script http-title 10.10.10.10
```

Run multiple scripts:

```bash
sudo nmap -p 25 --script banner,smtp-commands 10.10.10.10
```

Run vulnerability scripts in an authorized lab:

```bash
sudo nmap -sV --script vuln -p 80 10.10.10.10
```

!!! warning
    NSE scripts can be intrusive. Some scripts may brute force, fuzz, or interact heavily with services. Use them only in authorized environments.

### Common Script Categories

| Category | Purpose |
|---|---|
| `default` | Common safe scripts used with `-sC` |
| `safe` | Low-risk information gathering |
| `discovery` | Service and network discovery |
| `version` | Improve service fingerprinting |
| `auth` | Authentication-related checks |
| `vuln` | Vulnerability checks |
| `brute` | Brute-force related scripts |
| `intrusive` | Higher-risk active testing |

---

## Output and Reporting

Always save scan output during practical testing. This helps with documentation, comparison, and reporting.

```bash
# Normal output
nmap -oN scan.nmap 10.10.10.10

# Grepable output
nmap -oG scan.gnmap 10.10.10.10

# XML output
nmap -oX scan.xml 10.10.10.10

# Save all formats
nmap -oA scan 10.10.10.10
```

### Output Formats

| Option | Format | Use Case |
|---|---|---|
| `-oN` | Normal | Human-readable notes |
| `-oG` | Grepable | Parsing with command-line tools |
| `-oX` | XML | Importing into tools or automation |
| `-oA` | All formats | Best default for documentation |

### Example Organized Output

```bash
mkdir -p scans

sudo nmap -sC -sV -oA scans/initial 10.10.10.10
sudo nmap -p- -oA scans/full-tcp 10.10.10.10
sudo nmap -sU --top-ports 20 -oA scans/top-udp 10.10.10.10
```

---

## Parsing Results

Grepable output can be useful for extracting targets or services.

```bash
# Extract hosts with port 80 open
grep "80/open" scan.gnmap | cut -d " " -f 2
```

XML output can be imported into tools or converted to reports.

```bash
# Convert XML output into HTML
xsltproc scan.xml -o scan.html
```

---

## Verbosity and Debugging

Verbose mode shows scan progress and additional information.

```bash
nmap -v 10.10.10.10
nmap -vv 10.10.10.10
```

Debug mode is useful when troubleshooting scan behavior.

```bash
nmap -d 10.10.10.10
nmap -d2 10.10.10.10
```

Useful debugging flags:

| Flag | Purpose |
|---|---|
| `-v` | Increase verbosity |
| `-vv` | More verbose output |
| `-d` | Debug output |
| `--reason` | Show why Nmap marked a host or port a certain way |
| `--packet-trace` | Show sent and received packets |
| `--stats-every=5s` | Print progress at intervals |

Example:

```bash
sudo nmap -sS --reason --packet-trace -p 80 10.10.10.10
```

---

## Performance Tuning

Nmap performance depends on timing, retries, latency, packet loss, and target responsiveness.

### Timing Templates

```bash
nmap -T0 10.10.10.10
nmap -T1 10.10.10.10
nmap -T2 10.10.10.10
nmap -T3 10.10.10.10
nmap -T4 10.10.10.10
nmap -T5 10.10.10.10
```

| Template | Name | Use Case |
|---|---|---|
| `-T0` | Paranoid | Very slow, IDS evasion testing |
| `-T1` | Sneaky | Slow, stealth-focused testing |
| `-T2` | Polite | Reduces bandwidth and noise |
| `-T3` | Normal | Default balance |
| `-T4` | Aggressive | Faster scans on reliable networks |
| `-T5` | Insane | Very fast, higher risk of missed results |

For labs and CTFs, `-T4` is commonly used:

```bash
sudo nmap -sC -sV -T4 10.10.10.10
```

### Packet Rate

```bash
# Minimum packet rate
sudo nmap --min-rate 1000 10.10.10.10

# Maximum packet rate
sudo nmap --max-rate 100 10.10.10.10
```

### Retries

```bash
# Reduce retries for speed
sudo nmap --max-retries 1 10.10.10.10
```

Lower retries can make scans faster but may miss open ports on unreliable networks.

### RTT Timeout

```bash
sudo nmap --initial-rtt-timeout 50ms --max-rtt-timeout 100ms 10.10.10.10
```

Aggressive timeout values can speed up scans, but they can also create false negatives.

---

## Firewall and IDS/IPS Testing

Firewalls and IDS/IPS systems can affect scan results. During authorized testing, Nmap can help identify filtering behavior.

!!! warning
    Use these techniques only in authorized labs, internal assessments, or environments where firewall testing is explicitly in scope.

### Filtered Ports

A filtered port usually means a firewall is dropping or rejecting traffic.

| Behavior | Meaning |
|---|---|
| No response | Packet may have been silently dropped |
| ICMP unreachable | Packet was rejected or blocked |
| TCP RST | Port may be closed or traffic may be allowed through |

### ACK Scan

An ACK scan does not identify open ports. It helps map firewall rules.

```bash
sudo nmap -sA -Pn -n -p 22,80,443 10.10.10.10
```

ACK scan results:

| Result | Meaning |
|---|---|
| `filtered` | Firewall likely blocked the ACK packet |
| `unfiltered` | Packet reached the host and received a response |

### Decoy Scan

Decoy scanning mixes your scan traffic with traffic that appears to come from other IP addresses.

```bash
sudo nmap -sS -D RND:5 -p 80 10.10.10.10
```

This can make source attribution harder in logs, but it is noisy and should be used only when authorized.

### Source Port Testing

Some weak firewall rules trust traffic from certain source ports, such as DNS port 53.

```bash
sudo nmap --source-port 53 -p 50000 10.10.10.10
```

If a port appears filtered normally but open when using a trusted source port, that may indicate a firewall rule issue.

### Fragmentation

Packet fragmentation can split probes into smaller packets.

```bash
sudo nmap -f 10.10.10.10
```

Modern firewalls often detect or normalize fragmented traffic, so results vary.

---

## Practical Scan Workflow

A clean workflow helps keep scans organized and repeatable.

### 1. Create a scan directory

```bash
mkdir -p scans
```

### 2. Check if the host is alive

```bash
sudo nmap -sn -oA scans/host-discovery 10.10.10.10
```

### 3. Run an initial TCP scan

```bash
sudo nmap -sC -sV -oA scans/initial 10.10.10.10
```

### 4. Scan all TCP ports

```bash
sudo nmap -p- --min-rate 1000 -oA scans/full-tcp 10.10.10.10
```

### 5. Run targeted enumeration

```bash
sudo nmap -sC -sV -p 22,80,443 -oA scans/targeted 10.10.10.10
```

### 6. Scan common UDP ports

```bash
sudo nmap -sU --top-ports 20 -oA scans/top-udp 10.10.10.10
```

### 7. Investigate interesting services

Use service-specific tools after Nmap identifies exposed services.

Examples:

```bash
# HTTP
whatweb http://10.10.10.10

# SMB
smbclient -L //10.10.10.10/

# DNS
dig axfr @10.10.10.10 example.local
```

---

## Quick Reference

| Goal | Command |
|---|---|
| Host discovery | `sudo nmap -sn TARGET` |
| Basic scan | `nmap TARGET` |
| SYN scan | `sudo nmap -sS TARGET` |
| TCP connect scan | `nmap -sT TARGET` |
| UDP scan | `sudo nmap -sU TARGET` |
| Service detection | `nmap -sV TARGET` |
| Default scripts | `nmap -sC TARGET` |
| OS detection | `sudo nmap -O TARGET` |
| Aggressive scan | `sudo nmap -A TARGET` |
| Skip discovery | `nmap -Pn TARGET` |
| Scan all TCP ports | `nmap -p- TARGET` |
| Scan selected ports | `nmap -p 22,80,443 TARGET` |
| Save all outputs | `nmap -oA scan TARGET` |
| Show reasons | `nmap --reason TARGET` |
| Packet trace | `nmap --packet-trace TARGET` |
| Fast lab scan | `sudo nmap -sC -sV -T4 TARGET` |

---

## Notes to Remember

- Use `sudo` for SYN scans, OS detection, and full Nmap capabilities.
- Save scan results with `-oA` during labs and assessments.
- Use `-Pn` when a host blocks ping but is known to be online.
- Use `-sV` after discovering open ports.
- Use `--reason` and `--packet-trace` to understand confusing results.
- UDP scanning is slower and often requires more validation.
- Firewall testing techniques must be authorized and in scope.