# YARA and Sigma

YARA and Sigma are detection rule formats used by defenders to describe malicious patterns and suspicious behavior.

YARA is commonly used for files, malware samples, and memory. Sigma is commonly used for logs and SIEM detections.

---

## Overview

| Tool | Focus | Common Use |
|---|---|---|
| YARA | File and memory pattern matching | Malware detection and hunting |
| Sigma | Log-based detection logic | SIEM and EDR detections |
| Chainsaw | Offline Sigma hunting | Local EVTX investigation |

Simple summary:

```text
YARA = find malicious content
Sigma = find malicious behavior in logs
```

---

## YARA

YARA is a pattern-matching engine used to detect and classify files or memory based on strings, byte patterns, and logical conditions.

Common uses:

- Malware classification
- Threat hunting
- Incident response
- Memory scanning
- File triage
- Email attachment scanning
- Malware family detection
- IOC-based hunting

---

## YARA Rule Structure

Basic structure:

```yara
rule Rule_Name
{
    meta:
        author = "Analyst"
        description = "Example YARA rule"

    strings:
        $s1 = "malicious string"
        $h1 = { 4D 5A }

    condition:
        all of them
}
```

Main sections:

| Section | Purpose |
|---|---|
| `rule` | Names the rule |
| `meta` | Documentation and context |
| `strings` | Text, hex, or regex patterns |
| `condition` | Logic required to match |

---

## Meta Section

The `meta` section does not affect matching. It documents the rule.

Common fields:

```yara
meta:
    author = "Analyst"
    description = "Detects suspicious sample"
    reference = "Internal case"
    date = "2026-05-06"
    hash = "..."
```

Use `meta` to track:

- Author
- Description
- Version
- Case number
- Malware family
- Hashes
- References

---

## Strings Section

YARA strings define what to search for.

Text string:

```yara
$s1 = "cmd.exe"
```

Case-insensitive:

```yara
$s1 = "mimikatz" nocase
```

Full word:

```yara
$s1 = "admin" fullword
```

Wide and ASCII:

```yara
$s1 = "PowerShell" ascii wide
```

Hex bytes:

```yara
$h1 = { 4D 5A 90 00 }
```

Regex:

```yara
$r1 = /https?:\/\/[a-z0-9.-]+\/[a-z0-9]+/ nocase
```

---

## Condition Section

The `condition` controls when the rule matches.

Examples:

```yara
condition:
    any of them
```

```yara
condition:
    all of them
```

```yara
condition:
    2 of ($s*)
```

```yara
condition:
    $s1 and $s2 and filesize < 500KB
```

---

## PE Magic Bytes

Windows PE files start with `MZ`.

YARA condition:

```yara
uint16(0) == 0x5A4D
```

This helps limit a rule to Windows executables.

Example:

```yara
condition:
    uint16(0) == 0x5A4D and any of them
```

---

## Example YARA Rule

```yara
rule Suspicious_Invoke_Mimikatz_PE
{
    meta:
        description = "Detects small PE files containing Invoke-Mimikatz"
        author = "Security Fieldbook"

    strings:
        $s1 = "Invoke-Mimikatz" ascii wide nocase
        $s2 = "sekurlsa::logonpasswords" ascii wide nocase

    condition:
        uint16(0) == 0x5A4D and filesize < 500KB and any of ($s*)
}
```

---

## YARA Modules

YARA modules extend detection capabilities.

Common modules:

| Module | Purpose |
|---|---|
| `pe` | Windows PE structure |
| `math` | Entropy and math functions |
| `dotnet` | .NET metadata |
| `hash` | Hashing support |

Example:

```yara
import "pe"

rule Suspicious_Imphash
{
    condition:
        uint16(0) == 0x5A4D and
        pe.imphash() == "414bbd566b700ea021cfae3ad8f4d9b9"
}
```

---

## Entropy

Entropy measures randomness.

High entropy may indicate:

- Packing
- Compression
- Encryption
- Embedded payloads

Entropy range:

```text
0 to 8
```

Very high entropy:

```text
> 7.5
```

Example:

```yara
import "math"

rule High_Entropy_PE
{
    condition:
        uint16(0) == 0x5A4D and
        math.entropy(0, filesize) > 7.5
}
```

---

## Imphash

An import hash clusters Windows PE files by imported APIs.

Useful when:

- Malware variants change strings
- Family still uses similar imports
- You need resilient clustering

Example:

```yara
import "pe"

rule Detect_By_Imphash
{
    condition:
        pe.imphash() == "414bbd566b700ea021cfae3ad8f4d9b9"
}
```

---

## .NET Detection

.NET assemblies include specific metadata.

Common indicator:

```text
BSJB
```

Example:

```yara
rule DotNet_Assembly_Indicator
{
    strings:
        $dotnet = "BSJB"

    condition:
        uint16(0) == 0x5A4D and $dotnet
}
```

Useful for detecting:

- C# tools
- In-memory assemblies
- Malware written in .NET
- Red-team tools such as Seatbelt/Rubeus-like assemblies

