# Gobuster

Gobuster is a fast enumeration tool written in Go. It is commonly used to discover hidden directories, files, subdomains, virtual hosts, and cloud storage buckets using wordlists.

It is useful during reconnaissance because many applications expose resources that are not linked in the main interface.

!!! warning
    Use Gobuster only against systems you own or have explicit permission to test.

---

## Overview

Gobuster uses brute force enumeration.

It takes a wordlist and tries each entry against a target.

Examples:

```text
/admin
/login
/uploads
/config.php
dev.example.com
api.example.com
```

A discovered path or hostname may reveal:

- Admin panels
- Hidden APIs
- Backup files
- Upload directories
- Development sites
- Forbidden but existing resources
- Misconfigured virtual hosts

---

## Enumeration vs. Brute Force

| Concept | Meaning |
|---|---|
| Enumeration | Systematically identifying available resources |
| Brute force | Trying many possibilities from a wordlist |

Gobuster uses brute force to perform enumeration.

---

## Main Modes

| Mode | Purpose |
|---|---|
| `dir` | Discover directories and files |
| `dns` | Discover subdomains through DNS |
| `vhost` | Discover virtual hosts through HTTP Host headers |
| `s3` | Discover Amazon S3 buckets |
| `gcs` | Discover Google Cloud Storage buckets |

Most beginner web enumeration uses `dir`, `dns`, and `vhost`.

---

## Basic Syntax

```bash
gobuster <mode> [options]
```

Example:

```bash
gobuster dir -u http://example.com -w /usr/share/wordlists/dirb/common.txt
```

---

## Common Flags

| Flag | Purpose |
|---|---|
| `-u` | Target URL |
| `-w` | Wordlist |
| `-t` | Number of threads |
| `-o` | Output file |
| `-x` | File extensions |
| `-k` | Skip TLS certificate validation |
| `-r` | Follow redirects |
| `-c` | Add cookies |
| `-H` | Add custom header |
| `--delay` | Delay between requests |

---

## dir Mode

`dir` mode discovers directories and files after the base URL.

Example:

```bash
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt
```

Gobuster will try:

```text
http://10.10.10.10/admin
http://10.10.10.10/login
http://10.10.10.10/uploads
```

---

## Directory Enumeration with Threads

Increase threads for speed:

```bash
gobuster dir -u http://10.10.10.10 \
  -w /usr/share/wordlists/dirb/common.txt \
  -t 64
```

!!! note
    More threads means more speed but also more noise and higher load on the target.

---

## File Extension Search

Use `-x` to search for files with extensions.

```bash
gobuster dir -u http://example.thm \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,txt,html,bak
```

This can find files like:

```text
/config.php
/backup.bak
/readme.txt
/index.html
```

Common extensions:

```text
php,txt,html,bak,zip,conf,config,log,old
```

---

## HTTPS and Self-Signed Certificates

In labs, HTTPS services may use self-signed certificates.

Use `-k` to skip TLS validation:

```bash
gobuster dir -u https://10.10.10.10 \
  -w /usr/share/wordlists/dirb/common.txt \
  -k
```

---

## Following Redirects

Use `-r` to follow redirects.

```bash
gobuster dir -u http://example.thm \
  -w wordlist.txt \
  -r
```

This is useful when discovered paths redirect to login pages or canonical URLs.

---

## Status Code Filtering

Gobuster results are based partly on HTTP status codes.

Useful codes:

| Code | Meaning |
|---|---|
| `200` | Found and accessible |
| `204` | Found, no content |
| `301/302` | Redirect |
| `401` | Authentication required |
| `403` | Forbidden but exists |
| `500` | Server error |

Show only selected status codes:

```bash
gobuster dir -u http://example.thm \
  -w wordlist.txt \
  -s 200,204,301,302,401,403
```

Blacklist noisy codes:

```bash
gobuster dir -u http://example.thm \
  -w wordlist.txt \
  -b 404
```

---

## Authenticated Enumeration

If you are authorized and have a valid session, pass cookies with `-c`.

```bash
gobuster dir -u http://internal.thm/dashboard \
  -w wordlist.txt \
  -c "session=123456789abcdef"
```

This can discover authenticated-only paths.

---

## Custom Headers

Use `-H` to add headers.

```bash
gobuster dir -u http://example.thm \
  -w wordlist.txt \
  -H "Authorization: Bearer TOKEN"
```

Other useful headers:

```text
X-Forwarded-For
User-Agent
Authorization
Host
```

---

## Non-Recursive Behavior

Gobuster does not recursively scan discovered directories by default.

If you find:

```text
/uploads
```

Run a new scan against that path:

```bash
gobuster dir -u http://example.thm/uploads \
  -w /usr/share/wordlists/dirb/common.txt
```

This helps avoid missing deeper content.

---

## Web Root and Base Path

Gobuster starts from the exact URL you provide.

Example:

```bash
gobuster dir -u http://example.thm/resources -w wordlist.txt
```

It will test:

```text
/resources/admin
/resources/login
/resources/uploads
```

Use the correct base path for the area you want to enumerate.

---

## Hostname vs. IP

When virtual hosting is involved, use the hostname instead of only the IP address.

Better:

```bash
gobuster dir -u http://target.thm -w wordlist.txt
```

Maybe wrong:

```bash
gobuster dir -u http://10.10.10.10 -w wordlist.txt
```

The web server may show different content depending on the `Host` header.

---

## dns Mode

