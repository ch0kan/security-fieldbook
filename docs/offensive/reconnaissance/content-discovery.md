# Content Discovery

Content discovery is the process of finding hidden, unlinked, forgotten, or protected resources on a web application.

It is usually performed after identifying a live web service and before deeper vulnerability testing. Good content discovery combines manual review with automated enumeration.

!!! warning
    Perform content discovery only on systems you own or have explicit permission to test.

---

## Overview

Web applications often expose more than what appears in the main navigation.

Interesting resources may include:

- Admin panels
- Login portals
- Backup files
- API endpoints
- Old pages
- Development routes
- Upload directories
- Configuration files
- Debug endpoints
- Hidden virtual hosts
- Legacy applications

Content discovery helps map the real attack surface.

---

## Manual Before Automated

Before running automated tools, perform quiet manual checks.

Manual discovery can reveal:

- Technology stack
- Hidden routes
- Server versions
- Backend language
- Default framework files
- Search-engine hints
- Publicly exposed documentation

Manual checks are also less noisy than brute-force tools.

---

## Check Common Files

Start with common web discovery files.

```text
/robots.txt
/sitemap.xml
/favicon.ico
/.well-known/
/security.txt
```

These files can reveal useful paths, technologies, or intended crawler behavior.

---

## robots.txt

`robots.txt` tells search engines what paths they should not crawl.

Example:

```text
User-agent: *
Disallow: /admin
Disallow: /backup
Disallow: /dev
```

Security relevance:

- It can reveal sensitive or hidden paths.
- It does not enforce access control.
- Attackers and testers can read it directly.

Example:

```bash
curl https://example.com/robots.txt
```

---

## sitemap.xml

`sitemap.xml` tells search engines what pages should be indexed.

Example location:

```text
https://example.com/sitemap.xml
```

It may reveal:

- Old pages
- Unlinked pages
- Legacy endpoints
- Deep application routes
- Pages not visible in the main UI

Example:

```bash
curl https://example.com/sitemap.xml
```

---

## Favicon Analysis

A favicon is the small icon shown in the browser tab.

Default favicons can reveal frameworks, products, or technologies. Some frameworks and appliances ship with recognizable default icons.

Download favicon:

```bash
curl https://example.com/favicon.ico -o favicon.ico
```

Calculate MD5 hash on Linux:

```bash
md5sum favicon.ico
```

PowerShell:

```powershell
Invoke-WebRequest https://example.com/favicon.ico -OutFile favicon.ico
Get-FileHash .\favicon.ico -Algorithm MD5
```

The hash can be compared with known favicon hash databases.

!!! note
    Favicon fingerprinting is only a hint. Confirm technology with additional evidence.

---

## HTTP Header Inspection

HTTP response headers can reveal technologies and configuration details.

View headers:

```bash
curl -I https://example.com
```

Verbose request and response:

```bash
curl -v https://example.com
```

Interesting headers:

| Header | What It May Reveal |
|---|---|
| `Server` | Web server software/version |
| `X-Powered-By` | Backend language/framework |
| `Set-Cookie` | Session technology or framework |
| `Location` | Redirect paths |
| `WWW-Authenticate` | Authentication mechanism |
| `Content-Security-Policy` | Security policy and allowed sources |

Examples:

```http
Server: nginx/1.18.0 (Ubuntu)
X-Powered-By: PHP/7.4.3
Set-Cookie: PHPSESSID=...
```

Security relevance:

- Exact versions can guide vulnerability research.
- Headers may reveal backend language.
- Cookies can reveal framework defaults.
- Redirects can reveal hidden routes.

---

## Page Source Review

Review HTML source and loaded JavaScript.

Look for:

- Comments
- Hidden links
- API endpoints
- Old routes
- Debug values
- Feature flags
- JavaScript route lists
- Source maps
- Hardcoded hostnames

Useful browser locations:

```text
View Source
Developer Tools -> Network
Developer Tools -> Sources
Developer Tools -> Application
```

Useful command:

```bash
curl https://example.com | less
```

---

## JavaScript Discovery

JavaScript files often reveal application routes and API endpoints.

Look for patterns like:

```text
/api/
/admin/
/internal/
/v1/
/graphql
/upload
/download
```

Download and inspect JavaScript:

```bash
curl https://example.com/static/app.js -o app.js
grep -Eo '(/[A-Za-z0-9_./-]+)' app.js | sort -u
```

Security relevance:

- Frontend routes may reveal hidden pages.
- API endpoints may exist even if not visible in UI.
- Client-side role checks may expose privileged paths.

---

## Technology Fingerprinting

Identify the stack before testing deeply.

Clues:

| Source | Possible Clue |
|---|---|
| Headers | Server and backend language |
| Cookies | Framework or session type |
| HTML source | Generator tags or framework assets |
| Favicon | Default framework or product |
| Error pages | Stack traces or server templates |
| URLs | File extensions such as `.php`, `.aspx`, `.jsp` |

Example extensions:

```text
.php
.aspx
.jsp
.do
.cgi
```

---

## Directory and File Discovery

Automated tools can brute-force paths using wordlists.

Common tools:

