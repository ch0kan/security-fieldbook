# Windows Command Line

The Windows Command Prompt, or **CMD**, is a core interface for navigating the file system, managing files, gathering host information, and performing basic system administration.

For security work, CMD is also useful for host reconnaissance, triage, and understanding what an attacker may do using built-in Windows tools.

---

## Overview

CMD can help answer common questions during administration, troubleshooting, or authorized security testing:

- Where am I in the file system?
- What files and directories exist?
- What user am I running as?
- What system am I on?
- What network configuration is present?
- What users, groups, shares, and resources exist?
- Where are sensitive files or configuration artifacts located?
- How can command output be filtered, redirected, or saved?

---

## Warning

Run enumeration commands only on systems you own, administer, or have explicit permission to assess.

Many commands in this page, such as `systeminfo`, `net user`, `net share`, and recursive file searches, may be monitored by defenders because they are commonly used during reconnaissance.

---

## Command Prompt Navigation

Navigating a Windows system through CMD requires understanding drives, paths, directories, and common navigation commands.

### Listing Directory Contents

The `dir` command lists files and subdirectories in the current location.

```cmd
dir
```

Common output fields include:

| Field | Meaning |
|---|---|
| `<DIR>` | The entry is a directory |
| Size | File size in bytes |
| Timestamp | Last modified date and time |

Useful options:

```cmd
:: Show hidden files
dir /ah

:: Recursively list files and directories
dir /s

:: Show all files, including hidden and system files
dir /a
```

### Current Working Directory

The current working directory is the folder your shell is currently operating from.

```cmd
cd
```

Running `cd` by itself prints the current directory.

### Absolute and Relative Paths

| Path Type | Description | Example |
|---|---|---|
| Absolute path | Starts from the drive root and includes the full path | `C:\Windows\System32` |
| Relative path | Starts from the current location | `.\Pictures` |

Examples:

```cmd
:: Move to an absolute path
cd C:\Windows\System32

:: Move to a relative path
cd .\Pictures
```

### Navigation Shortcuts

```cmd
:: Move up one directory
cd ..

:: Move to the root of the current drive
cd \

:: Move up multiple levels
cd ..\..

:: Change to another drive
D:
```

### Visualizing Directory Structure

The `tree` command displays a visual directory hierarchy.

```cmd
:: Show folders only
tree

:: Show folders and files
tree /f
```

Large directory trees can generate a lot of output. Use `Ctrl+C` to stop the command if needed.

---

## File and Directory Management

CMD provides built-in commands for creating, deleting, moving, copying, renaming, and viewing files.

### Creating Directories

```cmd
mkdir reports
md logs
```

`mkdir` and `md` perform the same function.

### Removing Directories

```cmd
:: Remove an empty directory
rmdir reports

:: Remove a directory and all contents
rmdir /s reports

:: Remove without confirmation prompts
rmdir /s /q reports
```

Be careful with `/s` and `/q`. They can recursively delete data without additional prompts.

### Moving and Copying Directories

| Tool | Best Use Case | Notes |
|---|---|---|
| `move` | Relocating files or folders | Moves source to destination |
| `xcopy` | Legacy recursive copying | Mostly replaced by `robocopy` |
| `robocopy` | Reliable file copy and sync | Preserves attributes, timestamps, and ACLs |

Examples:

```cmd
:: Move a folder
move C:\Users\Public\reports C:\Temp\reports

:: Copy a directory tree
xcopy C:\Source C:\Backup /e /i

:: Copy files with robocopy
robocopy C:\Source C:\Backup /e
```

### Mirroring Directories with Robocopy

```cmd
robocopy C:\Source C:\Backup /mir
```

**Warning:** `robocopy /mir` mirrors the source to the destination. Files in the destination that do not exist in the source may be deleted.

From a defender perspective, unexpected use of `robocopy /mir` can indicate large-scale data staging, unauthorized synchronization, or destructive cleanup activity.

---

## Viewing and Creating Files

### Displaying File Contents

