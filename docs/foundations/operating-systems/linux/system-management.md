# Linux System Management

Linux system management covers the daily administrative tasks needed to operate, secure, and maintain a Linux host. This includes managing users and privileges, controlling services and processes, scheduling automated jobs, configuring network-facing services, maintaining backups, managing storage, reviewing logs, and applying hardening controls.

For security work, these skills matter because the same mechanisms administrators use to maintain systems are often abused by attackers for privilege escalation, persistence, lateral movement, and data access.

---

## User Management and Privileges

Linux separates normal users from the administrative user known as **root**. Standard users are restricted from modifying sensitive system files and services, while root can change nearly anything on the system.

This separation is one of Linux’s core security controls.

### Root and Standard Users

A standard user can usually:

- Work inside their own home directory.
- Run normal applications.
- Read files they have permission to access.
- Execute commands available to them.

A root user can:

- Modify system configuration files.
- Install or remove software.
- Create, delete, and modify users.
- Read sensitive files such as `/etc/shadow`.
- Start, stop, enable, or disable system services.

### `sudo`

The `sudo` command allows a permitted user to run a command with elevated privileges, usually as root.

```bash
sudo cat /etc/shadow
```

Without `sudo`, a standard user trying to read `/etc/shadow` will normally receive a permission denied error.

```bash
cat /etc/shadow
```

The benefit of `sudo` is that users can perform administrative actions without logging in as root directly. This is safer, more auditable, and easier to control.

### `su`

The `su` command switches the current shell session to another user.

```bash
su -
```

Using `su -` attempts to switch to the root account and load root’s environment. Unlike `sudo`, this requires the target user’s password, usually the root password.

### Essential User and Group Commands

| Command | Purpose |
|---|---|
| `sudo` | Run a command as another user, usually root. |
| `su` | Switch to another user account. |
| `useradd` | Create a new user. |
| `userdel` | Delete a user. |
| `usermod` | Modify an existing user. |
| `passwd` | Change a user’s password. |
| `addgroup` | Create a new group. |
| `delgroup` | Delete a group. |

### Common Examples

Create a new user:

```bash
sudo useradd alex
```

Set or change a password:

```bash
sudo passwd alex
```

Modify a user and add them to a group:

```bash
sudo usermod -aG sudo alex
```

Delete a user:

```bash
sudo userdel alex
```

Delete a user and their home directory:

```bash
sudo userdel -r alex
```

### Security Notes

User and group management is important for both administration and security monitoring. Unexpected user creation, privilege changes, or password changes can indicate persistence or account compromise.

Important files to know:

| File | Purpose |
|---|---|
| `/etc/passwd` | Stores user account information. |
| `/etc/shadow` | Stores password hashes. Requires root privileges. |
| `/etc/group` | Stores group membership information. |
| `/etc/sudoers` | Controls who can use `sudo`. |

---

## Service and Process Management

Linux services, also called **daemons**, are background processes that provide system functionality. Examples include SSH, web servers, databases, logging services, and scheduled task managers.

Modern Linux distributions usually manage services with **systemd** using the `systemctl` command.

---

## Managing Services with `systemctl`

The `systemctl` command controls systemd services.

| Action | Command | Description |
|---|---|---|
| Start service | `systemctl start <service>` | Starts a service immediately. |
| Stop service | `systemctl stop <service>` | Stops a running service. |
| Restart service | `systemctl restart <service>` | Stops and starts a service again. |
| Enable service | `systemctl enable <service>` | Starts the service automatically at boot. |
| Disable service | `systemctl disable <service>` | Prevents the service from starting at boot. |
| Check status | `systemctl status <service>` | Shows service state, PID, and recent logs. |

### Examples

Start SSH:

```bash
sudo systemctl start ssh
```

Enable SSH at boot:

```bash
sudo systemctl enable ssh
```

Check SSH status:

```bash
systemctl status ssh
```

Stop Apache:

