# IDS, IPS, Snort, Suricata, and Zeek

Network intrusion detection tools inspect traffic for suspicious or malicious activity.

IDS and IPS tools such as Snort and Suricata use rules to alert on or block threats, while Zeek focuses on rich network security monitoring logs.

---

## IDS vs IPS

| Type | Meaning | Behavior |
|---|---|---|
| IDS | Intrusion Detection System | Monitors and alerts |
| IPS | Intrusion Prevention System | Monitors and blocks |
| NIDS | Network IDS | Watches network traffic |
| HIDS | Host IDS | Watches one endpoint |
| NIPS | Network IPS | Blocks network traffic inline |
| HIPS | Host IPS | Blocks activity on one endpoint |

---

## IDS

An IDS is passive.

It receives a copy of traffic, usually from:

- SPAN port
- Network tap
- Packet capture
- Mirrored interface

It can alert analysts, but it does not stop traffic directly.

Benefits:

- No latency impact
- Lower risk of breaking traffic
- Useful for monitoring and forensics

Limitations:

- Cannot block attacks by itself
- Requires analyst review
- Can generate false positives

---

## IPS

An IPS is active.

It sits inline with traffic and can:

- Drop packets
- Reject connections
- Block exploit attempts
- Stop known malicious traffic

Benefits:

- Can stop attacks in real time
- Useful at network choke points

Limitations:

- Adds latency
- False positives can break legitimate traffic
- Requires careful tuning

---

## Detection Methods

| Method | Description | Strength | Weakness |
|---|---|---|---|
| Signature-based | Matches known malicious patterns | High fidelity for known threats | Misses new variants |
| Behavior-based | Detects deviations from normal | Can catch unknown threats | More false positives |
| Policy-based | Alerts on policy violations | Good for compliance | Depends on policy quality |

---

## Snort

Snort is an open-source packet sniffer, packet logger, IDS, and IPS.

It can:

- Read live traffic
- Read PCAP files
- Log packets
- Apply rules
- Generate alerts
- Drop packets in inline mode

---

## Snort Operation Modes

| Mode | Purpose |
|---|---|
| Sniffer | Print packets to console |
| Logger | Save packets to disk |
| IDS | Alert on rule matches |
| IPS | Drop or reject malicious traffic |

---

## Snort Architecture

Snort processes traffic through a pipeline:

```text
Packet capture
  -> Decoder
  -> Preprocessors / Inspectors
  -> Detection engine
  -> Logging / alerting
```

| Component | Purpose |
|---|---|
| Packet decoder | Parses raw packets |
| Preprocessors / inspectors | Normalize traffic and inspect protocols |
| Detection engine | Compares traffic against rules |
| Alerting | Outputs alerts and logs |

---

## Snort Configuration

Snort 3 commonly uses:

```text
snort.lua
```

Older Snort deployments may use:

```text
snort.conf
```

Configuration defines:

- Network variables
- Rule paths
- Inspectors/modules
- Output settings
- DAQ configuration

Common network variables:

```text
HOME_NET
EXTERNAL_NET
```

Validate configuration:

```bash
snort -c /etc/snort/snort.lua
```

Older style test mode:

```bash
sudo snort -c /etc/snort/snort.conf -T
```

---

## Running Snort on a PCAP

```bash
sudo snort -c /etc/snort/snort.lua -r traffic.pcap
```

With alert output:

```bash
sudo snort -c /etc/snort/snort.lua -r malicious.pcap -A cmg
```

Load a specific rule file:

```bash
snort -c snort.lua -r test.pcap -R /path/to/local.rules
```

Load a rule directory:

```bash
snort -c snort.lua -r test.pcap --rule-path /path/to/rules_dir/
```

---

## Running Snort Live

```bash
sudo snort -c /etc/snort/snort.lua -i eth0
```

Run in background:

```bash
sudo snort -c /etc/snort/snort.conf -D
```

Check process:

```bash
ps -ef | grep snort
```

---

## Snort IPS Mode

IPS mode requires inline traffic.

Example:

```bash
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A console
```

Key flags:

| Flag | Meaning |
|---|---|
| `-Q` | Inline IPS mode |
| `--daq afpacket` | Linux packet acquisition |
| `-i eth0:eth1` | Inline bridge pair |
| `-A console` | Console alerts |

IPS rules usually use actions like:

```text
drop
reject
```

---

## Snort Alert Modes

| Mode | Description |
|---|---|
| `console` | Prints alerts to terminal |
| `fast` | Compact alert format |
| `full` | Detailed packet/header output |
| `cmg` | Alert plus packet payload |
| `csv` | CSV output |
| `unified2` | Binary high-performance format |
| `none` | Disable alert file |

Example:

```bash
sudo snort -c /etc/snort/snort.lua -r traffic.pcap -A fast
```

---

## Snort Logger Mode

Snort can save packets to disk.

Binary logging:

```bash
sudo snort -dev -l .
```

ASCII logging:

```bash
sudo snort -dev -K ASCII -l .
```

Read Snort binary logs:

```bash
sudo snort -r snort.log.1638459842
```

Read only ICMP:

```bash
sudo snort -r snort.log.1638459842 icmp
```

Read DNS traffic:

```bash
sudo snort -r snort.log.1638459842 'udp and port 53'
```

Default binary logs can also be opened with:

```bash
tcpdump -r snort.log.1638459842 -n
```

Or Wireshark.

---

## Snort Rule Structure

A Snort rule has two parts:

```text
Rule Header (traffic match) + Rule Options (detection logic)
```

General structure:

```text
action protocol source_ip source_port direction dest_ip dest_port (options;)
```

Example:

```bash
alert tcp any any -> any 80 (msg:"Sensitive File Access"; content:"passwords.txt"; nocase; sid:1000001; rev:1;)
```

---

## Rule Header

| Field | Description | Example |
|---|---|---|
| Action | What to do | `alert`, `log`, `drop`, `reject` |
| Protocol | Protocol | `tcp`, `udp`, `icmp`, `ip` |
| Source IP | Sender | `any`, `$HOME_NET`, `!192.168.1.1` |
| Source Port | Sender port | `any`, `80`, `1:1024` |
| Direction | Flow | `->`, `<>` |
| Destination IP | Receiver | `$EXTERNAL_NET`, `any` |
| Destination Port | Receiver port | `443`, `any` |

Example:

```bash
alert tcp $EXTERNAL_NET any -> $HOME_NET 80
```

---

## Rule Options

Rule options are inside parentheses and separated by semicolons.

Common options:

| Option | Purpose |
|---|---|
| `msg` | Alert message |
| `sid` | Signature ID |
| `rev` | Revision |
| `content` | Match text or bytes |
| `nocase` | Case-insensitive content match |
| `fast_pattern` | Preferred quick match |
| `flags` | TCP flag matching |
| `dsize` | Payload size |
| `flow` | Direction/session state |
| `pcre` | Regular expression |
| `detection_filter` | Thresholding |

---

## Snort Rule IDs

| SID Range | Meaning |
|---:|---|
| `< 100` | Reserved |
| `100 - 999999` | Standard/community rules |
| `>= 1000000` | Local custom rules |

For your own rules, use:

```text
sid:1000001;
```

Increment `rev` when you update a rule:

```text
rev:2;
```

---

## Content Matching

ASCII content:

```text
content:"GET";
```

Hex content:

```text
content:"|90 90 90|";
```

Case-insensitive:

```text
content:"malware"; nocase;
```

Fast pattern:

```text
content:"EvilUserAgent"; fast_pattern;
```

---

## Snort Practical Examples

Detect request for a sensitive file:

```bash
alert tcp any any -> any 80 (msg:"Sensitive File Access"; content:"GET"; http_method; content:"passwords.txt"; nocase; sid:1000001; rev:1;)
```

Reject large FTP payloads:

```bash
reject tcp any any -> 192.168.1.5 21 (msg:"FTP Buffer Overflow Attempt"; dsize:>500; sid:1000002; rev:1;)
```

Detect a simple ICMP packet:

```bash
alert icmp any any -> any any (msg:"ICMP Packet Found"; sid:1000003; rev:1;)
```

---

## Snort Sticky Buffers

Sticky buffers tell Snort where to inspect.

Examples:

| Buffer | Purpose |
|---|---|
| `http_method` | HTTP method |
| `http_uri` | URI |
| `http_header` | HTTP headers |
| `http_client_body` | POST body |

Efficient HTTP example:

```bash
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"Patchwork Beacon"; http_method; content:"POST"; http_uri; content:".php?profile="; http_client_body; content:"ddager=", depth 7; http_header; content:!"Accept"; sid:1000006; rev:1;)
```

