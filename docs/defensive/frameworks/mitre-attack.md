# MITRE ATT&CK

MITRE ATT&CK is a knowledge base of adversary tactics, techniques, and procedures based on real-world observations.

It gives security teams a shared language for describing attacker behavior, building detections, mapping incidents, and planning adversary emulation.

---

## Core Concepts

MITRE ATT&CK organizes adversary behavior into three important levels:

| Term | Meaning | Example |
|---|---|---|
| Tactic | The adversary's objective | Credential Access |
| Technique | How the adversary achieves the objective | OS Credential Dumping |
| Procedure | The specific implementation | Using Mimikatz to dump LSASS memory |

Simple breakdown:

```text
Tactic    = Why
Technique = How
Procedure = Exact steps
```

---

## APT

APT stands for Advanced Persistent Threat.

An APT is usually a threat group or organization that conducts long-term cyber operations.

APT characteristics:

- Long-term targeting
- Specific objectives
- Persistence
- Repeated access attempts
- Use of multiple techniques
- May be financially or politically motivated
- May be linked to criminal groups or nation-state activity

!!! note
    “Advanced” does not always mean the attacker uses unique or complex tools. Many advanced groups still use common techniques when they work.

---

## TTPs

TTP stands for Tactics, Techniques, and Procedures.

TTPs describe how adversaries operate.

| Layer | Description |
|---|---|
| Tactic | Goal or objective |
| Technique | Method used to achieve the goal |
| Procedure | Specific implementation used by an actor |

Example:

```text
Tactic: Credential Access
Technique: Brute Force
Procedure: Password spraying against exposed VPN login portal
```

TTPs are useful because they describe behavior, not just individual tools or indicators.

---

## ATT&CK Matrix

The ATT&CK Matrix is a visual representation of adversary behavior.

It organizes:

- Tactics across the top
- Techniques under each tactic
- Sub-techniques under techniques

Example:

```text
Reconnaissance
  -> Active Scanning
    -> Scanning IP Blocks
    -> Vulnerability Scanning
```

The matrix helps analysts understand where an observed behavior fits into an attack lifecycle.

---

## Common Enterprise Tactics

Common MITRE ATT&CK Enterprise tactics include:

| Tactic | Purpose |
|---|---|
| Reconnaissance | Gather information about the target |
| Resource Development | Prepare infrastructure or capabilities |
| Initial Access | Gain entry into the environment |
| Execution | Run malicious code |
| Persistence | Maintain access |
| Privilege Escalation | Gain higher permissions |
| Defense Evasion | Avoid detection |
| Credential Access | Steal credentials |
| Discovery | Learn about the environment |
| Lateral Movement | Move between systems |
| Collection | Gather data of interest |
| Command and Control | Communicate with attacker infrastructure |
| Exfiltration | Steal data |
| Impact | Disrupt, destroy, or manipulate systems/data |

---

## Technique Details

ATT&CK technique pages commonly include:

- Technique name
- Technique ID
- Description
- Platforms affected
- Example procedures
- Associated threat groups
- Associated software
- Detection ideas
- Mitigations
- References

Example technique format:

```text
T1053 - Scheduled Task/Job
T1053.005 - Scheduled Task
```

Technique IDs make communication clearer between teams.

---

## Practical Uses

MITRE ATT&CK is useful for many defensive and offensive security activities.

| Team | How ATT&CK Helps |
|---|---|
| CTI | Map threat actor behavior to TTPs |
| SOC | Add context to alerts |
| Detection Engineering | Map detections to adversary techniques |
| Incident Response | Reconstruct attack timelines |
| Threat Hunting | Hunt for behaviors linked to known techniques |
| Red Team | Build realistic adversary emulation plans |
| Purple Team | Compare attack simulation against detection coverage |

---

## Threat Intelligence

ATT&CK helps turn threat intelligence into actionable defense.

Example workflow:

```text
Threat report mentions scheduled tasks and phishing
    ↓
Map behavior to ATT&CK techniques
    ↓
Identify relevant logs
    ↓
Build SIEM/EDR detections
    ↓
Test detection coverage
```

This helps defenders move from “interesting report” to practical monitoring.

---

## Incident Response Mapping

During incident response, ATT&CK can be used to map attacker behavior over time.

Example:

| Observed Activity | ATT&CK Mapping |
|---|---|
| Phishing email delivered | Initial Access |
| Malicious attachment executed | Execution |
| Scheduled task created | Persistence |
| Credentials dumped | Credential Access |
| Data sent to external server | Exfiltration |

This makes the attack timeline easier to understand and communicate.

---

## Detection Engineering

Detection engineers can use ATT&CK to identify coverage gaps.

Example questions:

- Do we detect suspicious scheduled task creation?
- Do we detect credential dumping behavior?
- Do we monitor PowerShell execution?
- Do we log suspicious parent-child process relationships?
- Do we monitor lateral movement activity?

ATT&CK helps organize these questions by adversary behavior.

---

## MITRE CAR

MITRE Cyber Analytics Repository, or CAR, is a knowledge base of detection analytics based on ATT&CK.

CAR provides:

- Detection descriptions
- Pseudocode
- Tool-specific queries
- ATT&CK mappings
- Unit test ideas

Example use:

```text
ATT&CK Technique -> Detection Analytic -> SIEM Query -> Alert / Hunt
```

CAR can help defenders build detection logic faster.

---

## D3FEND

MITRE D3FEND is a framework focused on defensive techniques and countermeasures.

While ATT&CK describes adversary behavior, D3FEND describes defensive actions.

```text
ATT&CK = What attackers do
D3FEND = What defenders can do
```

D3FEND defensive tactics include:

| Tactic | Purpose |
|---|---|
| Model | Understand systems and relationships |
| Harden | Reduce attack surface |
| Detect | Identify malicious behavior |
| Isolate | Limit attacker movement |
| Deceive | Mislead or trap adversaries |
| Evict | Remove adversary access |
| Restore | Recover systems and operations |

Example D3FEND technique:

```text
Credential Rotation
```

This can counter risks from stolen or reused credentials.

---

## ATT&CK vs D3FEND

| Framework | Focus |
|---|---|
| ATT&CK | Adversary behavior |
| D3FEND | Defensive techniques |
| CAR | Detection analytics |

Used together:

```text
ATT&CK identifies the technique.
CAR helps detect it.
D3FEND helps defend against it.
```

---

## Practical Example

Scenario:

```text
An attacker creates a scheduled task to run malware after reboot.
```

Possible mappings:

| Framework | Mapping |
|---|---|
| ATT&CK | Scheduled Task/Job |
| CAR | Detection analytic for suspicious scheduled task activity |
| D3FEND | Execution prevention, credential rotation, hardening, monitoring |

Defensive data sources may include:

- Windows Event Logs
- Sysmon logs
- EDR process telemetry
- Scheduled task logs
- PowerShell logs

---

## Quick Reference

| Term | Meaning |
|---|---|
| APT | Long-term threat actor/group |
| TTP | Tactics, Techniques, and Procedures |
| Tactic | Attacker objective |
| Technique | Method used |
| Procedure | Specific implementation |
| ATT&CK Matrix | Visual map of tactics and techniques |
| CAR | Detection analytics repository |
| D3FEND | Defensive countermeasure framework |

---

## Notes to Remember

- ATT&CK gives defenders and attackers a shared language.
- Tactics describe goals.
- Techniques describe methods.
- Procedures describe exact implementations.
- ATT&CK helps map incidents and detections.
- CAR helps translate ATT&CK into analytics.
- D3FEND complements ATT&CK with defensive techniques.