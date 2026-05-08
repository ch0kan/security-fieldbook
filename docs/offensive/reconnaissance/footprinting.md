# Footprinting and Host-Based Enumeration

## Executive Summary

Footprinting is the process of identifying exposed services, understanding how they are configured, and determining whether those services leak useful information or provide a path to access. Unlike basic port scanning, host-based enumeration focuses on interacting with each discovered service to answer practical questions:

- What service is running?
- What version is exposed?
- Does it allow anonymous or guest access?
- Does it leak users, shares, files, hostnames, or credentials?
- Is it misconfigured in a way that could support exploitation?

This note covers common services encountered during internal and external assessments, including FTP, SMB, NFS, DNS, SMTP, IMAP, POP3, SNMP, databases, and IPMI.

---

## 1. FTP and TFTP Enumeration

### FTP Overview

**FTP** is an application-layer protocol used to transfer files between a client and a server. It commonly uses two channels:

| Channel | Port | Purpose |
|---|---:|---|
| Command channel | TCP 21 | Authentication and FTP commands |
| Data channel | TCP 20 or dynamic ports | File transfer traffic |

FTP often appears during enumeration because misconfigured servers may allow anonymous login, writable directories, or access to sensitive files.

### Active vs. Passive FTP

| Mode | How It Works | Security / Firewall Impact |
|---|---|---|
| Active mode | The client connects to TCP 21, then the server connects back to a client-side data port. | Often blocked by client-side firewalls. |
| Passive mode | The client connects to TCP 21, then connects to a server-provided data port. | More firewall-friendly because the client initiates both connections. |

### Dangerous FTP Settings

A common Linux FTP server is **vsFTPd**, configured through `/etc/vsftpd.conf`.

| Setting | Risk |
|---|---|
| `anonymous_enable=YES` | Allows anonymous login. |
| `anon_upload_enable=YES` | Allows anonymous uploads. This can become critical if uploaded files are web-accessible. |
| `write_enable=YES` | Allows filesystem-modifying commands such as upload, delete, and create directory. |
| `hide_ids=YES` | Hides real owner and group names in listings, reducing visibility for defenders and testers. |

### Manual FTP Interaction

Grab the FTP banner with Netcat:

```bash
nc -nv 10.129.14.136 21
```

Connect with an FTP client:

```bash
ftp 10.129.14.136
```

Try anonymous login:

```text
Username: anonymous
Password: anonymous
```

If authenticated, useful commands include:

```text
ls
ls -R
pwd
get filename
put filename
binary
passive
```

### FTP over TLS

If the FTP server requires TLS, use OpenSSL:

```bash
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

### Mirroring FTP Content

Use `wget` to recursively download accessible files:

```bash
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

### FTP Nmap Enumeration

Useful NSE scripts include:

| Script | Purpose |
|---|---|
| `ftp-anon` | Checks anonymous login. |
| `ftp-syst` | Retrieves server status and system information. |
| `ftp-brute` | Attempts credential brute forcing. |

Example scan:

```bash
sudo nmap -sV -sC -A -p 21 10.129.14.136 --script-trace
```

---

## 2. TFTP Enumeration

### TFTP Overview

**TFTP** is a simplified file transfer protocol that runs over UDP port 69. Unlike FTP, it does not provide authentication or directory listing.

| Feature | FTP | TFTP |
|---|---|---|
| Transport | TCP | UDP |
| Common ports | 21 / 20 | 69 |
| Authentication | Supported | None |
| Directory listing | Supported | Not supported |
| Common use | File management | PXE boot, device configs, firmware transfer |

### TFTP Enumeration Strategy

Because TFTP has no directory listing, enumeration usually requires guessing filenames.

Common targets include:

```text
startup-config
running-config
config.txt
backup.cfg
router.cfg
switch.cfg
pxelinux.0
```

Download a guessed file:

```bash
tftp 10.129.14.136
tftp> get startup-config
```

---

## 3. SMB and Samba Enumeration

## SMB Overview

**SMB** is used for file and printer sharing. It is native to Windows, while **Samba** allows Linux and Unix systems to provide SMB-compatible services.

| Port | Protocol | Purpose |
|---:|---|---|
| 137 | UDP | NetBIOS name service |
| 138 | UDP | NetBIOS datagram service |
| 139 | TCP | SMB over NetBIOS |
| 445 | TCP | Direct SMB over TCP |