---

## Detection Filter

Use `detection_filter` to alert only after repeated behavior.

Example:

```bash
alert udp $HOME_NET any -> $EXTERNAL_NET any (msg:"Cerber Check-in"; dsize:9; content:"hi", depth 2, fast_pattern; pcre:"/^[af0-9]{7}$/R"; detection_filter:track by_src, count 1, seconds 60; sid:2816763; rev:1;)
```

This tracks by source IP and alerts when the threshold is met.

---

## Suricata

Suricata is a high-performance open-source network analysis engine.

It supports:

- IDS
- IPS
- Network Security Monitoring
- Deep packet inspection
- Protocol metadata logging
- JSON output through `eve.json`

---

## Suricata Modes

| Mode | Role | Behavior |
|---|---|---|
| IDS | Silent observer | Alerts but does not block |
| IPS | Active defender | Inline and can drop traffic |
| IDPS | Hybrid | Passive with active responses |
| NSM | Logger | Records metadata and transactions |

---

## Suricata Inputs

| Input | Use |
|---|---|
| `-r` | Read PCAP file |
| LibPCAP | Standard live capture |
| AF_PACKET | High-performance Linux capture |
| NFQ | Inline Linux mode with iptables |

Run against PCAP:

```bash
suricata -r capture.pcap
```

Live AF_PACKET mode:

```bash
sudo suricata --af-packet=eth0
```

Validate config:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

---

## Suricata Logs

Common log path:

```text
/var/log/suricata/
```

Important files:

| File | Purpose |
|---|---|
| `eve.json` | Main JSON event output |
| `fast.log` | Simple alert log |
| `stats.log` | Performance stats |

Filter alerts with `jq`:

```bash
cat eve.json | jq -c 'select(.event_type == "alert")'
```

Filter DNS events:

```bash
cat eve.json | jq -c 'select(.event_type == "dns")'
```

---

## Suricata Configuration

Main config:

```text
/etc/suricata/suricata.yaml
```

Important variables:

```text
HOME_NET
EXTERNAL_NET
```

Update rules:

```bash
suricata-update
```

Live rule reload:

```bash
kill -usr2 $(pidof suricata)
```

---

## Suricata Rules

Suricata rules are similar to Snort rules.

General structure:

```text
action protocol source_ip source_port -> dest_ip dest_port (options;)
```

Example:

```bash
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"Possible PowerShell Empire"; flow:established,to_server; content:"GET"; http_method; content:"session="; http_cookie; sid:3000001; rev:1;)
```

---

## Suricata Rule Keywords

| Keyword | Purpose |
|---|---|
| `msg` | Alert message |
| `sid` | Signature ID |
| `rev` | Rule revision |
| `flow` | Session direction/state |
| `content` | Match string/bytes |
| `nocase` | Case-insensitive |
| `dsize` | Payload size |
| `offset` / `depth` | Absolute search location |
| `distance` / `within` | Relative search location |
| `http_uri` | HTTP URI buffer |
| `http_method` | HTTP method buffer |
| `http_user_agent` | User-Agent buffer |
| `pcre` | Regex |
| `detection_filter` | Thresholding |
| `ja3.hash` | TLS client fingerprint |

---

## Suricata and TLS Detection

Encrypted payloads are unreadable, but TLS metadata can still be useful.

Detection vectors:

| Vector | Suricata Keyword |
|---|---|
| Certificate subject | `tls_cert_subject` |
| Certificate issuer | `tls_cert_issuer` |
| JA3 fingerprint | `ja3.hash` |

JA3 example:

```bash
alert tls any any -> any any (msg:"Sliver C2 Implant"; ja3.hash; content:"473cd7cb9faa642487833865d516e578"; sid:1002; rev:1;)
```

---

## Suricata File Extraction

Enable file storage in `suricata.yaml`:

```yaml
file-store:
  enabled: yes
```

Example extraction rule:

```bash
alert http any any -> any any (msg:"FILE store all"; filestore; sid:2; rev:1;)
```

Files are saved in the filestore directory, often named by SHA256 hash.

Inspect files:

```bash
file extracted-file
xxd extracted-file | head
```

---

## Zeek

Zeek is a passive network traffic analyzer.

