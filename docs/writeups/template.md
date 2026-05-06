# Writeup Template

!!! warning
    This template is for authorized labs, CTFs, and systems you have explicit permission to test.

---

## Overview

| Field | Details |
|---|---|
| Platform | HTB / THM / CTF / Home Lab |
| Machine / Challenge |  |
| Difficulty |  |
| Operating System | Linux / Windows / Web / Other |
| Category | Web / Network / AD / Crypto / Forensics / Other |
| Date Completed |  |
| Status | Completed / In Progress |

### Summary

Briefly describe what the lab was about and the main attack path.

```text
Example:
Initial access was gained through a vulnerable web application.
Privilege escalation was achieved by abusing a misconfigured scheduled task.
```

---

## Scope

Define what was tested.

```text
Target IP:
Target hostname:
Allowed scope:
Out of scope:
```

Notes:

- Only test systems included in the lab or scope.
- Avoid attacking third-party services.
- Do not include real secrets, tokens, or private data.

---

## Attack Path

High-level chain:

```text
1. Reconnaissance
2. Enumeration
3. Vulnerability discovery
4. Initial access
5. Privilege escalation
6. Proof collection
```

Short version:

```text
Nmap -> Web enumeration -> Vulnerable endpoint -> Shell -> Privilege escalation -> Proof
```

---

## Reconnaissance

### Port Scan

Command:

```bash
nmap -sC -sV -oA scans/initial TARGET
```

Results:

| Port | Service | Version | Notes |
|---:|---|---|---|
|  |  |  |  |

### Full Port Scan

Command:

```bash
nmap -p- --min-rate 1000 -oA scans/all-ports TARGET
```

Results:

```text
Paste important results here.
```

### UDP Scan

Command:

```bash
sudo nmap -sU --top-ports 20 -oA scans/udp TARGET
```

Results:

```text
Paste important results here.
```

---

## Enumeration

### Service Enumeration

Document each interesting service.

#### HTTP / HTTPS

URL:

```text
http://TARGET/
```

Findings:

- 
- 
- 

Useful commands:

```bash
curl -I http://TARGET/
whatweb http://TARGET/
```

#### Directory Discovery

Command:

```bash
gobuster dir -u http://TARGET/ -w /usr/share/wordlists/dirb/common.txt -o scans/gobuster-root.txt
```

Results:

| Path | Status | Notes |
|---|---:|---|
|  |  |  |

#### Virtual Hosts

Command:

```bash
gobuster vhost -u http://TARGET_HOSTNAME/ -w subdomains.txt --append-domain -o scans/vhosts.txt
```

Results:

| Hostname | Status / Length | Notes |
|---|---|---|
|  |  |  |

#### Other Services

Service:

```text
Service name / port:
```

Findings:

- 
- 
- 

---

## Vulnerability Discovery

Describe the weakness found.

### Finding

| Field | Details |
|---|---|
| Vulnerability |  |
| Affected endpoint / service |  |
| Required privileges | None / User / Admin |
| Impact |  |
| Evidence |  |

Explanation:

```text
Explain why this behavior is vulnerable.
```

Relevant request:

```http
GET /example HTTP/1.1
Host: TARGET
```

Relevant response:

```http
HTTP/1.1 200 OK

Example response evidence
```

---

## Exploitation

### Initial Access

Explain how access was obtained.

Command or request:

```bash
# command here
```

Result:

```text
Paste relevant output here.
```

### Shell

Listener:

```bash
rlwrap nc -lvnp 443
```

Payload:

```bash
# payload here
```

Shell context:

```bash
whoami
id
hostname
pwd
```

Output:

```text
Paste output here.
```

!!! note
    Keep payloads and commands sanitized if the writeup will be public.

---

## Privilege Escalation

### Enumeration

Linux:

```bash
whoami
id
uname -a
sudo -l
find / -perm -4000 -type f 2>/dev/null
```

Windows:

```cmd
whoami
whoami /priv
systeminfo
ipconfig
net user
```

Findings:

- 
- 
- 

### Privilege Escalation Path

Explain the misconfiguration or vulnerability.

```text
Example:
The user could run a script as root without a password.
The script used a writable path, allowing command injection.
```

Command:

```bash
# privesc command here
```

Proof of elevated privileges:

```bash
whoami
id
```

Output:

```text
Paste proof here.
```

---

## Proof

### User Proof

Location:

```text
/home/user/user.txt
```

Proof:

```text
Do not publish real flags unless allowed.
```

### Root / Admin Proof

Location:

```text
/root/root.txt
```

Proof:

```text
Do not publish real flags unless allowed.
```

---

## Remediation Notes

Explain how the issue could be fixed.

| Issue | Recommendation |
|---|---|
|  |  |
|  |  |

Examples:

- Enforce server-side authorization checks.
- Sanitize and validate user input.
- Use parameterized queries.
- Disable dangerous file upload types.
- Apply least privilege.
- Remove unnecessary SUID binaries.
- Patch vulnerable software.
- Disable password authentication where possible.

---

## Lessons Learned

Key takeaways:

- 
- 
- 

Questions to review:

- What made the vulnerability possible?
- What evidence confirmed the finding?
- What could have prevented it?
- What would I check faster next time?

---

## Commands Used

```bash
# Recon
nmap -sC -sV -oA scans/initial TARGET

# Web discovery
gobuster dir -u http://TARGET/ -w /usr/share/wordlists/dirb/common.txt

# Listener
rlwrap nc -lvnp 443
```

---

## Timeline

| Time | Action |
|---|---|
|  | Started scan |
|  | Found web service |
|  | Found vulnerability |
|  | Got initial access |
|  | Escalated privileges |
|  | Completed writeup |

---

## References

- 
- 
- 