```bash
sudo systemctl stop apache2
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

---

## Troubleshooting Services with `journalctl`

Systemd logs service activity through the journal. The `journalctl` command allows administrators to inspect these logs.

View logs for a specific service:

```bash
journalctl -u ssh.service --no-pager
```

Follow logs live:

```bash
journalctl -u ssh.service -f
```

Show recent logs:

```bash
journalctl -u apache2.service -n 50
```

This is useful when a service fails to start or behaves unexpectedly.

---

## Process Management

Every running program on Linux is a **process**. Each process has a unique **PID**, or Process ID.

Useful commands include:

| Command | Purpose |
|---|---|
| `ps aux` | Show running processes. |
| `top` | Interactive real-time process viewer. |
| `htop` | Improved interactive process viewer, if installed. |
| `pgrep` | Find process IDs by name. |
| `kill` | Send a signal to a process. |
| `jobs` | Show background jobs in the current shell. |
| `fg` | Bring a background job to the foreground. |
| `bg` | Resume a suspended job in the background. |

### Process States

| State | Meaning |
|---|---|
| Running | The process is executing or ready to execute. |
| Waiting | The process is waiting for a resource, such as disk or network I/O. |
| Stopped | The process has been suspended. |
| Zombie | The process has finished but has not been cleaned up by its parent. |

### Signals

Processes are controlled using signals.

| Signal | ID | Shortcut / Command | Purpose |
|---|---:|---|---|
| `SIGINT` | 2 | `Ctrl+C` | Interrupts a process. |
| `SIGKILL` | 9 | `kill -9 <PID>` | Forcefully kills a process. |
| `SIGTERM` | 15 | `kill <PID>` | Gracefully asks a process to terminate. |
| `SIGTSTP` | 20 | `Ctrl+Z` | Suspends a process. |

### Examples

Find a process:

```bash
ps aux | grep apache
```

Find a PID by process name:

```bash
pgrep ssh
```

Gracefully stop a process:

```bash
kill <PID>
```

Force kill a process:

```bash
kill -9 <PID>
```

---

## Foreground and Background Jobs

A command normally runs in the foreground, meaning it occupies the terminal until it finishes.

Run a command in the background by adding `&`:

```bash
ping 8.8.8.8 &
```

List background jobs:

```bash
jobs
```

Suspend a running foreground process:

```text
Ctrl+Z
```

Resume it in the background:

```bash
bg
```

Bring it back to the foreground:

```bash
fg
```

---

## Command Chaining

Linux shells allow commands to be chained together.

| Operator | Meaning | Example |
|---|---|---|
| `;` | Run the next command regardless of success or failure. | `echo A; ls fakefile; echo B` |
| `&&` | Run the next command only if the previous command succeeds. | `mkdir test && cd test` |
| `||` | Run the next command only if the previous command fails. | `ls fakefile || echo "File missing"` |
| `|` | Pipe output from one command into another. | `ps aux | grep ssh` |

### Examples

Run commands unconditionally:

```bash
echo "Starting"; whoami; hostname
```

Run the second command only if the first succeeds:

```bash
mkdir reports && cd reports
```

Run fallback logic if a command fails:

```bash
cat missing.txt || echo "File does not exist"
```

Pipe output into another command:

```bash
ps aux | grep apache
```

---

## Task Scheduling: Cron and Systemd Timers

Task scheduling allows administrators to automate repeated work such as backups, updates, log rotation, and maintenance scripts.

From a security perspective, scheduled tasks are also important because attackers commonly abuse them for persistence.

Linux commonly uses two scheduling systems:

| Scheduler | Description |
|---|---|
| Cron | Traditional time-based scheduler. |
| Systemd Timers | Modern systemd-based scheduler with better logging and event-based triggers. |

---

## Cron

Cron uses text files called **crontabs** to define scheduled jobs.

Edit the current user’s crontab:

```bash
crontab -e
```

List the current user’s crontab:

```bash
crontab -l
```

### Crontab Format

A cron job uses five time fields followed by the command.

```text
* * * * * command
│ │ │ │ │
│ │ │ │ └── day of week, 0-7
│ │ │ └──── month, 1-12
│ │ └────── day of month, 1-31
│ └──────── hour, 0-23
└────────── minute, 0-59
```

### Common Cron Examples

| Schedule | Syntax |
|---|---|
| Every minute | `* * * * * /path/script.sh` |
| Every 6 hours | `0 */6 * * * /path/script.sh` |
| Every day at midnight | `0 0 * * * /path/script.sh` |
| Every Sunday at midnight | `0 0 * * 0 /path/script.sh` |
| First day of every month | `0 0 1 * * /path/script.sh` |
| At reboot | `@reboot /path/script.sh` |

### Security Notes for Cron

Cron is a common persistence location. Review these locations during investigations:

```text
/etc/crontab
/etc/cron.d/
/etc/cron.daily/
/etc/cron.hourly/
/etc/cron.weekly/
/var/spool/cron/
```

Suspicious cron jobs often:

- Run from `/tmp`, `/dev/shm`, or hidden directories.
- Use `curl`, `wget`, `bash`, or `python` to download payloads.
- Use `@reboot` for persistence.
- Run as root unexpectedly.

---

## Systemd Timers

Systemd timers are a modern alternative to cron. They use two files:

| File | Purpose |
|---|---|
| `.service` | Defines what command or script runs. |
| `.timer` | Defines when it runs. |

### Example Timer Unit

Location:

```text
/etc/systemd/system/mytimer.timer
```

Example:

```ini
[Unit]
Description=Runs my script every hour

