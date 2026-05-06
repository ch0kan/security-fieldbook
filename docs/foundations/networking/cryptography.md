# Cryptography

Cryptography is the practice of protecting information using mathematical techniques. It is used to provide confidentiality, integrity, authentication, and non-repudiation in modern systems.

Cryptography appears everywhere in cybersecurity: HTTPS, SSH, VPNs, password storage, digital signatures, certificates, file integrity checks, and secure messaging.

---

## Core Concepts

| Term | Meaning |
|---|---|
| Plaintext | Original readable data before encryption |
| Ciphertext | Scrambled unreadable data after encryption |
| Cipher | Algorithm used to transform plaintext and ciphertext |
| Key | Secret value used by the cryptographic algorithm |
| Encryption | Converting plaintext into ciphertext |
| Decryption | Converting ciphertext back into plaintext |

Simple flow:

```text
Plaintext + Cipher + Key -> Ciphertext
Ciphertext + Cipher + Key -> Plaintext
```

---

## Security Goals

Cryptography supports several security goals.

| Goal | Description |
|---|---|
| Confidentiality | Prevent unauthorized reading of data |
| Integrity | Detect whether data was changed |
| Authentication | Verify identity or origin |
| Non-repudiation | Prevent a sender from denying an action |
| Key exchange | Securely agree on shared secrets |

---

## Symmetric Encryption

Symmetric encryption uses the **same key** for encryption and decryption.

```text
Shared key encrypts data
Shared key decrypts data
```

Examples:

| Algorithm | Status |
|---|---|
| DES | Broken / obsolete |
| 3DES | Deprecated / legacy |
| AES | Modern standard |

### Advantages

- Fast
- Efficient for large data
- Common for disk, file, and network encryption

### Disadvantages

- Key distribution is difficult
- Anyone with the key can decrypt the data
- Key compromise exposes protected data

### Common Use

Symmetric encryption is often used for bulk data encryption after a secure key exchange.

Example:

```text
TLS uses asymmetric methods to agree on keys,
then symmetric encryption protects the session data.
```

---

## Asymmetric Encryption

Asymmetric encryption uses a **key pair**:

| Key | Purpose |
|---|---|
| Public key | Shared publicly |
| Private key | Kept secret |

Data encrypted with one key can only be decrypted with the matching key.

### Advantages

- Easier key distribution
- Enables digital signatures
- Useful for authentication and key exchange

### Disadvantages

- Slower than symmetric encryption
- Requires larger key sizes
- Depends heavily on private key protection

### Common Algorithms

| Algorithm | Use |
|---|---|
| RSA | Encryption and digital signatures |
| Diffie-Hellman | Key exchange |
| ECC | Efficient public-key cryptography |
| Ed25519 | Modern digital signatures |

---

## Symmetric vs. Asymmetric Encryption

| Feature | Symmetric | Asymmetric |
|---|---|---|
| Keys | One shared key | Public/private key pair |
| Speed | Faster | Slower |
| Best for | Bulk data encryption | Key exchange, identity, signatures |
| Key distribution | Harder | Easier |
| Example | AES | RSA, Diffie-Hellman, ECC |

Modern systems often combine both:

```text
Asymmetric cryptography -> securely agree on a shared key
Symmetric cryptography  -> encrypt the actual data
```

---

## XOR

XOR is a basic binary operation used in many cryptographic concepts.

Truth table:

| A | B | A XOR B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

Important properties:

```text
A XOR A = 0
A XOR 0 = A
A XOR B = B XOR A
```

Simple XOR encryption:

```text
Ciphertext = Plaintext XOR Key
Plaintext  = Ciphertext XOR Key
```

!!! note
    XOR by itself is not secure unless used correctly. Reusing XOR keys can completely break confidentiality.

---

## Modulo

Modulo returns the remainder after division.

Examples:

```text
25 mod 5 = 0
23 mod 6 = 5
23 mod 7 = 2
```

Modulo arithmetic is important in public-key cryptography, especially RSA and Diffie-Hellman.

---

## RSA

RSA is a public-key algorithm based on the difficulty of factoring large numbers.

At a high level:

1. Choose two large prime numbers.
2. Multiply them together.
3. Use the result to create public and private keys.
4. Encryption and decryption rely on modular arithmetic.

Key idea:

```text
Easy: multiply two large primes
Hard: factor their product back into the original primes
```

RSA values commonly seen in CTFs and cryptography challenges:

