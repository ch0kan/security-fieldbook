# Burp Suite

---

## Executive Summary

Burp Suite is a web application testing proxy used to intercept, inspect, modify, replay, and automate HTTP/S requests.

Use this cheatsheet for quick Burp workflow reminders during web testing.

## Core Tools

| Tool | Purpose |
|---|---|
| Proxy | Intercept and inspect browser traffic |
| Target | Map hosts, paths, parameters, and site structure |
| Repeater | Manually modify and resend requests |
| Intruder | Automate payload injection and fuzzing |
| Decoder | Encode and decode data |
| Comparer | Compare responses or payloads |
| Logger | Review HTTP history |
| Sequencer | Analyze token randomness |
| Collaborator | Detect out-of-band interactions |

## Browser Proxy Setup

Default Burp listener:

```text
127.0.0.1:8080
```

Browser proxy:

```text
HTTP Proxy: 127.0.0.1
Port: 8080
```

Install Burp CA certificate:

```text
http://burp
```

## Proxy Workflow

| Step | Action |
|---|---|
| Configure browser | Set browser proxy to Burp |
| Install CA cert | Allow HTTPS interception |
| Turn intercept on | Capture requests |
| Send to tools | Forward interesting requests to Repeater or Intruder |
| Review history | Check Proxy HTTP history |
| Scope target | Limit noise and avoid unrelated traffic |

## Useful Right-Click Actions

| Action | Use |
|---|---|
| Send to Repeater | Manual testing |
| Send to Intruder | Fuzzing or brute force testing |
| Send to Comparer | Compare responses |
| Do intercept | Intercept matching requests |
| Add to scope | Keep testing focused |
| Copy URL | Save endpoint |
| Copy as curl | Reproduce request in terminal |

## Repeater Workflow

Use Repeater for manual request testing.

Common tests:

- Change HTTP method.
- Modify parameters.
- Remove parameters.
- Change cookies.
- Change headers.
- Replay requests.
- Test access control.
- Test input validation.
- Compare response codes and lengths.

Useful things to watch:

| Signal | Meaning |
|---|---|
| Status code | Access changes, redirects, errors |
| Response length | Different backend behavior |
| Headers | Auth, cache, redirects, server behavior |
| Body content | Error messages, data exposure |
| Timing | Possible blind or time-based behavior |

## Intruder Workflow

Use Intruder for controlled payload automation.

Common attack positions:

```text
GET /user?id=§1§ HTTP/1.1
```

Payload examples:

- IDs
- Usernames
- Passwords
- Filenames
- Paths
- Headers
- Cookies
- JSON values

Common filters:

- Status code
- Response length
- Response time
- Error keywords
- Redirect location
- Grep match

## Common Header Tests

Change Host header:

```http
Host: admin.example.com
```

Add forwarded header:

```http
X-Forwarded-For: 127.0.0.1
```

Spoof client IP:

```http
X-Real-IP: 127.0.0.1
```

Change content type:

```http
Content-Type: application/json
```

Test authorization:

```http
Authorization: Bearer TOKEN
```

## Common Access Control Tests

| Test | Action |
|---|---|
| Horizontal access | Change object ID to another user’s object |
| Vertical access | Use low-privileged user against admin endpoint |
| Missing auth | Remove cookies or tokens |
| Forced browsing | Request hidden/admin paths directly |
| Method change | Try `GET`, `POST`, `PUT`, `DELETE`, `PATCH` |
| Header trust | Test `X-Forwarded-For` and similar headers |

## Decoder

Common decoding tasks:

- URL decode
- Base64 decode
- HTML decode
- Hex decode
- JWT decode
- Gzip decode

Common encoding tasks:

- URL encode
- Base64 encode
- HTML encode
- Hex encode

## Comparer

Use Comparer to check differences between:

- Two user responses.
- Admin and normal user responses.
- Successful and failed login responses.
- Baseline and fuzzed responses.
- Token formats.

## Collaborator

Use Collaborator to detect:

- Blind SSRF.
- Blind XSS.
- XML external entity callbacks.
- Server-side template injection callbacks.
- Out-of-band DNS or HTTP interactions.

Payload idea:

```text
https://COLLABORATOR-ID.oastify.com
```

## Scope Tips

Add only authorized targets to scope.

Useful filters:

- Show only in-scope items.
- Hide images, CSS, fonts, and static files.
- Highlight interesting status codes.
- Filter by MIME type.
- Filter by extension.

## Practical Workflow

| Phase | Action |
|---|---|
| Setup | Configure proxy and certificate |
| Scope | Add authorized hosts |
| Map | Browse application normally |
| Review | Check HTTP history and target map |
| Test manually | Use Repeater |
| Automate carefully | Use Intruder |
| Decode | Use Decoder for tokens and payloads |
| Compare | Use Comparer for response differences |
| OOB testing | Use Collaborator where appropriate |

## Notes

- Keep testing scoped to authorized targets.
- Repeater is usually the most important manual testing tool.
- Response length changes often reveal useful behavior.
- Always test with at least two user roles when checking access control.
- Avoid noisy Intruder attacks unless rate limits and authorization allow them.