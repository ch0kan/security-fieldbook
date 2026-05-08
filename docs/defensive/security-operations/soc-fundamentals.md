# SOC Fundamentals

A Security Operations Center, or SOC, is a centralized team responsible for detecting, analyzing, and responding to cybersecurity threats.

A SOC combines people, process, and technology to maintain visibility across an organization and reduce the impact of attacks.

---

## SOC Mission

The SOC has two core missions:

| Mission | Description |
|---|---|
| Detection | Identify suspicious or malicious activity |
| Response | Contain, escalate, and support remediation |

---

# Detection

Detection is the process of identifying events that may represent risk.

SOC teams look for:

- Vulnerabilities
- Intrusions
- Malware
- Credential abuse
- Unauthorized activity
- Policy violations
- Data exfiltration
- Suspicious network traffic
- Suspicious endpoint behavior

---

## Detection Examples

| Detection Area | Example |
|---|---|
| Vulnerability | Unpatched internet-facing server |
| Unauthorized activity | Login from unusual country |
| Policy violation | User uploads sensitive data to personal storage |
| Intrusion | Web shell activity from external IP |
| Malware | Endpoint runs known malicious file |
| Credential abuse | Password spray against many users |

---

# Response

Response begins after a suspicious event is confirmed or escalated.

SOC response may include:

- Creating a ticket
- Gathering evidence
- Enriching indicators
- Isolating an endpoint
- Blocking an IP or domain
- Escalating to incident response
- Recommending remediation
- Documenting root cause

---

## SOC and Incident Response

The SOC often detects and triages the event first.

If the event is serious, the SOC escalates to incident response.

Example:

```text
SIEM alert detects suspicious PowerShell
  -> Tier 1 triages alert
  -> Tier 2 confirms malicious behavior
  -> Incident Response isolates host
  -> Forensics and malware analysis begin
```

---

# People, Process, and Technology

A mature SOC depends on three pillars.

| Pillar | Description |
|---|---|
| People | Analysts, engineers, responders, managers |
| Process | Playbooks, escalation paths, triage methods |
| Technology | SIEM, EDR, firewalls, logs, threat intel tools |

A SOC with tools but no process becomes noisy.

A SOC with process but no visibility misses attacks.

---

# Alert Triage

Alert triage is the first review of an alert.

The goal is to determine:

- Is it a true positive?
- Is it a false positive?
- How severe is it?
- What is affected?
- Should it be escalated?

---

## The 5 Ws of Triage

| Question | Analyst Objective |
|---|---|
| What? | What happened? |
| When? | When did it happen? |
| Where? | Which host, account, or system was affected? |
| Who? | Which user or process was involved? |
| Why? | Why did this happen or why is it suspicious? |

---

## Triage Example

Alert:

```text
PowerShell downloaded content from a rare external domain.
```

5 Ws:

| Question | Answer |
|---|---|
| What? | PowerShell executed a web download |
| When? | 2026-02-01 14:32 UTC |
| Where? | Workstation `HR-LAPTOP-07` |
| Who? | User `alice` |
| Why? | PowerShell download from rare domain may indicate malware staging |

---

# SOC Technology Stack

SOC tools collect, enrich, correlate, and respond to security events.

Common technologies:

- SIEM
- EDR
- Firewall
- IDS/IPS
- Email security gateway
- Threat intelligence platform
- Vulnerability scanner
- Ticketing system
- SOAR platform

---

## SIEM

A SIEM is the central log collection and correlation platform.

SIEM stands for:

```text
Security Information and Event Management
```

The SIEM collects logs from:

- Endpoints
- Servers
- Firewalls
- Cloud platforms
- Authentication systems
- Applications
- IDS/IPS
- DNS and proxy systems

---

## SIEM Use Cases

| Use Case | Example |
|---|---|
| Correlation | Failed logins followed by successful RDP |
| Alerting | PowerShell encoded command |
| Investigation | Search logs across many systems |
| Reporting | Weekly incident metrics |
| Threat hunting | Query for suspicious behavior |

---

## EDR

Endpoint Detection and Response focuses on workstations and servers.

EDR provides visibility into:

- Process execution
- Parent-child relationships
- File creation
- Network connections
- Registry changes
- Memory behavior
- Suspicious command lines

EDR can also respond by:

- Killing processes
- Quarantining files
- Isolating hosts
- Collecting forensic data

---

## Firewall

A firewall filters traffic based on rules.

In SOC context, firewall logs help answer:

- Which IP connected?
- Which port was used?
- Was traffic allowed or blocked?
- Was there unusual outbound traffic?
- Was a known bad IP contacted?

---

# Reporting and Escalation

Confirmed or high-risk alerts should be documented in tickets.

A good SOC ticket includes:

- Summary
- Severity
- Timeline
- Affected host or user
- Evidence
- Log snippets
- Indicators
- Analysis notes
- Recommended actions
- Escalation status

---

## Escalation Levels

SOC teams are often tiered.

| Tier | Typical Role |
|---|---|
| Tier 1 | Initial triage and alert validation |
| Tier 2 | Deeper investigation and containment support |
| Tier 3 | Threat hunting, advanced analysis, detection engineering |
| Incident Response | Major incident containment and recovery |

---

## Good Triage Habits

- Preserve evidence.
- Do not assume the alert is correct.
- Check surrounding timeline.
- Identify the affected identity and asset.
- Look for related activity before and after.
- Document decisions clearly.
- Escalate when impact or uncertainty is high.

---

## Common Alert Outcomes

| Outcome | Meaning |
|---|---|
| True Positive | Malicious or policy-violating activity occurred |
| False Positive | Benign activity triggered the alert |
| Benign True Positive | Alert is technically correct but expected |
| Needs Tuning | Rule is too noisy or missing context |
| Escalated | Requires deeper investigation or IR |

---

## Quick Reference

| SOC Concept | Meaning |
|---|---|
| Detection | Identifying suspicious activity |
| Response | Taking action to reduce impact |
| Triage | Initial alert investigation |
| SIEM | Central log correlation platform |
| EDR | Endpoint visibility and response |
| Ticket | Formal investigation record |
| Escalation | Hand-off to more advanced support |

---

## Notes to Remember

- A SOC is not just a toolset; it is people, process, and technology.
- Detection without response is incomplete.
- The 5 Ws make triage structured and repeatable.
- Good tickets make investigations easier to continue and review.
- SIEM gives broad visibility; EDR gives deep endpoint visibility.