```cmd
:: Print a text file to the terminal
type notes.txt

:: View one screen at a time
more notes.txt

:: Compress multiple blank lines while viewing
more /s notes.txt
```

### Writing Output to Files

```cmd
:: Create or overwrite a file
echo test > file.txt

:: Append to a file
echo another line >> file.txt
```

### Creating a File of a Specific Size

```cmd
fsutil file createnew test.bin 1048576
```

This creates a 1 MB file named `test.bin`.

### Renaming Files

```cmd
ren oldname.txt newname.txt
rename oldname.txt newname.txt
```

### Deleting Files by Attribute

The `del` and `erase` commands can target file attributes.

```cmd
:: Delete hidden files
del /a:h hidden.txt

:: Delete read-only files
del /a:r readonly.txt

:: Find hidden files
dir /a:h
```

Hidden files can be normal system artifacts, but they can also be used by malware or unauthorized tools to reduce visibility.

---

## Redirection and Command Chaining

Redirection controls where command input and output go.

### Redirection Operators

| Operator | Purpose |
|---|---|
| `>` | Redirect output to a file and overwrite it |
| `>>` | Append output to a file |
| `<` | Use a file as command input |
| `\|` | Pipe output from one command into another command |

Examples:

```cmd
:: Save output to a file
ipconfig /all > network.txt

:: Append output to an existing file
whoami /groups >> enum.txt

:: Filter output
ipconfig | find "IPv4"
```

### Logical Operators

| Operator | Purpose |
|---|---|
| `&` | Run command A, then command B |
| `&&` | Run command B only if command A succeeds |
| `\|\|` | Run command B only if command A fails |

Examples:

```cmd
:: Run both commands
hostname & whoami

:: Run second command only if first succeeds
mkdir logs && echo created > logs\status.txt

:: Run second command only if first fails
dir C:\Missing || echo directory not found
```

---

## Environment Variables

Environment variables are dynamic values used by Windows and applications. In CMD, they are referenced with percent signs.

```cmd
echo %USERNAME%
echo %USERPROFILE%
echo %TEMP%
```

Environment variables are useful because paths can vary between systems and users.

### Variable Scope

| Scope | Visibility | Persistence | Registry Location |
|---|---|---|---|
| System | All users | Persistent | `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment` |
| User | Current user | Persistent | `HKEY_CURRENT_USER\Environment` |
| Process | Current CMD session | Temporary | Memory only |

### Viewing Variables

```cmd
:: List all variables
set

:: Show variables beginning with PATH
set PATH

:: Print one variable
echo %COMPUTERNAME%
```

### Temporary Variables with `set`

The `set` command changes variables only for the current CMD session.

```cmd
:: Create or update a variable
set TARGET_IP=10.10.10.5

:: Use the variable
echo %TARGET_IP%

:: Remove the variable
set TARGET_IP=
```

### Persistent Variables with `setx`

The `setx` command writes variables persistently.

```cmd
setx TARGET_IP 10.10.10.5
```

Changes made with `setx` appear in new terminal sessions, not the current one.

### Useful Environment Variables

| Variable | Expands To | Security Relevance |
|---|---|---|
| `%PATH%` | Search path for executables | Useful for identifying path hijacking risk |
| `%SYSTEMROOT%` | Usually `C:\Windows` | Core Windows directory |
| `%TEMP%` | Current user's temp directory | Common location for temporary files and dropped tools |
| `%USERPROFILE%` | Current user's profile path | Quick access to Desktop, Documents, Downloads |
| `%PUBLIC%` | `C:\Users\Public` | Shared location accessible by multiple users |
| `%LOGONSERVER%` | Logon server or local machine | Can help identify domain membership |
| `%ProgramFiles%` | 64-bit Program Files path | Useful for installed software enumeration |
| `%ProgramFiles(x86)%` | 32-bit Program Files path | Indicates 64-bit Windows when present |

### Identifying Domain Membership

```cmd
echo %LOGONSERVER%
echo %COMPUTERNAME%
```

