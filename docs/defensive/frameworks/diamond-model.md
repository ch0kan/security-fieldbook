# Diamond Model

The Diamond Model of Intrusion Analysis is a framework for analyzing cyber intrusion events.

It describes each intrusion event using four core features: adversary, infrastructure, capability, and victim.

---

## Overview

The Diamond Model helps analysts understand relationships in an intrusion.

The four core features are:

```text
Adversary
Infrastructure
Capability
Victim
```

These form a diamond because each feature relates to the others.

Example:

```text
Adversary uses Infrastructure to deliver Capability against Victim
```

---

## The Four Core Features

| Feature | Meaning |
|---|---|
| Adversary | The attacker or threat actor |
| Infrastructure | Systems and resources used by the adversary |
| Capability | Tools, techniques, and methods used |
| Victim | The target of the attack |

---

## Adversary

The adversary is the actor responsible for the intrusion.

They may be:

- Individual attacker
- Criminal group
- Nation-state actor
- Insider threat
- Hacktivist group
- Ransomware affiliate

The adversary is often difficult to identify early in an investigation.

---

## Adversary Operator vs Customer

The Diamond Model separates two adversary roles.

| Role | Meaning |
|---|---|
| Adversary Operator | Person or group conducting the intrusion |
| Adversary Customer | Entity that benefits from the intrusion |

These may be the same or different.

Example:

```text
A contractor performs the intrusion.
A separate organization benefits from the stolen data.
```

This distinction helps with attribution and intent analysis.

---

## Victim

The victim is the target of the adversary activity.

Victims can include:

- Organizations
- People
- Email addresses
- IP addresses
- Domains
- Hosts
- Cloud accounts
- Social media accounts

There is always a victim in an intrusion event.

---

## Victim Personae vs Victim Assets

The Diamond Model separates victims into two categories.

| Category | Meaning |
|---|---|
| Victim Personae | People or organizations being targeted |
| Victim Assets | Systems, accounts, hosts, and infrastructure being targeted |

Example:

```text
Victim Personae: Finance department employees
Victim Assets: Email accounts, laptops, file shares
```

This helps analysts understand both who and what is being attacked.

---

## Capability

Capability describes the tools, techniques, and procedures used by the adversary.

Capabilities can include:

- Malware
- Phishing kits
- Exploits
- Web shells
- Credential theft tools
- Ransomware
- Manual commands
- Social engineering methods

Capability is closely related to TTPs.

---

## Capability Capacity

Capability capacity describes what a capability can exploit or accomplish.

Example:

```text
A phishing kit can harvest credentials.
A remote exploit can execute code on a vulnerable service.
A credential dumper can extract hashes from memory.
```

---

## Adversary Arsenal

The adversary arsenal is the full set of capabilities available to an adversary.

Example:

```text
Phishing templates
Credential harvesting site
C2 framework
Malware loader
Privilege escalation exploit
Data exfiltration script
```

A more complete arsenal usually gives an adversary more options.

---

## Infrastructure

Infrastructure includes the systems and resources used to deliver capabilities or maintain control.

Examples:

- IP addresses
- Domains
- C2 servers
- Email accounts
- Staging servers
- Malware hosting
- Compromised websites
- Cloud resources
- Malicious USB devices

Infrastructure helps connect intrusion events.

---

## Type 1 and Type 2 Infrastructure

| Type | Description |
|---|---|
| Type 1 Infrastructure | Directly controlled or owned by the adversary |
| Type 2 Infrastructure | Controlled by an intermediary or compromised third party |

Example Type 1:

```text
Attacker-owned VPS running a C2 server
```

Example Type 2:

```text
Compromised website used to host malware
```

Type 2 infrastructure can make attribution harder.

---

## Service Providers

Service providers can support adversary infrastructure.

Examples:

- ISPs
- Domain registrars
- Hosting providers
- Webmail providers
- Cloud providers
- CDN providers

They may not be malicious, but their services can be abused.

---

## Event Meta-Features

Meta-features add context to an intrusion event.

The six common meta-features are:

| Meta-Feature | Meaning |
|---|---|
| Timestamp | When the event occurred |
| Phase | Attack stage |
| Result | Success, failure, or unknown |
| Direction | Flow of the activity |
| Methodology | General type of intrusion activity |
| Resources | External resources needed |