| Variable | Meaning |
|---|---|
| `p`, `q` | Prime numbers |
| `n` | Modulus, usually `p * q` |
| `e` | Public exponent |
| `d` | Private exponent |
| `m` | Plaintext message |
| `c` | Ciphertext |

RSA formulas:

```text
c = m^e mod n
m = c^d mod n
```

Security depends on using strong key sizes and safe padding schemes.

---

## Diffie-Hellman Key Exchange

Diffie-Hellman allows two parties to agree on a shared secret over an insecure channel.

The shared secret is not directly transmitted.

Basic process:

1. Both parties agree on public values.
2. Each party chooses a private value.
3. Each party calculates a public value.
4. They exchange public values.
5. Each party independently calculates the same shared secret.

Simplified flow:

```text
Alice private value + Bob public value -> shared secret
Bob private value + Alice public value -> same shared secret
```

Security relevance:

- Used for secure key exchange
- Often appears in TLS and VPNs
- Must be authenticated to prevent man-in-the-middle attacks

---

## SSH Key Authentication

SSH can use public-key authentication instead of passwords.

Generate a key pair:

```bash
ssh-keygen
```

Use a private key to connect:

```bash
ssh -i id_rsa user@host
```

Copy a public key to a server:

```bash
ssh-copy-id user@host
```

Important files:

| File | Purpose |
|---|---|
| `id_rsa` | Private key |
| `id_rsa.pub` | Public key |
| `~/.ssh/authorized_keys` | Public keys allowed to log in |

Private key permissions should be restricted:

```bash
chmod 600 id_rsa
```

!!! warning
    Treat private keys like passwords. Do not publish them in notes, repositories, screenshots, or writeups.

---

## Digital Signatures

Digital signatures verify authenticity and integrity.

They help prove:

- Who signed the data
- That the data was not changed after signing

Basic process:

```text
Sender signs data with private key
Receiver verifies signature with public key
```

Digital signatures are used in:

- Software updates
- Code signing
- TLS certificates
- Secure email
- Document signing

---

## Certificates and Certificate Authorities

Certificates bind a public key to an identity.

In HTTPS, certificates help prove that the server you are connecting to is the correct server.

Certificate chain:

```text
Website Certificate
    ↓
Intermediate CA
    ↓
Root CA trusted by browser/OS
```

Important terms:

| Term | Meaning |
|---|---|
| CA | Certificate Authority |
| Root CA | Highly trusted certificate authority |
| Intermediate CA | CA between root and site certificate |
| TLS certificate | Certificate used by HTTPS |
| Chain of trust | Validation path from site certificate to trusted root |

Security relevance:

- Expired certificates cause trust errors.
- Self-signed certificates are not automatically trusted.
- Weak certificate validation can enable man-in-the-middle attacks.

---

## TLS and HTTPS

HTTPS is HTTP protected by TLS.

TLS provides:

- Encryption
- Server authentication
- Integrity protection
- Secure key exchange

Simplified TLS flow:

```text
Client connects to server
Server presents certificate
Client validates certificate
Client and server agree on session keys
Encrypted communication begins
```

---

## PGP and GPG

PGP and GPG are used for encrypting and signing messages or files.

| Term | Meaning |
|---|---|
| PGP | Pretty Good Privacy |
| GPG | GNU Privacy Guard, open-source OpenPGP implementation |

Common uses:

- Email encryption
- File encryption
- Digital signatures
- Verifying software releases

Example commands:

```bash
# Import a key
gpg --import public.key

# Encrypt a file
gpg --encrypt --recipient user@example.com file.txt

# Decrypt a file
gpg --decrypt file.txt.gpg
```

---

## Hash Functions

A hash function creates a fixed-size digest from input data.

Example:

```text
Input:  hello
Output: hash digest
```

Hash functions are one-way. They are designed to be easy to compute and hard to reverse.

Properties of good hash functions:

| Property | Meaning |
|---|---|
| Deterministic | Same input gives same hash |
| Fast to compute | Efficient to generate |
| Hard to reverse | Cannot easily recover input |
| Collision resistant | Hard to find two inputs with same hash |
| Avalanche effect | Small input change causes large hash change |

---

## Hashing vs. Encryption

Hashing and encryption are different.

| Feature | Hashing | Encryption |
|---|---|---|
| Direction | One-way | Reversible |
| Key required | Usually no | Yes |
| Output | Fixed-size digest | Ciphertext |
| Main use | Integrity, password storage | Confidentiality |