`dns` mode brute-forces subdomains by querying DNS.

Basic syntax:

```bash
gobuster dns -d example.com -w subdomains.txt
```

Example logic:

```text
admin + example.com = admin.example.com
dev + example.com = dev.example.com
api + example.com = api.example.com
```

Useful flags:

| Flag | Purpose |
|---|---|
| `-d` | Target domain |
| `-w` | Wordlist |
| `-i` | Show IP addresses |
| `-c` | Show CNAME records |
| `-r` | Use custom resolver |

Show IPs:

```bash
gobuster dns -d example.com -w subdomains.txt -i
```

Use a custom resolver:

```bash
gobuster dns -d example.com -w subdomains.txt -r 1.1.1.1
```

---

## vhost Mode

`vhost` mode discovers virtual hosts by changing the HTTP `Host` header.

This is different from DNS mode.

| Mode | Question |
|---|---|
| `dns` | Does this name resolve in DNS? |
| `vhost` | Does the web server respond differently to this Host header? |

Example:

```bash
gobuster vhost -u http://example.thm -w subdomains.txt --append-domain
```

Gobuster sends requests like:

```http
GET / HTTP/1.1
Host: admin.example.thm
```

---

## vhost False Positives

Some servers return the same default page for every invalid host.

If all results have the same response size, filter by length.

Example:

```bash
gobuster vhost -u http://example.thm \
  -w subdomains.txt \
  --append-domain \
  --exclude-length 3054
```

Workflow:

1. Run the scan.
2. Notice the common response length.
3. Exclude that length.
4. Review remaining differences.

---

## dns vs. vhost

| Feature | dns Mode | vhost Mode |
|---|---|---|
| Layer | DNS | HTTP |
| Uses DNS resolver | Yes | No, not necessarily |
| Tests name resolution | Yes | No |
| Tests web server host routing | No | Yes |
| Good for public subdomains | Yes | Sometimes |
| Good for CTF/internal hidden hosts | Sometimes | Yes |

Use both when appropriate.

---

## Wordlists

Common wordlists:

```text
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/
/usr/share/seclists/Discovery/DNS/
```

Install SecLists if needed:

```bash
sudo apt install seclists
```

Wordlist choice matters. A small list is faster. A larger list finds more but creates more noise.

---

## Output Files

Save results with `-o`.

```bash
gobuster dir -u http://example.thm \
  -w /usr/share/wordlists/dirb/common.txt \
  -o gobuster-dir.txt
```

Organized lab example:

```bash
mkdir -p scans/gobuster

gobuster dir -u http://example.thm \
  -w /usr/share/wordlists/dirb/common.txt \
  -o scans/gobuster/root-dir.txt
```

---

## Practical Workflow

A clean Gobuster workflow:

1. Confirm target URL and hostname.
2. Start with a small wordlist.
3. Review status codes and response lengths.
4. Search common extensions.
5. Re-scan interesting directories.
6. Try authenticated enumeration if authorized.
7. Run DNS or vhost enumeration if hostname-based routing is likely.
8. Save outputs.
9. Manually verify interesting findings.

---

## Example Command Set

```bash
# Basic directory scan
gobuster dir -u http://target.thm -w /usr/share/wordlists/dirb/common.txt

# Extension scan
gobuster dir -u http://target.thm \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,bak,zip

# HTTPS with invalid certificate
gobuster dir -u https://target.thm \
  -w /usr/share/wordlists/dirb/common.txt \
  -k

# Authenticated scan
gobuster dir -u http://target.thm/dashboard \
  -w wordlist.txt \
  -c "session=abc123"

# DNS subdomain brute force
gobuster dns -d target.thm -w subdomains.txt -i

# Virtual host brute force
gobuster vhost -u http://target.thm \
  -w subdomains.txt \
  --append-domain

# Vhost with length filtering
gobuster vhost -u http://target.thm \
  -w subdomains.txt \
  --append-domain \
  --exclude-length 3054
```

---

## Safety and Noise

Gobuster can generate many requests quickly.

Reduce noise with:

```bash
--delay 500ms
-t 5
```

Safer habits:

- Use permissioned targets.
- Start with small wordlists.
- Use reasonable thread counts.
- Avoid unnecessary recursion.
- Save outputs.
- Stop if the target becomes unstable.

---

## Quick Reference

| Goal | Command |
|---|---|
| Directory scan | `gobuster dir -u URL -w wordlist` |
| Add extensions | `gobuster dir -u URL -w wordlist -x php,txt` |
| Skip TLS validation | `gobuster dir -u URL -w wordlist -k` |
| Follow redirects | `gobuster dir -u URL -w wordlist -r` |
| Add cookie | `gobuster dir -u URL -w wordlist -c "session=value"` |
| Add header | `gobuster dir -u URL -w wordlist -H "Header: value"` |
| DNS enum | `gobuster dns -d domain -w wordlist` |
| Vhost enum | `gobuster vhost -u URL -w wordlist --append-domain` |
| Save output | `-o results.txt` |

---

## Notes to Remember

- Gobuster finds hidden paths, files, subdomains, and virtual hosts.
- `dir` mode scans URL paths.
- `dns` mode queries DNS.
- `vhost` mode changes the HTTP Host header.
- A `403` can still mean the resource exists.
- Use hostnames when virtual hosts are involved.
- Gobuster is not recursive by default.
- Wordlists determine scan quality.
- Threads increase speed but also noise.