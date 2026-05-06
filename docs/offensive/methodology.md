# Methodology

A security assessment should be repeatable, authorized, documented, and risk-aware.

This page summarizes a practical offensive security workflow inspired by NIST SP 800-115, which describes technical security assessments as a process involving planning, execution, and post-execution activities.

!!! warning
    Only perform testing on systems you own or have explicit written permission to assess.

---

## Assessment Phases

NIST SP 800-115 describes a phased assessment methodology with three major phases:

| Phase | Purpose |
|---|---|
| Planning | Define scope, objectives, rules, resources, and constraints |
| Execution | Identify, analyze, and validate vulnerabilities |
| Post-Execution | Analyze findings, recommend mitigations, and report results |

This fieldbook uses that structure as a practical guide for labs, CTFs, internal practice, and authorized assessments.

---

## 1. Scope and Authorization

Before any technical testing, define what is allowed.

Important questions:

- What systems are in scope?
- What systems are out of scope?
- What testing methods are allowed?
- Are automated tools allowed?
- Are brute-force attacks allowed?
- Are denial-of-service tests prohibited?
- What time window is approved?
- Who should be contacted if something breaks?
- How should sensitive data be handled?

Example scope statement:

```text
Testing is limited to 10.10.10.10 and target.thm.
No denial-of-service testing.
No persistence.
No testing outside the provided lab network.
```

---

## 2. Reconnaissance

Reconnaissance gathers information about the target.

Common activities:

- Identify hostnames
- Identify IP addresses
- Review public information
- Check DNS records
- Inspect HTTP headers
- Review `robots.txt`
- Review `sitemap.xml`
- Identify technologies
- Collect possible usernames or emails when in scope

Useful tools and techniques:

```text
whois
dig
nslookup
curl
browser DevTools
content discovery
favicon analysis
```

---

## 3. Enumeration

Enumeration actively identifies exposed services, paths, users, technologies, and application behavior.

Examples:

- Port scanning
- Service detection
- Web directory discovery
- Virtual host discovery
- Subdomain enumeration
- Login form analysis
- API route discovery

Useful tools:

```text
nmap
gobuster
Burp Suite
curl
```

Example:

```bash
nmap -sC -sV -oA scans/initial 10.10.10.10
```

---

## 4. Vulnerability Testing

Vulnerability testing checks whether discovered services or application features are affected by weaknesses.

Common web tests:

- Access control testing
- IDOR testing
- SQL injection testing
- Path traversal testing
- File upload testing
- SSRF testing
- Authentication testing
- Session handling review

Useful tools:

```text
Burp Suite
Repeater
SQLMap
Gobuster
manual payloads
```

Good practice:

```text
Change one variable at a time.
Compare responses carefully.
Record exact requests and responses.
Avoid destructive payloads.
```

---

## 5. Exploitation

Exploitation confirms whether a vulnerability has real impact.

Examples:

- Reading another user's data through IDOR
- Extracting limited proof-of-concept data through SQLi
- Reading a harmless local file through path traversal
- Triggering a callback for SSRF
- Uploading a safe proof-of-concept file in a lab

The goal is to prove impact safely, not to cause unnecessary damage.

!!! danger
    Do not delete data, disrupt services, modify production records, or maintain persistence unless explicitly authorized.

---

## 6. Post-Exploitation Basics

Post-exploitation is the controlled analysis performed after gaining limited access in a lab or authorized assessment.

Common activities:

- Identify current user
- Identify hostname
- Check operating system
- Review network interfaces
- Understand privilege level
- Locate evidence of impact
- Avoid unnecessary changes

Linux examples:

```bash
whoami
id
hostname
pwd
uname -a
ip addr
```

Windows examples:

```cmd
whoami
hostname
ipconfig
whoami /priv
```

---

## 7. Documentation

Documentation turns technical activity into useful evidence.

Record:

- Target
- Scope
- Date and time
- Tool commands
- Requests and responses
- Screenshots
- Findings
- Impact
- Reproduction steps
- Recommended remediation

Example finding note:

```text
Finding: Insecure Direct Object Reference
Target: /account?id=123
Evidence: Changing id=123 to id=124 returned another user's email address.
Impact: A normal user can access other users' account data.
Recommendation: Enforce server-side object ownership checks.
```

---

## 8. Reporting

A good report should be understandable and actionable.

Each finding should include:

| Section | Purpose |
|---|---|
| Title | Short issue name |
| Severity | Risk level |
| Summary | What the issue is |
| Impact | Why it matters |
| Evidence | Proof from testing |
| Steps to Reproduce | How to verify |
| Remediation | How to fix |
| References | Supporting material |

Severity should consider:

- Exploitability
- Impact
- Required privileges
- Exposure
- Data sensitivity
- Compensating controls

---

## 9. Cleanup

Cleanup depends on the scope and rules of engagement.

Possible cleanup tasks:

- Remove test files
- Remove uploaded proof-of-concept files
- Stop listeners
- Delete temporary accounts if created
- Remove generated artifacts
- Provide hashes or paths of files created
- Confirm cleanup with the system owner

Example cleanup note:

```text
Removed uploaded test file:
/uploads/poc-test.txt

No persistence mechanisms were created.
No production data was modified.
```

---

## Practical Workflow

A simple workflow for labs:

```text
1. Confirm scope
2. Create scan directory
3. Run initial Nmap scan
4. Review exposed services
5. Enumerate web content
6. Inspect requests with Burp Suite
7. Test likely vulnerabilities manually
8. Use tools carefully when useful
9. Confirm impact safely
10. Document findings
11. Clean up artifacts
12. Write summary
```

---

## Tool Mapping

| Phase | Tools |
|---|---|
| Reconnaissance | `curl`, browser, DNS tools |
| Enumeration | `nmap`, `gobuster`, Burp Suite |
| Vulnerability testing | Burp Repeater, manual payloads |
| Exploitation validation | Burp Suite, SQLMap, controlled payloads |
| Shell handling | Netcat, rlwrap, socat |
| Authentication testing | Hydra |
| Documentation | Markdown notes, screenshots, logs |

---

## Rules of Thumb

- Get authorization first.
- Define scope before touching the target.
- Start broad, then go deep.
- Prefer manual understanding before automation.
- Use noisy tools carefully.
- Validate findings safely.
- Document exact evidence.
- Report remediation, not just exploitation.
- Clean up after testing.

---

## Notes to Remember

- Methodology keeps testing consistent.
- Planning reduces risk.
- Enumeration drives good exploitation.
- Exploitation should prove impact safely.
- Documentation is part of the work, not an afterthought.
- Reporting should help someone fix the issue.
- Cleanup protects both the tester and the system owner.