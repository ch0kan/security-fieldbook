# Lab Architecture

---

## Executive Summary

I built this lab architecture with VMware Workstation, a pfSense firewall VM, and a small number of virtual machines.

My goal was to create a practical environment that is segmented, repeatable, and realistic enough for hands-on cybersecurity practice without becoming too complex to maintain.

## Lab Objective

I wanted the architecture to support:

- Safe attack simulation.
- Windows endpoint investigation.
- Network traffic analysis.
- Web application testing.
- Password security practice.
- Firewall rule testing.
- Evidence collection and documentation.

## Physical Setup

I kept the physical setup simple.

| Component | How I Use It |
|---|---|
| Laptop 1 | Main lab host running VMware Workstation and the core VMs |
| Laptop 2 | Secondary device for notes, research, screenshots, and management access |
| VMware Workstation | Virtualization platform for the lab |
| pfSense VM | Virtual firewall and router |
| Home network | Provides internet access when I allow it |

## High-Level Design

The lab sits behind a pfSense VM.

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

## Why I Chose This Design

I originally considered a larger design with more networks, but I simplified it to make the lab easier to build and troubleshoot.

This three-network layout gives me the separation I need:

| Network | Why I Use It |
|---|---|
| Attack Network | Keeps Kali and testing activity separated from the targets |
| Lab LAN | Holds the systems I investigate or test against |
| Logging Network | Gives me a dedicated place for analysis, captures, and logs |

This design is small enough to run at home but still lets me practice realistic segmentation and firewalling.

## Network Plan

| Network | Subnet | Purpose |
|---|---|---|
| Attack Network | `10.10.20.0/24` | Kali and controlled testing tools |
| Lab LAN | `10.10.30.0/24` | Windows, Linux, and web target systems |
| Logging Network | `10.10.50.0/24` | Analysis tools, logs, and packet captures |

## pfSense VM

I run pfSense as a virtual machine inside VMware Workstation.

pfSense acts as the central router and firewall for the lab. All traffic between the lab networks has to pass through pfSense, which gives me a practical place to control and observe traffic.

## pfSense Interface Plan

| pfSense Interface | Connected To | Example IP |
|---|---|---|
| WAN | VMware NAT or Bridged | DHCP from VMware or home router |
| LAN | Attack Network | `10.10.20.1/24` |
| OPT1 | Lab LAN | `10.10.30.1/24` |
| OPT2 | Logging Network | `10.10.50.1/24` |

For the first version of the lab, I prefer using VMware NAT for the WAN side because it is simpler and safer than exposing pfSense directly to the home network.

## VMware Network Mapping

I map each internal lab network to a separate VMware virtual network.

| VMware Network | Connected Systems |
|---|---|
| VMnet20 | pfSense LAN and Kali VM |
| VMnet30 | pfSense OPT1, Windows Client, Linux Server, Web App VM |
| VMnet50 | pfSense OPT2 and Analysis / Logging VM |

This makes the network boundaries clear and easy to reason about.

## Virtual Machine Placement

| VM | Network | Example IP |
|---|---|---|
| Kali VM | Attack Network | `10.10.20.10` |
| Windows Client VM | Lab LAN | `10.10.30.10` |
| Linux Server VM | Lab LAN | `10.10.30.20` |
| Web App VM | Lab LAN | `10.10.30.30` |
| Analysis / Logging VM | Logging Network | `10.10.50.10` |

## Hostname Plan

I use simple hostnames so the lab is easier to document.

| Hostname | System |
|---|---|
| `kali.lab.local` | Kali VM |
| `win-client.lab.local` | Windows Client VM |
| `linux-server.lab.local` | Linux Server VM |
| `webapp.lab.local` | Web App VM |
| `analysis.lab.local` | Analysis / Logging VM |
| `pfsense.lab.local` | pfSense VM |

## DHCP and DNS

I use pfSense for DHCP and DNS inside the lab.

This lets me centrally manage addressing and name resolution for the lab networks.

| Network | DHCP Range |
|---|---|
| Attack Network | `10.10.20.100 - 10.10.20.199` |
| Lab LAN | `10.10.30.100 - 10.10.30.199` |
| Logging Network | `10.10.50.100 - 10.10.50.199` |

For important systems, I use either static IPs or static DHCP mappings so the addresses stay consistent across lab sessions.

