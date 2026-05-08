# Splunk Queries

---

## Executive Summary

This cheatsheet contains quick Splunk SPL patterns for searching logs, filtering events, building timelines, and hunting suspicious activity.

Use these as starting points and adjust `index`, `sourcetype`, field names, and time ranges for your environment.

## Basic Search

Search an index:

```spl
index=main
```

Search with keyword:

```spl
index=main "powershell"
```

Search by host:

```spl
index=main host="HOSTNAME"
```

Search by source type:

```spl
index=main sourcetype="WinEventLog:Security"
```

Search last 24 hours:

```spl
index=main earliest=-24h
```

## Useful SPL Commands

| Command | Purpose |
|---|---|
| `table` | Display selected fields |
| `stats` | Aggregate results |
| `sort` | Sort results |
| `dedup` | Remove duplicates |
| `rename` | Rename fields |
| `eval` | Create calculated fields |
| `where` | Filter with expressions |
| `rex` | Extract fields with regex |
| `lookup` | Enrich with lookup data |
| `transaction` | Group related events |
| `timechart` | Create time-based aggregation |

## Field Display

Show selected fields:

```spl
index=main
| table _time host user process command_line
```

Sort newest first:

```spl
index=main
| sort - _time
```

Remove duplicate commands:

```spl
index=main
| dedup command_line
```

## Statistics

Count by host:

```spl
index=main
| stats count by host
```

Count by user and host:

```spl
index=main
| stats count by user host
```

Top source IPs:

```spl
index=main
| top src_ip
```

Rare processes:

```spl
index=main process_name=*
| rare process_name
```

## Windows Logons

Successful logons:

```spl
index=main sourcetype="WinEventLog:Security" EventCode=4624
| table _time host Account_Name Logon_Type IpAddress
```

Failed logons:

```spl
index=main sourcetype="WinEventLog:Security" EventCode=4625
| table _time host Account_Name Failure_Reason IpAddress
```

RDP logons:

```spl
index=main sourcetype="WinEventLog:Security" EventCode=4624 Logon_Type=10
| table _time host Account_Name IpAddress
```

Explicit credentials used:

```spl
index=main sourcetype="WinEventLog:Security" EventCode=4648
| table _time host Account_Name Target_Server_Name Process_Name
```

## Windows Process Creation

Process creation:

```spl
index=main sourcetype="WinEventLog:Security" EventCode=4688
| table _time host Account_Name New_Process_Name Command_Line Parent_Process_Name
```

PowerShell execution:

```spl
index=main EventCode=4688 New_Process_Name="*powershell*"
| table _time host Account_Name Command_Line Parent_Process_Name
```

Suspicious download commands:

```spl
index=main EventCode=4688
(Command_Line="*Invoke-WebRequest*" OR Command_Line="*DownloadString*" OR Command_Line="*DownloadFile*" OR Command_Line="*WebClient*")
| table _time host Account_Name New_Process_Name Command_Line
```

Certutil download behavior:

```spl
index=main EventCode=4688 New_Process_Name="*certutil*"
(Command_Line="*urlcache*" OR Command_Line="*decode*" OR Command_Line="*http*")
| table _time host Account_Name Command_Line
```

## Suspicious Parent-Child Processes

Office spawning shell:

```spl
index=main EventCode=4688
(Parent_Process_Name="*winword.exe" OR Parent_Process_Name="*excel.exe" OR Parent_Process_Name="*powerpnt.exe")
(New_Process_Name="*cmd.exe" OR New_Process_Name="*powershell.exe" OR New_Process_Name="*wscript.exe" OR New_Process_Name="*cscript.exe")
| table _time host Account_Name Parent_Process_Name New_Process_Name Command_Line
```

Web server spawning shell:

```spl
index=main EventCode=4688
(Parent_Process_Name="*w3wp.exe" OR Parent_Process_Name="*httpd.exe" OR Parent_Process_Name="*nginx.exe" OR Parent_Process_Name="*apache*")
(New_Process_Name="*cmd.exe" OR New_Process_Name="*powershell.exe" OR New_Process_Name="*/bin/sh*" OR New_Process_Name="*/bin/bash*")
| table _time host Parent_Process_Name New_Process_Name Command_Line
```

## Services and Scheduled Tasks

Service installed:

```spl
index=main (EventCode=7045 OR EventCode=4697)
| table _time host Service_Name Service_File_Name Account_Name
```

Scheduled task created:

```spl
index=main EventCode=4698
| table _time host Account_Name Task_Name Task_Content
```

Scheduled task deleted:

```spl
index=main EventCode=4699
| table _time host Account_Name Task_Name
```

## PowerShell Logs

PowerShell script block logging:

```spl
index=main sourcetype="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104
| table _time host User ScriptBlockText
```

Suspicious PowerShell keywords:

```spl
index=main EventCode=4104
(ScriptBlockText="*IEX*" OR ScriptBlockText="*Invoke-Expression*" OR ScriptBlockText="*DownloadString*" OR ScriptBlockText="*FromBase64String*")
| table _time host User ScriptBlockText
```

## Network Hunting

Outbound connections by process:

```spl
index=main dest_ip=* dest_port=*
| stats count by host process_name dest_ip dest_port
| sort - count
```

Rare external destinations:

```spl
index=main dest_ip=*
| stats count by dest_ip
| where count < 5
```

Direct IP web requests:

```spl
index=proxy url="http://*"
| regex url="https?://\d{1,3}(\.\d{1,3}){3}"
| table _time src_ip user url user_agent
```

Suspicious User-Agents:

```spl
index=proxy
(user_agent="*WindowsPowerShell*" OR user_agent="*Microsoft-CryptoAPI*" OR user_agent="*Microsoft BITS*" OR user_agent="*curl*" OR user_agent="*Wget*" OR user_agent="*python-requests*")
| table _time src_ip user url user_agent
```

## DNS Hunting

Top queried domains:

```spl
index=dns query=*
| top query
```

NXDOMAIN volume:

```spl
index=dns response_code="NXDOMAIN"
| stats count by src_ip
| sort - count
```

Long DNS queries:

```spl
index=dns query=*
| eval query_length=len(query)
| where query_length > 80
| table _time src_ip query query_length
```

Rare domains:

```spl
index=dns query=*
| stats count by query
| where count < 3
| sort count
```

## Web Attack Hunting

Path traversal:

```spl
index=web
(uri="*../*" OR uri="*%2e%2e%2f*" OR uri="*..%2f*")
| table _time src_ip status uri user_agent
```

SQL injection patterns:

```spl
index=web
(uri="*union*select*" OR uri="*sleep(*" OR uri="*benchmark(*" OR uri="*' OR '1'='1*")
| table _time src_ip status uri user_agent
```

Common exposed files:

```spl
index=web
(uri="*/.env*" OR uri="*/config*" OR uri="*/backup*" OR uri="*/phpinfo.php*" OR uri="*/.git/*")
| table _time src_ip status uri user_agent
```

## Timeline Building

Timeline by host:

```spl
index=main host="HOSTNAME"
| table _time sourcetype EventCode user process command_line src_ip dest_ip
| sort _time
```

Timeline by user:

```spl
index=main user="USERNAME"
| table _time host sourcetype EventCode process command_line src_ip dest_ip
| sort _time
```

## Notes

- Replace `index=main` with the correct index.
- Field names vary by data source and parser.
- Use `table` early while testing, then optimize later.
- Always validate query results with raw events.
- Tune detections to your environment before alerting.