### SMB Versions

| Version | Notes |
|---|---|
| SMB 1 / CIFS | Legacy and insecure. Associated with major vulnerabilities such as EternalBlue. |
| SMB 2.x | Better performance and reduced protocol overhead. |
| SMB 3.x | Adds stronger security features such as encryption and improved integrity. |

### Dangerous Samba Settings

Samba is commonly configured in `/etc/samba/smb.conf`.

| Setting | Risk |
|---|---|
| `browseable = yes` | Share is visible during enumeration. |
| `writable = yes` | Users may be able to create or modify files. |
| `guest ok = yes` | Allows guest access. |
| `map to guest = bad user` | Failed logins may be mapped to guest access instead of being rejected. |

---

## 4. SMB Manual Enumeration

### List Shares with a Null Session

```bash
smbclient -N -L //10.129.14.128
```

### Connect to a Share

```bash
smbclient //10.129.14.128/notes
```

Inside the SMB prompt:

```text
ls
pwd
get filename
put filename
recurse ON
prompt OFF
mget *
```

Run local commands from inside `smbclient` using `!`:

```text
!ls
!pwd
```

### RPC Enumeration

Connect with an empty username:

```bash
rpcclient -U "" 10.129.14.128
```

Useful `rpcclient` commands:

| Command | Purpose |
|---|---|
| `srvinfo` | Shows server information. |
| `enumdomusers` | Lists domain users if allowed. |
| `enumdomgroups` | Lists domain groups if allowed. |
| `queryuser <RID>` | Shows detailed information about a user. |
| `querygroup <RID>` | Shows detailed information about a group. |

### RID Cycling

If direct user listing is blocked, try querying common RID ranges:

```bash
for rid in $(seq 500 1100); do
  rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $rid)" 2>/dev/null | grep -E "User Name|user_rid"
done
```

---

## 5. SMB Automated Enumeration

### Nmap SMB Scripts

```bash
sudo nmap -p 139,445 --script smb-os-discovery,smb-enum-shares,smb-enum-users 10.129.14.128
```

Check for SMB vulnerabilities:

```bash
sudo nmap -p 445 --script smb-vuln* 10.129.14.128
```

### SMBMap

```bash
smbmap -H 10.129.14.128
```

With credentials:

```bash
smbmap -H 10.129.14.128 -u username -p password
```

### CrackMapExec

