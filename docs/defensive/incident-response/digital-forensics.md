# Digital Forensics

Digital forensics is the process of preserving, collecting, examining, analyzing, and reporting digital evidence.

The goal is to reconstruct events in a reliable, repeatable, and legally defensible way.

---

## What Digital Forensics Is

Digital forensics helps answer:

- What happened?
- When did it happen?
- Who was involved?
- What systems were affected?
- What data was accessed or changed?
- How did the attacker gain access?
- What evidence supports the conclusion?

---

## NIST Digital Forensics Framework

The NIST model uses four major phases:

```text
Collection
  -> Examination
  -> Analysis
  -> Reporting
```

---

## 1. Collection

Collection identifies and preserves potential evidence.

Examples:

- Laptops
- Desktops
- Servers
- Mobile phones
- USB drives
- Cloud snapshots
- Logs
- Memory images
- Network captures

Key principle:

```text
Preserve the original evidence and avoid modification.
```

---

## Collection Goals

During collection:

- Identify evidence sources
- Document the scene
- Photograph or record physical context when needed
- Acquire evidence using approved tools
- Preserve original media
- Start chain of custody
- Hash evidence when possible

---

## 2. Examination

Examination filters and prepares raw evidence.

Raw evidence is often large and noisy.

Examples:

- Extract files from disk images
- Filter logs by time window
- Parse browser history
- Recover deleted files
- Extract registry hives
- Identify relevant user profiles
- Convert artifacts into readable formats

---

## 3. Analysis

Analysis interprets the examined evidence.

Analysts correlate artifacts to build a timeline and answer investigative questions.

Common analysis tasks:

- Build event timeline
- Identify initial access
- Trace user activity
- Identify malware execution
- Confirm data access
- Confirm persistence
- Identify lateral movement
- Determine scope

---

## 4. Reporting

Reporting communicates findings clearly.

A good forensic report includes:

- Executive summary
- Scope
- Evidence collected
- Methods used
- Timeline
- Findings
- Impact
- Limitations
- Recommendations
- Technical appendices

Reports should be understandable to both technical and non-technical audiences.

---

## Specialized Forensics Types

| Type | Focus | Key Evidence |
|---|---|---|
| Computer Forensics | Desktops and laptops | Files, logs, registry, deleted data |
| Mobile Forensics | Phones and tablets | SMS, calls, GPS, app data |
| Network Forensics | Network traffic | PCAP, NetFlow, firewall logs |
| Cloud Forensics | Cloud infrastructure | API logs, snapshots, IAM events |
| Database Forensics | Databases | Queries, records, transaction logs |
| Email Forensics | Email systems | Headers, attachments, routing data |
| Memory Forensics | RAM | Processes, injected code, credentials |
| Malware Forensics | Malicious files | Capabilities, IOCs, persistence |

---

## Evidence Acquisition Principles

Evidence acquisition must be controlled and documented.

Important principles:

- Get proper authorization
- Minimize changes to original evidence
- Use forensic copies when possible
- Hash evidence before and after acquisition
- Document every action
- Store evidence securely

---

## Proper Authorization

Before collecting evidence, ensure authorization exists.

Authorization may come from:

- Corporate policy
- Incident response authority
- Legal counsel
- Law enforcement warrant
- Written consent
- Contractual authority

Without authorization, evidence may be inadmissible or violate privacy laws.

---

## Chain of Custody

Chain of custody is the paper trail for evidence.

It should record:

- Evidence identifier
- Description
- Source system/device
- Collector name
- Collection time
- Hash values
- Storage location
- Transfers
- Access history
- Reason for access

Example chain-of-custody fields:

```text
Evidence ID:
Description:
Collected by:
Date/time collected:
Location collected:
Hash:
Storage location:
Transferred to:
Reason:
Signature:
```

---

## Write Blockers

Write blockers prevent accidental modification of evidence.

They allow read access while blocking write commands.

Types:

| Type | Description |
|---|---|
| Hardware write blocker | Physical device between drive and forensic workstation |
| Software write blocker | Software-controlled write prevention |

Use cases:

- Disk imaging
- USB evidence collection
- Suspect drive acquisition

---

## Evidence Integrity

Hashing helps prove evidence integrity.

Common hashes:

```text
MD5
SHA1
SHA256
```

Preferred:

```text
SHA256
```

Example:

```bash
sha256sum disk_image.E01
```

Windows:

