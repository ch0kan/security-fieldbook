# Hydra

---

## Executive Summary

Hydra is an online password attack tool used to test username and password combinations against network services.

Use this cheatsheet for quick syntax in authorized labs and assessments.

## Basic Syntax

Single username and password list:

```bash
hydra -l USERNAME -P passwords.txt TARGET_IP SERVICE
```

Username list and single password:

```bash
hydra -L users.txt -p PASSWORD TARGET_IP SERVICE
```

Username list and password list:

```bash
hydra -L users.txt -P passwords.txt TARGET_IP SERVICE
```

Specify port:

```bash
hydra -L users.txt -P passwords.txt TARGET_IP -s PORT SERVICE
```

## Common Services

SSH:

```bash
hydra -L users.txt -P passwords.txt ssh://TARGET_IP
```

FTP:

```bash
hydra -L users.txt -P passwords.txt ftp://TARGET_IP
```

RDP:

```bash
hydra -L users.txt -P passwords.txt rdp://TARGET_IP
```

SMB:

```bash
hydra -L users.txt -P passwords.txt smb://TARGET_IP
```

Telnet:

```bash
hydra -L users.txt -P passwords.txt telnet://TARGET_IP
```

VNC:

```bash
hydra -P passwords.txt vnc://TARGET_IP
```

## HTTP Basic Auth

```bash
hydra -L users.txt -P passwords.txt TARGET_IP http-get /protected/
```

HTTPS Basic Auth:

```bash
hydra -L users.txt -P passwords.txt TARGET_IP https-get /protected/
```

## HTTP Forms

Basic POST form:

```bash
hydra -L users.txt -P passwords.txt TARGET_IP http-post-form "/login:username=^USER^&password=^PASS^:Invalid"
```

HTTPS POST form:

```bash
hydra -L users.txt -P passwords.txt TARGET_IP https-post-form "/login:username=^USER^&password=^PASS^:Invalid"
```

With cookie:

```bash
hydra -L users.txt -P passwords.txt TARGET_IP http-post-form "/login:username=^USER^&password=^PASS^:Invalid:H=Cookie: session=VALUE"
```

## Form Syntax

```text
/path:parameters:failure_condition
```

Example:

```text
/login:username=^USER^&password=^PASS^:Invalid password
```

| Part | Meaning |
|---|---|
| `/login` | Login path |
| `username=^USER^` | Username injection point |
| `password=^PASS^` | Password injection point |
| `Invalid password` | Text that means login failed |

## Useful Options

| Option | Purpose |
|---|---|
| `-l` | Single username |
| `-L` | Username list |
| `-p` | Single password |
| `-P` | Password list |
| `-s` | Custom port |
| `-t` | Number of parallel tasks |
| `-f` | Stop after first valid credential |
| `-F` | Stop entire scan after first valid credential |
| `-V` | Verbose attempts |
| `-I` | Ignore restore file warning |
| `-o` | Save output |

## Rate Control

Lower task count:

```bash
hydra -L users.txt -P passwords.txt -t 4 ssh://TARGET_IP
```

Stop after first valid result:

```bash
hydra -L users.txt -P passwords.txt -f ssh://TARGET_IP
```

Save output:

```bash
hydra -L users.txt -P passwords.txt -o hydra-results.txt ssh://TARGET_IP
```

## Username and Password Lists

Single known user:

```bash
hydra -l admin -P passwords.txt ssh://TARGET_IP
```

Password spraying style:

```bash
hydra -L users.txt -p "Password123!" ssh://TARGET_IP
```

Default credentials:

```bash
hydra -C default-creds.txt ssh://TARGET_IP
```

Credential pair file format:

```text
username:password
```

## Practical Workflow

| Step | Action |
|---|---|
| Identify service | Confirm port and protocol |
| Build user list | Use known or discovered usernames |
| Choose password strategy | Wordlist, default creds, or single spray password |
| Control rate | Use lower `-t` to avoid lockouts |
| Run Hydra | Test only authorized targets |
| Validate result | Manually confirm valid credentials |
| Document | Save command and output |

## Notes

- Online password attacks can lock accounts.
- Use low rates when testing real environments.
- Always understand password policy before spraying.
- Confirm valid credentials manually.
- Do not run Hydra outside authorized systems.