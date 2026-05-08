# Log Sources and SIEM

Security monitoring depends on collecting useful logs, normalizing them, and searching across them at scale.

A SIEM, or Security Information and Event Management platform, helps analysts centralize logs, detect suspicious activity, investigate alerts, and build reports.

---

## Why Logs Matter

Logs help answer:

```text
Who did what?
From where?
On which system?
At what time?
Was it successful?
What changed afterward?
```

Logs are used for:

- Threat detection
- Incident response
- Forensics
- Compliance
- Troubleshooting
- Asset visibility
- User activity tracking
- Attack timeline reconstruction

---

## Logs vs Network Traffic

Logs and packet captures answer different questions.

| Source | Shows | Limitation |
|---|---|---|
| Logs | Events, metadata, actions | May not show payload content |
| PCAP | Raw communication and payloads | Can be large and hard to retain |
| EDR | Endpoint behavior | Depends on sensor visibility |
| SIEM | Cross-source correlation | Depends on data quality |

Example:

```text
Firewall logs may show DNS queries.
PCAP may reveal DNS TXT records carrying encoded C2 data.
```

---

## Common Security Log Sources

| Source | Useful For |
|---|---|
| Windows Security Logs | Logons, account changes, Kerberos, NTLM |
| Sysmon | Process, network, registry, DNS, file activity |
| PowerShell Logs | Script execution and encoded commands |
| Linux auth logs | SSH logins, sudo activity |
| Web server logs | HTTP requests, user agents, status codes |
| Firewall logs | Allow/deny network traffic |
| DNS logs | Domain lookups, tunneling indicators |
| Proxy logs | Web requests and URL access |
| VPN logs | Remote access activity |
| EDR telemetry | Endpoint behavior and detections |
| Cloud logs | IAM, API calls, storage access |

---

## Log Quality

Useful logs should include:

- Accurate timestamps
- Hostname
- Source IP
- Destination IP
- Username
- Process name
- Command line
- Event type
- Result or status
- Unique IDs when possible

Poor logs make investigations harder.

Common problems:

- Missing timestamps
- Different time zones
- No username
- No source IP
- No command line
- Logs not retained long enough
- Excessive noise
- Inconsistent field names

---

## Centralized Logging

Centralized logging collects logs from many systems into one place.

Benefits:

- Search across many hosts
- Preserve evidence if endpoint is wiped
- Correlate events across systems
- Build detections
- Create dashboards
- Support incident response

Example flow:

```text
Endpoint -> Forwarder -> Indexer / Collector -> SIEM Search Interface
```

---

## What Is Splunk?

Splunk is a data analytics platform often used as a SIEM.

It can:

- Ingest machine data
- Index logs
- Search large datasets
- Build dashboards
- Create alerts
- Support incident investigations
- Correlate events across sources

Splunk searches are written using SPL, or Search Processing Language.

---

## Splunk Architecture

| Component | Function |
|---|---|
| Forwarder | Collects and sends data |
| Indexer | Stores, parses, and indexes data |
| Search Head | User interface for searching and dashboards |
| Deployment Server | Manages forwarder configurations |
| Cluster Manager | Coordinates clustered indexers |

---

## Forwarders

Forwarders collect data and send it to Splunk.

| Type | Description |
|---|---|
| Universal Forwarder | Lightweight agent for raw data collection |
| Heavy Forwarder | Can parse, filter, or route data before forwarding |

Universal Forwarders are common on endpoints and servers.

Heavy Forwarders may be used to:

- Filter data
- Mask sensitive fields
- Route data
- Perform parsing before indexing

---

## Indexers

Indexers receive and store data.

They:

- Parse incoming data
- Create searchable indexes
- Compress data into buckets
- Handle search requests
- Store historical logs

Indexes are used to separate data logically.

Examples:

```text
index=main
index=windows
index=firewall
index=edr
index=cloud
```

---

## Search Heads

Search Heads provide the user interface for analysts.

They are used to:

