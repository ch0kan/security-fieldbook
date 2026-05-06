# Hydra

Hydra is a network logon testing tool used to perform dictionary attacks against authentication services.

It supports many protocols, including SSH, FTP, HTTP forms, SMB, RDP, and more. In authorized labs, Hydra is useful for testing weak credentials and understanding how authentication brute forcing works.

!!! warning
    Use Hydra only against systems you own or have explicit permission to test. Brute forcing can lock accounts, trigger alerts, or disrupt services.

---

## Overview

Hydra automates login attempts by combining usernames and passwords.

Common targets:

- SSH
- FTP
- HTTP login forms
- SMB
- RDP
- Telnet
- VNC
- Databases

Typical use cases:

- Testing password policy strength
- Validating weak default credentials
- Practicing authentication testing in labs
- Checking whether rate limiting or lockout controls exist

---

## Core Concepts

Hydra needs three main things:

| Requirement | Example |
|---|---|
| Target | `10.10.10.10` |
| Protocol or module | `ssh`, `ftp`, `http-post-form` |
| Credentials to try | Username/password or wordlists |

Hydra can test:

```text
One username + many passwords
Many usernames + one password
Many usernames + many passwords
```

---

## Username and Password Flags

| Flag | Meaning |
|---|---|
| `-l` | Single username |
| `-L` | Username list |
| `-p` | Single password |
| `-P` | Password list |

Examples:

```bash
# Single username, password list
hydra -l admin -P passwords.txt 10.10.10.10 ssh

# Username list, single password
hydra -L users.txt -p Password123 10.10.10.10 ssh

# Username list, password list
hydra -L users.txt -P passwords.txt 10.10.10.10 ssh
```

---

## Basic Syntax

```bash
hydra [options] <target> <service>
```

Example:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.10.10 ssh
```

This tests the username `root` against SSH using passwords from `rockyou.txt`.

---

## SSH Brute Force

SSH example with one username:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.10.10 ssh
```

Add threads:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.10.10 -t 4 ssh
```

Verbose output:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.10.10 -t 4 -V ssh
```

!!! note
    High thread counts can trigger rate limits, cause lockouts, or make testing noisy.

---

## FTP Brute Force

FTP example:

```bash
hydra -l admin -P passwords.txt 10.10.10.10 ftp
```

With a username list:

```bash
hydra -L users.txt -P passwords.txt 10.10.10.10 ftp
```

---

## HTTP POST Form Brute Force

Web forms require more specific syntax.

General format:

```bash
hydra -l <user> -P <passlist> <target> http-post-form "<path>:<parameters>:<failure-condition>"
```

Example:

```bash
hydra -l admin -P passwords.txt 10.10.10.10 http-post-form "/login.php:username=^USER^&password=^PASS^:F=incorrect" -V
```

Breakdown:

| Part | Meaning |
|---|---|
| `/login.php` | Path where the login form submits |
| `username=^USER^` | Username field with Hydra placeholder |
| `password=^PASS^` | Password field with Hydra placeholder |
| `F=incorrect` | Failure condition |
| `-V` | Show each attempt |

---

## Placeholders

Hydra uses placeholders inside web form data.

| Placeholder | Replaced With |
|---|---|
| `^USER^` | Current username |
| `^PASS^` | Current password |

Example form body:

```text
username=^USER^&password=^PASS^
```

Hydra replaces those values during each attempt.

---

## Failure and Success Conditions

Hydra needs to know whether a login attempt failed or succeeded.

Common condition types:

| Prefix | Meaning |
|---|---|
| `F=` | Text that indicates failure |
| `S=` | Text that indicates success |

Failure example:

```bash
hydra -l admin -P passwords.txt 10.10.10.10 http-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid password"
```

Success example:

```bash
hydra -l admin -P passwords.txt 10.10.10.10 http-post-form "/login.php:username=^USER^&password=^PASS^:S=Dashboard"
```

Use Burp Suite or browser Developer Tools to inspect the real login request and response.

---

## Finding Form Parameters

To build a Hydra web form command:

1. Open the login page.
2. Submit a test login.
3. Capture the request in Burp Suite or browser DevTools.
4. Identify the form path.
5. Identify parameter names.
6. Identify failed-login response text.
7. Build the Hydra command.

Example HTML:

```html
<input name="username">
<input name="password">
```

Corresponding Hydra body:

```text
username=^USER^&password=^PASS^
```

---

## HTTP GET Form

Some forms send credentials in the query string.

General format:

