# Burp Suite

Burp Suite is a web application security testing platform used to intercept, inspect, modify, and replay HTTP and HTTPS traffic.

It is one of the core tools for testing authentication, access control, input validation, file uploads, APIs, sessions, and many common web vulnerabilities.

!!! warning
    Use Burp Suite only on systems you own or have explicit permission to test.

---

## Overview

Burp Suite works as an intercepting proxy.

```text
Browser -> Burp Suite -> Target Web Server
Browser <- Burp Suite <- Target Web Server
```

This lets you:

- View HTTP requests and responses
- Modify requests before sending them
- Replay requests
- Test parameters manually
- Inspect cookies and headers
- Map application endpoints
- Fuzz inputs
- Compare server responses
- Analyze authentication and authorization behavior

---

## Core Tools

| Tool | Purpose |
|---|---|
| Proxy | Intercept browser traffic |
| Target | View site map and scope |
| Repeater | Modify and resend individual requests |
| Intruder | Automate payload-based testing |
| Decoder | Decode and encode data |
| Comparer | Compare requests or responses |
| Logger / HTTP history | Review captured traffic |
| Dashboard | View scan and issue activity |

---

## Proxy

The Proxy tool intercepts HTTP and HTTPS traffic between the browser and the application.

Common uses:

- Capture login requests
- Modify parameters
- Inspect cookies
- Change headers
- Observe redirects
- Test client-side controls
- Send requests to Repeater or Intruder

Typical workflow:

1. Configure browser to use Burp as proxy.
2. Visit the target application.
3. Capture traffic in Proxy.
4. Review requests in HTTP history.
5. Send interesting requests to Repeater.

---

## HTTP History

HTTP history is often more useful than live interception.

It lets you review:

- All requests
- All responses
- Status codes
- Request methods
- Parameters
- Cookies
- Headers
- Response lengths
- API calls

Things to look for:

```text
/admin
/api/
/upload
/download
/id=
/user/
/account
/debug
```

---

## Target and Scope

The Target tab shows a site map of discovered paths.

Use scope to avoid accidentally testing unrelated systems.

Scope helps:

- Filter noise
- Keep testing organized
- Avoid third-party domains
- Focus Burp tools on the approved target

Good practice:

```text
Only include domains and IPs that are explicitly in scope.
```

---

## Repeater

Repeater is used to manually modify and resend individual requests.

It is useful for testing:

- SQL injection
- Access control
- IDOR
- SSRF
- File upload behavior
- Header manipulation
- Authentication logic
- Parameter tampering
- API behavior

Example workflow:

1. Capture request in Proxy.
2. Send request to Repeater.
3. Modify one value.
4. Send request.
5. Compare the response.
6. Repeat with controlled changes.

---

## Example: Parameter Testing

Original request:

```http
GET /account?id=123 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

Modified request:

```http
GET /account?id=124 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

If the response shows another user's account, this may indicate IDOR or broken access control.

---

## Example: SQL Injection Testing

Original request:

```http
GET /product?id=1 HTTP/1.1
Host: example.com
```

Test request:

```http
GET /product?id=1' HTTP/1.1
Host: example.com
```

Look for:

- SQL errors
- Response differences
- Status code changes
- Content-length changes
- Timing differences

---

## Intruder

Intruder automates sending many payloads to one or more positions in a request.

Common uses:

- Fuzzing parameters
- Testing usernames
- Testing IDs
- Testing hidden paths
- Checking input validation
- Enumerating values
- Testing rate limits in labs

!!! warning
    Intruder can generate many requests. Use it carefully and only where authorized.

---

## Intruder Positions

Intruder uses payload positions to define where values will be inserted.

Example:

```http
GET /account?id=§123§ HTTP/1.1
Host: example.com
Cookie: session=abc123
```

The marked value will be replaced by payloads.

Example payload list:

```text
1
2
3
4
5
```

---

## Decoder

Decoder converts data between formats.

Common tasks:

- URL decode
- URL encode
- Base64 decode
- Base64 encode
- HTML decode
- Hex decode
- Hash identification hints

Examples:

```text
%2f -> /
YWRtaW46YWRtaW4= -> admin:admin
```

Useful when analyzing:

- Cookies
- Tokens
- Encoded parameters
- Basic Auth headers
- URL-encoded payloads