```bash
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

With credentials:

```bash
crackmapexec smb 10.129.14.128 -u username -p password --shares
```

### Enum4Linux-ng

```bash
./enum4linux-ng.py 10.129.14.128 -A
```

---

## 6. NFS Enumeration

## NFS Overview

**NFS** is commonly used for file sharing between Linux and Unix systems. It allows remote directories to be mounted locally.

NFS relies on RPC services.

| Port | Service |
|---:|---|
| 111 TCP/UDP | RPCbind / Portmapper |
| 2049 TCP/UDP | NFS |

### NFS Versions

| Version | Notes |
|---|---|
| NFSv2 | Legacy, usually UDP. |
| NFSv3 | Supports TCP and larger files, still common. |
| NFSv4 | Stateful, uses TCP 2049, supports stronger security options such as Kerberos. |

### Dangerous NFS Export Options

NFS exports are configured in `/etc/exports`.

| Option | Risk |
|---|---|
| `rw` | Allows read and write access. |
| `ro` | Read-only access. Lower risk. |
| `root_squash` | Maps remote root to an unprivileged user. Safer default. |
| `no_root_squash` | Allows remote root to act as root on the share. Critical risk. |
| `insecure` | Allows connections from non-privileged ports. |

### NFS Nmap Enumeration

```bash
sudo nmap --script nfs* -sV -p 111,2049 10.129.14.128
```

Useful scripts include:

| Script | Purpose |
|---|---|
| `nfs-showmount` | Lists exported shares. |
| `nfs-ls` | Lists files if accessible. |
| `nfs-statfs` | Shows filesystem information. |

### Show Exports Manually

```bash
showmount -e 10.129.14.128
```

### Mount an NFS Share

```bash
mkdir target-NFS
sudo mount -t nfs 10.129.14.128:/mnt/nfs ./target-NFS/ -o nolock
cd target-NFS
ls -la
```

### UID and GID Matching

NFS often trusts numeric user IDs and group IDs. If a file is owned by UID `1000`, and your local user also has UID `1000`, the server may treat you as the file owner.

This can be abused by creating a local user with a matching UID:

```bash
sudo useradd -u 1000 tempuser
sudo su - tempuser
```

### `no_root_squash` Risk

If `no_root_squash` is enabled, remote root may create files on the share as root. This can support privilege escalation if the target later executes those files.

General attack pattern:

1. Mount the share.
2. Create or copy a binary as local root.
3. Set the SUID bit.
4. Execute it from the target context if accessible.

---

## 7. DNS Enumeration

## DNS Overview

**DNS** translates domain names into IP addresses. During footprinting, DNS can reveal subdomains, mail servers, internal hostnames, name servers, and sometimes entire zones.

| Record | Purpose |
|---|---|
| A | Hostname to IPv4 address. |
| AAAA | Hostname to IPv6 address. |
| CNAME | Alias to another hostname. |
| MX | Mail server for the domain. |
| NS | Authoritative name servers. |
| PTR | Reverse DNS lookup. |
| SOA | Zone authority and administrative metadata. |
| TXT | Arbitrary text, often SPF, DKIM, DMARC, or verification data. |

### BIND9 Configuration Locations

Common BIND9 files include:

| File | Purpose |
|---|---|
| `/etc/bind/named.conf.local` | Defines zones. |
| `/etc/bind/db.domain.com` | Forward zone records. |
| `/etc/bind/db.x.x.x` | Reverse zone records. |

### Dangerous DNS Settings

| Setting | Risk |
|---|---|
| `allow-transfer` | If too broad, attackers may perform full zone transfers. |
| `allow-recursion` | If open to the internet, may support DNS amplification attacks. |
| `allow-query` | Controls who can query the server. |

### Standard DNS Queries with `dig`

Query name servers:

```bash
dig ns inlanefreight.htb @10.129.14.128
```

Query all available records:

```bash
dig any inlanefreight.htb @10.129.14.128
```

Query SOA record:

```bash
dig soa inlanefreight.htb @10.129.14.128
```

Query BIND version if exposed:

```bash
dig CH TXT version.bind @10.129.14.128
```

### Zone Transfer

Attempt a zone transfer:

```bash
dig axfr inlanefreight.htb @10.129.14.128
```

If an internal subdomain is found, test it too:

```bash
dig axfr internal.inlanefreight.htb @10.129.14.128
```

### Subdomain Brute Forcing

```bash
for sub in $(cat subdomains.txt); do
  dig "$sub.inlanefreight.htb" @10.129.14.128 +short
done
```

Cleaner output:

```bash
for sub in $(cat subdomains.txt); do
  result=$(dig "$sub.inlanefreight.htb" @10.129.14.128 +short)
  if [ -n "$result" ]; then
    echo "$sub.inlanefreight.htb -> $result"
  fi
done
```

---

## 8. SMTP Enumeration

## SMTP Overview

**SMTP** is used to send and relay email. It usually runs on TCP port 25 for server-to-server mail transfer and TCP port 587 for authenticated client submission.

| Port | Purpose |
|---:|---|
| 25 | SMTP relay / server-to-server mail |
| 587 | Mail submission with STARTTLS |
| 465 | SMTPS / implicit TLS |

SMTP is useful during enumeration because it may leak valid usernames or allow open relay abuse.

### SMTP Commands

| Command | Purpose |
|---|---|
| `HELO` / `EHLO` | Starts the SMTP session. |
| `VRFY` | Attempts to verify whether a user exists. |
| `EXPN` | Expands mailing lists. |
| `MAIL FROM` | Defines sender address. |
| `RCPT TO` | Defines recipient address. |
| `DATA` | Begins the email body. |
| `STARTTLS` | Upgrades the connection to TLS. |

### Manual SMTP Interaction

```bash
telnet 10.129.14.128 25
```

Example:

```text
EHLO attacker.com
VRFY root
QUIT
```

Possible user enumeration responses:

| Response | Meaning |
|---|---|
| `250` | User likely exists. |
| `252` | Server cannot verify but accepts the user. |
| `550` | User unknown. |

### Open Relay Testing

An open relay allows unauthenticated third parties to send email through the server.

```text
EHLO attacker.com
MAIL FROM:<admin@corp.com>
RCPT TO:<victim@example.com>
DATA
Subject: Test