```bash
hydra -l admin -P passwords.txt 10.10.10.10 http-get-form "/login.php:username=^USER^&password=^PASS^:F=incorrect"
```

GET login forms are less common and insecure because credentials appear in URLs and logs.

---

## Custom Ports

Use `-s` to specify a non-standard port.

```bash
hydra -l admin -P passwords.txt -s 2222 10.10.10.10 ssh
```

Example for HTTP on port 8080:

```bash
hydra -l admin -P passwords.txt -s 8080 10.10.10.10 http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"
```

---

## Threads

Use `-t` to control parallel connections.

```bash
hydra -l admin -P passwords.txt -t 4 10.10.10.10 ssh
```

Guidance:

| Thread Count | Notes |
|---|---|
| Low | Slower, quieter |
| Medium | Balanced for labs |
| High | Faster but noisy and riskier |

---

## Output File

Save results with `-o`.

```bash
hydra -l admin -P passwords.txt 10.10.10.10 ssh -o hydra-results.txt
```

This helps with documentation.

---

## Common Flags

| Flag | Purpose |
|---|---|
| `-l` | Single username |
| `-L` | Username wordlist |
| `-p` | Single password |
| `-P` | Password wordlist |
| `-s` | Custom port |
| `-t` | Threads |
| `-V` | Show every attempt |
| `-vV` | Very verbose output |
| `-f` | Stop after first valid login |
| `-o` | Write output to file |

---

## Wordlists

Common password wordlist:

```text
/usr/share/wordlists/rockyou.txt
```

If compressed:

```bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

Common username ideas:

```text
admin
administrator
root
user
test
guest
support
```

Create a small test file:

```bash
cat > users.txt << EOF
admin
administrator
root
user
test
EOF
```

---

## Practical Workflow

A clean Hydra workflow:

1. Confirm authorization.
2. Identify the service and port.
3. Confirm the service is reachable.
4. Choose a small username/password list first.
5. Set a reasonable thread count.
6. Run Hydra.
7. Save output.
8. Verify any valid credential manually.
9. Stop after proof of impact if appropriate.

Example SSH workflow:

```bash
# Confirm SSH is open
nmap -p 22 -sV 10.10.10.10

# Run a controlled Hydra test
hydra -l admin -P passwords.txt -t 4 -f 10.10.10.10 ssh -o hydra-ssh.txt
```

---

## Defensive Considerations

Hydra helps demonstrate why strong authentication controls matter.

Defenses against brute forcing:

- Strong password policies
- Account lockout
- Rate limiting
- Multi-factor authentication
- Login monitoring
- Alerting on repeated failures
- CAPTCHA for risky flows
- IP throttling
- Blocking default credentials
- Disabling password login where possible

For SSH, stronger defenses include:

```text
SSH key authentication
Disable root login
Fail2ban or similar controls
MFA where possible
```

---

## Troubleshooting

| Problem | Possible Cause |
|---|---|
| No valid passwords found | Wrong username, wrong wordlist, account lockout |
| Connection errors | Service down, wrong port, firewall |
| Too slow | Low threads, slow service, network latency |
| Many false positives | Wrong failure/success string |
| Web form not working | Wrong path, wrong parameter names, missing cookie/token |
| Account locked | Lockout policy triggered |

For web forms, confirm the request in Burp Suite before running Hydra.

---

## Safety Notes

Brute forcing can be disruptive.

Avoid:

- Large wordlists without approval
- High thread counts on fragile services
- Testing accounts that may lock out real users
- Running against production without explicit authorization
- Ignoring rate limits or monitoring warnings

Better approach:

```text
Use small proof-of-concept wordlists.
Demonstrate weak credentials safely.
Stop after confirming impact.
```

---

## Quick Reference

| Goal | Command |
|---|---|
| SSH single user | `hydra -l user -P passwords.txt IP ssh` |
| SSH users list | `hydra -L users.txt -P passwords.txt IP ssh` |
| FTP | `hydra -l user -P passwords.txt IP ftp` |
| HTTP POST form | `hydra -l admin -P passwords.txt IP http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"` |
| Custom port | `-s PORT` |
| Threads | `-t 4` |
| Verbose attempts | `-V` |
| Stop on first hit | `-f` |
| Save output | `-o results.txt` |

---

## Notes to Remember

- Hydra tests authentication by trying credential combinations.
- `-l` and `-L` are for usernames.
- `-p` and `-P` are for passwords.
- Web forms require the correct path, parameters, and failure/success condition.
- Use Burp Suite to capture form requests.
- High thread counts increase noise and risk.
- Always confirm permission before brute forcing.