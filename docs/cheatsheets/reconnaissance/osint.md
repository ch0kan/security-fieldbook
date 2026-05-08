# OSINT

---

## Executive Summary

OSINT is the collection of publicly available information about a target, organization, domain, person, technology stack, or infrastructure.

Use this cheatsheet for quick passive reconnaissance before active scanning.

## Core OSINT Targets

| Target | Examples |
|---|---|
| Domains | Root domains, subdomains, DNS records |
| Infrastructure | IP ranges, hosting providers, cloud assets |
| People | Employees, usernames, emails, roles |
| Technologies | Web servers, frameworks, CMS platforms |
| Documents | PDFs, Office files, metadata |
| Repositories | Source code, secrets, internal names |
| Breaches | Leaked credentials, reused emails |
| Social presence | LinkedIn, GitHub, Twitter/X, forums |

## Search Operators

Search within a site:

```text
site:example.com
```

Find specific file types:

```text
site:example.com filetype:pdf
```

Search page titles:

```text
intitle:"admin"
```

Search URLs:

```text
inurl:login
```

Search exact phrase:

```text
"example phrase"
```

Exclude term:

```text
site:example.com -www
```

Find exposed indexes:

```text
site:example.com intitle:"index of"
```

Find login pages:

```text
site:example.com inurl:login
```

Find documents:

```text
site:example.com filetype:pdf OR filetype:docx OR filetype:xlsx
```

## Useful Search Ideas

| Goal | Query |
|---|---|
| Find subdomains | `site:*.example.com` |
| Find PDFs | `site:example.com filetype:pdf` |
| Find spreadsheets | `site:example.com filetype:xlsx` |
| Find login pages | `site:example.com inurl:login` |
| Find admin pages | `site:example.com intitle:admin` |
| Find exposed directories | `site:example.com intitle:"index of"` |
| Find backups | `site:example.com filetype:bak OR filetype:old` |
| Find config files | `site:example.com filetype:env OR filetype:config` |

## Domain and Network Resources

| Resource | Use |
|---|---|
| WHOIS | Domain registration details |
| RDAP | Modern domain/IP registration lookup |
| ARIN / RIPE / APNIC | IP ownership and netblocks |
| BGP tools | ASN and routing information |
| Shodan | Internet-exposed services |
| Censys | Internet-wide certificates and services |
| GreyNoise | Internet noise and scanner context |
| MXToolbox | DNS, mail, and blacklist checks |
| SecurityTrails | DNS and historical infrastructure |

## WHOIS and RDAP

WHOIS lookup:

```bash
whois example.com
```

IP ownership lookup:

```bash
whois TARGET_IP
```

Useful details:

- Registrar
- Name servers
- Creation date
- Contact organization
- Netblocks
- ASN
- Abuse contact

## Email Discovery

Common sources:

- Company websites
- GitHub commits
- LinkedIn
- Breach data
- Certificate records
- Public documents
- Press releases

Common formats:

```text
first.last@example.com
flast@example.com
first@example.com
first_last@example.com
```

## TheHarvester

Search for emails, hosts, and domains:

```bash
theHarvester -d example.com -b all
```

Use specific source:

```bash
theHarvester -d example.com -b bing
```

Save output:

```bash
theHarvester -d example.com -b all -f results
```

## GitHub OSINT

Search for organization mentions:

```text
"example.com"
```

Search for secrets:

```text
"example.com" "password"
```

Search for API keys:

```text
"example.com" "api_key"
```

Search for environment files:

```text
"example.com" ".env"
```

Useful items to review:

- Repositories
- Commit history
- Issues
- Wiki pages
- Actions/workflows
- Configuration files
- Hardcoded URLs
- Internal hostnames

## Document Metadata

Download public files and check metadata.

PDF metadata:

```bash
exiftool file.pdf
```

Office metadata:

```bash
exiftool file.docx
```

Find documents with search:

```text
site:example.com filetype:pdf OR filetype:docx OR filetype:xlsx
```

Metadata may reveal:

- Usernames
- Author names
- Software versions
- Internal paths
- Department names
- Document history

## Breach and Credential Sources

Look for:

- Reused emails
- Breached passwords
- Old usernames
- Password patterns
- Exposed API tokens
- Leaked source code

Useful checks:

- Public breach databases
- GitHub leaks
- Paste sites
- Dark web reports
- Password reuse across services

## Technology OSINT

Identify technologies from:

- HTTP headers
- TLS certificates
- JavaScript files
- BuiltWith-style services
- Shodan/Censys results
- Job postings
- GitHub repositories
- Documentation pages

Useful search:

```text
site:example.com "wp-content"
```

```text
site:example.com "X-Powered-By"
```

```text
site:example.com "Jenkins"
```

## Practical Workflow

| Phase | Action |
|---|---|
| Domain review | WHOIS, RDAP, name servers, MX records |
| Infrastructure | ASN, netblocks, cloud providers, exposed services |
| People | Emails, names, roles, usernames |
| Documents | PDFs, Office files, metadata |
| Code | GitHub, exposed repos, commits, secrets |
| Web presence | Subdomains, technologies, login portals |
| Risk review | Breaches, leaked credentials, exposed assets |

## Notes

- OSINT should be passive unless explicitly authorized otherwise.
- Public information may still be sensitive.
- Validate findings before acting on them.
- Do not assume old data is still accurate.
- Keep track of sources so findings can be verified later.