If `%LOGONSERVER%` differs from `%COMPUTERNAME%`, the host may be joined to a domain.

Example:

```cmd
:: Domain-joined example
LOGONSERVER=\\DC01
COMPUTERNAME=WS-01
```

---

## Useful Directories for Reconnaissance

Certain Windows directories are commonly reviewed during administration, incident response, and authorized security testing.

| Variable | Physical Path | Why It Matters |
|---|---|---|
| `%SYSTEMROOT%\Temp` | `C:\Windows\Temp` | Temporary system files; may allow broad write access |
| `%PUBLIC%` | `C:\Users\Public` | Shared user-accessible location |
| `%TEMP%` | `C:\Users\<user>\AppData\Local\Temp` | User-specific temporary files |
| `%USERPROFILE%` | `C:\Users\<user>` | User documents, downloads, desktop, and profile data |
| `%ProgramFiles%` | `C:\Program Files` | Installed 64-bit applications |
| `%ProgramFiles(x86)%` | `C:\Program Files (x86)` | Installed 32-bit applications |

These directories are also commonly abused for staging files, hiding tools, or discovering sensitive data.

---

## Gathering System Information

Host enumeration is the process of collecting information about the operating system, network configuration, users, groups, privileges, and shared resources.

### General System Information

```cmd
:: Comprehensive system information
systeminfo

:: Computer name
hostname

:: Windows version
ver
```

`systeminfo` is especially useful because it shows:

- Hostname
- OS name and version
- Build number
- System type
- Domain or workgroup
- Installed hotfixes
- Boot time
- Network adapters

### Network Configuration

```cmd
:: Full TCP/IP configuration
ipconfig /all

:: ARP cache
arp -a
```

`ipconfig /all` can reveal:

- IP addresses
- Subnet masks
- Default gateways
- DNS servers
- DHCP status
- MAC addresses

`arp -a` shows hosts the machine has recently communicated with on the local network.

---

## Identity and Privilege Analysis

### Current User

```cmd
whoami
```

This displays the current user and domain or workgroup context.

### User Privileges

```cmd
whoami /priv
```

This lists assigned privileges such as:

- `SeBackupPrivilege`
- `SeRestorePrivilege`
- `SeDebugPrivilege`
- `SeImpersonatePrivilege`

Some privileges may have security implications if enabled or abusable.

### Group Membership

```cmd
whoami /groups
```

Group membership helps determine local and domain-level access.

---

## Enumerating Users and Groups

```cmd
:: List local users
net user

:: List local groups
net localgroup

:: List local administrators
net localgroup Administrators

:: Inspect a specific user
net user username
```

Useful groups to review include:

| Group | Why It Matters |
|---|---|
| Administrators | Full local administrative control |
| Remote Desktop Users | May log in via RDP |
| Backup Operators | May access sensitive files through backup rights |
| Power Users | Legacy elevated capabilities |
| Event Log Readers | Can read local event logs |

---

## Network Shares and Resources

```cmd
:: List shares hosted by the current machine
net share

:: View reachable systems or resources
net view
```

Shared folders may contain sensitive files, scripts, backups, credentials, or configuration data.

---

## Finding Files

### Locating Files with `where`

The `where` command searches the current directory and directories in the `PATH`.

```cmd
where notepad.exe
```

### Recursive Searches

Use `/r` to search a directory and its subdirectories.

```cmd
:: Find all CSV files under a user's profile
where /r C:\Users\student *.csv

:: Find text files
where /r C:\Users *.txt
```

### Wildcards

```cmd
:: Find files beginning with password
where /r C:\Users password*

:: Find configuration files
where /r C:\ *.config
```

Recursive searches across large directories can be slow and noisy.

---

## Searching File Contents

### Basic String Search with `find`

```cmd
:: Search for an exact string
find "password" notes.txt

:: Ignore case
find /i "password" notes.txt

:: Show line numbers
find /n "password" notes.txt

:: Show lines that do not contain the string
find /v "password" notes.txt
```

