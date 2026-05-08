# IDS/IPS

---

## Executive Summary

An **IDS** and **IPS** are security monitoring solutions used to detect suspicious or malicious network activity. An **Intrusion Detection System (IDS)** observes traffic and generates alerts, while an **Intrusion Prevention System (IPS)** can actively block or drop malicious traffic.

These systems help defenders identify attacks such as port scans, exploit attempts, malware callbacks, brute-force activity, command-and-control traffic, and policy violations. They are often deployed near firewalls, network chokepoints, server segments, or critical infrastructure.

---

## IDS vs IPS

Although IDS and IPS technologies are closely related, their main difference is how they respond to suspicious activity.

| Technology | Full Name | Main Role | Action |
|---|---|---|---|
| IDS | Intrusion Detection System | Detect suspicious activity | Alert only |
| IPS | Intrusion Prevention System | Detect and stop suspicious activity | Alert and block |

An IDS is passive. It watches traffic and reports what it sees.

An IPS is active. It sits inline with traffic and can prevent malicious packets from reaching their destination.

---

## Intrusion Detection System

An **IDS** monitors traffic or system activity for signs of compromise or attack.

When it sees suspicious behavior, it generates an alert for analysts to investigate.

Common IDS use cases include:

- Detecting port scans
- Detecting exploit attempts
- Detecting suspicious DNS traffic
- Detecting malware command-and-control callbacks
- Detecting brute-force attempts
- Detecting unusual protocol usage
- Detecting policy violations

IDS tools are useful because they provide visibility, but they do not stop the attack by themselves.

---

## Intrusion Prevention System

An **IPS** performs detection like an IDS, but it can also enforce a response.

Because an IPS is usually deployed inline, traffic must pass through it before reaching the destination.

Common IPS actions include:

| Action | Description |
|---|---|
| Alert | Generate a security alert |
| Drop | Silently discard malicious packets |
| Reject | Block the traffic and notify the sender |
| Reset | Terminate the connection |
| Block IP | Temporarily or permanently block a source |
| Rate limit | Slow down suspicious traffic |

An IPS can reduce risk quickly, but it must be tuned carefully. A false positive on an IPS can block legitimate business traffic.

---

## Network-Based IDS/IPS

A **Network-Based IDS/IPS** monitors traffic moving across the network.

It is commonly placed:

- At the network perimeter
- Between internal network segments
- In front of critical servers
- Near domain controllers
- Near database servers
- Near DMZ systems

Examples of network-based IDS/IPS tools include:

| Tool | Description |
|---|---|
| Snort | Popular open-source IDS/IPS rule engine |
| Suricata | High-performance IDS/IPS and network security monitoring engine |
| Zeek | Network security monitoring tool focused on protocol analysis and metadata |
| Security Onion | Defensive platform combining tools like Suricata, Zeek, and Elastic |

---

## Host-Based IDS

A **Host-Based IDS (HIDS)** monitors activity on a specific endpoint or server instead of watching the network.

It can detect events that may not be visible in network traffic, especially if traffic is encrypted.

Common HIDS data sources include:

- File integrity changes
- Process execution
- Logon events
- Registry changes
- Privilege escalation attempts
- Suspicious command execution
- Malware persistence locations

Examples of host-based monitoring tools include:

| Tool | Description |
|---|---|
| Wazuh | Open-source security monitoring and HIDS platform |
| OSSEC | Host-based intrusion detection system |
| Sysmon | Windows telemetry tool often used with SIEMs |
| Auditd | Linux auditing framework |

---

## Detection Methods

IDS/IPS tools use different detection methods to identify malicious behavior.

### Signature-Based Detection

Signature-based detection compares traffic or activity against known attack patterns.

Example:

```text
Alert when traffic matches a known EternalBlue exploit pattern.
```

Advantages:

- Accurate for known threats
- Easy to understand
- Good for known malware, exploits, and scans

Limitations:

- Weak against new or modified attacks
- Requires frequent rule updates
- Attackers can evade simple signatures

---

### Anomaly-Based Detection

Anomaly-based detection compares activity against a baseline of normal behavior.

Example:

```text
Alert when a workstation suddenly sends thousands of DNS requests per minute.
```

Advantages:

- Can detect unknown attacks
- Useful for behavioral monitoring
- Helps identify compromised systems

Limitations:

- Can generate many false positives
- Requires tuning
- Depends on a good baseline

---

### Protocol-Based Detection

Protocol-based detection checks whether traffic follows expected protocol behavior.

Example:

```text
Alert when HTTP traffic contains malformed headers or suspicious methods.
```

Advantages:

- Useful for detecting protocol abuse
- Can identify malformed traffic
- Helps detect evasion attempts

Limitations:

- Requires deep protocol understanding
- May miss attacks hidden in allowed protocol behavior

---

## IDS/IPS Placement

Placement determines what the IDS/IPS can see and what it can protect.

| Placement | Visibility |
|---|---|
| Internet edge | External attacks, scanning, exploit attempts |
| DMZ | Attacks against public-facing servers |
| Internal network | Lateral movement, internal scanning, compromised hosts |
| Data center | Attacks against critical servers |
| Cloud network | Traffic between cloud workloads |
| Endpoint | Local activity, encrypted traffic after decryption |

A single IDS/IPS at the perimeter is not enough. Many attacks happen after the attacker is already inside the network.

---

## Inline vs Passive Deployment

IDS/IPS sensors can be deployed in two main ways.

| Deployment | Description | Common Use |
|---|---|---|
| Passive | Receives copied traffic through a TAP or SPAN port | IDS monitoring |
| Inline | Traffic flows directly through the device | IPS blocking |

Passive deployment is safer because it cannot interrupt traffic.

Inline deployment provides prevention but introduces risk if the IPS fails or blocks legitimate traffic.

---

## Common Alert Types

IDS/IPS alerts often map to different phases of an attack.

| Alert Type | Example |
|---|---|
| Reconnaissance | Port scan, service enumeration |
| Exploitation | SQL injection attempt, buffer overflow payload |
| Credential Access | Brute-force login attempts |
| Command and Control | Beaconing to suspicious domains |
| Lateral Movement | SMB scanning, RDP brute force |
| Data Exfiltration | Large outbound transfers, DNS tunneling |
| Policy Violation | Cleartext passwords, unauthorized protocols |

---

## Rules and Signatures

IDS/IPS tools rely heavily on rules.

A rule defines what traffic should trigger an alert or action.

A simplified IDS rule may include:

- Source IP
- Destination IP
- Source port
- Destination port
- Protocol
- Message
- Content match
- Flow direction
- Severity
- Action

Example Snort-style rule:

```text
alert tcp any any -> any 80 (msg:"Possible malicious HTTP request"; content:"/etc/passwd"; sid:1000001;)
```

This rule alerts when TCP traffic to port 80 contains `/etc/passwd`, which may indicate a path traversal attempt.

---

## IDS/IPS Tuning

Tuning is the process of adjusting rules and alerts so the system is useful in a real environment.

Without tuning, IDS/IPS platforms can generate too many alerts, overwhelming analysts.

Common tuning actions include:

| Tuning Action | Purpose |
|---|---|
| Disable noisy rules | Reduce low-value alerts |
| Add allowlists | Avoid alerts for known legitimate systems |
| Add suppressions | Ignore repeated benign alerts |
| Adjust severity | Prioritize important detections |
| Customize rules | Match the organization’s environment |
| Add asset context | Increase priority for critical systems |

Good tuning reduces alert fatigue and improves detection quality.

---

## False Positives and False Negatives

IDS/IPS systems are not perfect.

| Term | Meaning |
|---|---|
| False Positive | Benign activity incorrectly flagged as malicious |
| False Negative | Malicious activity missed by detection |
| True Positive | Real malicious activity correctly detected |
| True Negative | Benign activity correctly ignored |

A good security program works to reduce both false positives and false negatives.

False positives waste analyst time.

False negatives allow attackers to operate unnoticed.

---

## Encryption Challenges

Encrypted traffic limits what a network IDS/IPS can inspect.

For example, HTTPS protects web traffic from eavesdropping, but it also hides payload content from network inspection tools unless decryption is configured.

Common approaches include:

- TLS inspection at a proxy
- Endpoint detection after decryption
- Metadata analysis
- JA3/JA4 fingerprinting
- DNS monitoring
- Certificate inspection
- Flow analysis

Even when content is encrypted, defenders can still analyze timing, destination, volume, certificate details, and connection patterns.

---

## IDS/IPS Evasion

Attackers may attempt to avoid detection by changing how traffic appears.

Common evasion techniques include:

| Technique | Description |
|---|---|
| Fragmentation | Splitting payloads across packets |
| Encoding | Obfuscating strings or payloads |
| Encryption | Hiding content inside encrypted channels |
| Protocol tunneling | Hiding traffic inside allowed protocols |
| Low-and-slow scanning | Reducing scan speed to avoid thresholds |
| User-Agent changes | Making traffic look like normal browsers |
| Payload modification | Changing exploit patterns to avoid signatures |