Unlike Snort and Suricata, Zeek focuses less on signature alerts and more on rich transaction logs.

It is excellent for:

- Network forensics
- Threat hunting
- Connection logging
- DNS analysis
- HTTP analysis
- TLS metadata
- File transfer tracking
- Lateral movement investigation

---

## Zeek Architecture

Zeek has two major layers:

| Layer | Purpose |
|---|---|
| Event Engine | Parses traffic and emits events |
| Script Interpreter | Applies logic to events |

The event engine is policy-neutral.

Example:

```text
Zeek sees an HTTP request.
It generates an http_request event.
A Zeek script decides what to log or alert on.
```

---

## Zeek Logs

| Log | Content | Useful Fields |
|---|---|---|
| `conn.log` | Connections | `id.orig_h`, `id.resp_h`, `duration`, `orig_bytes`, `service` |
| `dns.log` | DNS activity | `query`, `qtype`, `answers` |
| `http.log` | HTTP transactions | `host`, `uri`, `user_agent`, `status_code`, `method` |
| `files.log` | File transfers | `mime_type`, `md5`, `sha1`, `source` |
| `ssl.log` | TLS metadata | `server_name`, `issuer`, `subject`, `ja3` |
| `smb_files.log` | SMB file activity | `path`, `name`, `action` |
| `dce_rpc.log` | RPC activity | Service control and remote management |

---

## Running Zeek on PCAP

```bash
zeek -C -r capture.pcap
```

`-C` ignores checksum validation.

---

## zeek-cut

`zeek-cut` extracts columns from Zeek logs.

Example:

```bash
cat conn.log | zeek-cut id.orig_h id.resp_h duration
```

Unique user agents:

```bash
cat http.log | zeek-cut user_agent | sort | uniq -c | sort -nr
```

Longest connections:

```bash
cat conn.log | zeek-cut id.orig_h id.resp_h duration | sort -k 3 -rn | head
```

File transfers by MIME type:

```bash
cat files.log | zeek-cut mime_type | sort | uniq -c
```

---

## Zeek Hunting Use Cases

### Beaconing

Look for repeated connections to the same destination at regular intervals.

Useful log:

```text
conn.log
```

Fields:

```text
id.orig_h
id.resp_h
id.resp_p
duration
orig_bytes
```

---

### DNS Exfiltration

Look for:

- Long random subdomains
- High query volume to one domain
- Repeated TXT queries
- High entropy names

Useful command:

```bash
cat dns.log | zeek-cut query | sort | uniq -c | sort -nr
```

---

### TLS Exfiltration

Find large outbound transfers:

```bash
cat conn.log | zeek-cut id.orig_h id.resp_h orig_bytes | datamash -g 1,2 sum 3 | sort -k 3 -rn
```

---

### PsExec Lateral Movement

Useful logs:

```text
smb_files.log
dce_rpc.log
```

Indicators:

- `PSEXESVC.exe` written to `ADMIN$`
- Service Control Manager activity
- RPC calls over `IPC$`
- Named pipe activity

---

## Snort vs Suricata vs Zeek

| Tool | Main Strength |
|---|---|
| Snort | Mature rule-based IDS/IPS |
| Suricata | High-performance IDS/IPS/NSM with JSON logs |
| Zeek | Rich network metadata and behavioral analysis |

Simple summary:

```text
Snort/Suricata = alert on signatures
Zeek = explain network behavior
```

---

## Quick Reference

| Goal | Tool / Command |
|---|---|
| Run Snort on PCAP | `snort -c snort.lua -r file.pcap` |
| Snort console payload alerts | `-A cmg` |
| Snort inline IPS | `-Q --daq afpacket` |
| Run Suricata on PCAP | `suricata -r file.pcap` |
| Suricata alert JSON | `eve.json` |
| Filter Suricata alerts | `jq 'select(.event_type=="alert")'` |
| Run Zeek on PCAP | `zeek -C -r file.pcap` |
| Extract Zeek fields | `zeek-cut` |

---

## Notes to Remember

- IDS alerts but does not block.
- IPS can block but must be tuned carefully.
- Snort and Suricata use rules.
- Suricata `eve.json` is excellent for SIEM ingestion.
- Zeek produces rich protocol logs instead of only alerts.
- Rule quality depends on precision, performance, and false-positive tuning.
- Test custom rules against PCAPs before deploying.