```powershell
Get-FileHash -Algorithm SHA256 .\evidence.E01
```

---

## Common Evidence Sources

| Source | Examples |
|---|---|
| Windows logs | Security, System, Application, Sysmon |
| File system | Timestamps, deleted files, user directories |
| Registry | Autoruns, USB history, user activity |
| Browser | History, downloads, cookies |
| Memory | Processes, network connections, injected code |
| Network | PCAP, DNS, proxy, firewall |
| Email | Headers, links, attachments |
| Cloud | IAM, object access, audit logs |
| EDR | Process tree, alerts, telemetry |

---

## Volatile Evidence

Volatile evidence disappears when a system shuts down.

Examples:

- RAM
- Running processes
- Network connections
- Logged-in users
- Clipboard contents
- Temporary files
- Encryption keys
- Process command lines

Collect volatile evidence before powering off if safe and authorized.

---

## Order of Volatility

A common collection priority:

```text
CPU/register/cache
RAM
Network connections
Running processes
Temporary files
Disk
Remote logs
Backups
```

The more volatile the evidence, the sooner it should be collected.

---

## Disk Forensics

Disk forensics examines storage media.

Common findings:

- Deleted files
- File timestamps
- Malware drops
- User documents
- Prefetch files
- Shortcut files
- Browser downloads
- Recycle Bin artifacts
- Persistence files

---

## Memory Forensics

Memory forensics examines RAM.

Useful for finding:

- Running malware
- Injected code
- Process hollowing
- Network connections
- Decrypted strings
- Credentials in memory
- Encryption keys
- Hidden processes

Common tooling:

```text
Volatility
Rekall
YARA
```

---

## Network Forensics

Network forensics examines communication between systems.

Evidence sources:

- PCAP
- NetFlow
- Firewall logs
- DNS logs
- Proxy logs
- Zeek logs
- Suricata alerts

Useful for identifying:

- C2 traffic
- Data exfiltration
- Lateral movement
- Scanning
- DNS tunneling
- Malware downloads

---

## Email Forensics

Email forensics focuses on message metadata and content.

Useful evidence:

- Sender
- Reply-To
- Return-Path
- Received headers
- SPF/DKIM/DMARC results
- Attachment hashes
- URLs
- Message-ID
- Timestamp chain

Common use cases:

- Phishing
- Business email compromise
- Malware delivery
- Credential theft

---

## Cloud Forensics

Cloud forensics focuses on cloud activity and provider logs.

Useful evidence:

- API calls
- IAM changes
- Object access logs
- Login events
- MFA changes
- Snapshot creation
- Security group changes
- External sharing events

Examples:

```text
AWS CloudTrail
Azure Activity Logs
Microsoft 365 Unified Audit Log
Google Workspace audit logs
```

---

## Timeline Analysis

Timeline analysis reconstructs activity over time.

Common timeline sources:

- File timestamps
- Event logs
- Browser history
- EDR events
- Prefetch
- Scheduled tasks
- Registry last write times
- Network logs

Example timeline:

```text
08:12 - User received phishing email
08:15 - Attachment downloaded
08:16 - PowerShell executed
08:17 - Payload written to Temp
08:18 - C2 connection started
08:22 - Archive file created
08:30 - Outbound upload detected
```

---

## Reporting Best Practices

A forensic report should be:

- Clear
- Accurate
- Evidence-based
- Repeatable
- Neutral
- Well-structured
- Understandable to non-technical readers

Avoid unsupported claims.

Use wording like:

```text
The evidence indicates...
The logs show...
No evidence was found to support...
The available data is insufficient to determine...
```

---

## Quick Reference

| Task | Tool / Method |
|---|---|
| Hash file | `sha256sum`, `Get-FileHash` |
| Preserve disk | Forensic image |
| Prevent writes | Write blocker |
| Analyze Windows logs | Event Viewer, Get-WinEvent, Chainsaw |
| Analyze memory | Volatility |
| Analyze network | Wireshark, Zeek, Suricata |
| Scan for malware | YARA |
| Build timeline | Log and artifact correlation |
| Preserve evidence trail | Chain of custody |

---

## Notes to Remember

- Forensics must preserve evidence integrity.
- Do not modify original evidence when avoidable.
- Chain of custody matters.
- Use write blockers for physical drive acquisition.
- Collect volatile evidence early.
- Hash evidence to prove integrity.
- Reporting must separate facts from assumptions.