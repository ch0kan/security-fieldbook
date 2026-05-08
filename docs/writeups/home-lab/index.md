# Home Lab

---

## Executive Summary

I built this home lab to demonstrate practical cybersecurity skills in a controlled environment.

The lab uses VMware Workstation, a pfSense firewall VM, and a small set of attacker, victim, and analysis systems. My goal was to create a realistic environment where I could safely simulate suspicious activity, collect evidence, investigate what happened, and document the results.

This section is written as a practical portfolio-style lab series, not as a generic guide. Each write-up shows what I built, what I tested, what I observed, and what I learned.

## Why I Built This Lab

I wanted a lab that would let me practice both offensive and defensive workflows.

My goals were to:

- Build a segmented virtual network.
- Use pfSense as a virtual firewall and router.
- Practice safe attack simulation inside an isolated environment.
- Investigate Windows endpoint activity with built-in tools.
- Review network traffic and firewall logs.
- Practice persistence detection.
- Test password security concepts.
- Simulate common web attack patterns.
- Document findings in a clear and repeatable way.

## Lab Overview

I kept the architecture intentionally simple so the lab would be realistic to maintain at home.

```text
Home Network / Internet
        |
   VMware NAT / Bridged
        |
    pfSense VM
        |
        +-- Attack Network
        |     +-- Kali VM
        |
        +-- Lab LAN
        |     +-- Windows Client VM
        |     +-- Linux Server VM
        |     +-- Web App VM
        |
        +-- Logging Network
              +-- Analysis / Logging VM
```

## Core Systems

| System | Purpose |
|---|---|
| VMware Workstation | Hosts the virtual machines |
| pfSense VM | Provides routing, firewalling, DHCP, DNS, and segmentation |
| Kali VM | Used for controlled testing and attack simulation |
| Windows Client VM | Used for endpoint triage and Windows investigation |
| Linux Server VM | Used for Linux service and network testing |
| Web App VM | Used as a safe target for web attack simulation |
| Analysis / Logging VM | Used for packet capture, log review, and investigation notes |

## Network Design

I used three internal lab networks behind pfSense.

| Network | Example Subnet | Purpose |
|---|---|---|
| Attack Network | `10.10.20.0/24` | Kali and offensive testing tools |
| Lab LAN | `10.10.30.0/24` | Windows, Linux, and web target systems |
| Logging Network | `10.10.50.0/24` | Analysis tools, logs, and packet captures |

This gives me enough segmentation to practice realistic workflows without overcomplicating the build.

## What I Practiced

This home lab helped me practice:

| Area | Practical Work |
|---|---|
| Network design | Built segmented VMware networks behind pfSense |
| Firewalling | Controlled traffic between attacker, victim, and logging systems |
| Endpoint triage | Investigated Windows processes, command lines, and network connections |
| Persistence review | Checked services, registry Run keys, local users, groups, and scheduled tasks |
| Traffic analysis | Reviewed DNS, firewall logs, packet captures, and suspicious connections |
| Password security | Tested password hash concepts and cracking workflows in a lab setting |
| Web security | Simulated common web attack patterns against a local target |
| Documentation | Wrote findings, evidence, lessons learned, and detection ideas |

## Write-Up Series

This section is organized into the following write-ups:

| Write-Up | What I Demonstrate |
|---|---|
| Lab Architecture | How I built the VMware and pfSense lab environment |
| Windows Live Triage Lab | How I investigated a live Windows endpoint |
| Persistence Investigation Lab | How I reviewed common Windows persistence locations |
| Network Traffic Investigation Lab | How I analyzed suspicious network activity |
| Password Security Lab | How I tested password hashing and cracking concepts |
| Web Attack Simulation Lab | How I safely tested common web attack patterns |
| Lessons Learned | What I improved, what worked, and what I would change next |

## Documentation Format

For each lab, I document:

| Section | Purpose |
|---|---|
| Objective | What I wanted to test or demonstrate |
| Environment | Systems, network, and tools I used |
| Scenario | The activity I simulated or investigated |
| Actions Taken | What I did during the lab |
| Evidence Collected | Commands, logs, screenshots, or artifacts |
| Findings | What I observed and concluded |
| Detection Opportunities | Signals that could be monitored or alerted on |
| Lessons Learned | What I learned from the exercise |
| Skills Demonstrated | The practical skills shown by the lab |

## Tools Used

Examples of tools I used across the lab:

| Tool | Purpose |
|---|---|
| VMware Workstation | Virtual machine management |
| pfSense | Routing, firewalling, DHCP, DNS, and segmentation |
| Kali Linux | Controlled testing and attack simulation |
| Windows | Endpoint triage and investigation |
| Linux | Server and service testing |
| PowerShell | Windows investigation and automation |
| Wireshark | Packet analysis |
| Tcpdump | Command-line packet capture |
| Nmap | Network discovery and validation |
| Burp Suite | Web request inspection |
| Hashcat / John | Password hash testing in controlled scenarios |

## Safety Boundaries

I designed the lab to stay isolated and safe.

I used:

- Private IP ranges.
- VMware virtual networks.
- pfSense firewall rules.
- Local-only targets.
- Test data only.
- VM snapshots before major changes.
- No real credentials.
- No real personal files.
- No testing against third-party systems.

## Final Goal

My goal with this lab is to show practical, first-hand cybersecurity skills.

This includes the ability to:

- Build and segment a lab network.
- Simulate suspicious activity safely.
- Collect endpoint and network evidence.
- Investigate what happened.
- Identify detection opportunities.
- Explain findings clearly.
- Improve the lab over time.