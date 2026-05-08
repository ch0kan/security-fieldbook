# Linux Structure and Workflow

---

## Executive Summary

Linux is built around a simple but powerful idea: everything is organized through a single filesystem tree, and most system interaction happens through files, processes, users, permissions, and the command line. Understanding the Linux structure and workflow is essential for administration, troubleshooting, and cybersecurity work because it explains where important files live, how commands are executed, how users interact with the system, and how permissions protect sensitive resources.

---

## Linux Filesystem Hierarchy

Unlike Windows, which uses drive letters such as `C:\` and `D:\`, Linux uses a single root directory represented by `/`.

Everything on the system exists somewhere under `/`, including files, folders, devices, mounted drives, logs, and configuration files.

Common directories include:

| Directory | Purpose |
|---|---|
| `/` | Root of the entire filesystem |
| `/home` | User home directories |
| `/root` | Home directory for the root user |
| `/etc` | System-wide configuration files |
| `/bin` | Essential user commands |
| `/sbin` | Essential system administration commands |
| `/usr` | User applications, libraries, and shared files |
| `/var` | Variable data such as logs, caches, mail, and web content |
| `/tmp` | Temporary files |
| `/opt` | Optional third-party software |
| `/dev` | Device files |
| `/proc` | Virtual filesystem exposing process and kernel information |
| `/mnt` | Temporary mount points |
| `/media` | Auto-mounted removable media |

---

## Important System Paths

Some directories are especially important during administration and security work.

| Path | Why It Matters |
|---|---|
| `/etc/passwd` | Lists local user accounts |
| `/etc/shadow` | Stores password hashes; readable only by root |
| `/etc/group` | Lists local groups |
| `/var/log/auth.log` | Authentication and sudo activity on Debian/Ubuntu |
| `/var/log/syslog` | General system logs |
| `/var/www/html` | Common Apache web root |
| `/home/<user>` | User files, SSH keys, shell history, and personal configs |
| `/tmp` | World-writable temporary directory often abused by attackers |
| `/proc/<pid>` | Runtime information about a specific process |

---

## Users, Root, and Privileges

Linux separates normal users from the administrator account.

The administrator account is called **root** and has full control over the system.

Normal users usually have limited access and cannot modify critical system files unless they are granted elevated permissions.

The most common way to run administrative commands is with `sudo`.

```bash
sudo apt update
```

The `sudo` command allows a permitted user to run a command with elevated privileges without logging in directly as root.

You can check your current identity with:

```bash
whoami
id
```

The `id` command shows your user ID, group ID, and group memberships.

---

## File Ownership and Permissions

Linux permissions control who can read, write, or execute files.

Every file has:

- An owner
- A group
- Permissions for the owner, group, and others

You can view permissions with:

```bash
ls -la
```

Example output:

```text
-rw-r--r-- 1 user user 1200 Jan 10 12:00 notes.txt
```

The permission section breaks down like this:

| Symbol | Meaning |
|---|---|
| `r` | Read |
| `w` | Write |
| `x` | Execute |
| `-` | Permission not granted |

For example:

```text
-rwxr-xr--
```

Means:

| Entity | Permissions |
|---|---|
| Owner | Read, write, execute |
| Group | Read, execute |
| Others | Read only |

---

## Changing Permissions

The `chmod` command changes file permissions.

```bash
chmod +x script.sh
```

This makes a script executable.

Numeric permissions are also common:

```bash
chmod 644 file.txt
chmod 755 script.sh
```

Common permission values:

| Value | Meaning |
|---|---|
| `600` | Owner can read/write; no access for others |
| `644` | Owner can read/write; others can read |
| `700` | Owner has full access; no access for others |
| `755` | Owner has full access; others can read/execute |
| `777` | Everyone has full access; dangerous |

> [!WARNING]
> Avoid using `chmod 777` unless you fully understand the risk. It gives every user full control over the file or directory.

---

## Changing Ownership

The `chown` command changes the owner of a file or directory.

```bash
sudo chown user:user file.txt
```

To change ownership recursively:

```bash
sudo chown -R user:user /path/to/folder
```

This is useful when fixing broken permissions or assigning files to the correct service account.

---

## Working with Files and Directories

Basic navigation and file management are part of everyday Linux workflow.

| Command | Purpose |
|---|---|
| `pwd` | Show current directory |
| `ls` | List files |
| `ls -la` | List all files with details |
| `cd` | Change directory |
| `mkdir` | Create directory |
| `touch` | Create empty file |
| `cp` | Copy files |
| `mv` | Move or rename files |
| `rm` | Delete files |
| `cat` | Print file contents |
| `less` | Read long files page by page |
| `head` | Show first lines of a file |
| `tail` | Show last lines of a file |

Examples:

```bash
pwd
ls -la
cd /var/log
cat auth.log
tail -f /var/log/syslog
```

The `tail -f` command is especially useful for watching logs in real time.

---

## Searching the Filesystem

Linux provides several tools for finding files and searching content.

| Command | Purpose |
|---|---|
| `find` | Search for files by name, type, size, permissions, etc. |
| `locate` | Fast filename search using an indexed database |
| `grep` | Search text inside files |
| `which` | Show the path of a command |
| `whereis` | Locate binary, source, and manual files |

Examples:

```bash
find / -name "*.conf" 2>/dev/null
find / -perm -4000 2>/dev/null
grep -Ri "password" /var/www 2>/dev/null
which bash
```

The `2>/dev/null` part hides permission errors, which keeps output cleaner.

---

## Processes and Jobs

A process is a running program. Every process has a unique Process ID, called a PID.

Useful commands include:

| Command | Purpose |
|---|---|
| `ps aux` | Show running processes |
| `top` | Live process viewer |
| `htop` | Improved interactive process viewer |
| `kill` | Send a signal to a process |
| `jobs` | Show background jobs in the current shell |
| `fg` | Bring a job to the foreground |
| `bg` | Resume a job in the background |

Examples:

```bash
ps aux
top
kill 1234
```

To run a command in the background, add `&`:

```bash
ping 8.8.8.8 &
```

To stop a foreground process, press:

```text
CTRL + C
```

To suspend it, press:

```text
CTRL + Z
```

---

## Services and Systemd

Modern Linux systems commonly use **systemd** to manage services.

A service is a background process, often called a daemon, that provides functionality such as SSH, web hosting, logging, or networking.

The main tool for managing services is `systemctl`.

| Command | Purpose |
|---|---|
| `systemctl status <service>` | Check service status |
| `sudo systemctl start <service>` | Start a service |
| `sudo systemctl stop <service>` | Stop a service |
| `sudo systemctl restart <service>` | Restart a service |
| `sudo systemctl enable <service>` | Start service automatically at boot |
| `sudo systemctl disable <service>` | Prevent service from starting at boot |

Examples:

```bash
systemctl status ssh
sudo systemctl restart apache2
sudo systemctl enable ssh
```

To view service logs:

```bash
journalctl -u ssh --no-pager
```

---

## Environment Variables

Environment variables store values used by the shell and programs.

Common variables include:

| Variable | Purpose |
|---|---|
| `$HOME` | Current user's home directory |
| `$USER` | Current username |
| `$PATH` | Directories searched for executable commands |
| `$SHELL` | Current shell |
| `$PWD` | Current working directory |

Examples:

```bash
echo $HOME
echo $PATH
echo $SHELL
```

The `$PATH` variable is important because it controls where Linux looks when you type a command.

---

## Shell Workflow

The shell is the command-line interface used to interact with Linux.

Common shells include:

| Shell | Description |
|---|---|
| `sh` | Basic Unix shell |
| `bash` | Common default Linux shell |
| `zsh` | Advanced interactive shell |
| `fish` | User-friendly shell with modern features |

A typical Linux workflow involves:

- Navigating directories
- Inspecting files
- Checking permissions
- Running commands
- Redirecting output
- Searching with `grep`
- Managing processes and services
- Reading logs

Example workflow:

```bash
whoami
id
pwd
ls -la
cd /var/log
sudo tail -f auth.log
```

---

## Input, Output, and Redirection

Linux commands use three standard data streams:

| Stream | Number | Purpose |
|---|---:|---|
| STDIN | `0` | Input |
| STDOUT | `1` | Normal output |
| STDERR | `2` | Error output |

Redirection allows you to control where output goes.

| Operator | Purpose |
|---|---|
| `>` | Redirect output and overwrite file |
| `>>` | Redirect output and append to file |
| `2>` | Redirect errors |
| `2>/dev/null` | Hide errors |
| `|` | Pipe output into another command |

Examples:

```bash
ls -la > files.txt
echo "new line" >> notes.txt
find / -name "*.conf" 2>/dev/null
cat auth.log | grep "Failed password"
```

---

## Package Management

Linux distributions use package managers to install, update, and remove software.

Debian and Ubuntu use `apt`.

```bash
sudo apt update
sudo apt install nginx
sudo apt remove nginx
```

Red Hat-based systems use `dnf` or `yum`.

```bash
sudo dnf install nginx
sudo yum install nginx
```

Package managers are important because they handle dependencies and install software from trusted repositories.

---

## Mounting and Devices

Linux treats devices as files, usually located under `/dev`.

Examples:

| Device | Description |
|---|---|
| `/dev/sda` | First disk |
| `/dev/sda1` | First partition on first disk |
| `/dev/null` | Discards output |
| `/dev/random` | Random data source |

Mounting attaches a filesystem to the Linux directory tree.

```bash
sudo mount /dev/sdb1 /mnt/usb
```

Unmount before removing devices:

```bash
sudo umount /mnt/usb
```

---

## Security Perspective

For security work, Linux structure and workflow help with both defense and offense.

Important security checks include:

```bash
id
sudo -l
find / -perm -4000 2>/dev/null
cat /etc/passwd
ls -la ~/.ssh
cat ~/.bash_history
ps aux
ss -tulpen
```

These commands help identify:

- Current privileges
- Sudo permissions
- SUID binaries
- Local users
- SSH keys
- Command history
- Running processes
- Listening ports

---

## Common Mistakes to Avoid

| Mistake | Why It Matters |
|---|---|
| Running everything as root | Increases damage if a command is wrong or compromised |
| Using `chmod 777` | Gives everyone full control |
| Ignoring logs | Misses signs of errors or compromise |
| Leaving secrets in history | Exposes passwords and tokens |
| Misconfiguring services | Can expose sensitive ports or files |
| Not checking permissions | Can create privilege escalation paths |

---

## Quick Reference

| Task | Command |
|---|---|
| Show current user | `whoami` |
| Show user and groups | `id` |
| Show current directory | `pwd` |
| List files | `ls -la` |
| View file | `cat file.txt` |
| Read large file | `less file.txt` |
| Search text | `grep -i "text" file.txt` |
| Find files | `find / -name file.txt 2>/dev/null` |
| Show processes | `ps aux` |
| Show listening ports | `ss -tulpen` |
| Check service | `systemctl status ssh` |
| View logs | `journalctl -xe` |
| Update packages | `sudo apt update` |
| Install package | `sudo apt install <package>` |

---

## Key Takeaways

Linux uses a single filesystem tree starting at `/`.

Most system configuration lives in `/etc`, logs live in `/var/log`, and user data lives in `/home`.

Permissions are central to Linux security.

The shell is the main workflow tool for administration, automation, troubleshooting, and security testing.

Understanding Linux structure makes later topics like privilege escalation, service management, log analysis, and scripting much easier.