[Timer]
OnBootSec=3min
OnUnitActiveSec=1hour

[Install]
WantedBy=timers.target
```

### Example Service Unit

Location:

```text
/etc/systemd/system/mytimer.service
```

Example:

```ini
[Unit]
Description=My scheduled task

[Service]
ExecStart=/usr/local/bin/backup.sh

[Install]
WantedBy=multi-user.target
```

### Activating a Timer

Reload systemd after creating or modifying unit files:

```bash
sudo systemctl daemon-reload
```

Start the timer:

```bash
sudo systemctl start mytimer.timer
```

Enable it at boot:

```bash
sudo systemctl enable mytimer.timer
```

List active timers:

```bash
systemctl list-timers
```

### Cron vs Systemd Timers

| Feature | Cron | Systemd Timers |
|---|---|---|
| Simplicity | Very simple | More complex |
| Granularity | Minute-based | Seconds and event-based |
| Logging | Limited | Integrated with `journalctl` |
| Configuration | Single crontab line | Requires `.timer` and `.service` |
| Security controls | Basic | Can use systemd sandboxing options |

---

## Network Services

Network services expose functionality over the network. They are essential for administration and application hosting, but they also increase attack surface.

Common Linux network services include:

| Service | Purpose |
|---|---|
| SSH | Encrypted remote command-line access. |
| NFS | Network file sharing. |
| Apache / Nginx | Web hosting. |
| VPN | Encrypted remote network access. |

---

## SSH

SSH is the standard protocol for secure remote administration.

Install OpenSSH server:

```bash
sudo apt install openssh-server -y
```

Check service status:

```bash
systemctl status ssh
```

Connect to a server:

```bash
ssh user@192.168.1.10
```

Connect using a private key:

```bash
ssh -i id_rsa user@192.168.1.10
```

### SSH Configuration

The SSH server configuration file is:

```text
/etc/ssh/sshd_config
```

Common hardening settings include:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

After editing the SSH configuration, restart SSH:

```bash
sudo systemctl restart ssh
```

---

## NFS

NFS, or Network File System, allows a remote directory to be mounted locally.

The NFS server defines shared directories in:

```text
/etc/exports
```

Example export:

```text
/var/backups 192.168.1.0/24(rw,sync)
```

### Common NFS Options

| Option | Meaning | Security Relevance |
|---|---|---|
| `rw` | Read/write access. | Dangerous if broadly allowed. |
| `ro` | Read-only access. | Safer for shared data. |
| `sync` | Writes are committed before replies. | Safer for data integrity. |
| `no_root_squash` | Remote root remains root on the share. | Very dangerous if misconfigured. |

Mount an NFS share:

```bash
mkdir local_mount
sudo mount 192.168.1.10:/shared local_mount
```

### Security Note

The `no_root_squash` option is especially dangerous. If an attacker can mount the share as root, they may be able to create files with root ownership or abuse SUID permissions.

---

## Web Services

Apache and Nginx are common Linux web servers. For quick testing and file transfer, Python can also run a simple temporary web server.

### Apache

Install Apache:

```bash
sudo apt install apache2 -y
```

Start Apache:

```bash
sudo systemctl start apache2
```

Enable Apache at boot:

```bash
sudo systemctl enable apache2
```

Default web root:

```text
/var/www/html
```

Default main configuration paths include:

```text
/etc/apache2/apache2.conf
/etc/apache2/sites-available/
```

### Python HTTP Server

Serve the current directory on port 8000:

```bash
python3 -m http.server 8000
```

Serve on port 80:

```bash
sudo python3 -m http.server 80
```

This is useful for quickly transferring files in labs and assessments.

---

## Command-Line Web Clients

Linux systems commonly use `curl` and `wget` to interact with web services.

### `curl`

`curl` transfers data to or from a server and prints output to the terminal by default.

Fetch a page:

```bash
curl http://example.com
```

View headers only:

```bash
curl -I http://example.com
```

Download a file:

```bash
curl -O http://example.com/file.txt
```

### `wget`

`wget` is designed for file downloads.

Download a file:

```bash
wget http://example.com/file.txt
```

Download recursively:

```bash
wget -r http://example.com
```

### `curl` vs `wget`

| Tool | Best For |
|---|---|
| `curl` | Inspecting responses, headers, APIs, and raw output. |
| `wget` | Downloading files and mirroring content. |

---

## Backup and Restore

Backups protect against accidental deletion, disk failure, ransomware, and destructive attacker activity.

Linux commonly uses tools such as:

| Tool | Description |
|---|---|
| `rsync` | Efficient file synchronization. |
| `duplicity` | Encrypted backup tool. |
| Deja Dup | User-friendly graphical backup tool based on duplicity. |

---

## Rsync

Rsync is a powerful tool for copying and synchronizing files. It only transfers differences between source and destination, making repeated backups efficient.

Install rsync:

```bash
sudo apt install rsync -y
```

Basic backup:

```bash
rsync -av /source/directory/ /backup/directory/
```

Remote backup over SSH:

```bash
rsync -avz -e ssh /source/directory/ user@backup-server:/backup/directory/
```

Useful options:

| Option | Purpose |
|---|---|
| `-a` | Archive mode. Preserves permissions, timestamps, and symlinks. |
| `-v` | Verbose output. |
| `-z` | Compress data during transfer. |
| `--delete` | Delete files from destination if they no longer exist in source. |
| `--backup` | Keep backup copies of overwritten or deleted files. |
| `--backup-dir` | Store changed/deleted files in a separate backup directory. |

Example with deleted files preserved:

```bash
rsync -avz --backup --backup-dir=/backup/old --delete /source/ user@backup-server:/backup/current/
```

---

## Automating Backups

Backups should be automated to avoid relying on manual execution.

### SSH Key Setup

Generate a key pair:

```bash
ssh-keygen -t rsa -b 2048
```

Copy the public key to the backup server:

```bash
ssh-copy-id user@backup-server
```

### Backup Script

Example script:

```bash
#!/bin/bash
rsync -avz -e ssh /home/user/documents/ user@backup-server:/backups/documents/
```

Make it executable:

```bash
chmod +x backup.sh
```

Schedule it with cron:

```bash
crontab -e
```

Example cron entry to run every hour:

```text
0 * * * * /path/to/backup.sh
```

---

## File System Management

Linux stores all files under a single directory tree beginning at `/`. Storage devices are attached to this tree through **mounting**.

---

## File Systems

Common Linux file systems include:

| File System | Description | Common Use |
|---|---|---|
| `ext4` | Stable, journaling file system. | Default for many Linux systems. |
| `ext2` | Older file system without journaling. | Flash drives or simple use cases. |
| `XFS` | High performance with large files. | Enterprise storage and databases. |
| `Btrfs` | Supports snapshots and advanced features. | Systems needing snapshots or integrity features. |
| `NTFS` | Windows file system. | Shared drives or dual-boot environments. |

---

## Inodes

An **inode** stores metadata about a file, including:

- Owner.
- Permissions.
- Timestamps.
- File size.
- Pointers to data blocks.

The filename itself is stored in the directory entry, not inside the inode.

A system can run out of inodes even if disk space remains, especially when millions of tiny files are created.

Check inode usage:

```bash
df -i
```

---

## Disk and Partition Management

List disks and partitions:

```bash
sudo fdisk -l
```

List block devices:

```bash
lsblk
```

Show disk usage:

```bash
df -h
```

Show directory size:

```bash
du -sh /var/log
```

---

## Mounting and Unmounting

Create a mount point:

```bash
sudo mkdir /mnt/usb
```

Mount a device:

```bash
sudo mount /dev/sdb1 /mnt/usb
```

Unmount the device:

```bash
sudo umount /mnt/usb
```

If unmounting fails because the target is busy, find processes using the mount:

```bash
lsof | grep /mnt/usb
```

---

## Persistent Mounts with `/etc/fstab`

To make mounts persist after reboot, configure:

```text
/etc/fstab
```

Example:

```text
UUID=3d6a020d-0000-0000-0000-000000000000 /data ext4 defaults 0 2
```

Typical fields are:

| Field | Meaning |
|---|---|
| Device | UUID or device path. |
| Mount point | Where the device is mounted. |
| File system type | Example: `ext4`, `xfs`, `ntfs`. |
| Options | Example: `defaults`, `noauto`, `ro`. |
| Dump | Backup utility field, usually `0`. |
| Pass | File system check order. |

---

## Swap Space

Swap is disk space used as overflow memory when RAM is full.

Create swap on a partition:

```bash
sudo mkswap /dev/vda2
```

Enable swap:

```bash
sudo swapon /dev/vda2
```

View swap usage:

```bash
swapon --show
```

Swap is slower than RAM, but it can prevent crashes when memory is exhausted.

---

## Containerization

Containerization packages an application and its dependencies into an isolated runtime environment.

Unlike a virtual machine, a container does not run a full separate operating system. It shares the host kernel while isolating user space.

---

## Docker

Docker is the most widely used container platform.

Core concepts:

| Concept | Meaning |
|---|---|
| Image | Read-only template used to create containers. |
| Container | Running instance of an image. |
| Dockerfile | Text file containing image build instructions. |
| Registry | Repository where images are stored, such as Docker Hub. |

### Example Dockerfile

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y apache2 openssh-server

RUN useradd -m docker-user && echo "docker-user:password" | chpasswd

EXPOSE 80 22

CMD service ssh start && apache2ctl -D FOREGROUND
```

