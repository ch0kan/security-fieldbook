# Web Attack Simulation Lab

---

## Executive Summary

I used this lab to practice safe web attack simulation against a local target inside my home lab.

The goal was to understand how common web attack patterns appear from both the attacker and defender perspective. I tested requests against a local Web App VM, reviewed the responses, and checked related server and network logs.

## Lab Objective

I performed this lab to practice:

- Inspecting HTTP requests and responses.
- Testing common web attack patterns safely.
- Using Burp Suite to intercept and modify requests.
- Reviewing web server logs.
- Correlating web requests with pfSense traffic.
- Documenting web findings and detection opportunities.

## Environment

| System | Role |
|---|---|
| Kali VM | Testing system with browser, Burp Suite, and web tools |
| Web App VM | Local web application target |
| pfSense VM | Firewall/router and network visibility point |
| Analysis VM | Optional packet capture and log review system |

## Network Placement

| System | Network | Example IP |
|---|---|---|
| Kali VM | Attack Network | `10.10.20.10` |
| Web App VM | Lab LAN | `10.10.30.30` |
| Analysis VM | Logging Network | `10.10.50.10` |
| pfSense Lab LAN Gateway | Lab LAN | `10.10.30.1` |

## Scenario I Simulated

I treated the Web App VM as an intentionally vulnerable local target.

I used the Kali VM to send controlled web requests, inspect application behavior, and review how those requests appeared in logs.

All testing was limited to my local lab systems.

## Tools I Used

| Tool | How I Used It |
|---|---|
| Browser | Manual application testing |
| Burp Suite | Intercepted and modified HTTP requests |
| curl | Sent simple repeatable HTTP requests |
| Nmap | Validated exposed web services |
| Gobuster | Tested controlled content discovery |
| Web server logs | Reviewed request evidence |
| pfSense logs | Validated network traffic |
| Wireshark | Optional packet review |

## Service Discovery

I started by confirming that the Web App VM was reachable from Kali.

```bash
ping 10.10.30.30
```

I checked for open web ports.

```bash
nmap -sV -p 80,443,8080 10.10.30.30
```

I documented the exposed services.

| Host | Port | Service | Notes |
|---|---:|---|---|
| `10.10.30.30` | 80 | HTTP |  |
| `10.10.30.30` | 443 | HTTPS |  |
| `10.10.30.30` | 8080 | HTTP alternate |  |

## HTTP Baseline

Before testing, I created a baseline of normal application behavior.

```bash
curl -i http://10.10.30.30/
```

I reviewed:

- Status code.
- Server header.
- Cookies.
- Redirects.
- Forms.
- Input fields.
- Authentication pages.
- Interesting paths.

## Burp Suite Setup

I used Burp Suite to intercept browser traffic.

My workflow was:

| Step | Action |
|---|---|
| 1 | Started Burp Suite on Kali |
| 2 | Configured the browser proxy to `127.0.0.1:8080` |
| 3 | Visited the Web App VM |
| 4 | Intercepted requests |
| 5 | Sent interesting requests to Repeater |
| 6 | Modified parameters safely |
| 7 | Documented request and response behavior |

## Content Discovery

I performed controlled content discovery against the local target.

```bash
gobuster dir -u http://10.10.30.30/ -w /usr/share/wordlists/dirb/common.txt
```

I documented interesting paths.

| Path | Status Code | Notes |
|---|---:|---|
| `/login` |  |  |
| `/admin` |  |  |
| `/uploads` |  |  |
| `/api` |  |  |

## Input Testing

I reviewed application input points.

Examples:

| Input Point | Example |
|---|---|
| URL parameter | `/item?id=1` |
| Search field | `q=test` |
| Login form | username/password |
| File upload | uploaded file name and type |
| API parameter | JSON body or query parameter |

For each input, I documented:

- Request method.
- Parameter name.
- Normal value.
- Modified value.
- Application response.
- Server-side log entry.

## SQL Injection Simulation

I tested SQL injection patterns only against the local vulnerable application.

Simple test payloads:

```text
'
```

```text
" 
```

```text
1 OR 1=1
```

I watched for:

- SQL error messages.
- Authentication bypass behavior.
- Different response sizes.
- Different status codes.
- Unexpected data exposure.
- Server log errors.

## Command Injection Simulation

I tested command injection behavior only where the lab application intentionally accepted OS-like input.

Simple separators I tested:

```text
;
```

```text
&&
```

```text
|
```

I focused on identifying whether input affected server-side command execution.

I documented:

| Field | Notes |
|---|---|
| Input tested |  |
| Payload pattern |  |
| Response change |  |
| Server log entry |  |
| Risk |  |

