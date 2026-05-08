# Unified Kill Chain

The Unified Kill Chain is a modern attack lifecycle framework that expands on earlier models like the Cyber Kill Chain and complements MITRE ATT&CK.

It describes a broader set of attack phases, including reconnaissance, initial access, post-exploitation, lateral movement, exfiltration, and impact.

---

## Overview

The Unified Kill Chain was created to represent modern attacks more completely.

Key ideas:

- Attacks are not always linear.
- Attackers may repeat phases.
- Post-exploitation is a major part of real intrusions.
- Motivation and final objectives matter.
- Internal propagation is often as important as initial access.

---

## Why It Matters

The Unified Kill Chain helps defenders:

- Understand full attack paths
- Map attacker behavior
- Build detection coverage
- Identify control gaps
- Improve incident response
- Understand post-exploitation activity
- Connect technical actions to attacker objectives

---

## High-Level Structure

The Unified Kill Chain can be grouped into three broad areas:

| Area | Purpose |
|---|---|
| Initial Foothold | Gain first access |
| Network Propagation | Expand through the environment |
| Action on Objectives | Complete attacker goals |

---

## Initial Foothold

The initial foothold phase focuses on gaining access to a system or network.

Common phases include:

- Reconnaissance
- Weaponization
- Social Engineering
- Exploitation
- Persistence
- Defense Evasion
- Command and Control
- Pivoting

---

## Reconnaissance

Reconnaissance gathers information about the target.

Examples:

- Services
- Employees
- Credentials
- Network topology
- Technologies
- Public documents
- External attack surface

Related MITRE tactic:

```text
Reconnaissance
TA0043
```

Defensive focus:

- Reduce public exposure
- Monitor scanning
- Watch for credential leaks
- Harden externally exposed services

---

## Weaponization

Weaponization prepares attack infrastructure or payloads.

Examples:

- Payload creation
- Phishing infrastructure
- C2 server setup
- Malware hosting
- Exploit preparation

Defensive focus:

- Threat intelligence
- Block malicious infrastructure
- Monitor known indicators
- Detect suspicious payloads

---

## Social Engineering

Social engineering manipulates people into performing unsafe actions.

Examples:

- Phishing
- Credential harvesting
- Malicious attachments
- Impersonation
- Helpdesk abuse

Defensive focus:

- User awareness
- Email security
- MFA
- Reporting processes
- Phishing-resistant authentication

---

## Exploitation

Exploitation abuses a vulnerability or weakness to execute code or gain access.

Examples:

- Uploading a reverse shell
- Exploiting vulnerable services
- Command injection
- Abusing weak credentials
- Exploiting insecure scripts

Defensive focus:

- Patch management
- Secure coding
- Input validation
- EDR
- WAF
- Vulnerability management

---

## Persistence

Persistence allows an attacker to maintain access.

Examples:

- Backdoors
- Scheduled tasks
- Startup items
- Web shells
- C2 integration
- New services
- Registry run keys

Defensive focus:

- Monitor persistence locations
- Detect unusual service creation
- File integrity monitoring
- EDR alerts
- Scheduled task auditing

---

## Defense Evasion

Defense evasion attempts to bypass security controls.

Targets may include:

- Firewalls
- Antivirus
- EDR
- IDS/IPS
- Logging
- Application controls

Examples:

- Obfuscation
- Disabling security tools
- Masquerading
- Timestomping
- Living-off-the-land techniques

Defensive focus:

- Tamper protection
- Centralized logs
- Alert on security tool changes
- Behavioral detections

---

## Command and Control

Command and Control, or C2, enables remote communication between the attacker and compromised system.

Examples:

- HTTPS beaconing
- DNS tunneling
- Encrypted C2 channels
- Cloud-based C2
- Periodic callbacks

Defensive focus:

- DNS monitoring
- Proxy logs
- Egress filtering
- Beaconing detection
- Network anomaly detection

---

## Pivoting

Pivoting uses a compromised system as a bridge to reach other systems.

Example:

```text
Internet -> DMZ Web Server -> Internal Network
```

Defensive focus:

- Network segmentation
- Firewall rules
- Internal monitoring
- Limit outbound access
- Detect unusual internal scanning

---

## Network Propagation

Network propagation describes how attackers expand control after initial access.

Common phases include:

- Pivoting
- Discovery
- Privilege Escalation
- Execution
- Credential Access
- Lateral Movement

