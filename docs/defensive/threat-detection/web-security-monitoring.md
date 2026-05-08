# Web Security Monitoring

Web security monitoring focuses on detecting suspicious activity against web applications and web servers.

This includes web shells, suspicious uploads, command execution through HTTP requests, scanning, brute force activity, and DoS/DDoS patterns.

---

## Web Shells

A web shell is a malicious server-side script uploaded to a web server to give an attacker remote command execution through HTTP.

Common languages:

```text
PHP
ASP
ASPX
JSP
```

Web shells are often used for:

- Initial access
- Persistence
- Reconnaissance
- Command execution
- File browsing
- Data exfiltration
- Lateral movement

---

## How Web Shells Are Deployed

Common deployment vectors:

| Vector | Description |
|---|---|
| File upload vulnerability | Application accepts executable script uploads |
| Misconfiguration | Web server allows dangerous file types |
| CMS compromise | Plugin/theme/editor abuse |
| RCE vulnerability | Attacker writes shell file through command execution |
| Stolen admin credentials | Attacker uploads through admin panel |

High-risk folders:

```text
/uploads/
/images/
/tmp/
/cache/
/wp-content/uploads/
/admin/
```

---

## Web Shell Capabilities

A web shell may allow:

- Command execution
- File upload/download
- Database access
- Credential theft
- Reverse shell staging
- File deletion
- Log tampering
- Internal network pivoting

Common server-side execution functions include:

```text
system
exec
shell_exec
passthru
eval
assert
base64_decode
```

---

## Web Shell Examples in the Wild

| Web Shell | Notes |
|---|---|
| China Chopper | Small server payload with rich client |
| WSO | Feature-rich PHP shell |
| C99 / R57 | Classic PHP shells |
| Exchange web shells | Seen in ProxyLogon-style intrusions |

---

## Web Server Log Analysis

Web server access logs are the first place to look.

Common log fields:

| Field | Use |
|---|---|
| Source IP | Identify requester |
| Timestamp | Build timeline |
| Method | GET, POST, PUT, HEAD |
| URI | Requested path |
| Status code | Success/failure |
| Bytes | Response size |
| Referrer | How request was reached |
| User-Agent | Browser/tool identity |

---

## Suspicious Request Patterns

Watch for:

- Repeated requests to unknown `.php`, `.aspx`, or `.jsp` files
- POST requests to files in upload directories
- Direct access with no referrer
- Command-like query parameters
- Unusual User-Agent strings
- Repeated access from one IP
- Access outside normal hours
- New script files receiving traffic shortly after upload

---

## Suspicious Request Methods

| Method | Normal Use | Suspicious Use |
|---|---|---|
| GET | Retrieve page/resource | Trigger shell via query string |
| POST | Submit forms | Send commands to shell |
| PUT | Update/upload resource | Upload web shell |
| HEAD | Retrieve metadata | Probe if shell exists |

---

## Suspicious Query Strings

Look for parameters and commands like:

```text
cmd=
exec=
command=
shell=
whoami
id
uname
cat /etc/passwd
powershell
cmd.exe
base64
```

Encoded values are also suspicious.

Example:

```text
cmd=d2hvYW1p
```

This may decode to:

```text
whoami
```

---

## Suspicious User-Agents

Examples:

```text
curl
wget
python-requests
python-urllib
Go-http-client
Mozilla/4.0
empty User-Agent
```

Tool-like User-Agents are more suspicious when paired with sensitive paths or command parameters.

---

## System-Level Confirmation

Web logs show requests, but they do not prove command execution.

Correlate with system logs.

Example attack chain:

```text
Apache access log:
POST /uploads/image.php

Linux audit log:
www-data executes /bin/sh

Conclusion:
The uploaded PHP file likely executed system commands.
```

---

## Auditd for Web Shell Detection

Auditd can confirm file creation or command execution.

Useful targets to monitor:

```text
/var/www/
/usr/share/nginx/html/
/var/www/html/uploads/
/etc/passwd
/etc/shadow
/bin/sh
/bin/bash
```

Example searches:

```bash
ausearch -k web_shell
ausearch -f /var/www/html/uploads/
ausearch -x bash
ausearch -x sh
```

---

## File System Hunting

Web shells must usually exist somewhere on disk.

Look for:

- New PHP/ASPX/JSP files in upload directories
- Random file names
- Double extensions
- Recently modified scripts
- Suspicious functions
- Encoded blobs
- Unexpected files owned by web server user

Double-extension examples:

```text
image.jpg.php
avatar.png.jsp
invoice.pdf.aspx
```

---

## Find Recently Modified Web Files

```bash
find /var/www -type f -name "*.php" -newerct "2025-07-01" ! -newerct "2025-08-01"
```