This is a relay test.
.
QUIT
```

### SMTP Nmap Scanning

Basic SMTP enumeration:

```bash
sudo nmap -sV -sC -p 25 10.129.14.128
```

Open relay detection:

```bash
sudo nmap -p 25 --script smtp-open-relay -v 10.129.14.128
```

---

## 9. IMAP and POP3 Enumeration

## IMAP and POP3 Overview

SMTP sends mail. **IMAP** and **POP3** retrieve mail.

| Protocol | Cleartext Port | TLS Port | Behavior |
|---|---:|---:|---|
| IMAP | 143 | 993 | Server-side mailbox management. |
| POP3 | 110 | 995 | Simple download-oriented retrieval. |

### IMAP Commands

IMAP commands are usually tagged.

| Command | Purpose |
|---|---|
| `1 LOGIN <user> <pass>` | Authenticate. |
| `1 LIST "" *` | List mailboxes. |
| `1 SELECT INBOX` | Select inbox. |
| `1 FETCH <id> all` | Fetch message metadata or content. |

### POP3 Commands

| Command | Purpose |
|---|---|
| `USER <username>` | Provide username. |
| `PASS <password>` | Provide password. |
| `STAT` | Show message count and total size. |
| `LIST` | List message IDs and sizes. |
| `RETR <id>` | Retrieve a message. |

### Nmap Email Retrieval Scan

```bash
sudo nmap -sV -sC -p 110,143,993,995 10.129.14.128
```

Pay attention to TLS certificate information. It may reveal internal hostnames or mail domains.

### OpenSSL Interaction

IMAP over TLS:

```bash
openssl s_client -connect 10.129.14.128:993
```

POP3 over TLS:

```bash
openssl s_client -connect 10.129.14.128:995
```

### cURL IMAP Interaction

List IMAP folders with credentials:

```bash
curl -k 'imaps://10.129.14.128' --user user:password
```

### Dangerous Dovecot Settings

| Setting | Risk |
|---|---|
| `auth_debug_passwords` | May log plaintext passwords. Critical issue. |
| `auth_verbose` | May assist username enumeration. |
| `auth_anonymous_username` | May enable anonymous access patterns if misconfigured. |

---

## 10. SNMP Enumeration

## SNMP Overview

**SNMP** is used to monitor and manage network devices, servers, printers, and appliances.

| Port | Protocol | Purpose |
|---:|---|---|
| 161 | UDP | SNMP polling |
| 162 | UDP | SNMP traps |

SNMP can leak large amounts of information, including processes, software, interfaces, routing tables, usernames, and contact details.

### SNMP Concepts

| Component | Description |
|---|---|
| MIB | Management Information Base. Defines available management data. |
| OID | Object Identifier. Numeric path to a specific data point. |
| Community string | Shared string used like a password in SNMPv1/v2c. |
| Trap | Unsolicited alert sent from agent to manager. |

### SNMP Versions

| Version | Security |
|---|---|
| SNMPv1 | Insecure. Plaintext community strings. |
| SNMPv2c | Insecure. Still plaintext, but very common. |
| SNMPv3 | Supports authentication and encryption. Preferred. |

### Dangerous SNMP Settings

| Setting | Risk |
|---|---|
| `rwuser noauth` | Read/write access without authentication. |
| `rwcommunity public 0.0.0.0/0` | Anyone can use the `public` community string with write access. |

### Brute Force Community Strings

```bash
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128
```

### Walk the SNMP Tree

```bash
snmpwalk -v2c -c public 10.129.14.128
```

### Query Specific OID Ranges with `braa`

```bash
braa public@10.129.14.128:.1.3.6.*
```

---

## 11. MySQL and MariaDB Enumeration

## MySQL Overview

**MySQL** and **MariaDB** are relational database systems commonly used in web stacks such as LAMP and LEMP. They usually listen on TCP port 3306.

### Dangerous MySQL Settings

| Setting | Risk |
|---|---|
| `bind-address = 0.0.0.0` | Exposes MySQL on all interfaces. |
| Weak database credentials | Allows direct data access. |
| Verbose debug settings | May leak query details or errors. |
| Misconfigured `secure_file_priv` | May allow file read/write abuse with SQL commands. |

### Nmap MySQL Enumeration

```bash
sudo nmap -sV -sC -p 3306 --script mysql* 10.129.14.128
```

Always manually verify high-impact findings such as empty root passwords.

### MySQL Manual Login

```bash
mysql -u root -pP4SSw0rd -h 10.129.14.128
```

Note: There is no space between `-p` and the password when providing the password inline.

### Useful MySQL Commands

```sql
select version();
show databases;
use database_name;
show tables;
show columns from table_name;
select * from table_name;
select user, password from mysql.user;
```

High-value databases:

| Database | Purpose |
|---|---|
| `information_schema` | Metadata about databases, tables, and columns. |
| `mysql` | Users, passwords, privileges, and internal server data. |
| `sys` | Performance and troubleshooting views. |

---

## 12. MSSQL Enumeration

## MSSQL Overview

**Microsoft SQL Server** is a Microsoft relational database system commonly found in Windows and Active Directory environments. It usually listens on TCP port 1433.

### Authentication Modes

| Mode | Description |
|---|---|
| Windows Authentication | Uses domain or local Windows credentials. |
| SQL Server Authentication | Uses database-specific usernames and passwords, such as `sa`. |

### Default System Databases

| Database | Purpose |
|---|---|
| `master` | Main system database and server configuration. |
| `model` | Template for new databases. |
| `msdb` | SQL Server Agent jobs and alerts. |
| `tempdb` | Temporary database recreated on restart. |

### Dangerous MSSQL Misconfigurations

| Issue | Risk |
|---|---|
| Weak `sa` credentials | Full database compromise. |
| Unencrypted connections | Credentials or data may be sniffed. |
| Enabled `xp_cmdshell` | Allows OS command execution from SQL. |
| Overprivileged service account | May turn DB compromise into system or domain compromise. |

### Nmap MSSQL Enumeration

```bash
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-ntlm-info -p 1433 10.129.201.248
```

### Metasploit MSSQL Discovery

```bash
msfconsole
use auxiliary/scanner/mssql/mssql_ping
set RHOSTS 10.129.201.248
run
```

### Impacket MSSQL Client

Windows authentication:

```bash
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
```

SQL authentication:

```bash
python3 mssqlclient.py sa@10.129.201.248
```

### Useful MSSQL Commands

```sql
select name from sys.databases;
use database_name;
select * from information_schema.tables;
```

With Impacket shortcuts:

```text
enable_xp_cmdshell
xp_cmdshell whoami
```

---

## 13. Oracle TNS Enumeration

## Oracle TNS Overview

**Oracle Transparent Network Substrate (TNS)** is used by Oracle databases for client-server communication. It commonly listens on TCP port 1521.

Oracle enumeration often requires discovering the correct **SID** or service name before authentication is possible.

### Key Oracle Files

| File | Purpose |
|---|---|
| `listener.ora` | Server-side listener configuration. |
| `tnsnames.ora` | Client-side name resolution for Oracle services. |

### Common Oracle Misconfigurations

| Issue | Risk |
|---|---|
| Default credentials | Older accounts such as `scott/tiger` may work. |
| Unprotected listener | Legacy systems may allow remote listener management. |
| SYSDBA privileges | Full administrative control of the database. |

### Nmap Oracle SID Brute Force

```bash
sudo nmap -p 1521 -sV --open --script oracle-sid-brute 10.129.204.235
```

### ODAT Enumeration

```bash
./odat.py all -s 10.129.204.235
```

### Connect with SQLPlus

```bash
sqlplus scott/tiger@10.129.204.235/XE
```

Connect as SYSDBA if permitted:

```bash
sqlplus scott/tiger@10.129.204.235/XE as sysdba
```

### Useful Oracle SQL Commands

```sql
select table_name from all_tables;
select * from user_role_privs;
select name, password from sys.user$;
```

### ODAT File Upload Example

If privileges allow file writing, ODAT can test file upload paths:

```bash
./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```

---

## 14. IPMI Enumeration

## IPMI Overview

**IPMI** is used for out-of-band server management through a Baseboard Management Controller, such as Dell iDRAC, HP iLO, or Supermicro IPMI. It usually listens on UDP port 623.

Access to IPMI can be equivalent to physical access to the server because it may allow remote reboot, console access, virtual media mounting, and OS reinstallation.

### Common BMC Interfaces

| Vendor | Product | Default Username | Default Password |
|---|---|---|---|
| Dell | iDRAC | `root` | `calvin` |
| HP | iLO | `Administrator` | Randomized factory password |
| Supermicro | IPMI | `ADMIN` | `ADMIN` |

### Nmap IPMI Discovery

```bash
sudo nmap -sU --script ipmi-version -p 623 10.129.42.195
```

### Metasploit IPMI Version Discovery

```bash
msfconsole
use auxiliary/scanner/ipmi/ipmi_version
set RHOSTS 10.129.42.195
run
```

### IPMI 2.0 Hash Disclosure

IPMI 2.0 uses the RAKP authentication process. During authentication, the server may provide a salted password hash for a requested valid user before the client has authenticated. This allows offline password cracking.

Dump hashes with Metasploit:

```bash
msfconsole
use auxiliary/scanner/ipmi/ipmi_dumphashes
set RHOSTS 10.129.42.195
run
```

Crack IPMI hashes with Hashcat mode 7300:

```bash
hashcat -m 7300 ipmi_hash.txt rockyou.txt
```

For HP iLO-style 8-character uppercase alphanumeric factory passwords:

```bash
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

