# Web Reconnaissance

---

## Executive Summary

Web reconnaissance focuses on discovering web-facing assets, technologies, subdomains, virtual hosts, exposed paths, and application behavior before exploitation.

The goal is to understand what exists, how it is hosted, what technologies are in use, and where hidden or forgotten attack surface may be exposed.

## Core Areas

| Area | Purpose |
|---|---|
| Subdomain discovery | Find additional hosts belonging to the target |
| DNS enumeration | Identify records, name servers, and misconfigurations |
| Virtual host discovery | Find hidden websites hosted on the same server |
| Certificate transparency | Discover domains from public certificate logs |
| Fingerprinting | Identify web servers, frameworks, CMS platforms, and WAFs |
| Crawling | Map linked pages, endpoints, files, and directories |
| Metadata discovery | Review `robots.txt`, `.well-known`, comments, and headers |

## Subdomain Discovery

Subdomain discovery helps identify additional systems connected to a root domain.

Examples:

```text
www.example.com
dev.example.com
api.example.com
staging.example.com
vpn.example.com
```

Subdomains may reveal:

- Development environments
- Admin panels
- APIs
- VPN portals
- Mail services
- Forgotten legacy systems
- Staging or testing applications

## Subdomain Brute Forcing

Subdomain brute forcing uses a wordlist to guess valid subdomains.

Common tools include:

- `dnsenum`
- `dnsrecon`
- `fierce`
- `amass`
- `assetfinder`
- `puredns`

Example with `dnsenum`:

```bash
dnsenum --enum example.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```

Useful options:

| Option | Purpose |
|---|---|
| `--enum` | Runs standard enumeration checks |
| `-f` | Specifies a wordlist |
| `-r` | Enables recursive brute forcing |

## DNS Zone Transfers

A DNS zone transfer copies DNS records from one name server to another. It is legitimate for DNS replication, but dangerous if exposed to unauthorized users.

If misconfigured, a zone transfer can reveal:

- Full subdomain lists
- Mail servers
- Name servers
- Internal hostnames
- Infrastructure naming patterns
- IP address mappings

Test for a zone transfer with `dig`:

```bash
dig axfr @NAMESERVER example.com
```

Example:

```bash
dig axfr @ns1.example.com example.com
```

Successful zone transfers are rare, but they are high-value findings when present.

## Virtual Hosts

Virtual hosting allows one web server to host multiple websites on the same IP address.

A server may host:

```text
www.example.com
admin.example.com
dev.example.com
internal.example.com
```

All of them may point to the same IP address, but the web server decides which site to serve based on the HTTP `Host` header.

## VHost Discovery

Virtual host discovery finds hidden sites that may not have public DNS records.

Gobuster example:

