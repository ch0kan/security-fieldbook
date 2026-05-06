# Network Fundamentals

Networking is the foundation for both offensive and defensive security. Before scanning, exploiting, monitoring, or responding to incidents, it is important to understand how devices communicate, how traffic is addressed, and how data moves through network layers.

This page covers the core concepts: network types, network components, OSI/TCP-IP models, packets, frames, ports, and basic data flow.

---

## What Is a Network?

A network is a group of connected devices that communicate and share resources.

Common networked devices include:

- Laptops
- Desktops
- Servers
- Phones
- Printers
- Routers
- Switches
- IoT devices
- Virtual machines

A basic network has three main parts:

| Component | Purpose |
|---|---|
| Nodes | Devices that send, receive, or forward data |
| Links | Wired or wireless paths that carry traffic |
| Protocols | Rules that define how communication happens |

Examples of links include Ethernet cables, fiber optics, Wi-Fi, and cellular radio signals.

---

## LAN, WAN, and VPN

Networks are often described by their size, ownership, and purpose.

### Local Area Network

A Local Area Network, or LAN, connects devices in a limited area such as a home, office, lab, or school.

LAN characteristics:

- Usually privately owned
- Often uses private IP address ranges
- Commonly uses Ethernet or Wi-Fi
- Usually has low latency
- Often managed by one person or organization

Examples:

```text
Home network
Office network
School computer lab
Internal company network
```

### Wireless LAN

A Wireless LAN, or WLAN, is a LAN that uses Wi-Fi instead of only wired connections.

```text
Laptop -> Wi-Fi access point -> Router -> Internet
```

### Wide Area Network

A Wide Area Network, or WAN, connects networks across larger distances.

WAN characteristics:

- Covers cities, countries, or global regions
- Often uses ISP infrastructure
- Usually has higher latency than LANs
- Connects multiple LANs together

The internet is the largest example of a WAN.

### Virtual Private Network

A Virtual Private Network, or VPN, creates an encrypted tunnel across another network.

Common VPN types:

| Type | Purpose |
|---|---|
| Remote access VPN | Connects one user to a private network |
| Site-to-site VPN | Connects two networks together |
| Split-tunnel VPN | Sends only private-network traffic through the VPN |
| Full-tunnel VPN | Sends all traffic through the VPN |

VPNs are common in labs, remote work, and internal assessments.

---

## Network Components

Networks are built from endpoint devices, intermediary devices, interfaces, and media.

| Category | Examples | Purpose |
|---|---|---|
| End devices | Laptops, phones, servers, printers | Source or destination of data |
| Intermediary devices | Switches, routers, firewalls, access points | Forward, filter, or manage traffic |
| Interfaces | NICs, wireless adapters | Connect devices to the network |
| Media | Ethernet, fiber, Wi-Fi | Carry signals between devices |

### Switches

A switch connects devices inside a LAN.

Switches usually operate at Layer 2 and use MAC addresses to forward frames to the correct device.

```text
PC1 -> Switch -> PC2
```

Modern switches are more efficient than hubs because they send traffic only where it needs to go.

### Routers

A router connects different networks.

Routers usually operate at Layer 3 and use IP addresses to forward packets between networks.

```text
LAN -> Router -> ISP -> Internet
```

### Network Interface Cards

A Network Interface Card, or NIC, connects a host to the network.

Each NIC has a MAC address used for local network communication.

Example MAC address:

```text
00:1A:2B:3C:4D:5E
```

---

## Physical vs. Logical Communication

Network communication has both physical and logical parts.

| Type | Description | Example |
|---|---|---|
| Physical | Actual signal or medium | Ethernet cable, Wi-Fi radio |
| Logical | Addressing and routing decisions | IP addresses, routing tables |

A device may physically connect to a switch, but logically communicate with a server across many networks.

---

## OSI Model

The OSI model is a seven-layer model used to understand network communication.

| Layer | Name | Main Function | Examples |
|---|---|---|---|
| 7 | Application | User-facing network services | HTTP, DNS, FTP, SMTP |
| 6 | Presentation | Data formatting, encryption, compression | TLS, MIME, JPEG |
| 5 | Session | Session creation and management | RPC, NFS |
| 4 | Transport | End-to-end communication | TCP, UDP |
| 3 | Network | Logical addressing and routing | IP, ICMP |
| 2 | Data Link | Local delivery and MAC addressing | Ethernet, Wi-Fi |
| 1 | Physical | Raw signal transmission | Cables, radio, fiber |

The OSI model is not something you usually configure directly. It is a mental model that helps you troubleshoot and understand where a network issue occurs.

---

## OSI Layers in Practical Terms

### Layer 1: Physical

Layer 1 handles the physical transmission of bits.

Examples:

- Ethernet cables
- Fiber optic cables
- Wi-Fi radio signals
- Hubs
- Repeaters
- Physical ports

Security relevance:

- Physical access matters.
- Cable taps and rogue devices can expose traffic.
- Broken cables or weak wireless signals can cause connectivity issues.