---

## Discovery

Discovery helps attackers understand the internal environment.

Attackers may enumerate:

- Users
- Groups
- Permissions
- Hosts
- Shares
- Applications
- Browser data
- Network configuration
- Domain information

Defensive focus:

- Monitor unusual enumeration commands
- Detect abnormal LDAP queries
- Monitor network scans
- Alert on discovery tool usage

---

## Privilege Escalation

Privilege escalation increases permissions on a system or domain.

Targets:

- Local administrator
- Root
- SYSTEM
- Domain administrator
- Cloud administrator
- Privileged service accounts

Defensive focus:

- Least privilege
- Patch privilege escalation flaws
- Monitor privilege changes
- Harden sudo/admin rights
- Protect service accounts

---

## Execution

Execution runs attacker-controlled code.

Examples:

- Remote commands
- Scripts
- Scheduled tasks
- Malware execution
- PowerShell
- WMI
- PsExec-like behavior

Defensive focus:

- Process monitoring
- Script block logging
- Application control
- EDR detections
- Parent-child process analysis

---

## Credential Access

Credential access steals or captures credentials.

Examples:

- Keylogging
- Credential dumping
- Password spraying
- Browser credential theft
- Token theft
- LSASS memory dumping

Defensive focus:

- MFA
- Credential Guard
- LSASS protection
- Password hygiene
- Alert on dumping tools
- Reduce credential exposure

---

## Lateral Movement

Lateral movement allows attackers to move from one system to another.

Examples:

- Remote services
- Stolen credentials
- RDP
- SMB
- WMI
- PsExec
- SSH
- Pass-the-hash

Defensive focus:

- Network segmentation
- Monitor remote logons
- Restrict admin protocols
- Use tiered administration
- Detect unusual authentication patterns

---

## Action on Objectives

Action on objectives is where the attacker completes their strategic goal.

Common phases:

- Collection
- Exfiltration
- Impact

---

## Collection

Collection gathers data of interest.

Targets:

- Documents
- Databases
- Emails
- Browser data
- Screenshots
- Audio/video
- File shares
- Source code

Defensive focus:

- Monitor mass file access
- Data classification
- DLP
- File access auditing
- Limit access to sensitive data

---

## Exfiltration

Exfiltration removes data from the environment.

Examples:

- Upload to external server
- C2 channel transfer
- Cloud storage upload
- DNS tunneling
- Compressed/encrypted archives

Defensive focus:

- Egress monitoring
- DLP
- Proxy logs
- DNS monitoring
- Alert on unusual outbound volume

---

## Impact

Impact disrupts, destroys, or manipulates systems or data.

Examples:

- Ransomware
- Disk wiping
- Defacement
- Denial of service
- Account deletion
- Data destruction
- Operational disruption

Defensive focus:

- Backups
- Recovery testing
- Immutable storage
- Incident response plans
- Privileged access control

---

## Non-Linear Attacks

A major strength of the Unified Kill Chain is that it recognizes attacks are not always linear.

Example:

```text
Initial access
  -> Discovery
  -> Lateral movement
  -> More discovery
  -> Privilege escalation
  -> More lateral movement
  -> Collection
```

Attackers often loop back to earlier phases as they learn more.

---

## Comparison to Other Frameworks

| Framework | Focus |
|---|---|
| Cyber Kill Chain | Simple sequential attack stages |
| MITRE ATT&CK | Detailed tactics and techniques |
| Unified Kill Chain | Broader modern attack lifecycle |

The frameworks can be used together.

---

## Defensive Use

Use the Unified Kill Chain to ask:

- Where could we detect this phase?
- What logs are needed?
- Which controls interrupt this phase?
- Which phases are weakly covered?
- Where would an attacker pivot next?
- What would the attacker need to reach their objective?

---

## Quick Reference

| Area | Example Phases |
|---|---|
| Initial Foothold | Reconnaissance, Exploitation, Persistence, C2 |
| Network Propagation | Discovery, Privilege Escalation, Credential Access, Lateral Movement |
| Action on Objectives | Collection, Exfiltration, Impact |

---

## Notes to Remember

- The Unified Kill Chain is broader than the original Cyber Kill Chain.
- It better represents modern, non-linear attacks.
- It includes post-exploitation and lateral movement.
- It helps defenders map detection and prevention controls.
- It works well alongside MITRE ATT&CK.