## Path Traversal Simulation

I tested path traversal patterns against local file-related parameters.

Example patterns:

```text
../
```

```text
../../../../etc/passwd
```

```text
..\..\..\..\windows\win.ini
```

I watched for:

- File disclosure.
- Error messages.
- Path normalization.
- Access denied messages.
- Differences between Linux and Windows style paths.

## SSRF Simulation

I tested SSRF-style behavior only if the lab application had a URL-fetching feature.

Example safe targets:

```text
http://127.0.0.1/
```

```text
http://10.10.30.30/
```

```text
http://10.10.30.20/
```

I documented whether the server attempted outbound requests and whether pfSense or server logs showed the traffic.

## File Upload Simulation

If the application had upload functionality, I tested basic upload controls.

I reviewed:

- Allowed file extensions.
- MIME type checks.
- File size limits.
- Upload directory behavior.
- Whether uploaded files were executable.
- Whether the application renamed files.
- Whether direct access to uploaded files was possible.

I documented:

| Test | Result |
|---|---|
| Upload normal image |  |
| Upload renamed text file |  |
| Upload blocked extension |  |
| Access uploaded file directly |  |

## Access Control Testing

I tested whether pages and functions were properly restricted.

Examples:

| Test | What I Checked |
|---|---|
| Direct browsing | Accessing a protected page without login |
| Role change | Accessing admin functionality as a normal user |
| ID change | Changing object IDs in a request |
| Session logout | Reusing session after logout |
| Forced browsing | Accessing hidden paths directly |

## Web Server Log Review

On the Web App VM, I reviewed server logs.

Apache:

```bash
sudo tail -f /var/log/apache2/access.log
```

```bash
sudo tail -f /var/log/apache2/error.log
```

Nginx:

```bash
sudo tail -f /var/log/nginx/access.log
```

```bash
sudo tail -f /var/log/nginx/error.log
```

I reviewed:

- Source IP.
- Timestamp.
- HTTP method.
- Requested path.
- Status code.
- User-Agent.
- Referrer.
- Repeated failed paths.
- Suspicious parameters.

## pfSense Validation

I used pfSense to confirm traffic between the Attack Network and Lab LAN.

I checked:

```text
Status > System Logs > Firewall
```

I reviewed:

| Field | Why I Checked It |
|---|---|
| Source IP | Confirmed traffic came from Kali |
| Destination IP | Confirmed traffic reached the Web App VM |
| Destination port | Confirmed HTTP/HTTPS service |
| Timestamp | Correlated with Burp and server logs |
| Action | Confirmed whether the firewall allowed or blocked the traffic |

## Evidence I Collected

| Artifact | Why I Collected It |
|---|---|
| Burp requests | Showed exact request and parameters |
| Burp responses | Showed application behavior |
| curl output | Provided repeatable request evidence |
| Web server access logs | Confirmed server-side request details |
| Web server error logs | Captured application or SQL errors |
| pfSense logs | Confirmed network path |
| Screenshots | Supported documentation |

## Findings Template

I used this structure to document web findings.

```text
Finding:
A web application weakness was identified in the local lab target.

Evidence:
- Target:
- URL:
- Parameter:
- HTTP method:
- Test payload:
- Response status:
- Response behavior:
- Server log evidence:
- pfSense log evidence:

Assessment:
Explain why the behavior matters.

Recommendation:
- Validate input.
- Enforce access control server-side.
- Avoid shell execution with user input.
- Use parameterized queries.
- Restrict file uploads.
- Log suspicious requests.
```

## Detection Opportunities

This lab gave me detection ideas for:

- Repeated 404 responses from content discovery.
- SQL error messages in application logs.
- Suspicious characters in query strings.
- Requests containing traversal patterns.
- Web requests with tool-like User-Agents.
- Unexpected server-side outbound requests.
- File upload attempts with risky extensions.
- Authentication bypass attempts.
- Access to admin paths from unauthorized users.

## Lessons Learned

This lab helped me connect web testing with defensive evidence.

The most useful workflow was:

1. Send a controlled request from Kali.
2. Capture and modify it in Burp.
3. Observe the application response.
4. Review web server logs.
5. Validate the network path in pfSense.
6. Document the finding and detection idea.

I learned that the request itself is only part of the story. The strongest documentation comes from correlating client-side testing, server-side logs, and network evidence.

## Skills Demonstrated

This lab demonstrates practical skills in:

- Web application testing.
- HTTP request analysis.
- Burp Suite usage.
- Content discovery.
- Input validation testing.
- Web log review.
- Firewall log correlation.
- Evidence collection.
- Detection planning.
- Technical documentation.