### Advanced String Search with `findstr`

`findstr` is similar to `grep` on Linux and supports more advanced pattern matching.

```cmd
:: Search text files recursively, ignoring case
findstr /s /i "password" *.txt

:: Search for multiple terms
findstr /s /i "password secret token key" *.txt

:: Show matching line numbers
findstr /s /n /i "admin" *.ini
```

Security analysts often use `findstr` to locate:

- Hardcoded passwords
- API keys
- Connection strings
- Internal hostnames
- Sensitive configuration values
- Suspicious script content

---

## Comparing Files

File comparison is useful for identifying tampering, configuration drift, or script changes.

| Command | Comparison Type | Best For |
|---|---|---|
| `comp` | Byte-by-byte | Checking whether two files are identical |
| `fc` | Line-by-line | Reviewing text differences |

Examples:

```cmd
:: Byte-level comparison
comp file1.txt file2.txt

:: Text comparison with line numbers
fc /n file1.txt file2.txt
```

---

## Sorting Data

The `sort` command organizes text output.

```cmd
:: Sort a file alphabetically
sort users.txt

:: Save sorted output
sort users.txt /o sorted_users.txt

:: Remove duplicate lines
sort users.txt /unique
```

Sorting is useful when working with:

- User lists
- Host lists
- IP addresses
- Log extracts
- Password wordlists
- Discovered filenames

---

## Practical Enumeration Workflow

A basic CMD-based host review might look like this:

```cmd
mkdir enum

hostname > enum\system.txt
ver >> enum\system.txt
systeminfo >> enum\system.txt

whoami > enum\identity.txt
whoami /priv >> enum\identity.txt
whoami /groups >> enum\identity.txt

ipconfig /all > enum\network.txt
arp -a >> enum\network.txt

net user > enum\users.txt
net localgroup > enum\groups.txt
net localgroup Administrators > enum\admins.txt

net share > enum\shares.txt
```

---

## Defender Notes

Watch for unusual or high-volume use of commands such as:

```cmd
systeminfo
whoami /priv
whoami /groups
net user
net localgroup Administrators
net share
net view
where /r C:\Users *
findstr /s /i "password" *.txt
robocopy /mir
del /a:h
```

These commands are legitimate administrative tools, but they can also indicate reconnaissance, staging, collection, or cleanup activity.

---

## Quick Reference

| Goal | Command |
|---|---|
| Show current directory | `cd` |
| List files | `dir` |
| Show hidden files | `dir /a:h` |
| Move up one directory | `cd ..` |
| Move to drive root | `cd \` |
| Show directory tree | `tree /f` |
| Create directory | `mkdir name` |
| Remove directory | `rmdir /s name` |
| View file | `type file.txt` |
| Page through file | `more file.txt` |
| Write to file | `echo text > file.txt` |
| Append to file | `echo text >> file.txt` |
| Rename file | `ren old.txt new.txt` |
| Delete file | `del file.txt` |
| Show variables | `set` |
| Print variable | `echo %TEMP%` |
| Show system info | `systeminfo` |
| Show current user | `whoami` |
| Show privileges | `whoami /priv` |
| Show groups | `whoami /groups` |
| Show network config | `ipconfig /all` |
| Show ARP cache | `arp -a` |
| List users | `net user` |
| List groups | `net localgroup` |
| List admins | `net localgroup Administrators` |
| List shares | `net share` |
| Find files | `where /r C:\Users *.txt` |
| Search file contents | `findstr /s /i "password" *.txt` |
| Compare files | `fc /n file1.txt file2.txt` |
| Sort file | `sort input.txt /o output.txt` |

---

## Notes to Remember

- Use CMD to understand the system before making changes.
- Environment variables make commands more portable across different users and hosts.
- Recursive searches can be noisy and slow.
- Built-in Windows commands are powerful for both administration and attacker tradecraft.
- Defenders should monitor command-line activity in context rather than treating every built-in command as malicious.