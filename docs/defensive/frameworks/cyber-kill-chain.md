# Cyber Kill Chain

The Cyber Kill Chain is a framework developed by Lockheed Martin to describe the stages of a cyberattack.

It helps defenders understand how attacks progress and where security controls can interrupt the attacker.

---

## Overview

The Cyber Kill Chain breaks an intrusion into sequential phases.

The main idea:

```text
If defenders interrupt any stage, the attack may fail.
```

It is useful for:

- Understanding attacker goals
- Building layered defenses
- Identifying control gaps
- Structuring incident timelines
- Training SOC analysts
- Explaining ransomware and APT activity

---

## The Seven Phases

| Phase | Description |
|---|---|
| Reconnaissance | Attacker gathers information |
| Weaponization | Attacker prepares payload and infrastructure |
| Delivery | Payload is delivered to the target |
| Exploitation | Vulnerability is triggered |
| Installation | Malware or backdoor is installed |
| Command and Control | Compromised host communicates with attacker |
| Actions on Objectives | Attacker completes their goal |

---

## 1. Reconnaissance

Reconnaissance is the information-gathering phase.

Attackers may collect:

- Domains
- IP addresses
- Employee names
- Email addresses
- Technologies used
- Exposed services
- Public documents
- Social media information
- Leaked credentials

Defensive opportunities:

- Reduce public exposure
- Monitor for scanning
- Watch for credential leaks
- Limit public metadata
- Harden exposed services

---

## 2. Weaponization

Weaponization is where the attacker prepares the attack package.

Examples:

- Malicious document
- Exploit payload
- Phishing kit
- Malware loader
- Backdoor
- Exploit chain
- Command and control infrastructure

Defensive opportunities:

- Threat intelligence
- Malware analysis
- Detection engineering
- Block known malicious infrastructure
- Monitor suspicious file types

---

## 3. Delivery

Delivery is how the attack reaches the victim.

Common delivery methods:

- Phishing email
- Malicious attachment
- Malicious link
- Drive-by download
- USB device
- Exposed service exploitation
- Supply chain compromise

Defensive opportunities:

- Email filtering
- Web filtering
- Attachment sandboxing
- User awareness training
- Blocking risky file types
- Network monitoring

---

## 4. Exploitation

Exploitation occurs when the attacker triggers a vulnerability or abuses a weakness.

Examples:

- Exploiting a vulnerable web application
- Running malicious macro code
- Exploiting a public-facing service
- Abusing weak credentials
- Exploiting browser or plugin vulnerabilities

Defensive opportunities:

- Patch management
- Secure configuration
- Input validation
- Exploit protection
- EDR monitoring
- Web application firewalls

---

## 5. Installation

Installation is where the attacker establishes a foothold.

Examples:

- Installing malware
- Dropping a web shell
- Creating a scheduled task
- Modifying registry run keys
- Creating a malicious service
- Adding startup folder entries

This phase often overlaps with persistence.

---

## Persistence

Persistence allows an attacker to regain access after reboot, detection, or loss of initial access.

Examples:

| Method | Description |
|---|---|
| Web shell | Server-side script used to execute commands |
| Backdoor payload | Malware or remote-control payload |
| Windows service | Malicious service created or modified |
| Run keys | Registry entries that execute at login |
| Startup folder | Files that run when a user logs in |

Example MITRE mapping:

```text
Create or Modify System Process: Windows Service
T1543.003
```

Defensive opportunities:

- Monitor service creation
- Monitor registry run keys
- Monitor startup folders
- Detect unusual web files
- Review scheduled tasks
- Use EDR and file integrity monitoring

---

## 6. Command and Control

Command and Control, or C2, is the communication channel between compromised systems and attacker infrastructure.

C2 allows attackers to:

- Send commands
- Receive output
- Download tools
- Exfiltrate data
- Move laterally
- Maintain remote control

Common C2 channels:

| Channel | Notes |
|---|---|
| HTTP/HTTPS | Blends with web traffic |
| DNS | Can be used for DNS tunneling |
| IRC | Older C2 method |
| Cloud services | May blend with legitimate traffic |

Defensive opportunities:

- DNS monitoring
- Proxy logs
- EDR network telemetry
- Beaconing detection
- Block suspicious domains
- TLS inspection where appropriate
- Egress filtering

---

## C2 Beaconing

Beaconing occurs when malware repeatedly contacts attacker infrastructure.

Beacon indicators:

- Regular repeated connections
- Same destination over time
- Strange user agents
- Unusual DNS query patterns
- Connections to newly registered domains
- Connections during odd hours

Example:

```text
Host -> C2 server every 60 seconds
```

---

## 7. Actions on Objectives

Actions on objectives are the attacker's final goals.

Examples:

- Data theft
- Ransomware deployment
- System destruction
- Account takeover
- Financial fraud
- Espionage
- Defacement
- Business disruption

Defensive opportunities:

- Data loss prevention
- Network segmentation
- Backup and recovery
- Monitoring sensitive data access
- Privileged access management
- Incident response playbooks

---

## Anti-Forensics

Attackers may attempt to hide activity.

Example technique:

```text
Timestomping
```

Timestomping modifies file timestamps to make malicious files appear older or legitimate.

Defensive opportunities:

- File integrity monitoring
- Timeline analysis
- EDR telemetry
- Centralized logging
- Compare timestamps across sources

---

## Threat Modeling Connection

The Cyber Kill Chain can support threat modeling by helping defenders identify where attacks may happen.

A simple threat modeling process:

1. Identify systems and assets.
2. Assess weaknesses and possible attack paths.
3. Plan mitigations.
4. Implement prevention and detection controls.

Related frameworks:

- STRIDE
- DREAD
- CVSS
- MITRE ATT&CK
- Unified Kill Chain

---

## Detection by Phase

| Phase | Example Detection |
|---|---|
| Reconnaissance | Web scanning, DNS enumeration |
| Weaponization | Malware sample analysis |
| Delivery | Phishing email detection |
| Exploitation | Exploit signatures, suspicious process behavior |
| Installation | New services, scheduled tasks, web shells |
| Command and Control | Beaconing, suspicious DNS |
| Actions on Objectives | Mass file access, exfiltration, encryption activity |

---

## Strengths

The Cyber Kill Chain is useful because it:

- Is easy to understand
- Shows attack progression
- Helps identify defensive opportunities
- Supports incident response timelines
- Encourages layered defense

---

## Limitations

The model has limitations:

- Attacks are not always linear.
- Modern attacks may skip or repeat phases.
- Insider threats may not fit cleanly.
- Cloud and identity attacks may require additional models.
- It is less detailed than MITRE ATT&CK or Unified Kill Chain.

---

## Quick Reference

| Phase | Attacker Goal |
|---|---|
| Reconnaissance | Learn about target |
| Weaponization | Prepare attack |
| Delivery | Send payload |
| Exploitation | Trigger vulnerability |
| Installation | Establish foothold |
| Command and Control | Maintain remote communication |
| Actions on Objectives | Complete mission |

---

## Notes to Remember

- The Cyber Kill Chain describes attack stages.
- Defenders can disrupt attacks at multiple points.
- C2 enables remote attacker control.
- Persistence allows access to survive reboot or remediation attempts.
- Actions on objectives describe the final attacker goal.
- Modern attacks may not follow the chain perfectly.