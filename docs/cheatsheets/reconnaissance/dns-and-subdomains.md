# DNS and Subdomains

---

## Executive Summary

DNS and subdomain enumeration help identify hosts, services, environments, and infrastructure connected to a target domain.

Use this cheatsheet for quick DNS lookups, zone transfer testing, subdomain discovery, and virtual host fuzzing.

## Common DNS Record Types

| Record | Purpose |
|---|---|
| `A` | Maps hostname to IPv4 address |
| `AAAA` | Maps hostname to IPv6 address |
| `CNAME` | Alias to another hostname |
| `MX` | Mail server |
| `NS` | Authoritative name server |
| `TXT` | Text records, SPF, DKIM, verification values |
| `SOA` | Start of authority record |
| `PTR` | Reverse DNS lookup |
| `SRV` | Service location record |

## Basic DNS Lookups

Query A record:

```bash
dig example.com A
```

Query all common records:

```bash
dig example.com ANY
```

Query name servers:

```bash
dig example.com NS
```

Query mail servers:

```bash
dig example.com MX
```

Query TXT records:

```bash
dig example.com TXT
```

Reverse lookup:

```bash
dig -x TARGET_IP
```

Use a specific DNS server:

```bash
dig @DNS_SERVER example.com
```

## Nslookup

Basic lookup:

```bash
nslookup example.com
```

Query record type:

```bash
nslookup -type=MX example.com
```

Use specific DNS server:

```bash
nslookup example.com DNS_SERVER
```

Interactive mode:

```text
nslookup
server DNS_SERVER
set type=ANY
example.com
exit
```

## Zone Transfer Testing

Attempt zone transfer:

```bash
dig axfr @NAMESERVER example.com
```

Example:

```bash
dig axfr @ns1.example.com example.com
```

Nslookup zone transfer:

```text
nslookup
server NAMESERVER
set type=ANY
ls -d example.com
exit
```

A successful zone transfer may expose:

- Subdomains
- Internal naming patterns
- Mail servers
- Name servers
- Infrastructure IP addresses
- Forgotten hosts

## Subdomain Discovery

Common tools:

| Tool | Use |
|---|---|
| `amass` | Passive and active subdomain enumeration |
| `subfinder` | Fast passive subdomain discovery |
| `assetfinder` | Lightweight passive discovery |
| `dnsenum` | DNS enumeration and brute forcing |
| `dnsrecon` | DNS enumeration and zone transfer checks |
| `ffuf` | Subdomain and VHost fuzzing |
| `gobuster` | DNS and VHost brute forcing |

## Dnsenum

Basic enumeration:

```bash
dnsenum example.com
```

Use wordlist:

```bash
dnsenum --enum example.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

Recursive brute forcing:

```bash
dnsenum --enum example.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```

## Dnsrecon

Standard enumeration:

```bash
dnsrecon -d example.com
```

Zone transfer test:

```bash
dnsrecon -d example.com -t axfr
```

Brute force subdomains:

```bash
dnsrecon -d example.com -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -t brt
```

## Amass

Passive enumeration:

```bash
amass enum -passive -d example.com
```

Active enumeration:

```bash
amass enum -active -d example.com
```

Save output:

```bash
amass enum -passive -d example.com -o amass.txt
```

## Subfinder

Run subfinder:

```bash
subfinder -d example.com
```

Save output:

```bash
subfinder -d example.com -o subdomains.txt
```

## Certificate Transparency

Search certificate logs:

```bash
curl -s "https://crt.sh/?q=example.com&output=json" | jq -r '.[].name_value' | sort -u
```

Filter for specific keyword:

```bash
curl -s "https://crt.sh/?q=example.com&output=json" | jq -r '.[].name_value' | grep dev | sort -u
```

Certificate logs may reveal:

- Old hosts
- Staging systems
- Development environments
- Wildcard certificates
- Forgotten subdomains

## Gobuster DNS

DNS brute force:

```bash
gobuster dns -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

Show IP addresses:

```bash
gobuster dns -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -i
```

## Virtual Host Discovery

VHost fuzzing checks hidden sites hosted on the same IP or web server.

Gobuster VHost:

```bash
gobuster vhost -u http://example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain
```

FFUF VHost:

```bash
ffuf -u http://TARGET_IP/ -H "Host: FUZZ.example.com" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

Filter by response size:

```bash
ffuf -u http://TARGET_IP/ -H "Host: FUZZ.example.com" -w wordlist.txt -fs SIZE
```

## Resolving Subdomains

Resolve a list:

```bash
while read sub; do host "$sub"; done < subdomains.txt
```

Extract IPs:

```bash
while read sub; do dig +short "$sub"; done < subdomains.txt | sort -u
```

## Practical Workflow

| Step | Command |
|---|---|
| Get NS records | `dig example.com NS` |
| Test zone transfer | `dig axfr @NAMESERVER example.com` |
| Passive subdomains | `subfinder -d example.com` |
| CT logs | `crt.sh` query |
| Brute force DNS | `gobuster dns -d example.com -w WORDLIST` |
| Resolve results | `dig +short sub.example.com` |
| VHost fuzzing | `ffuf -H "Host: FUZZ.example.com"` |

## Notes

- DNS subdomains and VHosts are related but not the same.
- A VHost may exist without a public DNS record.
- Zone transfers are rare but high-value.
- Certificate logs are passive and often reveal forgotten hosts.
- Always validate discovered subdomains before prioritizing them.