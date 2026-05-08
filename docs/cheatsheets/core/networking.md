# Networking

---

## Executive Summary

This cheatsheet is a quick reference for IP addressing, routing, DNS, connectivity checks, and packet inspection.

Use it when troubleshooting network reachability, reviewing scan results, or validating basic connectivity during labs and assessments.

## IP Addressing

| Range | Purpose |
|---|---|
| `10.0.0.0/8` | Private IPv4 |
| `172.16.0.0/12` | Private IPv4 |
| `192.168.0.0/16` | Private IPv4 |
| `127.0.0.0/8` | Loopback |
| `169.254.0.0/16` | Link-local/APIPA |
| `224.0.0.0/4` | Multicast |
| `0.0.0.0` | Any address / default route |
| `255.255.255.255` | Limited broadcast |

## CIDR Quick Reference

| CIDR | Hosts | Subnet Mask |
|---:|---:|---|
| `/24` | 254 | `255.255.255.0` |
| `/25` | 126 | `255.255.255.128` |
| `/26` | 62 | `255.255.255.192` |
| `/27` | 30 | `255.255.255.224` |
| `/28` | 14 | `255.255.255.240` |
| `/29` | 6 | `255.255.255.248` |
| `/30` | 2 | `255.255.255.252` |
| `/32` | 1 | Single host |

## Linux Network Commands

Show interfaces:

```bash
ip addr
```

Show routes:

```bash
ip route
```

Show listening ports:

```bash
ss -tulpen
```

Show ARP/neighbor table:

```bash
ip neigh
```

Test connectivity:

```bash
ping -c 4 TARGET_IP
```

Trace route:

```bash
traceroute TARGET_IP
```

Check DNS resolution:

```bash
dig example.com
```

```bash
nslookup example.com
```

## Windows Network Commands

Show IP configuration:

```cmd
ipconfig /all
```

Show routing table:

```cmd
route print
```

Show ARP table:

```cmd
arp -a
```

Show active connections:

```cmd
netstat -ano
```

Show DNS cache:

```cmd
ipconfig /displaydns
```

Test connectivity:

```cmd
ping TARGET_IP
```

Trace route:

```cmd
tracert TARGET_IP
```

PowerShell port test:

```powershell
Test-NetConnection TARGET_IP -Port PORT
```

## DNS Commands

Query A record:

```bash
dig example.com A
```

Query MX record:

```bash
dig example.com MX
```

Query TXT record:

```bash
dig example.com TXT
```

Query name servers:

```bash
dig example.com NS
```

Reverse lookup:

```bash
dig -x TARGET_IP
```

Attempt zone transfer:

```bash
dig axfr @NAMESERVER example.com
```

## Netcat Connectivity Checks

Check TCP port:

```bash
nc -zv TARGET_IP PORT
```

Connect to service:

```bash
nc TARGET_IP PORT
```

Listen on port:

```bash
nc -lvnp PORT
```

Send text over TCP:

```bash
echo "test" | nc TARGET_IP PORT
```

## Tcpdump Quick Reference

Capture traffic on interface:

```bash
sudo tcpdump -i INTERFACE
```

Capture host traffic:

```bash
sudo tcpdump -i INTERFACE host TARGET_IP
```

Capture port traffic:

```bash
sudo tcpdump -i INTERFACE port PORT
```

Capture to file:

```bash
sudo tcpdump -i INTERFACE -w capture.pcap
```

Read capture file:

```bash
tcpdump -r capture.pcap
```

## Useful Filters

| Filter | Purpose |
|---|---|
| `host TARGET_IP` | Traffic to or from host |
| `src TARGET_IP` | Source IP only |
| `dst TARGET_IP` | Destination IP only |
| `port 80` | Traffic using port 80 |
| `tcp` | TCP traffic |
| `udp` | UDP traffic |
| `icmp` | ICMP traffic |
| `net 192.168.1.0/24` | Traffic for subnet |

## Troubleshooting Flow

| Step | Check |
|---|---|
| Interface | Does the host have an IP address? |
| Route | Is there a route to the destination? |
| DNS | Does the hostname resolve? |
| ICMP | Does ping work? |
| TCP/UDP | Is the target port reachable? |
| Firewall | Is traffic filtered or blocked? |
| Service | Is the service actually listening? |

## Notes

- ICMP may be blocked even when services are reachable.
- DNS success does not guarantee the service is alive.
- A filtered port usually means traffic is being dropped.
- A refused connection usually means the host is reachable but the port is closed.
- Always check both local host routing and remote service availability.