- Run searches
- Build dashboards
- Create alerts
- Manage reports
- Visualize results
- Coordinate searches across indexers

---

## Data Discovery in Splunk

Before searching deeply, identify what data exists.

Useful SPL:

```spl
| eventcount summarize=false index=*
```

List sourcetypes:

```spl
| metadata type=sourcetypes
```

Summarize fields:

```spl
index=* 
| fieldsummary
```

Useful questions:

- Which indexes exist?
- Which sourcetypes exist?
- Which hosts are sending logs?
- When was the last event received?
- Which fields are available?

---

## Sourcetypes

A sourcetype identifies the format of data.

Examples:

```text
WinEventLog:Security
WinEventLog:Sysmon
XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
linux_secure
access_combined
pan:traffic
```

Sourcetypes help Splunk parse data consistently.

---

## Data Models

Data models organize fields into structured objects.

They help:

- Normalize data
- Build dashboards
- Use pivots
- Support accelerated searches
- Map different vendors into a common structure

Example:

```text
Authentication data model
Network traffic data model
Endpoint process data model
```

---

## SIEM Detection Pipeline

A basic detection pipeline:

```text
Log Source
  -> Collection
  -> Parsing
  -> Normalization
  -> Enrichment
  -> Detection Rule
  -> Alert
  -> Triage
  -> Investigation
  -> Response
```

---

## Enrichment

Enrichment adds context to events.

Examples:

| Enrichment | Value |
|---|---|
| Asset inventory | Identifies critical systems |
| User directory | Maps users to departments |
| Threat intelligence | Flags known bad IPs/domains |
| GeoIP | Adds geographic context |
| Lookup tables | Maps hashes, hosts, or users |
| Vulnerability data | Shows known exposure |

Example:

```text
Process hash -> malware lookup -> known suspicious hash
```

---

## Detection Approaches

There are two broad detection approaches.

| Approach | Description | Strength | Weakness |
|---|---|---|---|
| Known TTP detection | Search for known attacker behavior | High fidelity | May miss new behavior |
| Anomaly detection | Search for unusual behavior | Can find unknowns | More false positives |

Best practice:

```text
Use both known-behavior detections and anomaly-based detections.
```

---

## Alert Quality

Good alerts should include:

- What happened
- Why it matters
- Affected user
- Affected host
- Source and destination
- Time range
- Evidence fields
- Suggested triage steps
- Possible false positives
- Severity

Bad alerts are vague and hard to investigate.

---

## Common SIEM Use Cases

| Use Case | Data Needed |
|---|---|
| Brute force detection | Authentication logs |
| Password spraying | Failed logons across many users |
| Malware execution | Process creation and EDR logs |
| C2 beaconing | Network, DNS, proxy, EDR |
| Lateral movement | Authentication, SMB, process logs |
| Data exfiltration | Proxy, firewall, DNS, cloud logs |
| Privilege escalation | Account, group, sudo/admin logs |
| Persistence | Services, tasks, registry, startup logs |

---

## Practical Analyst Workflow

1. Identify the alert.
2. Confirm source log and timestamp.
3. Identify user and host.
4. Search around the event.
5. Check related process/network activity.
6. Check authentication history.
7. Look for lateral movement.
8. Validate true positive or false positive.
9. Document evidence.
10. Escalate or close.

---

## Quick Reference

| Concept | Meaning |
|---|---|
| SIEM | Central platform for log search and detection |
| Forwarder | Agent that sends logs |
| Indexer | Stores searchable data |
| Search Head | Analyst search interface |
| Sourcetype | Data format/category |
| SPL | Splunk search language |
| Data model | Normalized data structure |
| Enrichment | Adds context to events |

---

## Notes to Remember

- Log quality determines detection quality.
- Centralized logging preserves evidence.
- SIEM alerts need context to be useful.
- Splunk uses forwarders, indexers, and search heads.
- Sourcetypes identify log formats.
- Detection should combine known TTPs and anomaly hunting.
- Always understand the data source behind an alert.