Find scripts in upload directories:

```bash
find /var/www -type f \( -name "*.php" -o -name "*.jsp" -o -name "*.aspx" \)
```

---

## Search for Dangerous Functions

```bash
grep -R "eval(" /var/www
grep -R "system(" /var/www
grep -R "shell_exec" /var/www
grep -R "base64_decode" /var/www
```

These functions are not always malicious, but they deserve review when found in writable web directories.

---

## Network Traffic Analysis

PCAP analysis can reveal:

- Uploaded web shell source code
- Command parameters
- Encoded payloads
- Attacker IP
- File upload flow
- C2 staging
- Data exfiltration through HTTP

Useful Wireshark filters:

| Goal | Filter |
|---|---|
| POST requests | `http.request.method == "POST"` |
| PHP requests | `http.request.uri contains ".php"` |
| User-Agent review | `http.user_agent` |
| Known attacker IP | `ip.addr == <ATTACKER_IP>` |

---

## Web Shell Correlation Strategy

Strong evidence comes from correlation.

| Evidence | Meaning |
|---|---|
| POST to upload endpoint | Possible shell upload |
| New `.php` in uploads | Suspicious file creation |
| POST to that `.php` | Shell interaction |
| `www-data` spawning shell | Command execution confirmed |
| Outbound connection after command | Payload staging or exfiltration |

---

## DoS and DDoS Monitoring

Denial-of-Service attacks attempt to make a service unavailable.

| Type | Description |
|---|---|
| DoS | One source overwhelms target |
| DDoS | Many distributed sources overwhelm target |
| Application-layer DoS | Abuses expensive web endpoints |

---

## DoS/DDoS Indicators in Web Logs

| Indicator | Example |
|---|---|
| High request rate | One IP sends hundreds of requests per second |
| Burst timestamps | Many requests in same second |
| Odd User-Agents | `curl`, `python-requests`, bot strings |
| Geo anomalies | Sudden traffic from unusual regions |
| 5xx spike | Server begins failing |
| Heavy endpoint abuse | `/login`, `/search`, `/checkout` |
| Large query parameters | `limit=999999` |

---

## Expensive Endpoints

Attackers may target endpoints that cost more server resources.

Examples:

```text
/login
/register
/search
/products
/api/*
/cart
/checkout
```

These may trigger:

- Database queries
- Password hashing
- Session handling
- Token validation
- Payment API calls
- Search indexing

---

## Web Log Triage Questions

Ask:

- Which IPs generated the most requests?
- Which endpoints were targeted?
- Did status codes change?
- Are there 5xx spikes?
- Are User-Agents normal?
- Is traffic geographically unusual?
- Is a single endpoint receiving abnormal traffic?
- Did suspicious uploads happen before shell activity?
- Did the web server spawn OS commands?

---

## Example Hunting Ideas

Top source IPs:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

Top requested paths:

```bash
awk '{print $7}' access.log | sort | uniq -c | sort -nr | head
```

Look for shell-like parameters:

```bash
grep -Ei "cmd=|exec=|whoami|/etc/passwd|base64" access.log
```

Look for POSTs to scripts:

```bash
grep '"POST ' access.log | grep -Ei "\.php|\.aspx|\.jsp"
```

---

## Response Actions

For suspected web shell:

- Preserve web logs
- Preserve suspicious files
- Hash suspicious files
- Isolate server if needed
- Remove exposed shell after evidence collection
- Review upload validation
- Review web directory permissions
- Search for other shells
- Rotate credentials
- Patch initial vulnerability
- Monitor for reinfection

For DoS/DDoS:

- Confirm traffic pattern
- Identify top sources and endpoints
- Apply rate limiting
- Block obvious malicious sources
- Use CDN/WAF controls
- Scale if needed
- Preserve logs for investigation
- Review application bottlenecks

---

## Quick Reference

| Goal | Command / Filter |
|---|---|
| Find POST requests in PCAP | `http.request.method == "POST"` |
| Find PHP traffic in PCAP | `http.request.uri contains ".php"` |
| Find suspicious functions | `grep -R "system(" /var/www` |
| Find recent PHP changes | `find /var/www -type f -name "*.php" -newerct <date>` |
| Find suspicious parameters | `grep -Ei "cmd=|exec=|whoami" access.log` |
| Confirm execution | Auditd process execution logs |
| Find DoS top IPs | Count source IPs in access logs |

---

## Notes to Remember

- Web logs show requests, not proof of execution.
- System logs confirm whether the web process spawned commands.
- Web shells often hide in writable upload directories.
- Double extensions are suspicious in web roots.
- DoS investigations rely heavily on request rate, endpoint, and status-code patterns.