---

## yarGen

`yarGen` helps generate YARA rules automatically from malware samples.

It extracts strings and filters out common goodware strings.

Example:

```bash
python3 yarGen.py -m malware_samples/ -o generated_rules.yar
```

Use generated rules as a starting point. Always review and tune them.

---

## YARA Scanning

Scan a file:

```bash
yara rule.yar suspicious.exe
```

Scan a directory recursively:

```bash
yara -r rule.yar /path/to/files/
```

Show matching strings:

```bash
yara -s rule.yar suspicious.exe
```

Compile rules:

```bash
yarac rules.yar rules.yrc
```

---

## YARA on Memory

YARA can scan process memory or memory dumps.

Scan a process by PID:

```bash
yara64.exe rule.yar 1234
```

Scan all processes with PowerShell:

```powershell
Get-Process | ForEach-Object { yara64.exe C:\Rules\meterpreter.yar $_.id }
```

Use cases:

- Injected shellcode
- Fileless malware
- Unpacked malware in memory
- Process hollowing
- Suspicious .NET payloads

---

## YARA with Volatility

Volatility can scan memory images with YARA and provide process context.

Inline string search:

```bash
vol.py -f memory.raw yarascan -U "malicious-domain.example"
```

Rule file scan:

```bash
vol.py -f memory.raw yarascan -y malware_rule.yar
```

Why Volatility helps:

```text
Standalone YARA gives raw offsets.
Volatility can map matches to processes and PIDs.
```

---

## YARA with ETW

YARA can be paired with ETW event streams using tools such as SilkETW.

Useful ETW providers:

| Provider | Hunting Value |
|---|---|
| Microsoft-Windows-PowerShell | Script activity |
| Microsoft-Windows-DNS-Client | DNS queries |
| Microsoft-Windows-Kernel-Process | Process activity |
| Microsoft-Windows-Kernel-Network | Network activity |

Example use cases:

- Detect PowerShell strings in script block events
- Match known C2 domains in DNS events
- Detect suspicious command patterns in event streams

---

## YARA Rule Quality

Good YARA rules should:

- Use unique strings
- Avoid common strings
- Combine multiple indicators
- Use file type checks
- Include metadata
- Avoid overly broad conditions
- Be tested against goodware and malware
- Be reviewed before production deployment

Weak rule:

```yara
condition:
    $s1
```

Better rule:

```yara
condition:
    uint16(0) == 0x5A4D and filesize < 1MB and 3 of ($s*)
```

---

## Sigma

Sigma is a generic rule format for log-based detections.

It lets analysts write one detection rule and convert it into SIEM-specific queries.

Common targets:

- Splunk
- Elastic
- Sentinel
- QRadar
- PowerShell
- Logpoint

---

## Sigma Rule Structure

Basic structure:

```yaml
title: Suspicious LSASS Access
id: 00000000-0000-0000-0000-000000000000
status: experimental
description: Detects suspicious access to LSASS
logsource:
  product: windows
  category: process_access
detection:
  selection:
    TargetImage|endswith: '\lsass.exe'
    GrantedAccess|endswith:
      - '0x1010'
      - '0x1410'
  condition: selection
level: high
```

---

## Sigma Sections

| Section | Purpose |
|---|---|
| `title` | Detection name |
| `id` | Unique rule ID |
| `status` | Rule maturity |
| `description` | What it detects |
| `references` | Supporting links |
| `tags` | ATT&CK mappings |
| `logsource` | Required log data |
| `detection` | Matching logic |
| `falsepositives` | Expected benign cases |
| `level` | Severity |

---

## Sigma Detection Logic

Simple selection:

```yaml
detection:
  selection:
    EventID: 4625
  condition: selection
```

Multiple values:

```yaml
detection:
  selection:
    Image|endswith:
      - '\cmd.exe'
      - '\powershell.exe'
  condition: selection
```

Exclusion filter:

```yaml
detection:
  selection:
    TargetImage|endswith: '\lsass.exe'
  filter_defender:
    SourceImage|contains: '\Windows Defender\'
  condition: selection and not filter_defender
```

Optional filters:

```yaml
condition: selection and not 1 of filter_optional_*
```

---

## Sigma Field Modifiers

| Modifier | Meaning |
|---|---|
| `contains` | Field contains value |
| `endswith` | Field ends with value |
| `startswith` | Field starts with value |
| `re` | Regex |
| `all` | All values must match |
| `base64` | Base64 encoded form |
| `windash` | Windows dash variants |

Example:

```yaml
Image|endswith: '\powershell.exe'
```

---

## LSASS Access Sigma Example

```yaml
title: Suspicious LSASS Process Access
status: experimental
description: Detects suspicious access to LSASS memory
logsource:
  product: windows
  category: process_access
detection:
  selection:
    TargetImage|endswith: '\lsass.exe'
    GrantedAccess|endswith:
      - '0x1010'
      - '0x1410'
  filter_optional_defender:
    SourceImage|contains: '\Windows Defender\'
  condition: selection and not 1 of filter_optional_*
falsepositives:
  - Security software
  - Backup tools
  - EDR products
level: high
```