---

## 15. Footprinting Workflow

Use a repeatable process when enumerating a host.

### Step 1: Identify Open Services

```bash
sudo nmap -sS -sV -sC -O -p- 10.129.14.128
```

### Step 2: Prioritize Services

Prioritize services that commonly leak data or provide authentication paths:

1. SMB
2. NFS
3. FTP
4. SNMP
5. DNS
6. Databases
7. Mail services
8. IPMI

### Step 3: Check Anonymous or Guest Access

Examples:

```bash
smbclient -N -L //10.129.14.128
ftp 10.129.14.128
showmount -e 10.129.14.128
snmpwalk -v2c -c public 10.129.14.128
```

### Step 4: Extract Useful Information

Look for:

- Usernames
- Hostnames
- Internal domains
- Shares
- Writable folders
- Configuration files
- Database names
- Email addresses
- Version numbers
- Credentials
- Password hashes

### Step 5: Reuse Findings Across Services

A username found through SMTP, SNMP, SMB, or NFS may be useful for:

- Password spraying
- SMB authentication
- SSH login
- Database login
- Web application login
- Email login

---

## 16. Quick Reference

| Service | Ports | Primary Tools | High-Value Findings |
|---|---:|---|---|
| FTP | 21, 20 | `ftp`, `wget`, `nmap`, `openssl` | Anonymous login, writable upload paths, sensitive files |
| TFTP | 69/UDP | `tftp`, wordlists | Config files, firmware, boot files |
| SMB | 139, 445 | `smbclient`, `rpcclient`, `smbmap`, `cme`, `enum4linux-ng` | Shares, users, groups, writable folders |
| NFS | 111, 2049 | `showmount`, `mount`, `nmap` | Exports, UID/GID access, `no_root_squash` |
| DNS | 53 | `dig`, `host`, `nmap` | Subdomains, zone transfers, internal hosts |
| SMTP | 25, 587, 465 | `telnet`, `nc`, `nmap` | Valid users, open relay |
| IMAP / POP3 | 143, 993, 110, 995 | `openssl`, `curl`, `nmap` | Mailboxes, credentials, internal communications |
| SNMP | 161/UDP, 162/UDP | `onesixtyone`, `snmpwalk`, `braa` | Users, processes, software, interfaces |
| MySQL | 3306 | `mysql`, `nmap` | Databases, tables, credentials, hashes |
| MSSQL | 1433 | `mssqlclient.py`, `nmap`, Metasploit | Databases, Windows auth, `xp_cmdshell` |
| Oracle TNS | 1521 | `odat`, `sqlplus`, `nmap` | SID, default creds, SYSDBA access |
| IPMI | 623/UDP | `nmap`, Metasploit, Hashcat | BMC access, dumped hashes |

---

## Key Takeaways

- Footprinting is about understanding exposed services, not just finding open ports.
- Anonymous access and guest access should always be tested carefully.
- Misconfigured file-sharing services such as SMB, FTP, and NFS often leak credentials or sensitive documents.
- DNS can reveal internal structure through records, subdomains, and zone transfers.
- SNMP can leak an entire system profile if weak community strings are used.
- Database services should be checked for exposed ports, weak credentials, and dangerous execution features.
- IPMI is especially high impact because BMC access can equal physical server control.