# Protocols

Network protocols are standardized rules that allow devices, services, and applications to communicate. They define how data is addressed, transmitted, routed, encrypted, requested, and interpreted.

This page covers common protocols and terminology used in networking, system administration, web security, and cybersecurity labs.

---

## Overview

A protocol answers questions like:

- How should two systems start communication?
- What port should a service listen on?
- How should data be formatted?
- How should errors be reported?
- How should names resolve to IP addresses?
- How should devices receive network configuration?

Common protocol categories:

| Category | Examples |
|---|---|
| Transport | TCP, UDP |
| Diagnostic | ICMP |
| Addressing / Configuration | DHCP, ARP, NAT |
| Name Resolution | DNS |
| Web | HTTP, HTTPS |
| Remote Access | SSH, RDP, VPN |
| File Sharing | FTP, SMB, NFS |
| Routing / Switching | OSPF, BGP, RIP, STP, VLAN |

---

## TCP

Transmission Control Protocol, or TCP, is a connection-oriented transport protocol.

TCP is used when reliability matters.

Common TCP use cases:

- Web browsing
- SSH
- File transfers
- Email
- Remote desktop
- SMB file sharing

### TCP Characteristics

| Characteristic | Description |
|---|---|
| Connection-oriented | Establishes a session before sending data |
| Reliable | Uses acknowledgements and retransmission |
| Ordered | Reassembles data in the correct order |
| Stateful | Tracks connection state |
| Higher overhead | More control traffic than UDP |

### TCP Three-Way Handshake

Before data is exchanged, TCP establishes a connection.

```text
Client -> SYN -> Server
Client <- SYN/ACK <- Server
Client -> ACK -> Server
```

Common TCP flags:

| Flag | Purpose |
|---|---|
| `SYN` | Starts a connection |
| `ACK` | Acknowledges received data |
| `FIN` | Gracefully closes a connection |
| `RST` | Resets or abruptly terminates a connection |
| `PSH` | Pushes data to the application quickly |

Security relevance:

- Port scanners use TCP behavior to infer open, closed, and filtered ports.
- Firewalls often track TCP connection states.
- Unexpected `RST` packets can indicate closed ports, filtering, or connection resets.

---

## UDP

User Datagram Protocol, or UDP, is a connectionless transport protocol.

UDP is used when speed matters more than guaranteed delivery.

Common UDP use cases:

- DNS
- VoIP
- Video streaming
- Online gaming
- Some VPN traffic
- Service discovery

### UDP Characteristics

| Characteristic | Description |
|---|---|
| Connectionless | No handshake before sending data |
| Fast | Lower overhead than TCP |
| Stateless | Does not track sessions by default |
| No delivery guarantee | Packets may be lost |
| No ordering guarantee | Packets may arrive out of order |

Security relevance:

- UDP scans are harder to interpret than TCP scans.
- No response does not always mean a UDP port is closed.
- Some UDP services can be abused for amplification attacks.

---

## ICMP

Internet Control Message Protocol, or ICMP, is used for network diagnostics and error reporting.

ICMP does not usually carry application data. Instead, it reports network conditions.

Common ICMP uses:

- Testing reachability
- Measuring round-trip time
- Reporting unreachable destinations
- Tracing network paths

### Ping

`ping` uses ICMP Echo Request and Echo Reply messages.

```bash
ping 8.8.8.8
```

Useful for checking:

- Whether a host is reachable
- Round-trip time
- Packet loss
- Basic network connectivity

!!! note
    A host can be online even if it does not respond to ping. Firewalls often block ICMP.

### Traceroute

Traceroute maps the path packets take through routers.

```bash
# Linux/macOS
traceroute 8.8.8.8

# Windows
tracert 8.8.8.8
```

Traceroute uses the TTL field. Each router decreases TTL by 1. When TTL reaches 0, the router drops the packet and sends an ICMP Time Exceeded message.

### Common ICMP Concepts

| Concept | Purpose |
|---|---|
| Echo Request | Sent by ping client |
| Echo Reply | Sent by responding host |
| Time Exceeded | Used by traceroute |
| Destination Unreachable | Reports unreachable host, network, or port |
| TTL | Prevents packets from looping forever |

