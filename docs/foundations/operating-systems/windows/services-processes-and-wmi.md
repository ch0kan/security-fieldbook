# Windows Services, Processes, and WMI

Windows systems rely on services, processes, and management interfaces to operate. These components are essential for administration, but they are also commonly abused by attackers for persistence, privilege escalation, discovery, and lateral movement.

## Processes

A process is a running instance of a program.

Each process contains:

- Process ID
- Virtual address space
- Threads
- Handles
- Access token
- Loaded DLLs

Processes are central to both normal system activity and malware behavior.

## Critical Windows Processes

| Process | Purpose |
|---|---|
| smss.exe | Session Manager Subsystem |
| csrss.exe | Client Server Runtime Process |
| wininit.exe | Starts critical background services |
| services.exe | Service Control Manager |
| lsass.exe | Handles authentication and security policy |
| winlogon.exe | Handles interactive logon |
| svchost.exe | Hosts Windows services implemented as DLLs |

## LSASS

`lsass.exe` is one of the most important security processes on Windows.

It is responsible for:

- Verifying logons
- Enforcing security policy
- Creating access tokens
- Handling password changes
- Storing credential material in memory for single sign-on

Because LSASS may contain credential material, it is a major target for tools such as Mimikatz and credential dumping malware.

## Services

Windows services are background programs managed by the Service Control Manager.

Services often start automatically at boot and may run with high privileges.

Common service accounts include:

- LocalSystem
- LocalService
- NetworkService
- Managed Service Accounts
- Domain service accounts

# Windows Services, Processes, and WMI

Windows systems rely on services, processes, and management interfaces to operate. These components are essential for administration, but they are also commonly abused by attackers for persistence, privilege escalation, discovery, and lateral movement.

## Processes

A process is a running instance of a program.

Each process contains:

- Process ID
- Virtual address space
- Threads
- Handles
- Access token
- Loaded DLLs

Processes are central to both normal system activity and malware behavior.

## Critical Windows Processes

| Process | Purpose |
|---|---|
| smss.exe | Session Manager Subsystem |
| csrss.exe | Client Server Runtime Process |
| wininit.exe | Starts critical background services |
| services.exe | Service Control Manager |
| lsass.exe | Handles authentication and security policy |
| winlogon.exe | Handles interactive logon |
| svchost.exe | Hosts Windows services implemented as DLLs |

## LSASS

`lsass.exe` is one of the most important security processes on Windows.

It is responsible for:

- Verifying logons
- Enforcing security policy
- Creating access tokens
- Handling password changes
- Storing credential material in memory for single sign-on

Because LSASS may contain credential material, it is a major target for tools such as Mimikatz and credential dumping malware.

## Services

Windows services are background programs managed by the Service Control Manager.

Services often start automatically at boot and may run with high privileges.

Common service accounts include:

- LocalSystem
- LocalService
- NetworkService
- Managed Service Accounts
- Domain service accounts

## Managing Services

### GUI

Open the Services management console:

```powershell
services.msc
```

### Command Line

Query services with `sc.exe`:

```powershell
sc.exe query
sc.exe qc <service_name>
```

### PowerShell

List and filter services:

```powershell
Get-Service
Get-Service | Where-Object { $_.Status -eq "Running" }
```