## Firewall Approach

I use pfSense firewall rules to make traffic intentional.

My approach is:

- Allow Kali to reach lab targets only when I am testing.
- Allow Lab LAN systems to send logs or traffic to the Logging Network.
- Allow the Analysis VM to reach selected lab systems for investigation.
- Block lab systems from reaching personal home devices.
- Allow internet access only when needed for updates or tool installation.

## Initial Firewall Policy

| Source | Destination | Action | Why |
|---|---|---|---|
| Attack Network | Lab LAN | Allow limited | Controlled testing |
| Lab LAN | Logging Network | Allow | Log forwarding and analysis |
| Logging Network | Lab LAN | Allow limited | Investigation and management |
| Lab Networks | Internet | Allow optional | Updates only |
| Lab Networks | Home LAN | Block | Protect personal devices |
| Unknown traffic | Any | Block | Default deny mindset |

## Logging and Visibility

I designed the lab so I can observe activity from both the endpoint and the network.

| Source | What I Can Review |
|---|---|
| pfSense firewall logs | Allowed and blocked connections |
| pfSense DNS logs | Name resolution activity |
| Windows Event Logs | Authentication, services, PowerShell, scheduled tasks |
| Linux logs | SSH, sudo, service, and system activity |
| Web server logs | Requests, paths, status codes, and User-Agents |
| Packet captures | Full traffic for selected tests |

## Traffic Capture Points

I can capture traffic from several locations depending on the scenario.

| Capture Point | Why I Use It |
|---|---|
| Kali VM | To see traffic generated by the testing system |
| pfSense interface | To observe traffic crossing network boundaries |
| Windows Client VM | To capture endpoint-specific traffic |
| Linux or Web App VM | To capture service-side activity |
| Analysis VM | To store and review packet captures |

## VM Resource Plan

I kept the VM sizes practical so the lab can run on a normal laptop.

| VM | CPU | RAM | Disk |
|---|---:|---:|---:|
| pfSense | 1-2 cores | 1-2 GB | 20 GB |
| Kali | 2 cores | 4 GB | 40-80 GB |
| Windows Client | 2 cores | 4-6 GB | 60-100 GB |
| Linux Server | 1-2 cores | 2 GB | 20-40 GB |
| Web App VM | 1-2 cores | 2-4 GB | 20-40 GB |
| Analysis VM | 2 cores | 4-8 GB | 60-100 GB |

## Snapshot Strategy

I use snapshots to make the lab repeatable.

| Snapshot | When I Take It |
|---|---|
| Clean Install | After installing the operating system |
| Tools Installed | After updates and tool setup |
| Baseline State | Before simulating activity |
| Pre-Investigation | Before collecting evidence |
| Post-Lab | After completing a scenario |

Snapshots let me repeat the same scenario, reset broken systems, and compare before-and-after behavior.

## Build Process

I built the lab in this order:

| Step | Action |
|---|---|
| 1 | Installed VMware Workstation |
| 2 | Created the VMware virtual networks |
| 3 | Deployed the pfSense VM |
| 4 | Added WAN, LAN, OPT1, and OPT2 adapters to pfSense |
| 5 | Configured pfSense interfaces |
| 6 | Enabled DHCP and DNS for the lab networks |
| 7 | Created initial firewall rules |
| 8 | Deployed Kali |
| 9 | Deployed the Windows Client |
| 10 | Deployed Linux and web target systems |
| 11 | Deployed the Analysis / Logging VM |
| 12 | Took baseline snapshots |
| 13 | Validated routing and firewall behavior |

## Validation

Before running lab scenarios, I validated that:

- pfSense WAN had internet access.
- pfSense internal interfaces had the correct IP addresses.
- Kali received an Attack Network address.
- Windows received a Lab LAN address.
- The Analysis VM received a Logging Network address.
- Kali could reach permitted lab targets.
- Lab systems could not reach personal home devices.
- DNS worked where intended.
- pfSense firewall logs showed allowed and blocked traffic.
- Snapshots existed for the main VMs.

## Skills Demonstrated

This architecture demonstrates:

- VMware virtual networking.
- pfSense firewall deployment.
- Network segmentation.
- DHCP and DNS configuration.
- Firewall rule planning.
- Safe lab isolation.
- Traffic visibility planning.
- Practical home lab design.