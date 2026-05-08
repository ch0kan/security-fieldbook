# Incident Response Process

Incident response is the structured process used to prepare for, detect, contain, eradicate, recover from, and learn from security incidents.

The goal is not only to remove the immediate threat, but also to understand the root cause, prevent re-entry, preserve evidence, and improve future defenses.

---

## Incident Response Frameworks

Two common incident response frameworks are:

| Framework | Phases |
|---|---|
| SANS PICERL | Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned |
| NIST | Preparation, Detection & Analysis, Containment/Eradication/Recovery, Post-Incident Activity |

Both cover the same lifecycle, but SANS separates the active response phases more granularly.

---

## SANS PICERL

PICERL stands for:

```text
Preparation
Identification
Containment
Eradication
Recovery
Lessons Learned
```

---

## 1. Preparation

Preparation happens before an incident.

The goal is to build the people, processes, and technology needed to respond effectively.

Key actions:

- Create an Incident Response Plan
- Define roles and responsibilities
- Establish communication channels
- Prepare legal and escalation contacts
- Deploy logging and monitoring
- Maintain forensic tools
- Run tabletop exercises
- Train users against phishing
- Create playbooks for common incidents

Examples:

```text
Phishing playbook
Ransomware playbook
Credential compromise playbook
Data exfiltration playbook
Malware infection playbook
```

---

## 2. Identification

Identification answers:

```text
Is this actually an incident?
What happened?
Who and what is affected?
How severe is it?
```

Common identification sources:

- SIEM alerts
- EDR detections
- User reports
- Firewall logs
- DNS logs
- Email security alerts
- Cloud audit logs
- Threat intelligence
- Network traffic analysis

---

## Detection and Analysis

Detection and analysis is the investigative core of incident response.

The goal is to determine:

- Initial access method
- Root cause
- Scope of compromise
- Impacted users
- Impacted hosts
- Attacker actions
- Persistence mechanisms
- Data accessed or exfiltrated
- Timeline of events

!!! warning
    Fixing only the obvious symptom is not enough. If the root cause is not identified, the attacker may re-enter through the same path.

---

## The Investigation Cycle

Investigations are rarely linear.

A common loop:

```text
Create and use IOCs
  -> Identify new leads
  -> Collect and analyze evidence
  -> Discover new artifacts
  -> Create new IOCs
```

---

## Indicators of Compromise

Indicators of Compromise, or IOCs, are artifacts that suggest malicious activity.

Examples:

| IOC Type | Examples |
|---|---|
| File | Filename, path, hash |
| Network | IP, domain, URL, JA3 |
| Host | Registry key, scheduled task, service |
| Identity | Suspicious user, logon ID |
| Process | Process name, command line |
| Malware | Mutex, embedded string, C2 path |

---

## IOC Formats

Common formats and standards:

| Format | Use |
|---|---|
| YARA | File and memory pattern matching |
| Sigma | Log detection rules |
| STIX | Threat intelligence exchange |
| OpenIOC | Structured IOC documentation |

Example STIX-style file indicator:

```json
{
  "type": "file",
  "hashes": {
    "MD5": "40e609840ef3f7fea94d53998ec9f97f",
    "SHA-256": "3461da3a2ddcced4a00f87dcd7650af48f97998a3ac9ca649d7ef3b7332bd997"
  },
  "name": "osvmhdfl.dll",
  "extensions": {
    "windows-pebinary-ext": {
      "pe_type": "dll"
    }
  }
}
```

---

## Operational Security During Investigation

Never expose high-privilege credentials to a compromised machine.

Attackers may dump cached credentials from memory using credential theft tools.

| Method | Safe? | Reason |
|---|---|---|
| WinRM / Network Logon Type 3 | Yes | Does not normally cache credentials on the remote host |
| PsExec with current token | Usually safer | Uses existing session token |
| PsExec with explicit credentials | No | Can expose typed credentials on the target |

!!! danger
    Avoid logging into a suspected compromised host using domain admin credentials.

---

## Evidence Collection

Once a system is identified as impacted, collect evidence carefully.

Common evidence sources:

- Memory image
- Disk image
- Event logs
- Running processes
- Network connections
- Scheduled tasks
- Services
- Registry autoruns
- Browser artifacts
- File system timeline
- EDR telemetry
- Firewall/proxy/DNS logs

---

## Live Response

Live response captures volatile data while the system is running.

Examples:

- RAM
- Network connections
- Running processes
- Open handles
- Loaded DLLs
- Logged-in users
- Active sessions
- Temporary files

Live response is important because volatile evidence may disappear after shutdown.

---

## Forensic Analysis

Forensic analysis may include:

| Area | Focus |
|---|---|
| Disk forensics | Files, timeline, deleted artifacts |
| Memory forensics | Injected code, credentials, malware |
| Malware analysis | Capabilities, persistence, C2 |
| Log analysis | Timeline and attacker actions |
| Network forensics | Exfiltration, C2, lateral movement |

---

## Chain of Custody

Chain of custody documents evidence handling.

Track:

- Who collected the evidence
- When it was collected
- Where it was collected from
- Hashes of collected files/images
- Storage location
- Who accessed it
- Any transfers or changes

This matters for legal, regulatory, and investigative integrity.

---

## 3. Containment

Containment stops the incident from spreading or causing more damage.

Containment must be coordinated. If attackers notice partial containment, they may change tactics, destroy evidence, or trigger destructive actions.

---

## Short-Term Containment

Short-term containment limits immediate damage while preserving evidence.

Examples:

- Isolate host from network
- Move host to quarantine VLAN
- Disable compromised accounts
- Block malicious IPs/domains
- Sinkhole C2 domains
- Stop known malicious processes
- Preserve forensic image before major changes

---

## Long-Term Containment

Long-term containment allows business operations to continue while reducing attacker access.

Examples:

- Patch exploited vulnerabilities
- Reset credentials
- Apply firewall restrictions
- Remove exposed services
- Disable risky accounts
- Rebuild known compromised systems
- Deploy temporary monitoring rules
- Harden affected systems

---

## 4. Eradication

Eradication removes the attacker and the root cause.

Activities:

- Remove malware
- Remove persistence
- Delete unauthorized accounts
- Remove malicious scheduled tasks
- Remove rogue services
- Patch exploited vulnerabilities
- Close exposed ports
- Revoke stolen tokens
- Rotate compromised credentials
- Rebuild systems when needed

!!! note
    Rebuilding is often safer than cleaning when system integrity is no longer trusted.

---

## 5. Recovery

Recovery restores systems to normal business operation.

Steps:

1. Restore from clean backups.
2. Rebuild or reimage compromised hosts.
3. Patch and harden.
4. Validate functionality with business owners.
5. Return systems to production.
6. Monitor heavily for signs of reinfection.

---

## Enhanced Monitoring During Recovery

Recovered systems should be monitored closely.

Watch for:

- Unusual logons
- Unknown processes
- New services
- New scheduled tasks
- Registry autoruns
- DNS queries to known bad domains
- Connections to suspicious IPs
- Reappearance of old IOCs

---

## 6. Lessons Learned

Lessons Learned is the post-incident improvement phase.

It should happen soon after the incident while details are still fresh.

Goals:

- Identify what worked
- Identify what failed
- Improve playbooks
- Improve detections
- Improve controls
- Update training
- Justify new tools or resources

---

## Final Incident Report

A final report should answer:

| Question | Purpose |
|---|---|
| What happened? | Executive summary |
| When did it happen? | Timeline |
| How did it start? | Root cause |
| What was affected? | Scope |
| What data was impacted? | Business impact |
| What did the team do? | Response actions |
| Is the threat removed? | Recovery confidence |
| How can recurrence be prevented? | Recommendations |

---

## Example Incident Timeline

```text
08:15 - Phishing email delivered
08:23 - User opened attachment
08:24 - PowerShell spawned from Office process
08:25 - Payload downloaded
08:26 - C2 connection established
08:40 - Lateral movement attempt detected
09:05 - Host isolated
09:20 - Account disabled
10:30 - Memory and disk evidence collected
13:00 - Root cause confirmed
```

---

## AI in Incident Response

AI can assist with triage and analysis.

Useful capabilities:

- Alert clustering
- Timeline reconstruction
- Log summarization
- Similarity grouping
- Triage prioritization
- Investigation note drafting

Limitations:

- AI output must be validated
- AI may miss context
- Sensitive data handling must be controlled
- Evidence decisions remain human responsibility

---

## SANS vs NIST Mapping

| SANS Phase | NIST Equivalent |
|---|---|
| Preparation | Preparation |
| Identification | Detection & Analysis |
| Containment | Containment, Eradication & Recovery |
| Eradication | Containment, Eradication & Recovery |
| Recovery | Containment, Eradication & Recovery |
| Lessons Learned | Post-Incident Activity |

---

## Quick Reference

| Phase | Main Goal |
|---|---|
| Preparation | Be ready before incidents |
| Identification | Confirm and scope the incident |
| Containment | Stop the spread |
| Eradication | Remove threat and root cause |
| Recovery | Restore operations safely |
| Lessons Learned | Improve future response |

---

## Notes to Remember

- Incident response is cyclic, not purely linear.
- Identify root cause before declaring victory.
- Do not expose privileged credentials to compromised hosts.
- Preserve volatile evidence before shutdown when possible.
- Coordinate containment to avoid tipping off attackers.
- Recovery requires enhanced monitoring.
- Lessons learned should produce concrete improvements.