Build the image:

```bash
docker build -t my_web_server .
```

Run the container:

```bash
docker run -p 8080:80 -d my_web_server
```

### Docker Management Commands

| Command | Purpose |
|---|---|
| `docker ps` | List running containers. |
| `docker ps -a` | List all containers. |
| `docker stop <id>` | Stop a container. |
| `docker rm <id>` | Remove a stopped container. |
| `docker rmi <image>` | Remove an image. |
| `docker logs <id>` | View container logs. |

### Docker Security Notes

Containers provide isolation, not full virtualization. Security risks include:

- Shared kernel exposure.
- Containers running as root.
- Misconfigured privileged containers.
- Exposed Docker APIs.
- Sensitive host directories mounted into containers.

Avoid running containers with:

```bash
--privileged
```

unless absolutely necessary.

---

## LXC

LXC, or Linux Containers, is more system-oriented than Docker. It behaves more like a lightweight virtual machine and can run a full init system with multiple services.

| Feature | Docker | LXC |
|---|---|---|
| Focus | Application containers | System containers |
| Common use | Microservices and app deployment | Lightweight Linux environments |
| State | Often ephemeral | Often persistent |
| Management style | Run one main process | Manage like a small server |

Resource limits can be enforced with cgroups to prevent containers from exhausting the host.

