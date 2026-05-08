# Common Ports

---

## Executive Summary

Common ports help quickly identify likely services during scanning, enumeration, traffic analysis, and log review.

Use this page as a fast reference when reviewing Nmap results, firewall logs, packet captures, or SIEM events.

## Common TCP Ports

| Port | Service | Notes |
|---:|---|---|
| 20 | FTP Data | File transfer data channel |
| 21 | FTP Control | Cleartext file transfer login/control |
| 22 | SSH | Secure shell, SCP, SFTP |
| 23 | Telnet | Cleartext remote shell |
| 25 | SMTP | Mail transfer |
| 53 | DNS | Zone transfers and DNS queries over TCP |
| 80 | HTTP | Web traffic |
| 88 | Kerberos | Active Directory authentication |
| 110 | POP3 | Mail retrieval |
| 111 | RPCBind | NFS/RPC mapping |
| 135 | MSRPC | Windows RPC endpoint mapper |
| 139 | NetBIOS Session | Legacy Windows file sharing |
| 143 | IMAP | Mail retrieval |
| 389 | LDAP | Directory services |
| 443 | HTTPS | Encrypted web traffic |
| 445 | SMB | Windows file sharing and domain services |
| 465 | SMTPS | SMTP over SSL/TLS |
| 587 | SMTP Submission | Authenticated mail submission |
| 636 | LDAPS | LDAP over SSL/TLS |
| 993 | IMAPS | IMAP over SSL/TLS |
| 995 | POP3S | POP3 over SSL/TLS |
| 1433 | MSSQL | Microsoft SQL Server |
| 1521 | Oracle TNS | Oracle database listener |
| 2049 | NFS | Network File System |
| 3306 | MySQL | MySQL database |
| 3389 | RDP | Remote Desktop Protocol |
| 5432 | PostgreSQL | PostgreSQL database |
| 5900 | VNC | Remote desktop |
| 5985 | WinRM HTTP | Windows Remote Management |
| 5986 | WinRM HTTPS | Windows Remote Management over TLS |
| 6379 | Redis | In-memory database |
| 8080 | HTTP Alternate | Web proxy or alternate web server |
| 8443 | HTTPS Alternate | Alternate TLS web service |
| 9200 | Elasticsearch | Search/database API |
| 27017 | MongoDB | MongoDB database |

## Common UDP Ports

| Port | Service | Notes |
|---:|---|---|
| 53 | DNS | DNS queries |
| 67 | DHCP Server | IP address assignment |
| 68 | DHCP Client | IP address assignment |
| 69 | TFTP | Simple file transfer |
| 123 | NTP | Time synchronization |
| 137 | NetBIOS Name | Legacy Windows name service |
| 138 | NetBIOS Datagram | Legacy Windows browsing |
| 161 | SNMP | Network device management |
| 162 | SNMP Trap | SNMP alerts |
| 500 | IKE | VPN/IPsec negotiation |
| 514 | Syslog | Network logging |
| 520 | RIP | Routing protocol |
| 1900 | SSDP | UPnP discovery |
| 4500 | IPsec NAT-T | VPN/IPsec traversal |
| 5353 | mDNS | Multicast DNS |

## Active Directory Ports

| Port | Protocol | Service |
|---:|---|---|
| 53 | TCP/UDP | DNS |
| 88 | TCP/UDP | Kerberos |
| 135 | TCP | RPC Endpoint Mapper |
| 137-139 | TCP/UDP | NetBIOS |
| 389 | TCP/UDP | LDAP |
| 445 | TCP | SMB |
| 464 | TCP/UDP | Kerberos password change |
| 636 | TCP | LDAPS |
| 3268 | TCP | Global Catalog |
| 3269 | TCP | Global Catalog over SSL/TLS |
| 5985 | TCP | WinRM HTTP |
| 5986 | TCP | WinRM HTTPS |

## Web and Proxy Ports

| Port | Service |
|---:|---|
| 80 | HTTP |
| 443 | HTTPS |
| 8000 | Development web server |
| 8008 | Alternate HTTP |
| 8080 | HTTP proxy or alternate HTTP |
| 8081 | Alternate HTTP |
| 8443 | Alternate HTTPS |
| 8888 | Proxy or development server |

## Database Ports

| Port | Service |
|---:|---|
| 1433 | MSSQL |
| 1521 | Oracle |
| 3306 | MySQL / MariaDB |
| 5432 | PostgreSQL |
| 5984 | CouchDB |
| 6379 | Redis |
| 9200 | Elasticsearch |
| 27017 | MongoDB |

## Quick Scan Examples

Top ports:

```bash
nmap --top-ports 100 TARGET_IP
```

Service detection:

```bash
nmap -sV -sC TARGET_IP
```

All TCP ports:

```bash
nmap -p- --min-rate 1000 TARGET_IP
```

Common UDP scan:

```bash
sudo nmap -sU --top-ports 100 TARGET_IP
```

## Notes

- Open ports do not always mean exploitable services.
- Closed ports may still be useful for firewall mapping.
- Filtered ports suggest packet filtering, firewalls, or dropped traffic.
- Always pair port discovery with service/version detection.