- Gobuster
- ffuf
- dirsearch
- Feroxbuster

Example with Gobuster:

```bash
gobuster dir -u https://example.com -w /usr/share/wordlists/dirb/common.txt
```

Search for extensions:

```bash
gobuster dir -u https://example.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,bak,zip
```

---

## Interesting File Extensions

Some extensions are more likely to reveal sensitive information.

```text
.txt
.bak
.old
.zip
.tar.gz
.sql
.conf
.config
.env
.log
.php
.aspx
.jsp
```

Examples:

```text
/config.php.bak
/.env
/backup.zip
/database.sql
/debug.log
```

---

## Status Codes

Directory enumeration results must be interpreted carefully.

| Code | Meaning |
|---|---|
| `200` | Resource exists and is accessible |
| `301/302` | Redirect, may indicate directory or moved resource |
| `401` | Authentication required |
| `403` | Exists but access forbidden |
| `404` | Not found |
| `500` | Server error, may indicate interesting behavior |

A `403 Forbidden` can still be valuable because it confirms the resource exists.

---

## Virtual Host Discovery

A single IP can host multiple websites using the HTTP `Host` header.

Example:

```text
10.10.10.10 -> main.example.thm
10.10.10.10 -> dev.example.thm
10.10.10.10 -> admin.example.thm
```

Virtual host discovery checks whether hidden hostnames return different content.

Example with Gobuster:

```bash
gobuster vhost -u http://example.thm -w subdomains.txt --append-domain
```

---

## Subdomain Discovery

Subdomain discovery finds names like:

```text
admin.example.com
dev.example.com
api.example.com
staging.example.com
```

Common sources:

- DNS brute force
- Certificate transparency logs
- Search engines
- Public code repositories
- Company documentation
- Asset inventory leaks

Gobuster DNS example:

```bash
gobuster dns -d example.com -w subdomains.txt
```

---

## Wordlists

Wordlists are central to content discovery.

Common locations on Kali:

```text
/usr/share/wordlists/
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists/
```

Choose wordlists based on the target:

| Target | Wordlist Idea |
|---|---|
| Small app | Short common list |
| Large app | Medium directory list |
| API | API route wordlist |
| Technology-specific app | WordPress, IIS, PHP, ASP.NET lists |
| Subdomains | DNS/subdomain wordlist |

---

## Authenticated Content Discovery

Some content only appears after login.

If authorized, test authenticated areas using session cookies.

Example:

```bash
gobuster dir -u https://example.com/dashboard \
  -w wordlist.txt \
  -c "session=abc123"
```

Security relevance:

- Hidden authenticated routes may expose privilege issues.
- Role-specific paths may be accessible by lower-privileged users.
- APIs may return more than the UI displays.

---

## Practical Workflow

A clean content discovery workflow:

1. Identify web service and hostname.
2. Visit the site manually.
3. Check `robots.txt`.
4. Check `sitemap.xml`.
5. Download and hash favicon.
6. Inspect response headers.
7. Review page source.
8. Review JavaScript files.
9. Run light directory enumeration.
10. Search for interesting extensions.
11. Test discovered paths manually.
12. Run vhost or subdomain enumeration if relevant.
13. Repeat from interesting directories.

---

## Example Command Set

```bash
# Headers
curl -I https://example.com

# Verbose headers and TLS info
curl -v https://example.com

# Common files
curl https://example.com/robots.txt
curl https://example.com/sitemap.xml

# Favicon hash
curl https://example.com/favicon.ico -o favicon.ico
md5sum favicon.ico

# Directory discovery
gobuster dir -u https://example.com -w /usr/share/wordlists/dirb/common.txt

# Extensions
gobuster dir -u https://example.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,bak,zip
```

---

## What to Document

During content discovery, document:

- Target URL
- Hostnames
- Interesting paths
- Status codes
- Redirects
- Server headers
- Technologies identified
- Wordlists used
- Tool commands
- Screenshots or response evidence
- Follow-up testing notes

Example:

```text
/admin -> 403 Forbidden
/backup.zip -> 200 OK
X-Powered-By: PHP/7.4.3
/favicon.ico hash matched default framework icon
```

---

## Quick Reference

| Technique | Tool / Location | Reveals |
|---|---|---|
| Headers | `curl -I` | Server/backend hints |
| Favicon | `md5sum favicon.ico` | Framework/product hints |
| Sitemap | `/sitemap.xml` | Intended pages |
| Robots | `/robots.txt` | Disallowed paths |
| Source review | Browser / curl | Comments, endpoints |
| JS review | DevTools / grep | API routes |
| Directory brute force | Gobuster / ffuf | Hidden paths |
| Vhost brute force | Gobuster vhost | Hidden sites |
| DNS brute force | Gobuster dns | Subdomains |

---

## Notes to Remember

- Manual discovery should come before noisy automation.
- `robots.txt` and `sitemap.xml` can reveal hidden paths.
- Headers can leak server and framework details.
- JavaScript often exposes API routes.
- A `403` can still confirm that a resource exists.
- Use hostnames when virtual hosting is involved.
- Wordlist choice affects results heavily.
- Always document commands and findings.