---

## Network Configuration and Hardening

Linux networking can be managed with both legacy and modern tools.

| Action | Legacy Tool | Modern Tool |
|---|---|---|
| View interfaces | `ifconfig` | `ip addr` |
| Bring interface up | `ifconfig eth0 up` | `ip link set eth0 up` |
| Assign IP | `ifconfig eth0 192.168.1.2` | `ip addr add 192.168.1.2/24 dev eth0` |
| View routes | `route` | `ip route` |
| Add default gateway | `route add default gw <ip>` | `ip route add default via <ip>` |

### Common Network Commands

View IP addresses:

```bash
ip addr
```

View routes:

```bash
ip route
```

Test reachability:

```bash
ping 8.8.8.8
```

View listening sockets:

```bash
ss -tuln
```

Trace network path:

```bash
traceroute 8.8.8.8
```

---

## Linux Access Control Models

Linux security can be described through different access control models.

| Model | Meaning |
|---|---|
| DAC | Discretionary Access Control. File owners control permissions. |
| MAC | Mandatory Access Control. System policy restricts access. |
| RBAC | Role-Based Access Control. Permissions are based on roles. |

Standard Linux permissions are mostly DAC. SELinux and AppArmor provide MAC.

---

## SELinux and AppArmor

Mandatory Access Control systems restrict what applications can access, even if standard permissions would allow it.

### SELinux

