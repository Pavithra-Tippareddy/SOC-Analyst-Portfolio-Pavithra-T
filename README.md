# SOC Analyst Portfolio — Pavithra T

Hands-on, reproducible mini-labs showing Tier-1 SOC skills with Windows + Splunk.

## Lab Goals
- Generate real Windows security events (failed logins, PowerShell).
- Ingest them into Splunk.
- Detect and explain findings like a SOC analyst.

## Stack
- **OS:** Windows 11 (+ WSL: Ubuntu, Kali)
- **Logs:** Windows Security, System, **PowerShell/Operational**
- **SIEM:** Splunk Free (local)

## What’s inside
- `/setup` – how I enabled Windows auditing & added PowerShell logs without `gpedit.msc`.
- `/labs` – step-by-step exercises I ran.
- `/splunk_queries` – SPL I actually used.
- `/screenshots` – Event Viewer + Splunk evidence.

> **Note:** In queries, replace `index=wineventlog` with **your** index name if different, and use the field that exists in your data (`EventID` or `EventCode`).

## Quick Reproduce
1) Enable PowerShell Script Block Logging (no gpedit) → see `/setup/windows_logging.md`  
2) Tell Splunk to ingest `Microsoft-Windows-PowerShell/Operational` → see `/setup/splunk_inputs.md`  
3) Run a few PowerShell commands, then search:

