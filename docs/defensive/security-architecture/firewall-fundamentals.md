# Firewall Fundamentals

A firewall is a network security control that filters traffic based on rules.

Firewalls help prevent unauthorized access, segment networks, enforce policy, and reduce attack surface.

---

## Firewall Purpose

Firewalls answer questions like:

```text
Who can connect?
Where can they connect?
Which protocol can they use?
Which port is allowed?
Should this traffic be allowed, denied, or forwarded?
```

---

## Firewalls and the OSI Model

Different firewall types inspect different layers.

| Firewall Type | OSI Layer | Focus |
|---|---|---|
| Stateless firewall | Layer 3 and 4 | IPs, ports, protocols |
| Stateful firewall | Layer 3 and 4 | Connection state |
| Proxy firewall | Layer 7 | Application content |
| Next-generation firewall | Layer 3 through 7 | Advanced inspection and threat prevention |

---

# Stateless Firewalls

A stateless firewall treats every packet as independent.

It checks packet headers against a rule list.

Common criteria:

- Source IP
- Destination IP
- Protocol
- Source port
- Destination port

---

## Stateless Firewall Strengths

| Strength | Description |
|---|---|
| Fast | Minimal processing |
| Efficient | Good for high-volume filtering |
| Simple | Easy rule logic |

---

## Stateless Firewall Weaknesses

| Weakness | Description |
|---|---|
| No memory | Does not track connection context |
| Limited awareness | Cannot understand full session behavior |
| Easier to bypass | Cannot detect many state-based anomalies |

Example limitation:

```text
A packet may be allowed because its port matches the rule,
even if it is part of an abnormal or malicious connection pattern.
```

---

# Stateful Firewalls

A stateful firewall tracks active connections.

It maintains a state table.

For TCP, it understands connection flow such as:

```text
SYN
SYN-ACK
ACK
Established session
```

If traffic belongs to an approved session, it can be allowed based on state.

---

## Stateful Firewall Strengths

| Strength | Description |
|---|---|
| Tracks sessions | Understands connection state |
| More secure | Blocks traffic that does not belong to valid sessions |
| Practical | Common in enterprise networks |

---

## Stateful Firewall Weaknesses

| Weakness | Description |
|---|---|
| More resource usage | Requires memory and CPU for state table |
| State exhaustion risk | Many connections can overload state tracking |
| Still limited at app layer | Does not fully understand application content by default |

---

# Proxy Firewalls

A proxy firewall acts as an intermediary.

Instead of allowing a direct connection, it connects on behalf of the client.

Flow:

```text
Client -> Proxy firewall -> Internet service
```

This allows deeper inspection of application traffic.

---

## Proxy Firewall Strengths

| Strength | Description |
|---|---|
| Application visibility | Inspects Layer 7 traffic |
| Content filtering | Can block URLs, file types, or commands |
| Internal IP masking | External systems see the proxy |
| Strong policy control | Useful for web and application access |

---

## Proxy Firewall Weaknesses

| Weakness | Description |
|---|---|
| Slower | Application inspection adds overhead |
| Complex | More policy and certificate management |
| App-specific | May need protocol-aware configuration |

---

# Next-Generation Firewalls

Next-generation firewalls, or NGFWs, combine traditional firewalling with advanced security features.

Common NGFW features:

- Stateful filtering
- Deep packet inspection
- Intrusion prevention
- Application awareness
- User identity awareness
- Threat intelligence integration
- SSL/TLS inspection
- Malware detection
- URL filtering

---

## SSL/TLS Inspection

Encrypted traffic can hide threats.

Some NGFWs decrypt, inspect, and re-encrypt traffic.

This can detect:

- Malware downloads over HTTPS
- C2 communication
- Data exfiltration
- Policy violations

Important considerations:

- Privacy
- Legal requirements
- Certificate management
- Performance impact
- Sensitive categories that should not be decrypted

---

# Firewall Rule Anatomy

A firewall rule is an if-then decision.

Example:

```text
If source is 192.168.1.0/24
and destination is any
and protocol is TCP
and port is 443
then allow outbound traffic
```

---

## Rule Components

| Component | Description | Example |
|---|---|---|
| Source address | Where traffic starts | `192.168.1.50` |
| Destination address | Where traffic goes | `8.8.8.8` |
| Protocol | TCP, UDP, ICMP | `TCP` |
| Port | Service destination | `443` |
| Direction | Inbound, outbound, forward | `Outbound` |
| Action | Allow, deny, forward | `Allow` |

---

# Rule Actions

## Allow

Allows matching traffic.

Example:

```text
Allow internal users outbound TCP 443
```

Use case:

```text
Employees browsing HTTPS websites.
```

---

## Deny

Blocks matching traffic.

Example:

```text
Deny inbound TCP 22 from internet to database server
```