SELinux uses labels and policies to define which processes can access which files, ports, and resources.

It is powerful and granular, but can be complex to configure.

### AppArmor

AppArmor uses application profiles based on paths. It is common on Ubuntu and Debian-based systems and is generally easier to understand than SELinux.

These systems are useful because if a service like Apache is compromised, MAC can prevent it from reading sensitive files or launching unexpected programs.

---

## Linux Firewalls

Linux firewalls control network traffic entering, leaving, or passing through the host.

Common firewall tools include:

| Tool | Description |
|---|---|
| iptables | Traditional interface to the Netfilter firewall framework. |
| nftables | Modern replacement for iptables. |
| UFW | Simpler frontend commonly used on Ubuntu. |
| firewalld | Common on RHEL, CentOS, and Fedora. |

---

## Iptables Basics

Iptables uses tables, chains, rules, matches, and targets.

### Main Tables

| Table | Purpose |
|---|---|
| `filter` | Basic allow and deny rules. |
| `nat` | Network Address Translation. |
| `mangle` | Packet modification. |
| `raw` | Connection tracking exemptions. |

### Common Chains

| Chain | Purpose |
|---|---|
| `INPUT` | Packets destined for the local system. |
| `OUTPUT` | Packets generated by the local system. |
| `FORWARD` | Packets routed through the system. |
| `PREROUTING` | Packets before routing decisions. |
| `POSTROUTING` | Packets after routing decisions. |

### Common Targets

| Target | Meaning |
|---|---|
| `ACCEPT` | Allow the packet. |
| `DROP` | Silently discard the packet. |
| `REJECT` | Block and notify the sender. |
| `LOG` | Log the packet details. |

Allow incoming SSH:

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

Set default INPUT policy to DROP:

```bash
sudo iptables -P INPUT DROP
```

List rules with line numbers:

```bash
sudo iptables -L --line-numbers
```

---

## UFW Basics

UFW is easier to use than raw iptables.

Enable UFW:

```bash
sudo ufw enable
```

Allow SSH:

```bash
sudo ufw allow ssh
```

Allow HTTP:

```bash
sudo ufw allow 80/tcp
```

Check status:

```bash
sudo ufw status verbose
```

---

## Linux System Logs

Linux logs are usually stored in:

```text
/var/log/
```

They are essential for troubleshooting and incident response.

| Log | Purpose |
|---|---|
| `/var/log/syslog` | General system messages on Debian/Ubuntu. |
| `/var/log/auth.log` | Authentication and sudo events on Debian/Ubuntu. |
| `/var/log/kern.log` | Kernel messages. |
| `/var/log/ufw.log` | UFW firewall events. |
| `/var/log/apache2/access.log` | Apache access logs. |
| `/var/log/nginx/access.log` | Nginx access logs. |

### Authentication Log Example

```text
Feb 28 15:04:22 server sshd[3010]: Failed password for htb-student from 10.14.15.2 port 50223 ssh2
Feb 28 15:07:19 server sshd[3010]: Accepted password for htb-student from 10.14.15.2 port 50223 ssh2
```

A high number of failed login attempts followed by a successful login may indicate brute force activity.

### Useful Log Commands

View recent log entries:

```bash
tail /var/log/syslog
```

Follow a log live:

```bash
tail -f /var/log/auth.log
```

Search authentication failures:

```bash
grep "Failed password" /var/log/auth.log
```

Search successful SSH logins:

```bash
grep "Accepted" /var/log/auth.log
```

Search compressed rotated logs:

```bash
zgrep "Failed password" /var/log/auth.log.*.gz
```

---

## Log Management Best Practices

Logs should be protected and maintained.

Important practices include:

- Enable log rotation.
- Send logs to a remote log server.
- Monitor authentication logs.
- Monitor service failures.
- Review firewall logs.
- Protect logs from unauthorized modification.

The `logrotate` utility prevents logs from consuming excessive disk space by rotating and compressing old logs.

Remote logging is important because attackers often try to delete local logs after compromise.

---

## Linux Security and Hardening

Linux hardening reduces the attack surface and limits damage if compromise occurs.

### Core Hardening Practices