---

## ARP

Address Resolution Protocol, or ARP, maps an IPv4 address to a MAC address on a local network.

A device may know the destination IP address, but Ethernet and Wi-Fi still need a destination MAC address for local delivery.

Example:

```text
192.168.1.1 -> aa:bb:cc:dd:ee:ff
```

### How ARP Works

```text
Host: Who has 192.168.1.1?
Router: 192.168.1.1 is at aa:bb:cc:dd:ee:ff
```

View the ARP table:

```bash
arp -a
```

Security relevance:

- ARP is local-network only.
- ARP has no built-in authentication.
- ARP spoofing can be used in man-in-the-middle attacks.

---

## DHCP

Dynamic Host Configuration Protocol, or DHCP, automatically assigns network configuration to devices.

DHCP commonly provides:

- IP address
- Subnet mask
- Default gateway
- DNS server
- Lease duration

Without DHCP, administrators would need to configure IP settings manually.

---

## DHCP DORA Process

DHCP uses a four-step process often called DORA.

| Step | Message | Description |
|---|---|---|
| 1 | Discover | Client broadcasts to find DHCP servers |
| 2 | Offer | DHCP server offers an available IP configuration |
| 3 | Request | Client requests the offered IP address |
| 4 | Acknowledge | Server confirms the lease |

```text
Client -> DHCP Discover -> Broadcast
Server -> DHCP Offer
Client -> DHCP Request
Server -> DHCP ACK
```

DHCP uses UDP:

| Role | Port |
|---|---|
| DHCP Server | UDP 67 |
| DHCP Client | UDP 68 |

### DHCP Commands

```bash
# Windows: release and renew DHCP lease
ipconfig /release
ipconfig /renew
```

```bash
# Linux: release and request new lease
sudo dhclient -r
sudo dhclient
```

Capture DHCP traffic:

```bash
sudo tcpdump -i eth0 port 67 or port 68 -vv
```

Security relevance:

- Rogue DHCP servers can provide malicious gateways or DNS servers.
- DHCP starvation can exhaust the address pool.
- DHCP logs can help identify devices joining a network.

---

## DNS

Domain Name System, or DNS, translates human-readable names into IP addresses.

Example:

```text
example.com -> 93.184.216.34
```

DNS is important because users remember names more easily than IP addresses.

---

## DNS Hierarchy

DNS is hierarchical.

```text
www.example.com
│   │       │
│   │       └── Top-level domain: .com
│   └────────── Second-level domain: example
└────────────── Hostname/subdomain: www
```

Main levels:

| Level | Example | Description |
|---|---|---|
| Root | `.` | Top of the DNS hierarchy |
| TLD | `.com` | Top-level domain |
| Second-level domain | `example` | Registered domain name |
| Subdomain / host | `www` | Specific host or service |

---

## DNS Record Types

DNS can store different types of records.

| Record | Purpose |
|---|---|
| `A` | Maps a name to an IPv4 address |
| `AAAA` | Maps a name to an IPv6 address |
| `CNAME` | Creates an alias to another name |
| `MX` | Defines mail servers for a domain |
| `TXT` | Stores text values, often for verification or email security |
| `NS` | Identifies authoritative name servers |
| `PTR` | Reverse DNS lookup from IP to name |

Examples:

```bash
# Basic DNS lookup
nslookup example.com

# Query A records
dig example.com A

# Query MX records
dig example.com MX

# Query TXT records
dig example.com TXT
```

---

## DNS Resolution Process

When a client needs to resolve a domain, it usually follows this process:

1. Check local cache.
2. Ask the configured recursive resolver.
3. Resolver asks root servers.
4. Root servers point to the correct TLD servers.
5. TLD servers point to the authoritative name server.
6. Authoritative server returns the requested record.
7. Resolver caches the answer and returns it to the client.

```text
Client
  -> Recursive Resolver
    -> Root Server
      -> TLD Server
        -> Authoritative Name Server
```

### DNS Caching and TTL

DNS records include a TTL, or Time To Live.

TTL controls how long a resolver or client can cache the answer.

