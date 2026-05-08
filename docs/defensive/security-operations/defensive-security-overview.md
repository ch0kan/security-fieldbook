# Defensive Security Overview

Defensive security focuses on protecting systems, detecting threats, responding to incidents, and improving security posture over time.

It is not one single role. It includes monitoring, threat intelligence, digital forensics, incident response, and malware analysis.

---

## Major Areas of Defensive Security

| Area | Primary Goal |
|---|---|
| Security Operations Center | Monitor and respond to malicious activity |
| Threat Intelligence | Understand adversaries and their behavior |
| Digital Forensics | Preserve and analyze evidence |
| Incident Response | Contain, remove, and recover from incidents |
| Malware Analysis | Understand malicious code and behavior |

---

# Security Operations Center

A Security Operations Center, or SOC, is a team responsible for monitoring systems and networks for suspicious or malicious activity.

The SOC watches for:

- Vulnerabilities
- Policy violations
- Unauthorized activity
- Network intrusions
- Malware infections
- Suspicious authentication
- Data exfiltration
- Endpoint compromise

---

## SOC Focus Areas

| Focus Area | Description |
|---|---|
| Vulnerabilities | Identify and remediate weaknesses |
| Policy violations | Detect activity that breaks security policy |
| Unauthorized activity | Detect stolen credential use or unusual access |
| Network intrusions | Detect attacks from exploits, phishing, or malicious links |
| Endpoint activity | Detect suspicious processes, files, and registry changes |

---

## SOC Output

The SOC typically produces:

- Alerts
- Tickets
- Investigations
- Incident escalations
- Detection rules
- Reports
- Recommendations for remediation

---

# Threat Intelligence

Threat intelligence supports threat-informed defense.

Its goal is to understand adversaries before, during, and after attacks.

Threat intelligence helps answer:

```text
Who is attacking?
What are they targeting?
What tools do they use?
What infrastructure do they use?
What techniques do they prefer?
How should we defend?
```

---

## Threat Intelligence Process

The general process:

```text
Data collection
  -> Processing
  -> Analysis
  -> Actionable recommendations
```

Sources may include:

- Malware reports
- IP/domain reputation
- Vulnerability reports
- Threat actor profiles
- Open-source intelligence
- Internal incident data
- Dark web monitoring
- Vendor reports

---

## Threat Intelligence Outputs

Useful outputs include:

- Indicators of compromise
- Threat actor profiles
- TTP mappings
- Detection logic
- Blocking recommendations
- Risk assessments
- Incident preparation guidance

---

# Digital Forensics and Incident Response

Digital Forensics and Incident Response is often shortened to DFIR.

It combines:

| Area | Purpose |
|---|---|
| Digital Forensics | Investigate evidence |
| Incident Response | Manage and recover from incidents |

---

## Digital Forensics

Digital forensics focuses on preserving and analyzing evidence.

Evidence sources include:

| Source | Examples |
|---|---|
| File system | Created, deleted, modified files |
| Memory | Running malware, injected code, credentials |
| System logs | Logons, process activity, service changes |
| Network logs | Firewall logs, DNS logs, proxy logs |
| Packet captures | Full traffic content and protocol evidence |

---

## Incident Response

Incident response is the structured process for handling confirmed or suspected incidents.

A common methodology:

```text
Preparation
  -> Detection and Analysis
  -> Containment, Eradication, and Recovery
  -> Post-Incident Activity
```

---

## Incident Response Phases

| Phase | Goal |
|---|---|
| Preparation | Build people, tools, playbooks, and controls |
| Detection and Analysis | Identify and understand the incident |
| Containment | Stop the incident from spreading |
| Eradication | Remove the threat and root cause |
| Recovery | Restore safe operations |
| Post-Incident Activity | Report, learn, and improve |

---

# Malware Analysis

Malware analysis is the process of understanding malicious software.

Common malware types:

- Viruses
- Worms
- Trojans
- Ransomware
- Downloaders
- Backdoors
- Credential stealers
- Web shells
- Rootkits

---

## Static Analysis

Static analysis inspects malware without running it.

Examples:

- Hashing the file
- Extracting strings
- Inspecting headers
- Reviewing imports
- Checking PE sections
- Writing YARA rules

Benefits:

- Safer than execution
- Good for finding indicators
- Useful for quick triage

Limitations:

- Packed or obfuscated malware may hide logic
- Some behavior only appears at runtime

---

## Dynamic Analysis

Dynamic analysis runs malware in a controlled environment and observes behavior.

Examples:

- File creation
- Registry changes
- Network connections
- Process injection
- Persistence creation
- Command and control traffic

Benefits:

- Reveals actual behavior
- Useful for behavior-based detection
- Helps identify runtime indicators

Limitations:

- Requires sandboxing
- Malware may detect analysis environment
- Execution can be risky if containment fails

---

# How These Areas Work Together

Defensive security areas support each other.

Example flow:

```text
Threat intelligence reports a new malware campaign
  -> SOC creates detection rules
  -> SIEM detects suspicious activity
  -> Incident response scopes the incident
  -> Forensics preserves and analyzes evidence
  -> Malware analysis extracts IOCs
  -> Detections and controls are improved
```

---

## Quick Reference

| Area | Main Question |
|---|---|
| SOC | What is happening right now? |
| Threat Intelligence | Who is likely to attack and how? |
| Digital Forensics | What evidence proves what happened? |
| Incident Response | How do we contain and recover? |
| Malware Analysis | What does this malicious code do? |

---

## Notes to Remember

- Defensive security is a collection of disciplines.
- SOC work focuses on monitoring, triage, and escalation.
- Threat intelligence turns adversary knowledge into defensive action.
- DFIR preserves evidence and manages incidents.
- Malware analysis explains what malicious code does and how to detect it.