| Practice | Purpose |
|---|---|
| Keep systems updated | Patches known vulnerabilities. |
| Disable unused services | Reduces attack surface. |
| Enforce least privilege | Limits unnecessary access. |
| Disable SSH root login | Prevents direct root access. |
| Use SSH keys | Reduces password brute-force risk. |
| Use firewalls | Restricts network exposure. |
| Enable MAC systems | Limits service behavior after compromise. |
| Audit SUID/SGID files | Finds dangerous privilege escalation paths. |
| Centralize logs | Protects evidence from local tampering. |

Update the system:

```bash
sudo apt update && sudo apt dist-upgrade
```

Find SUID files:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Find SGID files:

```bash
find / -perm -2000 -type f 2>/dev/null
```

---

## Security Auditing Tools

Useful Linux auditing and security tools include:

| Tool | Purpose |
|---|---|
| Lynis | System hardening audit. |
| chkrootkit | Rootkit checks. |
| rkhunter | Rootkit and suspicious file checks. |
| auditd | Kernel-level auditing. |
| fail2ban | Blocks repeated failed login attempts. |

Install fail2ban:

```bash
sudo apt install fail2ban -y
```

Check fail2ban status:

```bash
sudo systemctl status fail2ban
```

---

## Remote Desktop Protocols on Linux

Although SSH is the standard for command-line access, Linux can also support graphical remote access.

Common protocols include:

| Protocol | Port | Purpose |
|---|---:|---|
| X11 | TCP 6000+ | Remote graphical application display. |
| VNC | TCP 5900+ | Full graphical desktop sharing. |
| XDMCP | UDP 177 | Legacy graphical login protocol. |
| RDP | TCP 3389 | Remote desktop protocol, available on Linux through `xrdp`. |

### X11 Forwarding

X11 can run a graphical application on a remote machine and display it locally through SSH.

```bash
ssh -X user@192.168.1.10
```

Then run a graphical application:

```bash
firefox
```

Native X11 is insecure if exposed directly. It should be tunneled through SSH.

### VNC

VNC provides full graphical desktop access. It is often used for remote support or GUI-based administration.

Because many VNC setups are weakly encrypted or password-only, VNC should be tunneled through SSH or restricted by firewall rules.

### XDMCP

XDMCP is a legacy protocol and should generally be avoided. It sends sensitive session data in cleartext and is vulnerable to interception.

---

## Terminal Shortcuts and Productivity

Terminal shortcuts make command-line work much faster.

### Navigation

| Shortcut | Action |
|---|---|
| `Tab` | Auto-complete commands, paths, and filenames. |
| `Ctrl+A` | Jump to the beginning of the line. |
| `Ctrl+E` | Jump to the end of the line. |
| `Alt+B` | Move backward one word. |
| `Alt+F` | Move forward one word. |

### Editing

| Shortcut | Action |
|---|---|
| `Ctrl+U` | Cut from cursor to start of line. |
| `Ctrl+K` | Cut from cursor to end of line. |
| `Ctrl+W` | Delete the word before the cursor. |
| `Ctrl+Y` | Paste the last cut text. |

### Process Control

| Shortcut | Action |
|---|---|
| `Ctrl+C` | Stop the current process. |
| `Ctrl+Z` | Suspend the current process. |
| `Ctrl+D` | Send EOF or close the shell. |
| `Ctrl+L` | Clear the terminal screen. |

### History

| Shortcut | Action |
|---|---|
| `Ctrl+R` | Reverse-search command history. |
| Up / Down | Move through previous commands. |

---

## Quick Reference

### Users and Privileges

```bash
whoami
id
sudo <command>
su -
sudo useradd <user>
sudo passwd <user>
sudo usermod -aG sudo <user>
```

### Services

```bash
systemctl status <service>
sudo systemctl start <service>
sudo systemctl stop <service>
sudo systemctl enable <service>
sudo systemctl disable <service>
journalctl -u <service>
```

### Processes

```bash
ps aux
top
pgrep <name>
kill <pid>
kill -9 <pid>
jobs
fg
bg
```

### Networking

```bash
ip addr
ip route
ss -tuln
ping <host>
traceroute <host>
```

### Logs

```bash
tail -f /var/log/auth.log
grep "Failed password" /var/log/auth.log
journalctl -xe
```

### Storage

```bash
lsblk
df -h
df -i
du -sh <path>
sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb
```

### Security

```bash
sudo apt update && sudo apt dist-upgrade
find / -perm -4000 -type f 2>/dev/null
sudo ufw status verbose
sudo iptables -L --line-numbers
```