### Layer 2: Data Link

Layer 2 handles local network delivery using MAC addresses.

Examples:

- Ethernet frames
- Switches
- MAC addresses
- ARP
- Wi-Fi frames

Security relevance:

- MAC spoofing
- ARP spoofing
- VLAN hopping
- Rogue devices on a LAN

### Layer 3: Network

Layer 3 handles logical addressing and routing.

Examples:

- IP addresses
- Routers
- ICMP
- Routing tables

Security relevance:

- Firewall rules often use IP addresses.
- Routing issues can break connectivity.
- ICMP can be used for diagnostics and discovery.

### Layer 4: Transport

Layer 4 handles communication between processes using TCP or UDP.

Examples:

- TCP
- UDP
- Ports
- Connection tracking

Security relevance:

- Port scanning targets this layer.
- Firewalls often filter by port and protocol.
- TCP connection states help explain scan results.

### Layers 5–7: Session, Presentation, Application

These layers handle sessions, formatting, encryption, and user-facing services.

Examples:

- HTTP
- HTTPS
- DNS
- SMTP
- SMB
- TLS

Security relevance:

- Web attacks happen mostly at Layer 7.
- TLS protects data in transit.
- Application logs often show user activity.

---

## TCP/IP Model

The TCP/IP model is a practical four-layer model used to describe internet communication.

| TCP/IP Layer | Rough OSI Equivalent | Examples |
|---|---|---|
| Application | OSI 5–7 | HTTP, DNS, SMTP |
| Transport | OSI 4 | TCP, UDP |
| Internet | OSI 3 | IP, ICMP |
| Network Interface | OSI 1–2 | Ethernet, Wi-Fi |

The TCP/IP model is simpler than OSI and maps more closely to real-world networking.

---

## Encapsulation

Encapsulation is the process of wrapping data with headers as it moves down the network stack.

```text
Application Data
    ↓
TCP/UDP Segment
    ↓
IP Packet
    ↓
Ethernet/Wi-Fi Frame
    ↓
Bits on the wire or radio signal
```

Each layer adds information needed for delivery.

| Layer | Data Unit | Addressing |
|---|---|---|
| Application | Data | Application-specific |
| Transport | Segment or Datagram | Ports |
| Network | Packet | IP addresses |
| Data Link | Frame | MAC addresses |
| Physical | Bits | Signals |

---

## Packets vs. Frames

Packets and frames are related, but they are not the same thing.

| Term | Layer | Contains |
|---|---|---|
| Packet | Layer 3 | IP header and payload |
| Frame | Layer 2 | MAC addresses and encapsulated packet |

Simple analogy:

```text
Frame  = Envelope used for local delivery
Packet = Letter inside the envelope
```

A packet can travel across many networks. A frame is rebuilt at each local network segment.

---

## MAC Addresses

A MAC address is a Layer 2 hardware identifier used for local network communication.

Example:

```text
7C:DF:A1:D3:8C:5C
```

Key points:

- Usually assigned by the NIC manufacturer
- Used inside a LAN
- Required for Ethernet and Wi-Fi frame delivery
- Can often be spoofed in software

MAC addresses help switches decide where to forward frames.

---

## IP Addresses

An IP address is a Layer 3 logical address used for routing between networks.

Example IPv4 address:

```text
192.168.1.50
```

Example IPv6 address:

```text
2001:db8::10
```

IP addresses help routers move packets toward the destination network.

### Private IP Ranges

Private IP addresses are used inside internal networks and are not directly routable on the public internet.

| Range | Common Use |
|---|---|
| `10.0.0.0/8` | Large private networks |
| `172.16.0.0/12` | Medium private networks |
| `192.168.0.0/16` | Home and small office networks |

---

## Ports

Ports identify specific services or processes on a host.

Port range:

```text
0 - 65535
```

Common ports:

| Service | Port |
|---|---|
| FTP | TCP 21 |
| SSH | TCP 22 |
| DNS | TCP/UDP 53 |
| HTTP | TCP 80 |
| HTTPS | TCP 443 |
| SMB | TCP 445 |
| RDP | TCP 3389 |

Ports matter because one IP address can host many services.

Example:

```text
192.168.1.10:22  -> SSH
192.168.1.10:80  -> HTTP
192.168.1.10:443 -> HTTPS
```

---

## TCP

TCP is a connection-oriented transport protocol.

TCP focuses on reliable delivery.

Characteristics:

- Connection-based
- Uses acknowledgements
- Supports retransmission
- Maintains order
- Provides flow control
- More reliable than UDP
- Usually slower than UDP

Common TCP services:

| Service | Port |
|---|---|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| SMB | 445 |

---

## TCP Three-Way Handshake

TCP uses a three-way handshake to establish a connection.

```text
Client -> SYN -> Server
Client <- SYN/ACK <- Server
Client -> ACK -> Server
```

After the handshake, data can be exchanged.

### Common TCP Flags

