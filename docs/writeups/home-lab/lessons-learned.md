# Lessons Learned

---

## Executive Summary

This page documents the main lessons I learned while building and using my home lab.

The biggest takeaway is that a useful cybersecurity lab does not need to be large or overly complex. A small, segmented environment with pfSense, VMware Workstation, and a few focused virtual machines is enough to practice realistic offensive, defensive, and investigative workflows.

## What I Built

I built a small virtual lab with:

| Component | Purpose |
|---|---|
| VMware Workstation | Hosted the lab systems |
| pfSense VM | Routed and filtered traffic between lab networks |
| Kali VM | Generated controlled test activity |
| Windows Client VM | Practiced endpoint triage and persistence investigation |
| Linux Server VM | Practiced service and network testing |
| Web App VM | Practiced web attack simulation |
| Analysis / Logging VM | Reviewed logs, captures, and evidence |

## Architecture Lessons

I originally considered a more complex design, but I simplified it to three internal networks:

| Network | Purpose |
|---|---|
| Attack Network | Testing system and offensive tooling |
| Lab LAN | Windows, Linux, and web targets |
| Logging Network | Analysis, packet capture, and evidence review |

This was a better design because it was easier to build, easier to troubleshoot, and still realistic enough to demonstrate segmentation.

## pfSense Lessons

Using pfSense as a VM gave me practical experience with:

- Interface assignment.
- DHCP and DNS configuration.
- Firewall rules.
- Network segmentation.
- Allowed and blocked traffic review.
- Log validation.
- Packet capture from a network boundary.

The most useful part was forcing traffic between lab networks to pass through pfSense. This gave me a clear place to control and observe traffic.

## VMware Lessons

VMware networking was one of the most important parts of the lab.

I learned that clear VMnet mapping prevents confusion:

| VMware Network | Purpose |
|---|---|
| VMnet20 | Attack Network |
| VMnet30 | Lab LAN |
| VMnet50 | Logging Network |

Keeping the network layout simple made troubleshooting much easier.

## Windows Triage Lessons

The Windows live triage lab reinforced that process names alone are not enough.

The most useful evidence came from:

- Process command lines.
- Parent-child process relationships.
- Executable paths.
- Active network connections.
- DNS cache entries.
- File hashes.
- pfSense validation.

The biggest lesson was that endpoint evidence becomes much stronger when it is correlated with network evidence.

## Persistence Investigation Lessons

The persistence investigation lab helped me understand how many persistence mechanisms are visible through built-in Windows tools.

The most useful areas to review were:

| Area | Why It Was Useful |
|---|---|
| Services | Showed auto-starting executables and service paths |
| Registry Run keys | Showed logon/startup autoruns |
| Local users | Helped identify unauthorized accounts |
| Administrators group | Helped identify privilege persistence |
| Scheduled tasks | Revealed timed or event-based execution |
| Event logs | Helped support the timeline |
| Baseline comparison | Made suspicious changes easier to find |

I learned that persistence investigation should focus on both the mechanism and the payload.

## Network Investigation Lessons

The network traffic lab reinforced the importance of correlation.

The most effective workflow was:

| Step | Action |
|---|---|
| 1 | Identify endpoint connection |
| 2 | Map connection to process |
| 3 | Check DNS cache or DNS logs |
| 4 | Validate traffic in pfSense |
| 5 | Review packet capture if needed |
| 6 | Check server-side logs |
| 7 | Document timeline and findings |

pfSense logs were useful, but they were much more valuable when combined with endpoint and application logs.

## Password Security Lessons

The password security lab reinforced that password length and predictability matter more than simple complexity.

The key lessons were:

- Short passwords are weak even with symbols.
- Predictable patterns are easy to test.
- Unsalted hashes are weaker against large-scale cracking.
- Long passphrases are stronger than short complex passwords.
- MFA reduces the impact of password compromise.
- Defenders should focus on preventing credential theft, not only improving password rules.

## Web Testing Lessons

The web attack simulation lab helped me connect application testing with defensive evidence.

The strongest documentation came from correlating:

| Evidence Source | Value |
|---|---|
| Burp request | Showed exact input sent |
| Burp response | Showed application behavior |
| Web access log | Confirmed server-side request |
| Web error log | Captured application errors |
| pfSense log | Confirmed network path |
| Packet capture | Preserved traffic details when needed |

The main lesson was that a finding is stronger when I can show the request, response, server-side evidence, and network evidence together.

## Documentation Lessons

Writing the lab as a first-person portfolio project made it more useful.

The structure that worked best was:

| Section | Purpose |
|---|---|
| Objective | What I wanted to practice |
| Environment | What systems I used |
| Scenario | What I simulated |
| Actions Taken | What I did |
| Evidence Collected | What artifacts I gathered |
| Findings | What I observed |
| Detection Opportunities | What could be monitored |
| Lessons Learned | What I improved |
| Skills Demonstrated | What practical ability the lab shows |

This format keeps the write-ups practical and evidence-driven.

## What Worked Well

The strongest parts of the lab were:

- Simple network segmentation.
- pfSense as the central control point.
- VMware snapshots.
- Windows triage with PowerShell.
- Correlating endpoint and network logs.
- Using local web targets for safe testing.
- Keeping test data separate from personal data.
- Writing each lab as a practical investigation.

## What I Would Improve Next

Future improvements:

- Add centralized Windows log forwarding.
- Add Sysmon to the Windows Client VM.
- Add a lightweight SIEM or log platform.
- Create repeatable attack simulation scripts.
- Add a vulnerable Active Directory environment.
- Build a dedicated malware-analysis-only network if needed.
- Create detection rules from observed behavior.
- Add screenshots and packet captures to each write-up.

## Skills Demonstrated

This home lab demonstrates practical skills in:

- Lab architecture design.
- VMware networking.
- pfSense firewall administration.
- Network segmentation.
- Windows endpoint triage.
- Persistence investigation.
- Network traffic analysis.
- Password security testing.
- Web application testing.
- Log correlation.
- Evidence collection.
- Detection thinking.
- Technical documentation.

## Final Reflection

The most important lesson from this lab is that practical cybersecurity skills come from repetition.

By building a small environment, simulating activity, collecting evidence, and writing down what I observed, I created a workflow that I can reuse and improve over time.

This lab gives me a safe place to practice both sides of security: how suspicious activity appears, and how to investigate and detect it.