You do not “decrypt” a hash. You crack or guess the original input by hashing guesses and comparing results.

---

## Common Hash Algorithms

| Algorithm | Status |
|---|---|
| MD5 | Insecure |
| SHA-1 | Insecure |
| SHA-256 | Common modern hash |
| SHA-512 | Common modern hash |
| bcrypt | Password hashing |
| scrypt | Password hashing |
| Argon2 | Modern password hashing |

!!! warning
    MD5 and SHA-1 should not be used for security-sensitive integrity or password storage.

---

## Hashing for Integrity

Hashes can verify that a file has not changed.

Example:

```bash
sha256sum file.iso
```

Compare the result with the hash published by the software provider.

If the hashes match, the file is likely unchanged.

---

## HMAC

HMAC stands for Hash-based Message Authentication Code.

It combines:

```text
Hash function + Secret key
```

HMAC verifies:

- Integrity
- Authenticity

Unlike a normal hash, HMAC proves that the sender knew the shared secret key.

Common uses:

- API authentication
- Message integrity
- Secure tokens
- Internal service communication

---

## Password Storage

Passwords should not be stored in plaintext or reversibly encrypted form.

Bad practices:

| Practice | Problem |
|---|---|
| Plaintext passwords | Database leak exposes all passwords |
| Reversible encryption | Key compromise exposes all passwords |
| Fast unsalted hashes | Vulnerable to cracking and rainbow tables |

Better practice:

```text
Password + unique salt -> slow password hashing algorithm -> stored hash
```

Recommended password hashing algorithms:

- Argon2
- bcrypt
- scrypt
- PBKDF2

---

## Salting

A salt is a unique random value added to a password before hashing.

Purpose:

- Prevents identical passwords from producing identical hashes
- Defeats precomputed rainbow tables
- Forces attackers to crack each password separately

Example:

```text
Password: CorrectHorseBatteryStaple
Salt:     random-user-specific-value
Hash:     hash(password + salt)
```

Salts do not need to be secret. They are commonly stored next to the password hash.

---

## Recognizing Password Hashes

Password hashes often include format indicators.

Linux password hashes are commonly stored in:

```text
/etc/shadow
```

Example format:

```text
$prefix$options$salt$hash
```

Common Linux hash prefixes:

| Prefix | Algorithm |
|---|---|
| `$y$` | yescrypt |
| `$7$` | scrypt |
| `$2b$`, `$2y$` | bcrypt |
| `$6$` | sha512crypt |
| `$1$` | md5crypt |

Windows password hashes are commonly associated with:

| Hash | Notes |
|---|---|
| NTLM | Modern Windows password hash format |
| LM | Legacy and insecure |

---

## Password Cracking Concept

Password cracking does not reverse a hash.

Instead:

1. Guess a password.
2. Hash the guess using the same algorithm and salt.
3. Compare the result to the target hash.
4. If they match, the password was found.

```text
guess -> hash function -> compare with target hash
```

Common tools:

| Tool | Use |
|---|---|
| Hashcat | GPU-accelerated cracking |
| John the Ripper | Flexible CPU/GPU cracking |

!!! warning
    Only crack hashes in authorized labs, assessments, or recovery scenarios.

---

## Quick Reference

| Concept | Purpose |
|---|---|
| Symmetric encryption | Fast bulk encryption |
| Asymmetric encryption | Key exchange, signatures, identity |
| AES | Modern symmetric encryption |
| RSA | Public-key cryptography |
| Diffie-Hellman | Shared secret agreement |
| SSH keys | Public-key authentication |
| Digital signatures | Authenticity and integrity |
| Certificates | Bind identity to public key |
| TLS | Secure network communication |
| PGP/GPG | File/email encryption and signing |
| Hashing | Integrity and password verification |
| HMAC | Keyed integrity/authenticity |
| Salt | Protect password hashes from precomputed attacks |

---

## Notes to Remember

- Encryption protects confidentiality.
- Hashing protects integrity and supports password verification.
- Symmetric encryption is fast but requires shared key protection.
- Asymmetric encryption solves key exchange and identity problems.
- Digital signatures are created with private keys and verified with public keys.
- Certificates help browsers trust HTTPS websites.
- Passwords should be stored with slow salted password hashing algorithms.
- Hashes are cracked by guessing, not decrypted.
- Private keys must never be shared or committed to Git.