# Project 2: Windows Persistence Triage Reporter

---

## The Problem

Windows persistence is often hidden in places that look normal at first glance: services, scheduled tasks, registry Run keys, and local group membership. I wanted a repeatable way to collect these artifacts and turn them into a clear triage report instead of manually copying screenshots from different tools.

## Repository

[View the GitHub repository](https://github.com/ch0kan/windows-persistence-triage-reporter)

## What Was Built/Done

I wrote a PowerShell script that collected common Windows persistence artifacts from a local system and generated a Markdown report.

The script collected:

- Auto-start services and executable paths
- Registry Run and RunOnce keys
- Scheduled tasks and task actions
- Local users
- Local Administrators group membership
- Recent persistence-related Windows Event IDs

The output was a report like:

```text
Finding:
Suspicious scheduled task action

Artifact:
Scheduled Task

Name:
ChromeUpdateCheck

Action:
powershell.exe -NoProfile -WindowStyle Hidden -File C:\Users\Public\update.ps1

Why it was flagged:
- Task launched PowerShell
- Script ran from a user-writable directory
- Task name resembled a browser updater

Suggested triage:
- Review task creation time
- Check related Event ID 4698
- Hash the referenced script
- Review network activity around the last run time
```

I also included a small `sample-report.md` generated from test data so the project could be reviewed without running the script.

## Why It's Not Trivial

The core challenge was deciding what made a persistence artifact suspicious without pretending every autorun entry was malicious.

For example, a scheduled task by itself is normal. A scheduled task that runs PowerShell from `C:\Users\Public\` with hidden window flags is much more suspicious.

I had to think in combinations:

```text
Weak signal:
A scheduled task exists.

Stronger signal:
A scheduled task launches PowerShell.

High-value signal:
A scheduled task launches PowerShell from a user-writable path with hidden execution flags.
```

I also had to normalize messy Windows output into a readable report. Scheduled task actions, service paths, and registry values are not formatted consistently, so I wrote parsing logic that extracted the fields I actually needed for triage.

## Skills Demonstrated

- PowerShell artifact collection
- Windows persistence mechanism analysis
- Scheduled task and service path inspection
- Registry autorun triage
- Windows Event Log querying with Get-WinEvent
- Markdown report generation from structured evidence

## The Deliverable

I created a GitHub repository with:

```text
windows-persistence-triage-reporter/
├── Invoke-PersistenceTriage.ps1
├── sample-report.md
├── config/
│   └── suspicious-paths.json
├── samples/
│   └── sample-events.json
├── docs/
│   └── detection-notes.md
├── README.md
└── screenshots/
```

The most important deliverable was `sample-report.md`. I made the portfolio page lead with the sample report so a reviewer could immediately understand what the tool did without reading the whole README.

The portfolio page included:

```text
- The generated sample report as the main evidence
- A screenshot of the report output
- A table of persistence locations checked
- Examples of flagged artifacts
- Explanation of the scoring logic
- A section on false positives and limitations
```

## Write-Up Angle

I framed the write-up around this idea:

> “Persistence triage is not about finding one magic registry key. It is about collecting small clues from multiple Windows locations and deciding which combinations deserve attention.”

The narrative walks through how I approached the project:

- I started by collecting obvious persistence locations.
- I realized raw output was too noisy to be useful.
- I grouped artifacts by persistence type.
- I added reasons for each flag instead of only assigning a score.
- I tested benign-looking entries against suspicious combinations.
- I made the generated sample report the center of the portfolio page.
- I documented why the script was a triage helper, not a malware detector.

I also explained what I intentionally scoped out. I did not include WMI event subscriptions in this version because they are a classic persistence mechanism that deserve their own dedicated analysis rather than being added as a shallow checkbox. That made the scope more honest and showed that I understood the difference between “covered well” and “mentioned briefly.”

The strongest section of the write-up explains how I handled false positives:

> My first version flagged every PowerShell-based scheduled task. That was too broad because administrators can legitimately automate work with PowerShell. I changed the logic so PowerShell became more suspicious when combined with user-writable paths, hidden execution, encoded commands, or unusual task names.

I also explained where the suggested triage steps came from: I derived them from standard incident response workflows for persistence review, including validating creation time, checking related event logs, hashing referenced files, and correlating with nearby network activity.

## Honest Scope

This script did not prove that an artifact was malicious. It helped me collect and prioritize Windows persistence evidence.

Limitations:

```text
- It did not inspect every possible Windows persistence mechanism.
- It intentionally scoped out WMI event subscriptions for a future focused project.
- It did not perform memory analysis.
- It did not replace EDR or forensic tooling.
- It depended on local permissions and available logs.
- It could flag legitimate administrative automation as suspicious.
```