---

## Comparer

Comparer helps identify differences between two requests or responses.

Useful for testing:

- User vs admin responses
- Valid vs invalid input
- True vs false SQLi conditions
- Successful vs failed login
- Access allowed vs access denied

Example:

```text
Response A: id=123
Response B: id=124
```

Compare:

- Status code
- Content length
- Error messages
- Reflected values
- Hidden fields
- Role-specific data

---

## Common Testing Workflows

### Access Control

1. Log in as a low-privileged user.
2. Capture sensitive requests.
3. Change object IDs.
4. Try admin paths.
5. Replay requests in Repeater.
6. Compare with expected authorization behavior.

### File Upload

1. Capture upload request.
2. Modify filename.
3. Modify extension.
4. Modify `Content-Type`.
5. Test file contents.
6. Check where the upload is stored.
7. Verify whether execution is possible in a lab.

### SSRF

1. Find a URL-fetching parameter.
2. Send request to Repeater.
3. Change URL to localhost or controlled callback.
4. Observe response or out-of-band interaction.
5. Test safely within scope.

### SQL Injection

1. Identify parameters.
2. Add quotes or boolean conditions.
3. Compare responses.
4. Use timing tests if needed.
5. Use SQLMap only when authorized.

---

## Useful Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + Shift + D` | Dashboard |
| `Ctrl + Shift + T` | Target |
| `Ctrl + Shift + P` | Proxy |
| `Ctrl + Shift + I` | Intruder |
| `Ctrl + Shift + R` | Repeater |
| `Ctrl + R` | Send request to Repeater |
| `Ctrl + I` | Send request to Intruder |
| `Ctrl + G` | Send request in Repeater |

!!! note
    On macOS, many shortcuts use `Command` instead of `Ctrl`.

---

## Capture-and-Analyze Workflow

A fast Burp workflow:

```text
Proxy -> Capture request
Ctrl + R -> Send to Repeater
Ctrl + Shift + R -> Jump to Repeater
Modify request
Ctrl + G -> Send request
Compare response
```

This workflow is useful for most manual web testing.

---

## What to Look For

When reviewing traffic, look for:

- IDs in parameters
- Role values
- Cookies
- Tokens
- Hidden API routes
- Debug endpoints
- File upload endpoints
- Redirects
- Verbose errors
- Sensitive response data
- Authorization headers
- Insecure cookie flags
- Missing security headers

---

## Useful Headers to Inspect

| Header | Why It Matters |
|---|---|
| `Cookie` | Session state |
| `Authorization` | Bearer tokens or Basic Auth |
| `Host` | Virtual host routing |
| `Content-Type` | Body parsing behavior |
| `Referer` | Possible data leakage |
| `Origin` | CORS behavior |
| `X-Forwarded-For` | Proxy/client IP handling |
| `Set-Cookie` | Cookie security attributes |

---

## Cookie Review

Check whether session cookies have secure attributes.

Example:

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax
```

Important attributes:

| Attribute | Purpose |
|---|---|
| `HttpOnly` | Prevents JavaScript from reading cookie |
| `Secure` | Sends cookie only over HTTPS |
| `SameSite` | Controls cross-site cookie behavior |

---

## Good Notes to Keep

When testing with Burp, document:

- Target URL
- User role
- Request path
- Parameter tested
- Original request
- Modified request
- Response difference
- Evidence screenshot or copied response
- Impact
- Remediation idea

Example note:

```text
Role: standard user
Request: GET /account?id=123
Modified: GET /account?id=124
Result: returned another user's email address
Issue: IDOR / broken access control
```

---

## Quick Reference

| Goal | Burp Tool |
|---|---|
| Capture traffic | Proxy |
| Review all requests | HTTP History |
| Map application | Target |
| Modify and replay | Repeater |
| Fuzz parameters | Intruder |
| Decode values | Decoder |
| Compare responses | Comparer |
| Review cookies | Proxy / Inspector |
| Test APIs | Repeater |

---

## Notes to Remember

- Burp is an intercepting proxy.
- Repeater is the main manual testing tool.
- HTTP history is often better than intercepting every request.
- Scope prevents accidental testing of unrelated systems.
- Intruder can be noisy.
- Decoder helps analyze encoded values.
- Always compare responses carefully.
- Document exact requests and responses as evidence.