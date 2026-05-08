# Nmap

---

## Executive Summary

Nmap is used for host discovery, port scanning, service detection, script scanning, and network mapping.

Use this cheatsheet for quick scan syntax during enumeration and validation.

## Basic Scans

Scan a single host:

```bash
nmap TARGET_IP
```

Scan multiple hosts:

```bash
nmap TARGET1 TARGET2 TARGET3
```

Scan a subnet:

```bash
nmap 192.168.1.0/24
```

Scan from a file:

```bash
nmap -iL targets.txt
```

## Host Discovery

Ping scan:

```bash
nmap -sn 192.168.1.0/24
```

Treat host as alive:

```bash
nmap -Pn TARGET_IP
```

ARP discovery on local network:

```bash
sudo nmap -PR -sn 192.168.1.0/24
```

## Port Scanning

Top 1000 TCP ports:

```bash
nmap TARGET_IP
```

Specific ports:

```bash
nmap -p 22,80,443 TARGET_IP
```

Port range:

```bash
nmap -p 1-1000 TARGET_IP
```

All TCP ports:

```bash
nmap -p- TARGET_IP
```

Fast all-port scan:

```bash
nmap -p- --min-rate 1000 TARGET_IP
```

UDP scan:

```bash
sudo nmap -sU TARGET_IP
```

Top UDP ports:

```bash
sudo nmap -sU --top-ports 100 TARGET_IP
```

## Scan Types

TCP SYN scan:

```bash
sudo nmap -sS TARGET_IP
```

TCP connect scan:

```bash
nmap -sT TARGET_IP
```

UDP scan:

```bash
sudo nmap -sU TARGET_IP
```

Null scan:

```bash
sudo nmap -sN TARGET_IP
```

FIN scan:

```bash
sudo nmap -sF TARGET_IP
```

Xmas scan:

```bash
sudo nmap -sX TARGET_IP
```

## Service and Version Detection

Service detection:

```bash
nmap -sV TARGET_IP
```

Default scripts and service detection:

```bash
nmap -sC -sV TARGET_IP
```

Aggressive scan:

```bash
nmap -A TARGET_IP
```

Version intensity:

```bash
nmap -sV --version-intensity 9 TARGET_IP
```

## NSE Scripts

Run default scripts:

```bash
nmap -sC TARGET_IP
```

Run specific script:

```bash
nmap --script SCRIPT_NAME TARGET_IP
```

Run script category:

```bash
nmap --script vuln TARGET_IP
```

Run scripts against specific port:

```bash
nmap -p 445 --script smb-enum-shares TARGET_IP
```

List local scripts:

```bash
ls /usr/share/nmap/scripts/
```

Search scripts:

```bash
ls /usr/share/nmap/scripts/ | grep smb
```

## Timing and Performance

Timing templates:

```bash
nmap -T4 TARGET_IP
```

Minimum packet rate:

```bash
nmap --min-rate 1000 TARGET_IP
```

Limit retries:

```bash
nmap --max-retries 2 TARGET_IP
```

Scan faster with no DNS:

```bash
nmap -n TARGET_IP
```

## Output

Normal output:

```bash
nmap -oN scan.txt TARGET_IP
```

Grepable output:

```bash
nmap -oG scan.gnmap TARGET_IP
```

XML output:

```bash
nmap -oX scan.xml TARGET_IP
```

All formats:

```bash
nmap -oA scan TARGET_IP
```

## Firewall and Evasion Options

Disable host discovery:

```bash
nmap -Pn TARGET_IP
```

Fragment packets:

```bash
sudo nmap -f TARGET_IP
```

Use decoys:

```bash
sudo nmap -D RND:5 TARGET_IP
```

Spoof source port:

```bash
sudo nmap --source-port 53 TARGET_IP
```

Append random data:

```bash
nmap --data-length 24 TARGET_IP
```

## Common Service Scans

SMB:

```bash
nmap -p 445 --script smb-enum-shares,smb-enum-users TARGET_IP
```

FTP:

```bash
nmap -p 21 --script ftp-anon,ftp-syst TARGET_IP
```

HTTP:

```bash
nmap -p 80,443 --script http-title,http-headers TARGET_IP
```

DNS:

```bash
nmap -p 53 --script dns-zone-transfer TARGET_IP
```

SNMP:

```bash
sudo nmap -sU -p 161 --script snmp-info TARGET_IP
```

RDP:

```bash
nmap -p 3389 --script rdp-enum-encryption TARGET_IP
```

## Practical Workflow

| Step | Command |
|---|---|
| Discover hosts | `nmap -sn SUBNET` |
| Scan all TCP ports | `nmap -p- --min-rate 1000 TARGET_IP` |
| Scan found ports | `nmap -sC -sV -p PORTS TARGET_IP` |
| Run service scripts | `nmap --script CATEGORY -p PORT TARGET_IP` |
| Save output | `nmap -oA scan TARGET_IP` |

## Notes

- Use `-Pn` when ICMP is blocked.
- Use `-n` to skip DNS and speed up scans.
- UDP scans are slower and often require root.
- Raw packet scans may fail through proxies or tunnels.
- Always save scan output for comparison and reporting.