| Flag | Purpose |
|---|---|
| SYN | Start a connection |
| ACK | Acknowledge received data |
| FIN | Gracefully close a connection |
| RST | Abruptly reset a connection |
| PSH | Push data to the application quickly |

Security relevance:

- Nmap SYN scans rely on TCP handshake behavior.
- Firewalls track TCP connection states.
- RST responses often indicate closed ports.

---

## UDP

UDP is a connectionless transport protocol.

UDP focuses on speed and simplicity.

Characteristics:

- No handshake
- No delivery guarantee
- No built-in retransmission
- No connection state
- Lower overhead than TCP

Common UDP uses:

| Use Case | Why UDP Fits |
|---|---|
| DNS | Small fast queries |
| VoIP | Real-time communication |
| Streaming | Speed matters more than perfect delivery |
| Gaming | Low latency is important |

Security relevance:

- UDP scanning is slower and harder to interpret.
- Lack of response does not always mean a port is closed.
- Some UDP services are abused for amplification attacks.

---

## Transmission Modes

Transmission modes describe how communication flows between devices.

| Mode | Description | Example |
|---|---|---|
| Simplex | One-way only | Keyboard to computer |
| Half-duplex | Two-way, one direction at a time | Walkie-talkie |
| Full-duplex | Two-way at the same time | Phone call |

---

## Network Topologies

A network topology describes how devices are arranged or how data flows.

### Physical Topology

Physical topology describes the actual layout of cables, devices, and wireless links.

### Logical Topology

Logical topology describes how traffic moves, even if the physical layout looks different.

Common topologies:

| Topology | Description | Notes |
|---|---|---|
| Point-to-point | Direct link between two devices | Simple and dedicated |
| Bus | Devices share one medium | Legacy and collision-prone |
| Star | Devices connect to a central device | Common in modern LANs |
| Ring | Devices form a loop | Used in some legacy/token systems |
| Mesh | Devices have multiple paths | High redundancy |
| Tree | Hierarchical star-like design | Common in larger networks |
| Hybrid | Combination of topologies | Common in real environments |

---

## Web Request Data Flow

When you visit a website, several networking steps happen behind the scenes.

Example:

```text
https://example.com
```

### 1. Network Configuration

The device needs basic network settings:

- IP address
- Subnet mask
- Default gateway
- DNS server

These are often provided by DHCP.

### 2. DNS Resolution

The browser needs the IP address for the domain.

```text
example.com -> 93.184.216.34
```

### 3. ARP for Local Delivery

If the destination is outside the local network, the device sends traffic to the default gateway.

To do that, it needs the gateway MAC address.

```text
Gateway IP -> Gateway MAC address
```

ARP performs this local IP-to-MAC mapping.

### 4. TCP Connection

For HTTPS, the browser usually connects to TCP port 443.

```text
Client ephemeral port -> Server TCP 443
```

### 5. TLS and HTTP

For HTTPS, TLS protects the communication. Then the browser sends the HTTP request through the encrypted connection.

### 6. Routing and NAT

The packet travels through the router, ISP, and internet. If NAT is used, the router translates the private source IP to a public IP.

### 7. Response and Decapsulation

The server responds, and each layer processes the headers until the browser receives the web content.

---

## Useful Commands

### View IP Configuration

```bash
# Linux
ip addr show

# Windows
ipconfig /all

# macOS/Linux legacy
ifconfig
```

### View MAC Addresses

```bash
# Linux
ip link show

# Windows
getmac /v

# macOS/Linux
ifconfig
```

### View Routing Table

```bash
# Linux/macOS
netstat -rn

# Linux modern
ip route

# Windows
route print
```

### View Active Connections

```bash
# Windows
netstat -ano

# Linux
ss -tulpn
```

### Test Connectivity

```bash
ping 8.8.8.8
```

### Trace Route

```bash
# Linux/macOS
traceroute 8.8.8.8

# Windows
tracert 8.8.8.8
```

### View ARP Table

```bash
arp -a
```

---

## Quick Reference

| Concept | Layer | Purpose |
|---|---|---|
| Cable / Wi-Fi signal | Layer 1 | Moves bits |
| MAC address | Layer 2 | Local delivery |
| Frame | Layer 2 | Data link container |
| IP address | Layer 3 | Routing between networks |
| Packet | Layer 3 | Network layer container |
| TCP/UDP port | Layer 4 | Service/process targeting |
| TCP | Layer 4 | Reliable transport |
| UDP | Layer 4 | Fast connectionless transport |
| HTTP/DNS/SMTP | Layer 7 | Application protocols |

---

## Notes to Remember

- A frame is used for local network delivery.
- A packet is routed across networks.
- MAC addresses work at Layer 2.
- IP addresses work at Layer 3.
- Ports work at Layer 4.
- TCP is reliable and connection-oriented.
- UDP is faster but does not guarantee delivery.
- Routers forward packets between networks.
- Switches forward frames inside a LAN.
- DNS turns names into IP addresses.
- ARP turns local IP addresses into MAC addresses.