```bash
gobuster vhost -u http://example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

Useful findings include:

- Admin portals
- Staging environments
- Internal dashboards
- Development applications
- Forgotten test sites

A valid VHost often produces a different response size, status code, title, or redirect behavior.

## Certificate Transparency Logs

Certificate Transparency logs are public records of issued TLS certificates.

They can reveal subdomains without touching the target directly.

Useful sources include:

- `crt.sh`
- Censys
- Google Certificate Transparency logs
- SecurityTrails
- Amass passive sources

Example using `crt.sh`:

```bash
curl -s "https://crt.sh/?q=example.com&output=json" | jq -r '.[].name_value' | sort -u
```

Certificate logs may expose:

- Old subdomains
- Development hosts
- Wildcard certificates
- Forgotten services
- Internal naming conventions

## Web Technology Fingerprinting

Fingerprinting identifies the technologies behind a web application.

Useful clues include:

- HTTP headers
- Cookies
- HTML source code
- JavaScript files
- Default paths
- Error pages
- Login pages
- CMS-specific directories

Manual header check:

```bash
curl -I https://example.com
```

Interesting headers:

| Header | Possible Value |
|---|---|
| `Server` | Web server and version |
| `X-Powered-By` | Backend language or framework |
| `Set-Cookie` | Application framework or session technology |
| `Location` | Redirect behavior |
| `Content-Security-Policy` | Security controls |

## CMS Fingerprinting

Common CMS indicators include:

| CMS | Indicators |
|---|---|
| WordPress | `/wp-content/`, `/wp-admin/`, `/wp-json/` |
| Joomla | `/administrator/`, Joomla-specific generator tags |
| Drupal | `/sites/default/`, Drupal headers or paths |
| Magento | `/skin/`, `/media/`, Magento cookies |

WordPress example indicators:

```text
/wp-login.php
/wp-content/
/wp-json/
```

## WAF Detection

A Web Application Firewall may block or alter suspicious requests.

Check for WAFs with:

```bash
wafw00f https://example.com
```

Possible signs of a WAF:

- Repeated `403 Forbidden` responses
- JavaScript challenges
- CAPTCHA pages
- Block pages
- Changed response headers
- Rate limiting
- Different responses to suspicious payloads

## Nikto Fingerprinting

Nikto can identify common web server issues and technology indicators.

Basic scan:

```bash
nikto -h https://example.com
```

Software identification-focused scan:

```bash
nikto -h https://example.com -Tuning b
```

Nikto may identify:

- Server versions
- Default files
- Dangerous HTTP methods
- Backup files
- Admin paths
- CMS indicators

## Web Crawling

Crawling maps pages and endpoints that are linked from the application.

Crawling can reveal:

- Login pages
- API endpoints
- Parameters
- Forms
- Linked files
- JavaScript routes
- Comments
- Hidden references

Common tools include:

- Burp Suite crawler
- OWASP ZAP spider
- Hakrawler
- Katana
- Custom scripts

## Robots.txt

`robots.txt` tells search engines what not to crawl.

Location:

```text
https://example.com/robots.txt
```

Example:

```text
User-agent: *
Disallow: /admin/
Disallow: /backup/
Disallow: /api/internal/
```

Important note:

`robots.txt` does not enforce access control. It only provides instructions to compliant crawlers.

For reconnaissance, it can reveal sensitive paths that administrators wanted hidden from search engines.

## Well-Known URIs

The `.well-known` directory stores standardized metadata.

Location:

```text
https://example.com/.well-known/
```

Useful endpoints include:

| Endpoint | Purpose |
|---|---|
| `/.well-known/security.txt` | Security contact and disclosure information |
| `/.well-known/openid-configuration` | OpenID Connect configuration |
| `/.well-known/jwks.json` | JSON Web Key Set |
| `/.well-known/change-password` | Password change URL |
| `/.well-known/apple-app-site-association` | iOS app association data |
| `/.well-known/assetlinks.json` | Android app link verification |

These files can reveal authentication endpoints, key locations, mobile app links, and security contacts.

## JavaScript Reconnaissance

JavaScript files often expose useful information.

Look for:

- API routes
- Internal endpoints
- Feature flags
- Cloud storage URLs
- Source maps
- Hardcoded keys
- Environment names
- Hidden functionality

Download and review JavaScript files:

```bash
curl -O https://example.com/static/app.js
```

Search for interesting terms:

```bash
grep -Ei "api|token|key|secret|admin|debug|internal|dev|staging" app.js
```

## Useful Recon Questions

During web reconnaissance, ask:

- What domains and subdomains exist?
- Which hosts are alive?
- Are there hidden virtual hosts?
- What technologies are running?
- Are versions exposed?
- Is a WAF present?
- What paths are linked or disallowed?
- Are authentication endpoints exposed?
- Are development or staging systems reachable?
- Are JavaScript files leaking sensitive routes?

## Practical Workflow

| Phase | Action |
|---|---|
| Passive discovery | Collect subdomains from CT logs and OSINT sources |
| DNS enumeration | Identify DNS records and test for zone transfers |
| Active discovery | Brute force subdomains and virtual hosts |
| Fingerprinting | Identify technologies, servers, CMS platforms, and WAFs |
| Crawling | Map linked pages, forms, files, and endpoints |
| Metadata review | Check `robots.txt`, `.well-known`, comments, and JavaScript |
| Prioritization | Focus on exposed admin panels, old systems, and unusual hosts |

## Defensive Perspective

Defenders should monitor and reduce exposed web attack surface by:

- Removing unused DNS records
- Restricting access to staging systems
- Preventing unauthorized zone transfers
- Avoiding sensitive information in certificates
- Reviewing `robots.txt` for accidental exposure
- Removing secrets from JavaScript files
- Hiding unnecessary version banners
- Monitoring for large-scale enumeration
- Protecting admin portals with strong authentication and access controls