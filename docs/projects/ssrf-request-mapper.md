# Project 1: SSRF Request Mapper

---

## Executive Summary

I built **SSRF Request Mapper** to demonstrate how Server-Side Request Forgery works in a controlled local environment.

The project includes a deliberately vulnerable-style Flask URL preview feature, a Python mapper script, a YAML payload list, a generated Markdown report, and mitigation notes explaining why naive SSRF filtering is not enough.

The core idea behind the project was simple:

> SSRF is not about whether a user can fetch a URL. It is about what the server can reach on the user's behalf.

## Repository

[View the GitHub repository](https://github.com/ch0kan/ssrf-request-mapper)

## Project Type

| Field | Value |
|---|---|
| Category | Offensive / Secure Coding |
| Language | Python |
| Framework | Flask |
| Output | Tool + report + mitigation notes |
| Scope | Local legal test environment |
| Repository | `ssrf-request-mapper` |

## The Problem

Server-Side Request Forgery is dangerous because a vulnerable application can be abused to make requests from the server side, not the attacker’s browser.

I wanted to demonstrate that I understood the underlying risk: the attacker is not just “fetching a URL,” they are testing what the server itself can reach.

## What I Built

I built a small Flask application with a URL preview feature.

The preview endpoint accepted a URL like this:

```text
http://127.0.0.1:5000/preview?url=http://example.com
```

Then I wrote a Python script that sent controlled test cases to the preview endpoint and documented how the application responded when asked to fetch:

- External URLs
- Localhost addresses
- Private IP ranges
- Blocked ports
- Invalid schemes
- Redirect-style scenarios
- DNS names resolving to private addresses

The mapper generated a Markdown report showing each payload, response behavior, HTTP status code, and whether the server attempted the request.

## Repository Structure

```text
ssrf-request-mapper/
├── vulnerable_app/
│   ├── app.py
│   └── requirements.txt
├── mapper/
│   ├── ssrf_mapper.py
│   └── payloads.yml
├── reports/
│   └── ssrf-test-report.md
├── screenshots/
├── README.md
└── mitigations.md
```

## Example Result

```text
Target:
http://127.0.0.1:5000/preview

Test:
http://127.0.0.1:8000/admin

Result:
Blocked

Reason:
Blocked internal or unsafe address: 127.0.0.1

Evidence:
- Payload category: Loopback
- HTTP status: 403
- Response: "Blocked internal or unsafe address"
```

## Why This Was Not Trivial

The main technical challenge was that SSRF defense is not just a string-matching problem.

My first version blocked URLs containing obvious strings such as:

```text
localhost
127.0.0.1
```

That was weak because attackers can change how a destination is represented.

I improved the logic by:

- Parsing the URL
- Resolving hostnames
- Checking the resolved IP address
- Blocking private, loopback, link-local, reserved, multicast, and unspecified addresses
- Restricting allowed schemes
- Disabling automatic redirects

The important design decision was where validation should happen:

```text
Weak approach:
Block obvious strings before making the request.

Better approach:
Parse the URL, resolve the host, inspect the resolved IP, enforce an allowlist, and limit redirects.
```

This showed the difference between filtering input superficially and understanding the network behavior behind the vulnerability.

## Redirect Handling

One of the most useful lessons was redirect handling.

A URL can look safe at first, but redirect the server toward an internal target.

Example risk:

```text
Initial URL:
http://allowed-looking.example/redirect

Final destination:
http://127.0.0.1:8000/admin
```

I disabled redirects in the demo application so an allowed-looking URL could not silently redirect the server toward an internal destination.

## Cloud Metadata Awareness

I did not query real cloud metadata services.

I included a metadata-style link-local address, `169.254.169.254`, only as a blocked local test case because cloud metadata SSRF is where this vulnerability class can become especially severe.

The purpose was to show awareness of the real-world impact while keeping the project safe and local.

## Skills Demonstrated

- Python HTTP request automation with `requests`
- Flask application development for controlled security testing
- URL parsing and hostname resolution
- Private, loopback, link-local, reserved, multicast, and unspecified IP range validation
- SSRF payload design in a legal local environment
- Markdown report generation
- Secure input validation and allowlist-based defense

## Deliverables

| Deliverable | Description |
|---|---|
| Flask application | Local URL preview feature used for controlled SSRF testing |
| Python mapper | Sends safe test cases to the preview endpoint |
| YAML payload file | Stores controlled SSRF test cases |
| Markdown report | Documents allowed and blocked behavior |
| Mitigation notes | Explains what I fixed and why |
| README | Explains setup, usage, scope, and lessons learned |

## Honest Scope

This project did not test real third-party applications or real cloud metadata services. I kept all testing inside my own local environment.

Limitations:

- It did not cover every possible URL parser edge case.
- It did not test real cloud metadata endpoints.
- It did not implement enterprise-grade egress filtering.
- It did not prove that all SSRF bypasses were prevented.
- It focused on HTTP/HTTPS behavior, not every possible protocol.
- It was a controlled demonstration of SSRF mechanics and mitigation, not a full web security scanner.

## Lessons Learned

The main lesson was:

```text
Do not ask, "Does this URL look safe?"
Ask, "Where will the server actually connect?"
```

This project helped me understand why SSRF validation should be based on the server-side destination, not just the original user-supplied string.