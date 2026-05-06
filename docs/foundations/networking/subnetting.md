# Subnetting

Subnetting is the process of dividing a larger IP network into smaller subnetworks.

It is used to organize networks, control traffic, improve address management, and separate systems by purpose, department, location, or security zone.

---

## Why Subnetting Matters

Without subnetting, every device could end up in one large flat network.

That can create problems:

- Too much broadcast traffic
- Poor organization
- Harder troubleshooting
- Weak separation between departments
- Inefficient IP address usage
- Larger impact if one segment is compromised

Subnetting helps create smaller, more manageable network segments.

Examples:

```text
Accounting subnet
HR subnet
Server subnet
Guest Wi-Fi subnet
Security camera subnet
Management subnet
```

---

## Core Concepts

Every IPv4 subnet has important address types.

| Address Type | Purpose | Example |
|---|---|---|
| Network address | Identifies the subnet itself | `192.168.1.0` |
| Usable host addresses | Assigned to devices | `192.168.1.10` |
| Default gateway | Routes traffic to other networks | `192.168.1.1` |
| Broadcast address | Sends traffic to all hosts in the subnet | `192.168.1.255` |

Example subnet:

```text
Network:        192.168.1.0/24
Usable hosts:   192.168.1.1 - 192.168.1.254
Broadcast:      192.168.1.255
```

---

## Subnet Mask

A subnet mask defines which part of an IP address is the network portion and which part is the host portion.

Example:

```text
IP address:     192.168.1.25
Subnet mask:    255.255.255.0
CIDR:           /24
```

In this example:

```text
Network portion: 192.168.1
Host portion:    25
```

So `192.168.1.25/24` belongs to the network:

```text
192.168.1.0/24
```

---

## CIDR Notation

CIDR notation shows how many bits are used for the network portion.

```text
192.168.1.0/24
```

The `/24` means the first 24 bits are the network portion.

Common CIDR examples:

| CIDR | Subnet Mask | Usable Hosts |
|---|---|---|
| `/24` | `255.255.255.0` | 254 |
| `/25` | `255.255.255.128` | 126 |
| `/26` | `255.255.255.192` | 62 |
| `/27` | `255.255.255.224` | 30 |
| `/28` | `255.255.255.240` | 14 |
| `/29` | `255.255.255.248` | 6 |
| `/30` | `255.255.255.252` | 2 |

!!! note
    The number of usable hosts is usually total addresses minus 2: one for the network address and one for the broadcast address.

---

## Host Calculation

IPv4 addresses are 32 bits.

Formula:

```text
Total addresses = 2^(host bits)
Usable hosts = 2^(host bits) - 2
```

Example:

```text
/24 means 24 network bits
32 - 24 = 8 host bits
2^8 = 256 total addresses
256 - 2 = 254 usable hosts
```

---

## Example: /24 Network

```text
Network:        192.168.10.0/24
Subnet mask:    255.255.255.0
Broadcast:      192.168.10.255
Usable range:   192.168.10.1 - 192.168.10.254
```

This is common in home and small office networks.

---

## Example: /26 Network

A `/26` subnet has 64 total addresses and 62 usable hosts.

```text
Network:        192.168.10.0/26
Subnet mask:    255.255.255.192
Broadcast:      192.168.10.63
Usable range:   192.168.10.1 - 192.168.10.62
```

The next `/26` subnet starts at:

```text
192.168.10.64/26
```

---

## Subnet Increments

The subnet increment tells you where each subnet starts.

For a `/26`:

```text
Subnet mask: 255.255.255.192
Block size:  256 - 192 = 64
```

So the subnets are:

```text
192.168.10.0/26
192.168.10.64/26
192.168.10.128/26
192.168.10.192/26
```

---

## Common Subnet Sizes

| CIDR | Total Addresses | Usable Hosts | Common Use |
|---|---:|---:|---|
| `/24` | 256 | 254 | Small LAN |
| `/25` | 128 | 126 | Split a /24 in half |
| `/26` | 64 | 62 | Department subnet |
| `/27` | 32 | 30 | Small team or lab segment |
| `/28` | 16 | 14 | Small service segment |
| `/29` | 8 | 6 | Network devices |
| `/30` | 4 | 2 | Point-to-point links |

---

## Default Gateway

The default gateway is the router interface used to reach other networks.

Example:

```text
Host IP:         192.168.1.50
Subnet mask:     255.255.255.0
Default gateway: 192.168.1.1
```

If the host needs to reach another subnet or the internet, it sends traffic to the default gateway.

```text
Host -> Default Gateway -> Other Network
```

---

## Security Benefits

Subnetting can improve security by separating systems.

Example:

| Subnet | Purpose |
|---|---|
| `192.168.10.0/24` | User devices |
| `192.168.20.0/24` | Servers |
| `192.168.30.0/24` | Guest Wi-Fi |
| `192.168.40.0/24` | Cameras / IoT |
| `192.168.50.0/24` | Management |

Benefits:

- Guest devices can be isolated from internal systems.
- Servers can be protected with firewall rules.
- IoT devices can be restricted.
- Sensitive systems can be placed in controlled segments.
- Monitoring can be organized by subnet.

!!! important
    Subnetting alone does not enforce security. You need routing rules, firewall rules, ACLs, VLANs, or other controls to restrict traffic between subnets.

---

## Subnetting and VLANs

Subnets and VLANs are often used together.

| Concept | Layer | Purpose |
|---|---|---|
| VLAN | Layer 2 | Separates broadcast domains |
| Subnet | Layer 3 | Defines IP network boundaries |

Common design:

```text
VLAN 10 -> 192.168.10.0/24 -> Users
VLAN 20 -> 192.168.20.0/24 -> Servers
VLAN 30 -> 192.168.30.0/24 -> Guest Wi-Fi
```

Traffic between VLANs usually requires routing.

---

## Quick Method

When subnetting, ask:

1. What network am I starting with?
2. How many subnets do I need?
3. How many hosts are needed per subnet?
4. What CIDR size supports that number of hosts?
5. What are the network, usable, and broadcast addresses?

---

## Quick Reference

| Need | Suggested CIDR |
|---|---|
| Around 250 hosts | `/24` |
| Around 125 hosts | `/25` |
| Around 60 hosts | `/26` |
| Around 30 hosts | `/27` |
| Around 14 hosts | `/28` |
| Around 6 hosts | `/29` |
| 2 hosts | `/30` |

---

## Notes to Remember

- Subnetting divides larger networks into smaller networks.
- The subnet mask separates the network portion from the host portion.
- CIDR notation shows how many bits belong to the network.
- The network address identifies the subnet.
- The broadcast address reaches all hosts in the subnet.
- The default gateway routes traffic to other networks.
- Smaller subnets reduce broadcast scope and improve organization.
- Subnetting helps security, but firewall rules enforce security.