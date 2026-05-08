# Encoding and Hashing

---

## Executive Summary

Encoding transforms data into another representation. Hashing produces a fixed-length fingerprint of data. Encryption protects data with a key.

This cheatsheet is for quick lookup when handling files, payloads, hashes, certificates, tokens, and encoded strings.

## Encoding vs Hashing vs Encryption

| Concept | Reversible | Key Required | Common Use |
|---|---:|---:|---|
| Encoding | Yes | No | Data formatting and transport |
| Hashing | No | No | Integrity checks and password storage |
| Encryption | Yes | Yes | Confidentiality |

## Base64

Encode a file:

```bash
base64 file.bin > file.b64
```

Encode without line wrapping:

```bash
base64 -w 0 file.bin
```

Decode:

```bash
base64 -d file.b64 > file.bin
```

Encode a string:

```bash
echo -n "text" | base64
```

Decode a string:

```bash
echo "dGV4dA==" | base64 -d
```

PowerShell encode:

```powershell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("text"))
```

PowerShell decode:

```powershell
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("dGV4dA=="))
```

## Hex

String to hex:

```bash
echo -n "text" | xxd -p
```

Hex to string:

```bash
echo "74657874" | xxd -r -p
```

Hex dump a file:

```bash
xxd file.bin
```

## URL Encoding

URL encode with Python:

```bash
python3 -c 'import urllib.parse; print(urllib.parse.quote("admin@test.com"))'
```

URL decode with Python:

```bash
python3 -c 'import urllib.parse; print(urllib.parse.unquote("admin%40test.com"))'
```

Common characters:

| Character | Encoded |
|---|---|
| Space | `%20` |
| `"` | `%22` |
| `#` | `%23` |
| `%` | `%25` |
| `&` | `%26` |
| `'` | `%27` |
| `/` | `%2F` |
| `:` | `%3A` |
| `=` | `%3D` |
| `?` | `%3F` |
| `@` | `%40` |

## Hashing Files

MD5:

```bash
md5sum file.bin
```

SHA1:

```bash
sha1sum file.bin
```

SHA256:

```bash
sha256sum file.bin
```

SHA512:

```bash
sha512sum file.bin
```

PowerShell hash:

```powershell
Get-FileHash .\file.bin -Algorithm SHA256
```

## Hashing Strings

MD5:

```bash
echo -n "password" | md5sum
```

SHA256:

```bash
echo -n "password" | sha256sum
```

Python SHA256:

```bash
python3 -c 'import hashlib; print(hashlib.sha256(b"password").hexdigest())'
```

## Identify Hash Lengths

| Hash | Hex Length |
|---|---:|
| MD5 | 32 |
| SHA1 | 40 |
| SHA256 | 64 |
| SHA512 | 128 |
| NTLM | 32 |
| bcrypt | Variable, often starts with `$2a$`, `$2b$`, or `$2y$` |
| SHA-crypt | Starts with `$5$` or `$6$` |

## Common Password Hash Indicators

| Format | Meaning |
|---|---|
| `$1$` | MD5-crypt |
| `$2a$`, `$2b$`, `$2y$` | bcrypt |
| `$5$` | SHA256-crypt |
| `$6$` | SHA512-crypt |
| `$krb5asrep$` | Kerberos AS-REP hash |
| `$krb5tgs$` | Kerberos TGS hash |
| `aad3b435b51404eeaad3b435b51404ee` | Empty LM hash |
| 32 hex chars | Could be MD5 or NTLM |

## OpenSSL Encryption

Encrypt file:

```bash
openssl enc -aes256 -pbkdf2 -iter 100000 -in file.txt -out file.enc
```

Decrypt file:

```bash
openssl enc -d -aes256 -pbkdf2 -iter 100000 -in file.enc -out file.txt
```

Generate random bytes:

```bash
openssl rand -hex 16
```

Generate random base64:

```bash
openssl rand -base64 32
```

## Certificates

View certificate details:

```bash
openssl x509 -in cert.pem -text -noout
```

Check remote TLS certificate:

```bash
openssl s_client -connect example.com:443
```

Show certificate dates:

```bash
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
```

Show certificate subject:

```bash
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -subject -issuer
```

## JWT Quick Checks

JWTs usually have three dot-separated parts:

```text
header.payload.signature
```

Decode JWT header:

```bash
echo 'HEADER_PART' | base64 -d
```

Decode JWT payload:

```bash
echo 'PAYLOAD_PART' | base64 -d
```

If padding is missing, add `=` characters until decoding works.

## File Integrity Workflow

| Step | Command |
|---|---|
| Hash original | `sha256sum file.bin` |
| Transfer file | Use chosen transfer method |
| Hash received copy | `sha256sum file.bin` |
| Compare hashes | Hashes should match exactly |

## Notes

- Base64 is not encryption.
- Hashes are not reversible.
- Fast hashes like MD5 and SHA1 are weak for password storage.
- Use SHA256 or stronger for integrity checks.
- Use authenticated encryption or trusted secure transfer methods for sensitive data.