---

## Password Spraying Sigma Concept

Password spraying can be detected by aggregation.

Idea:

```text
Count distinct target usernames by source workstation.
Alert when one workstation fails authentication for many users.
```

Relevant event:

```text
4776 - NTLM credential validation
4625 - Failed logon
```

Conceptual Sigma logic:

```yaml
condition: selection | count(TargetUserName) by Workstation > 3
```

---

## Sigma Rule Development Workflow

1. Understand the attack.
2. Identify required telemetry.
3. Execute or simulate in a lab.
4. Review logs.
5. Identify unique fields.
6. Write a basic selection.
7. Add filters for false positives.
8. Convert to SIEM query.
9. Test on known data.
10. Tune before production.

---

## Sigma Conversion

Sigma rules can be converted into SIEM queries using tools such as:

```text
sigmac
pySigma
```

Example concept:

```bash
sigmac -t splunk rule.yml
```

Converted outputs may target:

- Splunk SPL
- Elastic Query DSL
- Microsoft Sentinel KQL
- PowerShell
- QRadar AQL

---

## Chainsaw

Chainsaw is a command-line tool for hunting through Windows Event Logs using Sigma rules.

It is useful for:

- Offline incident response
- Local EVTX analysis
- Rapid triage
- Running Sigma detections without a SIEM
- Searching forensic evidence

---

## Chainsaw Commands

Core commands:

| Command | Purpose |
|---|---|
| `hunt` | Run rules against logs |
| `search` | Keyword or regex search |
| `dump` | Convert artifacts |
| `lint` | Validate rules |

Basic hunt:

```powershell
.\chainsaw_x86_64-pc-windows-msvc.exe hunt .\evtx\ -s .\sigma\ --mapping .\mapping.yml
```

Hunt with JSON output:

```powershell
chainsaw hunt evtx/ -s sigma/ --mapping map.yml --json
```

Search for keyword:

```powershell
chainsaw search mimikatz -i evtx/
```

Search for Event ID:

```powershell
chainsaw search -t 'Event.System.EventID: =4104' evtx/
```

---

## Chainsaw Mapping Files

Mapping files translate generic Sigma field names into actual Windows Event Log fields.

If a Sigma rule is valid but returns no results, check:

- Correct log source
- Correct Event ID
- Correct field mapping
- Correct EVTX file
- Correct timestamp range

Common problem:

```text
Sigma field NewProcessName is not mapped to the actual EVTX field.
```

---

## Brute Force Detection with Chainsaw

Use case:

```text
Multiple failed NTLM logins from a single workstation.
```

Relevant event:

```text
4776
```

Example finding:

```text
5 failed NTLM logins against user NOUSER from workstation FS01
```

---

## Abnormal PowerShell Detection with Chainsaw

Use case:

```text
Very long PowerShell command line suggests Base64 or obfuscation.
```

Relevant event:

```text
4688 - Process Creation
```

Rule idea:

```text
powershell.exe with CommandLine length greater than 1000 characters
```

If detection fails despite visible evidence, inspect the Chainsaw mapping file.

---

## YARA vs Sigma

| Feature | YARA | Sigma |
|---|---|---|
| Detects | File/memory content | Log events |
| Format | `.yar`, `.yara` | `.yml`, `.yaml` |
| Best for | Malware and artifacts | Behavior and telemetry |
| Example target | PE file, memory dump | Windows logs, Sysmon, SIEM |
| Common output | Matched file/process | Alert/search result |

---

## Combined Hunting Workflow

Example incident workflow:

```text
1. Sigma detects suspicious PowerShell.
2. Analyst collects process tree and files.
3. YARA scans dropped files.
4. YARA scans memory for injected payloads.
5. Sigma/Chainsaw hunts EVTX for lateral movement.
6. Findings are documented and converted into detections.
```

---

## Quick Reference

| Goal | Tool / Command |
|---|---|
| Scan file with YARA | `yara rule.yar file.exe` |
| Recursive YARA scan | `yara -r rule.yar directory/` |
| Show matching strings | `yara -s rule.yar file.exe` |
| Compile YARA rules | `yarac rules.yar rules.yrc` |
| Volatility YARA scan | `vol.py -f memory.raw yarascan -y rule.yar` |
| Inline Volatility search | `vol.py -f memory.raw yarascan -U "string"` |
| Sigma conversion | `sigmac -t splunk rule.yml` |
| Chainsaw hunt | `chainsaw hunt evtx/ -s sigma/ --mapping map.yml` |
| Chainsaw keyword search | `chainsaw search mimikatz -i evtx/` |

---

## Notes to Remember

- YARA detects static and memory patterns.
- Sigma detects suspicious log behavior.
- YARA rules need strong strings and careful conditions.
- Sigma rules need correct log sources and field mappings.
- False positive tuning is part of rule development.
- Chainsaw is useful for offline EVTX hunting.
- Use YARA and Sigma together for stronger detection coverage.