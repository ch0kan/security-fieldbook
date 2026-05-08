# Web Requests

---

## Executive Summary

This cheatsheet is a quick reference for HTTP requests, headers, methods, status codes, cookies, and common `curl` usage.

Use it when testing web applications, reproducing Burp requests, reviewing logs, or troubleshooting web behavior.

## HTTP Methods

| Method | Purpose |
|---|---|
| `GET` | Retrieve a resource |
| `POST` | Submit data |
| `PUT` | Replace or upload a resource |
| `PATCH` | Partially update a resource |
| `DELETE` | Delete a resource |
| `HEAD` | Retrieve headers only |
| `OPTIONS` | Show supported methods |

## Common Status Codes

| Code | Meaning |
|---:|---|
| `200` | OK |
| `201` | Created |
| `204` | No Content |
| `301` | Moved Permanently |
| `302` | Found / Redirect |
| `304` | Not Modified |
| `400` | Bad Request |
| `401` | Unauthorized |
| `403` | Forbidden |
| `404` | Not Found |
| `405` | Method Not Allowed |
| `429` | Too Many Requests |
| `500` | Internal Server Error |
| `502` | Bad Gateway |
| `503` | Service Unavailable |

## Common Headers

| Header | Purpose |
|---|---|
| `Host` | Target hostname |
| `User-Agent` | Client software |
| `Cookie` | Session or tracking data |
| `Authorization` | Auth token or credentials |
| `Content-Type` | Request body format |
| `Accept` | Expected response format |
| `Origin` | Source origin for CORS |
| `Referer` | Previous page |
| `X-Forwarded-For` | Original client IP, often proxy-related |
| `Location` | Redirect destination |
| `Set-Cookie` | Server sets cookie |

## Content Types

| Content-Type | Use |
|---|---|
| `application/x-www-form-urlencoded` | Standard HTML form body |
| `multipart/form-data` | File uploads |
| `application/json` | JSON API request |
| `application/xml` | XML request |
| `text/plain` | Plain text body |

## Basic Curl

GET request:

```bash
curl http://example.com
```

Show headers only:

```bash
curl -I http://example.com
```

Verbose output:

```bash
curl -v http://example.com
```

Follow redirects:

```bash
curl -L http://example.com
```

Save response to file:

```bash
curl -o output.html http://example.com
```

Ignore TLS certificate errors:

```bash
curl -k https://example.com
```

## Curl with Headers

Add header:

```bash
curl -H "Header-Name: value" http://example.com
```

Set User-Agent:

```bash
curl -A "Mozilla/5.0" http://example.com
```

Set Host header:

```bash
curl -H "Host: admin.example.com" http://TARGET_IP
```

Add cookie:

```bash
curl -H "Cookie: session=VALUE" http://example.com
```

Add bearer token:

```bash
curl -H "Authorization: Bearer TOKEN" http://example.com/api
```

## POST Requests

Form POST:

```bash
curl -X POST -d "username=admin&password=test" http://example.com/login
```

JSON POST:

```bash
curl -X POST http://example.com/api \
  -H "Content-Type: application/json" \
  -d '{"name":"test"}'
```

Upload file:

```bash
curl -X POST http://example.com/upload \
  -F "file=@example.txt"
```

PUT upload:

```bash
curl -T file.txt http://example.com/uploads/file.txt
```

## Cookies

Save cookies:

```bash
curl -c cookies.txt http://example.com/login
```

Use cookies:

```bash
curl -b cookies.txt http://example.com/profile
```

Send cookie manually:

```bash
curl -H "Cookie: session=VALUE" http://example.com
```

## Proxying Through Burp

Send curl traffic through Burp:

```bash
curl -x http://127.0.0.1:8080 http://example.com
```

For HTTPS with Burp:

```bash
curl -k -x http://127.0.0.1:8080 https://example.com
```

## Common Test Headers

Forwarded IP:

```http
X-Forwarded-For: 127.0.0.1
```

Real IP:

```http
X-Real-IP: 127.0.0.1
```

Original URL:

```http
X-Original-URL: /admin
```

Rewrite URL:

```http
X-Rewrite-URL: /admin
```

JSON content type:

```http
Content-Type: application/json
```

## Manual Request Example

```http
GET /admin HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Cookie: session=VALUE
Accept: */*
Connection: close
```

## Access Control Request Tests

| Test | Action |
|---|---|
| Remove auth | Delete `Cookie` or `Authorization` header |
| Change user ID | Modify `id`, `user`, or object parameters |
| Change method | Try `GET`, `POST`, `PUT`, `DELETE` |
| Change content type | Try JSON vs form encoding |
| Add proxy headers | Test `X-Forwarded-For` and similar headers |
| Direct path access | Request hidden or admin paths manually |

## Useful Grep Patterns

Search URLs:

```bash
grep -Eo 'https?://[^"]+' file.html
```

Search JavaScript endpoints:

```bash
grep -Eo '["'\''][/A-Za-z0-9._?=&-]+["'\'']' app.js
```

Search for secrets:

```bash
grep -Ei "api|token|secret|key|password|bearer" file.txt
```

## Notes

- `401` usually means authentication is required.
- `403` usually means authentication worked but access is denied.
- `302` after login may indicate success.
- Response length changes can reveal behavior even when status codes match.
- Always compare requests from different user roles when testing access control.