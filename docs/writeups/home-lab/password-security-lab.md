# Password Security Lab

---

## Executive Summary

I used this lab to practice password security concepts in a controlled environment.

The goal was to understand how password hashes are stored, how weak passwords can be tested offline in a lab, and what defensive controls make credential attacks harder.

## Lab Objective

I performed this lab to practice:

- Identifying common Windows and Linux hash formats.
- Understanding the difference between unsalted and salted password hashes.
- Testing password cracking workflows safely.
- Comparing weak passwords against stronger passphrases.
- Reviewing the defensive value of password length.
- Documenting password security lessons and mitigations.

## Environment

| System | Role |
|---|---|
| Kali VM | Password testing and cracking tools |
| Windows Client VM | Lab system for Windows hash concepts |
| Linux Server VM | Lab system for Linux hash concepts |
| Analysis VM | Notes, evidence review, and documentation |

## Network Placement

| System | Network | Example IP |
|---|---|---|
| Kali VM | Attack Network | `10.10.20.10` |
| Windows Client VM | Lab LAN | `10.10.30.10` |
| Linux Server VM | Lab LAN | `10.10.30.20` |
| Analysis VM | Logging Network | `10.10.50.10` |

## Scenario I Simulated

I treated this as a controlled password audit inside my lab.

The purpose was not to attack real accounts. I used lab-created test users, sample hashes, and known test passwords to understand how password storage and cracking workflows behave.

## Tools I Used

| Tool | How I Used It |
|---|---|
| John the Ripper | Tested basic password cracking workflows |
| Hashcat | Tested hash identification and cracking modes |
| Linux shell tools | Reviewed hash files and formats |
| PowerShell | Documented Windows-side concepts and evidence |
| Wordlists | Tested dictionary-based cracking in a controlled way |

## Password Hashing Concepts I Practiced

I reviewed the difference between storing passwords and storing hashes.

A password hash is a one-way representation of a password. During login, the system hashes the submitted password and compares it to the stored hash.

The main concepts I focused on were:

| Concept | Why It Matters |
|---|---|
| Hashing | Prevents plaintext password storage |
| Salting | Prevents identical passwords from producing identical hashes |
| Slow hashing | Makes large-scale cracking harder |
| Password length | Increases cracking difficulty |
| MFA | Reduces the value of a cracked password |

## Windows Hash Concepts

I reviewed Windows hash formats and focused on why legacy formats are weak.

Important points I documented:

| Format | Notes |
|---|---|
| LM hash | Legacy, weak, and should be disabled |
| NT hash | Still widely relevant in Windows environments |
| Empty LM value | Often appears as `aad3b435b51404eeaad3b435b51404ee` |
| Empty NT value | Often appears as `31d6cfe0d16ae931b73c59d7e0c089c0` |

I also documented the common Windows hash dump format:

```text
Username:RID:LM_Hash:NT_Hash:::
```

## Linux Hash Concepts

I reviewed Linux password hash formats from `/etc/shadow`.

Linux shadow hashes commonly use a `$id$salt$hash` structure.

Example format:

```text
user:$6$saltvalue$hashvalue:...
```

Common identifiers:

| ID | Algorithm |
|---|---|
| `$1` | MD5 |
| `$2` | Blowfish |
| `$5` | SHA-256 |
| `$6` | SHA-512 |

The key takeaway was that Linux hashes normally include salts, which makes precomputed cracking attacks much less effective.

## Test Hash File

For the lab, I used sample/test hashes only.

Example lab file:

```text
testuser1:1000:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::
testuser2:1001:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

I stored lab hashes in a dedicated working directory:

```bash
mkdir -p ~/lab/password-security
cd ~/lab/password-security
```

## John the Ripper Workflow

I used John the Ripper for a basic cracking workflow.

Example command for Windows NT hashes:

```bash
john --format=NT hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Show cracked results:

```bash
john --show --format=NT hashes.txt
```

I also noted that John stores cracked results in a pot file, so I kept the lab environment isolated and avoided using real credentials.

## Hashcat Workflow

I used Hashcat to practice controlled cracking modes.

Example NT hash mode:

```bash
hashcat -m 1000 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

Show cracked results:

```bash
hashcat -m 1000 hashes.txt --show
```

Example mask-style test:

```bash
hashcat -m 1000 -a 3 hashes.txt ?u?l?l?l?l?d?d
```

## Attack Modes I Practiced

| Mode | Tool Concept | What I Tested |
|---|---|---|
| Dictionary | Wordlist-based guesses | Common weak passwords |
| Mask | Pattern-based guessing | Predictable password formats |
| Hybrid | Wordlist plus suffix/prefix | Common human password patterns |
| Rules | Word mutation | Capitalization and substitutions |

## What I Observed

During the lab, I observed that weak and predictable passwords were much easier to recover.

The patterns that performed poorly included:

- Short passwords.
- Common words.
- Seasonal passwords.
- Passwords with predictable numbers.
- Capitalized dictionary words.
- Simple substitutions such as `a` to `@` or `e` to `3`.

The stronger examples were longer passphrases with less predictable structure.

## Defensive Lessons

The lab reinforced several defensive lessons.

| Defensive Control | Why It Helps |
|---|---|
| Disable LM hashes | Removes a weak legacy format |
| Use long passphrases | Increases cracking difficulty |
| Avoid forced predictable resets | Reduces seasonal password patterns |
| Block common passwords | Prevents easy dictionary wins |
| Use MFA | Reduces password-only compromise risk |
| Protect LSASS and credential stores | Reduces hash theft opportunities |
| Monitor credential dumping behavior | Improves detection of attacks before cracking |

## Detection Opportunities

This lab gave me detection ideas for:

- Credential dumping attempts.
- Access to sensitive files such as SAM, SYSTEM, or shadow equivalents.
- Suspicious use of tools that interact with LSASS.
- Unexpected copying of credential databases.
- Large outbound transfers of archive files.
- Password audit tools running on non-administrative systems.
- Repeated failed authentication attempts after cracking attempts.

## Evidence I Collected

| Artifact | Why I Collected It |
|---|---|
| Test hash files | Documented sample data used in the lab |
| Tool commands | Documented cracking workflow |
| Cracking results | Compared weak and stronger passwords |
| Screenshots | Supported documentation |
| Notes on formats | Captured Windows and Linux hash differences |

## Findings Template

I used this format to document password audit results.

```text
Finding:
A weak test password was recovered during the lab audit.

Evidence:
- Hash type:
- Test account:
- Tool used:
- Attack mode:
- Wordlist or mask:
- Time to recover:
- Password pattern:

Assessment:
Explain why this password was weak.

Defensive Recommendation:
- Increase password length.
- Block common passwords.
- Use MFA.
- Monitor credential access.
- Disable legacy hash storage.
```

## Lessons Learned

This lab helped me understand that password strength is not just about complexity.

The most important lessons were:

- Length matters more than simple symbol substitution.
- Unsalted hashes are easier to attack at scale.
- Weak password patterns are predictable.
- Password cracking is usually performed offline after credential material is stolen.
- MFA is critical because passwords alone are not enough.
- Defenders should focus on both preventing hash theft and reducing crackability.

## Skills Demonstrated

This lab demonstrates practical skills in:

- Password hash format recognition.
- Safe password audit workflows.
- John the Ripper usage.
- Hashcat usage.
- Wordlist and mask testing.
- Defensive password policy analysis.
- Credential attack detection planning.
- Technical documentation.