```text
Higher TTL = fewer lookups, slower changes
Lower TTL  = more lookups, faster changes
```

Security relevance:

- DNS poisoning can redirect users to malicious destinations.
- DNS logs are useful for threat hunting.
- Unusual DNS queries can indicate malware or data exfiltration.

---

## HTTP and HTTPS

Hypertext Transfer Protocol, or HTTP, is used for web communication.

HTTPS is HTTP protected by TLS encryption.

| Protocol | Default Port | Security |
|---|---|---|
| HTTP | TCP 80 | Unencrypted |
| HTTPS | TCP 443 | Encrypted with TLS |

HTTP is request-response based.

```text
Client -> HTTP Request -> Server
Client <- HTTP Response <- Server
```

### HTTP Request Example

```http
GET / HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: */*
```

### HTTP Response Example

```http
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
Content-Length: 1256
```

### Common HTTP Methods

| Method | Purpose |
|---|---|
| `GET` | Retrieve data |
| `POST` | Submit or create data |
| `PUT` | Update data |
| `PATCH` | Partially update data |
| `DELETE` | Delete data |
| `HEAD` | Retrieve headers only |
| `OPTIONS` | Show supported methods |

### Common HTTP Status Codes

| Code | Meaning |
|---|---|
| `200` | OK |
| `201` | Created |
| `301` | Moved Permanently |
| `302` | Found / temporary redirect |
| `400` | Bad Request |
| `401` | Unauthorized |
| `403` | Forbidden |
| `404` | Not Found |
| `405` | Method Not Allowed |
| `500` | Internal Server Error |
| `503` | Service Unavailable |

!!! note
    HTTP is covered here as a networking protocol. Deeper web request analysis belongs in the web fundamentals section.

---

## NAT

Network Address Translation, or NAT, maps private internal addresses to public addresses.

NAT is commonly used because IPv4 addresses are limited.

Example:

```text
Internal host: 192.168.1.25
Router public IP: 203.0.113.10
```

When the internal host accesses the internet, the router replaces the private source IP with the public IP and tracks the connection in a NAT table.

### Private IP Ranges

| Range | Common Use |
|---|---|
| `10.0.0.0/8` | Large private networks |
| `172.16.0.0/12` | Medium private networks |
| `192.168.0.0/16` | Home and small office networks |

### PAT / NAT Overload

Port Address Translation, or PAT, allows many private hosts to share one public IP by tracking unique source ports.

```text
192.168.1.10:51514 -> 203.0.113.10:40001
192.168.1.11:51515 -> 203.0.113.10:40002
```

Security relevance:

- NAT hides internal addressing from the public internet.
- NAT is not a full security control.
- Port forwarding exposes internal services externally.
- Some protocols can break or require special handling through NAT.

---

## Routing Protocols

Routing protocols help routers decide where to forward packets.

| Protocol | Type | Common Use |
|---|---|---|
| OSPF | Link-state | Internal enterprise routing |
| RIP | Distance-vector | Small or legacy networks |
| EIGRP | Advanced distance-vector | Cisco environments |
| BGP | Path-vector | Internet routing between organizations |

### OSPF

Open Shortest Path First shares network topology information and calculates efficient paths.

Common use:

```text
Internal enterprise networks
```

### RIP

Routing Information Protocol uses hop count to choose routes.

Limitations:

- Simple
- Older
- Less scalable
- Not ideal for large networks

### BGP

Border Gateway Protocol is the core routing protocol of the internet.

Common use:

```text
ISP routing
Cloud provider routing
Large organization internet edge
```

Security relevance:

- Route leaks can cause outages.
- BGP hijacking can redirect traffic.
- Internal routing visibility helps with network defense and troubleshooting.

---

## VLANs

A Virtual Local Area Network, or VLAN, logically separates a network at Layer 2.

VLANs allow multiple isolated broadcast domains to exist on the same physical switching infrastructure.

Example:

| VLAN | Purpose |
|---|---|
| VLAN 10 | Users |
| VLAN 20 | Servers |
| VLAN 30 | Guest Wi-Fi |
| VLAN 40 | Management |

Benefits:

- Segmentation
- Reduced broadcast noise
- Better organization
- Security boundaries between groups

### Trunking

A trunk link carries traffic for multiple VLANs between switches.

The common trunking standard is IEEE 802.1Q.

Security relevance:

- Misconfigured trunks can expose VLANs.
- Native VLAN misuse can create risk.
- Dynamic trunking should be disabled on user-facing ports.

---

## STP

Spanning Tree Protocol, or STP, prevents Layer 2 loops.

Layer 2 loops can cause broadcast storms and network instability.

STP works by blocking redundant paths while keeping backup paths available.

Security relevance:

- STP manipulation can affect traffic flow.
- Rogue switches can disrupt LAN stability.
- BPDU Guard is often used to protect access ports.

---

## VPN

A Virtual Private Network creates an encrypted tunnel over an untrusted network.

Common VPN types:

| Type | Purpose |
|---|---|
| Remote access VPN | Connects a user to a private network |
| Site-to-site VPN | Connects two networks |
| Full tunnel | Routes all traffic through the VPN |
| Split tunnel | Routes only selected traffic through the VPN |

### IPsec

IPsec secures IP traffic at Layer 3.

Important components:

| Component | Purpose |
|---|---|
| AH | Authentication and integrity |
| ESP | Encryption and optional authentication |
| IKE | Key exchange and negotiation |
| NAT-T | Helps IPsec work through NAT |

Common IPsec firewall requirements:

| Component | Port / Protocol |
|---|---|
| IKE | UDP 500 |
| NAT-T | UDP 4500 |
| ESP | IP protocol 50 |
| AH | IP protocol 51 |

Security relevance:

- VPNs protect traffic over untrusted networks.
- Weak VPN protocols should be avoided.
- Split tunneling affects monitoring and traffic visibility.

---

## Wireless Protocols

Wireless networks use radio frequencies instead of cables.

Common Wi-Fi bands:

| Band | Characteristics |
|---|---|
| 2.4 GHz | Longer range, better wall penetration, more interference |
| 5 GHz | Faster, shorter range, less congestion |
| 6 GHz | Newer, high performance, shorter range |

### Wi-Fi Security

| Protocol | Status |
|---|---|
| WEP | Broken / insecure |
| WPA | Legacy |
| WPA2 | Common modern baseline |
| WPA3 | Stronger modern option |

Security relevance:

- WEP should not be used.
- Hidden SSIDs are not real security.
- Weak Wi-Fi passwords can be attacked.
- Enterprise Wi-Fi often uses EAP-based authentication.

---

## Common Ports

| Protocol / Service | Port |
|---|---|
| FTP | TCP 21 |
| SSH | TCP 22 |
| Telnet | TCP 23 |
| SMTP | TCP 25 |
| DNS | TCP/UDP 53 |
| DHCP Server | UDP 67 |
| DHCP Client | UDP 68 |
| HTTP | TCP 80 |
| NTP | UDP 123 |
| SNMP | UDP 161/162 |
| HTTPS | TCP 443 |
| SMB | TCP 445 |
| RDP | TCP 3389 |

---

## Quick Reference

| Protocol | Main Purpose |
|---|---|
| TCP | Reliable transport |
| UDP | Fast connectionless transport |
| ICMP | Diagnostics and error reporting |
| ARP | IPv4-to-MAC resolution on local networks |
| DHCP | Automatic network configuration |
| DNS | Name resolution |
| HTTP | Web communication |
| HTTPS | Encrypted web communication |
| NAT | Address translation |
| OSPF | Internal routing |
| BGP | Internet routing |
| VLAN | Logical Layer 2 segmentation |
| STP | Loop prevention |
| VPN | Encrypted tunneling |

---

## Notes to Remember

- TCP is reliable and connection-oriented.
- UDP is fast and connectionless.
- ICMP is used for diagnostics, not normal application data.
- ARP resolves local IPv4 addresses to MAC addresses.
- DHCP automatically assigns IP configuration.
- DNS translates names to IP addresses.
- HTTPS is HTTP protected by TLS.
- NAT maps private addresses to public addresses.
- VLANs segment networks logically.
- VPNs create encrypted tunnels across untrusted networks.