This is why detection should not rely on one rule or one tool.

---

## IDS/IPS and SIEM Integration

IDS/IPS alerts become more valuable when forwarded to a SIEM.

The SIEM can correlate IDS/IPS alerts with:

- Firewall logs
- Endpoint logs
- Authentication events
- DNS logs
- Proxy logs
- Cloud logs
- EDR alerts
- Vulnerability data

Example correlation:

```text
IDS detects SMB exploit attempt
+
Windows logs show successful logon
+
Endpoint logs show suspicious process execution
=
High-confidence compromise
```

IDS/IPS alerts are strongest when combined with other telemetry.

---

## Example Investigation Workflow

When an IDS/IPS alert fires, an analyst should not immediately assume compromise. The alert must be validated.

A basic workflow:

| Step | Question |
|---|---|
| Identify alert | What rule triggered? |
| Check source | Who sent the traffic? |
| Check destination | What asset was targeted? |
| Review payload | What did the traffic contain? |
| Check context | Is the destination vulnerable? |
| Correlate logs | Did authentication or process activity follow? |
| Determine impact | Was the attack blocked, failed, or successful? |
| Respond | Contain, escalate, tune, or close |

---

## Common Defensive Use Cases

IDS/IPS tools help with many defensive operations.

### Detecting Reconnaissance

```text
A host scans many ports across many systems.
```

Possible indicators:

- Many connection attempts
- Sequential port access
- Failed connection bursts
- Nmap-like traffic patterns

---

### Detecting Exploitation

```text
An attacker sends a known exploit payload to a vulnerable service.
```

Possible indicators:

- Known exploit strings
- Suspicious HTTP parameters
- Abnormal SMB traffic
- Malformed packets

---

### Detecting Malware Communication

```text
A compromised host contacts a command-and-control server.
```

Possible indicators:

- Repeated beaconing
- Suspicious domains
- Strange User-Agent values
- Connections to known bad IPs
- Unusual outbound ports

---

### Detecting Data Exfiltration

```text
A host sends unusually large amounts of data outside the network.
```

Possible indicators:

- Large outbound transfers
- DNS tunneling
- Repeated uploads
- Connections to cloud storage
- Traffic outside business hours

---

## IDS/IPS Limitations

IDS/IPS tools are powerful, but they have limits.

| Limitation | Explanation |
|---|---|
| Encrypted traffic | Payloads may be hidden |
| Alert fatigue | Too many alerts can overwhelm analysts |
| False positives | Normal traffic may match signatures |
| Evasion | Attackers can modify traffic |
| Blind spots | Sensors only see traffic that reaches them |
| Lack of context | Alerts need asset and user context |
| Performance impact | Inline IPS can affect traffic flow |

IDS/IPS should be part of a layered defense strategy, not the only control.

---

## Best Practices

Strong IDS/IPS deployments follow several best practices.

| Practice | Why It Matters |
|---|---|
| Keep rules updated | Detect newer threats |
| Tune noisy alerts | Reduce alert fatigue |
| Monitor internal traffic | Detect lateral movement |
| Integrate with SIEM | Improve correlation |
| Use asset context | Prioritize critical systems |
| Test rules safely | Avoid blocking legitimate traffic |
| Review blocked traffic | Ensure IPS actions are correct |
| Combine with EDR | Cover endpoint visibility gaps |

---

## Quick Reference

| Concept | Meaning |
|---|---|
| IDS | Detects suspicious activity and alerts |
| IPS | Detects and blocks suspicious activity |
| Signature detection | Matches known attack patterns |
| Anomaly detection | Detects abnormal behavior |
| HIDS | Host-based intrusion detection |
| NIDS | Network-based intrusion detection |
| Inline | Traffic passes through the device |
| Passive | Device receives copied traffic |
| False positive | Benign activity flagged as malicious |
| False negative | Malicious activity missed |

---

## Key Takeaways

An IDS detects suspicious activity and alerts defenders.

An IPS can actively block or drop malicious traffic.

IDS/IPS tools are useful for detecting scans, exploits, malware callbacks, brute force, lateral movement, and exfiltration.

Tuning is essential because noisy alerts can overwhelm analysts.

Encrypted traffic and attacker evasion techniques limit what IDS/IPS tools can see.

IDS/IPS works best when integrated with SIEM, endpoint logs, firewall logs, DNS logs, and asset context.