# Windows Authentication and Access

Windows domain environments commonly rely on Kerberos and NetNTLM for authentication. These protocols allow users and computers to prove their identity and access resources across the network.

Kerberos is the default authentication protocol in modern Windows domains, while NetNTLM remains available for compatibility with older systems and certain network scenarios.

---

## Authentication in a Windows Domain

In a domain environment, authentication is usually handled by a Domain Controller.

The Domain Controller verifies identity and issues the information needed to access services such as:

- File shares
- Web applications
- Databases
- Remote administration services
- Internal business applications

The two main authentication protocols are:

| Protocol | Status | Purpose |
|---|---|---|
| Kerberos | Modern default | Ticket-based domain authentication |
| NetNTLM | Legacy compatibility | Challenge-response authentication |

---

## Kerberos

Kerberos is a ticket-based authentication protocol.

Instead of sending passwords across the network, Kerberos uses encrypted tickets to prove identity and request access to services.

The main Kerberos components are:

| Component | Meaning |
|---|---|
| KDC | Key Distribution Center |
| TGT | Ticket Granting Ticket |
| TGS | Ticket Granting Service ticket |
| SPN | Service Principal Name |
| krbtgt | Special account used to encrypt and sign TGTs |

In Active Directory, the KDC usually runs on the Domain Controller.

---

## Kerberos Authentication Flow

Kerberos authentication happens in three main stages.

---

## Step 1: Initial Authentication

The user authenticates to the Key Distribution Center.

The user sends an encrypted authentication request to the KDC. This request includes information such as the username and timestamp.

If authentication succeeds, the KDC returns:

| Item | Purpose |
|---|---|
| TGT | Proves the user has authenticated |
| Session Key | Used for future communication with the KDC |

The TGT is encrypted using the `krbtgt` account secret.

The user cannot directly read or modify the TGT, but can present it later when requesting access to services.

---

## Step 2: Requesting Access to a Service

When the user wants to access a specific service, such as a file share or web application, the user sends the TGT back to the KDC and requests a service ticket.

The KDC validates the TGT and returns:

| Item | Purpose |
|---|---|
| TGS | Ticket for the specific service |
| Service Session Key | Used to communicate with that service |

The TGS is encrypted using the service account’s password hash.

---

## Step 3: Service Authentication

The user presents the TGS to the target service.

The service decrypts the TGS using its own account secret. If the ticket is valid, the service grants access based on the user’s permissions.

This allows the user to authenticate to the service without sending their password to that service.

---

## Service Principal Names

A Service Principal Name, or SPN, identifies a service instance in Active Directory.

Examples of services that may use SPNs include:

- MSSQL
- HTTP web services
- CIFS file shares
- LDAP
- WinRM

Example format:

```text
service/host
```

Example:

```text
MSSQLSvc/sql01.domain.local
```

SPNs allow Kerberos to know which service account should be used to encrypt the service ticket.

---

## Why Kerberos Matters for Security

Kerberos is central to Windows domain security.

Many Active Directory attacks target Kerberos tickets or Kerberos-related configuration.

Examples include:

- Kerberoasting
- AS-REPRoasting
- Pass-the-Ticket
- Golden Ticket attacks
- Silver Ticket attacks
- Overpass-the-Hash

Understanding TGTs, TGSs, SPNs, and the KDC is essential before studying these attacks.

---

## NetNTLM

NetNTLM is a legacy challenge-response authentication protocol.

It is often still enabled for compatibility, even though Kerberos is preferred in modern Windows environments.

NetNTLM does not send the user’s password across the network. Instead, it proves knowledge of the password hash through a challenge-response exchange.

---

## NetNTLM Challenge-Response Flow

The NetNTLM process works like this:

1. The client sends an authentication request to the server.
2. The server sends back a random challenge.
3. The client calculates a response using the user’s NTLM hash and the challenge.
4. The server forwards the challenge and response to the Domain Controller.
5. The Domain Controller validates the response.
6. The server allows or denies access.

The password itself is not transmitted.

---

## Local vs Domain Authentication

Authentication depends on the account type.

| Account Type | Verified By |
|---|---|
| Local account | Local SAM database |
| Domain account | Domain Controller |

A local account only exists on one machine.

A domain account exists in Active Directory and can be used across domain-joined systems, depending on permissions.

---

## Kerberos vs NetNTLM

| Feature | Kerberos | NetNTLM |
|---|---|---|
| Authentication style | Ticket-based | Challenge-response |
| Default in modern domains | Yes | No, but often enabled |
| Requires KDC | Yes | Domain Controller validates responses |
| Uses service tickets | Yes | No |
| Common attack focus | Ticket abuse | Hash capture and relay |
| Compatibility | Modern Windows domains | Legacy or fallback scenarios |

---

## Key Takeaways

- Kerberos is the default authentication protocol in modern Windows domains.
- Kerberos uses tickets instead of repeatedly sending credentials.
- A TGT proves a user has authenticated.
- A TGS grants access to a specific service.
- SPNs identify services in Active Directory.
- NetNTLM is a legacy challenge-response protocol.
- Passwords are not directly sent over the network in either Kerberos or NetNTLM.
- Local accounts are verified locally, while domain accounts are verified by a Domain Controller.