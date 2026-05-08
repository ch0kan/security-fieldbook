# Password Cracking

---

## Executive Summary

Password cracking is used to recover plaintext passwords from captured hashes in authorized labs and assessments.

Use this cheatsheet for quick Hashcat and John the Ripper syntax, hash identification, wordlists, and rule-based cracking.

## Common Hash Indicators

| Format | Possible Type |
|---|---|
| 32 hex chars | MD5 or NTLM |
| 40 hex chars | SHA1 |
| 64 hex chars | SHA256 |
| 128 hex chars | SHA512 |
| `$1$` | MD5-crypt |
| `$2a$`, `$2b$`, `$2y$` | bcrypt |
| `$5$` | SHA256-crypt |
| `$6$` | SHA512-crypt |
| `$krb5asrep$` | Kerberos AS-REP |
| `$krb5tgs$` | Kerberos TGS |
| `aad3b435b51404eeaad3b435b51404ee` | Empty LM hash |

## Hashcat Basics

Show help:

```bash
hashcat --help
```

List example hashes:

```bash
hashcat --example-hashes
```

Benchmark:

```bash
hashcat -b
```

Crack with wordlist:

```bash
hashcat -m HASH_MODE -a 0 hashes.txt wordlist.txt
```

Show cracked hashes:

```bash
hashcat -m HASH_MODE hashes.txt --show
```

Remove cracked hashes:

```bash
hashcat -m HASH_MODE hashes.txt --left
```

## Common Hashcat Modes

| Mode | Hash Type |
|---:|---|
| `0` | MD5 |
| `100` | SHA1 |
| `1400` | SHA256 |
| `1700` | SHA512 |
| `1000` | NTLM |
| `3000` | LM |
| `3200` | bcrypt |
| `500` | MD5-crypt |
| `7400` | SHA256-crypt |
| `1800` | SHA512-crypt |
| `13100` | Kerberos 5 TGS-REP |
| `18200` | Kerberos 5 AS-REP |

## Hashcat Attack Modes

| Mode | Type |
|---:|---|
| `0` | Straight wordlist |
| `1` | Combination |
| `3` | Brute force / mask |
| `6` | Wordlist + mask |
| `7` | Mask + wordlist |

## Wordlist Attack

```bash
hashcat -m 1000 -a 0 ntlm.txt /usr/share/wordlists/rockyou.txt
```

With rules:

```bash
hashcat -m 1000 -a 0 ntlm.txt /usr/share/wordlists/rockyou.txt -r rules/best64.rule
```

## Mask Attack

Eight lowercase letters:

```bash
hashcat -m 1000 -a 3 hashes.txt ?l?l?l?l?l?l?l?l
```

Password pattern with year:

```bash
hashcat -m 1000 -a 3 hashes.txt Company?d?d?d?d
```

Common masks:

| Mask | Meaning |
|---|---|
| `?l` | Lowercase letter |
| `?u` | Uppercase letter |
| `?d` | Digit |
| `?s` | Symbol |
| `?a` | All printable characters |
| `?1` | Custom charset |

Custom charset:

```bash
hashcat -m 1000 -a 3 hashes.txt -1 ?l?d ?1?1?1?1?1?1
```

## John the Ripper Basics

Crack with wordlist:

```bash
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Show cracked passwords:

```bash
john hashes.txt --show
```

Use format:

```bash
john hashes.txt --format=NT
```

Use rules:

```bash
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt --rules
```

## John Common Formats

| Format | Use |
|---|---|
| `NT` | NTLM |
| `raw-md5` | MD5 |
| `raw-sha1` | SHA1 |
| `raw-sha256` | SHA256 |
| `raw-sha512` | SHA512 |
| `bcrypt` | bcrypt |
| `krb5tgs` | Kerberos TGS |
| `krb5asrep` | Kerberos AS-REP |

## Archive and File Hash Extraction

ZIP:

```bash
zip2john file.zip > zip.hash
john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

RAR:

```bash
rar2john file.rar > rar.hash
john rar.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

SSH private key:

```bash
ssh2john id_rsa > ssh.hash
john ssh.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

KeePass:

```bash
keepass2john database.kdbx > keepass.hash
john keepass.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

## Wordlists

Common locations:

```text
/usr/share/wordlists/
/usr/share/seclists/Passwords/
/usr/share/seclists/Usernames/
```

RockYou:

```bash
gunzip /usr/share/wordlists/rockyou.txt.gz
```

## Practical Workflow

| Step | Action |
|---|---|
| Identify hash | Check format and length |
| Choose tool | Hashcat for speed, John for flexibility |
| Select mode | Match hash type |
| Start wordlist attack | Use known or common wordlists |
| Add rules | Try mutations and common patterns |
| Try masks | Use target-specific patterns |
| Show results | Export cracked credentials |
| Validate safely | Confirm only where authorized |

## Notes

- Hash type identification can be ambiguous.
- NTLM and MD5 are both 32 hex characters.
- Rules are often more effective than pure brute force.
- Target-specific wordlists usually outperform generic lists.
- Never crack or use hashes outside authorized environments.