---

## Timestamp

Timestamp records when an event occurred.

Useful for:

- Timeline building
- Pattern analysis
- Campaign clustering
- Timezone analysis
- Identifying attacker working hours

Example:

```text
2026-05-06 14:35 UTC
```

---

## Phase

Phase maps an event to a stage in the intrusion lifecycle.

Example frameworks:

- Cyber Kill Chain
- Unified Kill Chain
- MITRE ATT&CK tactics

Example:

```text
Phase: Command and Control
```

---

## Result

Result records the outcome.

Possible values:

```text
Success
Failure
Unknown
```

Example:

```text
Phishing email delivered successfully.
Credential harvesting attempt failed.
Data exfiltration unknown.
```

Result may also describe impact on confidentiality, integrity, or availability.

---

## Direction

Direction describes how activity flows.

Examples:

| Direction | Meaning |
|---|---|
| Victim-to-Infrastructure | Victim connects to C2 |
| Infrastructure-to-Victim | Malware delivered to victim |
| Infrastructure-to-Infrastructure | C2 communicates with staging server |
| Adversary-to-Infrastructure | Operator logs into C2 |
| Infrastructure-to-Adversary | Stolen data sent to operator |
| Bidirectional | Communication flows both ways |
| Unknown | Direction unclear |

---

## Methodology

Methodology describes the general type of activity.

Examples:

- Phishing
- Port scan
- DDoS
- Breach
- Malware delivery
- Credential theft
- Data exfiltration
- Watering-hole attack

---

## Resources

Resources are external requirements needed for the intrusion.

Examples:

- Software
- Hardware
- Knowledge
- Credentials
- Money
- Infrastructure
- Access
- Facilities

All intrusions require one or more resources.

---

## Social-Political Component

The social-political component describes adversary motivation and intent.

Common motivations:

| Motivation | Description |
|---|---|
| Financial gain | Profit, fraud, extortion |
| Espionage | Stealing secrets or intelligence |
| Hacktivism | Political or social cause |
| Status | Recognition in a community |
| Disruption | Damaging operations or reputation |

Understanding motivation helps explain why the victim was targeted.

---

## Technology Component

The technology component describes the relationship between capability and infrastructure.

It explains how the adversary operates technically.

Example:

```text
Capability: Exploit kit
Infrastructure: Compromised website
Method: Watering-hole attack
```

In a watering-hole attack, adversaries compromise a legitimate website visited by the target audience and use it to deliver malware or exploits.

---

## Using the Diamond Model

The Diamond Model helps analysts answer:

- Who may be responsible?
- What infrastructure was used?
- What capability was used?
- Who or what was targeted?
- Are multiple events related?
- What campaign might this belong to?
- What can defenders block or monitor?

---

## Example Event

```text
A phishing email sends a user to a credential harvesting page.
```

Diamond mapping:

| Feature | Example |
|---|---|
| Adversary | Unknown phishing actor |
| Infrastructure | Phishing domain and hosting server |
| Capability | Credential harvesting page |
| Victim | Employee email account |

Meta-features:

| Meta-Feature | Example |
|---|---|
| Timestamp | Time email was received |
| Phase | Delivery / Credential Access |
| Result | User submitted credentials |
| Direction | Infrastructure-to-Victim, then Victim-to-Infrastructure |
| Methodology | Phishing |
| Resources | Domain, hosting, phishing kit |

---

## Defensive Value

The Diamond Model helps defenders:

- Correlate events
- Track campaigns
- Understand adversary behavior
- Identify infrastructure relationships
- Improve threat intelligence
- Plan mitigations
- Support attribution analysis
- Build incident timelines

---

## Quick Reference

| Feature | Question |
|---|---|
| Adversary | Who is responsible? |
| Infrastructure | What systems/resources were used? |
| Capability | What tools or techniques were used? |
| Victim | Who or what was targeted? |

---

## Notes to Remember

- The Diamond Model analyzes intrusion events.
- Every event has an adversary, infrastructure, capability, and victim.
- Meta-features add time, phase, result, direction, methodology, and resource context.
- The model helps correlate activity across incidents.
- It is useful for threat intelligence and intrusion analysis.