Use case:

```text
Prevent public SSH access to sensitive systems.
```

---

## Forward

Redirects traffic to another destination.

Common with NAT or port forwarding.

Example:

```text
Forward inbound TCP 443 on public IP to internal web server
```

Use case:

```text
Public website hosted behind a firewall.
```

---

# Directionality

## Inbound Rules

Inbound rules control traffic entering the network.

Examples:

- Internet to web server
- VPN users to internal systems
- Partner network to application server

Primary goal:

```text
Protect internal services from external threats.
```

---

## Outbound Rules

Outbound rules control traffic leaving the network.

Examples:

- Workstations to internet
- Servers to update repositories
- Internal hosts to DNS resolvers
- Database servers to approved app servers only

Primary goal:

```text
Limit what compromised internal systems can reach.
```

Outbound filtering is important for detecting and stopping:

- C2 connections
- Data exfiltration
- Spam from compromised hosts
- Unauthorized cloud storage usage

---

## Forward / Internal Rules

Forward rules control traffic between network segments.

Examples:

- User VLAN to server VLAN
- DMZ to database network
- Production to backup network

Primary goal:

```text
Enforce segmentation and least privilege.
```

---

# Implicit Deny

Professional firewalls usually have an implicit deny rule at the bottom.

Meaning:

```text
If traffic does not match an allow rule, block it.
```

This supports a default-deny security model.

Recommended approach:

```text
Deny by default.
Allow only what is required.
Log important denies.
Review rules regularly.
```

---

# Firewall Rule Order

Firewall rules are usually evaluated top to bottom.

The first matching rule wins.

Example:

```text
1. Allow admin workstation to SSH server
2. Deny all SSH to server
3. Allow HTTPS to web server
4. Deny all
```

If rule 2 came before rule 1, even the admin workstation could be blocked.

---

## Rule Hygiene

Good firewall management requires:

- Clear rule names
- Business justification
- Owner
- Expiration date where possible
- Change ticket reference
- Regular review
- Removal of unused rules
- Avoiding broad `any-any` rules

---

## Bad Rule Examples

Avoid rules like:

```text
Allow Any -> Any Any Port
Allow Internet -> Internal Any
Allow Workstations -> Servers Any
```

These create excessive exposure and weaken segmentation.

---

## Better Rule Examples

Prefer specific rules:

```text
Allow Helpdesk subnet -> Server subnet TCP 3389
Allow Web server -> Database server TCP 5432
Allow Internal DNS clients -> DNS resolver UDP/TCP 53
Deny Internet -> Internal RFC1918 ranges
```

---

# Firewall Logs

Firewall logs are useful for:

- Investigation
- Tuning rules
- Detecting scans
- Detecting blocked attacks
- Detecting suspicious outbound traffic
- Verifying whether traffic was allowed or denied

Common log fields:

| Field | Use |
|---|---|
| Timestamp | When the connection occurred |
| Source IP | Who initiated |
| Destination IP | Target |
| Source port | Client-side port |
| Destination port | Service |
| Protocol | TCP, UDP, ICMP |
| Action | Allow or deny |
| Rule name | Which rule matched |
| Bytes | Traffic volume |

---

## Detection Ideas

Look for:

- Denied inbound scans
- Repeated denied access attempts
- Outbound traffic to rare countries
- Internal hosts connecting to suspicious IPs
- Workstations using unusual ports
- Servers initiating unexpected outbound traffic
- Large outbound data transfers
- Allowed traffic that should have been denied

---

## Firewall Comparison

| Type | Layer | Main Benefit | Main Limitation |
|---|---|---|---|
| Stateless | 3/4 | Very fast | No session context |
| Stateful | 3/4 | Tracks connections | Limited app awareness |
| Proxy | 7 | Deep app inspection | More latency |
| NGFW | 3-7 | Advanced protection | More complex and resource-heavy |

---

## Quick Reference

| Concept | Meaning |
|---|---|
| Source | Where traffic starts |
| Destination | Where traffic goes |
| Protocol | TCP, UDP, ICMP |
| Port | Service endpoint |
| Inbound | Into the network |
| Outbound | Out of the network |
| Forward | Between networks or NAT destination |
| Allow | Permit traffic |
| Deny | Block traffic |
| Implicit deny | Default block if no rule matches |
| State table | Tracks active connections |
| DPI | Deep packet inspection |
| NGFW | Next-generation firewall |

---

## Notes to Remember

- Stateless firewalls are fast but lack memory.
- Stateful firewalls track connection state.
- Proxy firewalls inspect application content.
- NGFWs combine firewalling with advanced threat detection.
- Firewall rules should follow least privilege.
- The implicit deny rule blocks anything not explicitly allowed.
- Outbound filtering is important for stopping C2 and exfiltration.