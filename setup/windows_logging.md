# Enable Windows & PowerShell Logging (Windows Home – no gpedit)

## Why
SOC analysts need visibility into PowerShell usage. Script Block Logging writes events:
- **4103** – module logging
- **4104** – script block logging

## How (Registry Method)
Open `regedit` and create:

`HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging`
- `EnableScriptBlockLogging` (DWORD) = `1`

(Optional, more detail)
`HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging`
- `EnableModuleLogging` (DWORD) = `1`
- Subkey: `ModuleLogging\ModuleNames`
  - String value: `*` = `*`

**Reboot** (or `gpupdate /force`).

## Verify
Event Viewer → `Applications and Services Logs/Microsoft/Windows/PowerShell/Operational`  
Run in PowerShell:
```powershell
Get-Process
Write-Output "Testing logging"

We should see 4103/4104 events appear.

---

### `setup/splunk_inputs.md`
```markdown
# Ingest PowerShell/Operational into Splunk

## Why
The GUI doesn’t list PowerShell/Operational by default. We add it manually.

## Steps (Windows Splunk local install)
Edit:

C:\Program Files\Splunk\etc\system\local\inputs.conf

Add:
```ini
[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
index = wineventlog
renderXml = 0

Restart the Splunkd service.

Test SPL

Try each, depending on your field names:

index=wineventlog (EventID=4103 OR EventID=4104)
index=wineventlog (EventCode=4103 OR EventCode=4104)
index=wineventlog "4104"


Pitfalls I hit

My index name differed from examples → no results until I fixed it.

Field might be EventCode instead of EventID.


---

### `labs/failed_login_detection.md`
```markdown
# Lab: Failed Login Detection (4625)

## Objective
Detect failed user authentication attempts.

## Generate Data
- Use wrong password a few times:

runas /user:.\Administrator notepad

(enter incorrect password repeatedly)

## Event Source
- Windows Event Viewer → Security → **4625**

## SPL
```spl
index=wineventlog (EventID=4625 OR EventCode=4625)
| stats count by Account_Name, Workstation_Name, Source_Network_Address
| sort - count


Findings & Analyst Notes
Repeated 4625 from the same IP/user can indicate brute force.
Check for subsequent 4624 (success) to see if attacker eventually got in.


---

### `labs/powershell_execution.md`
```markdown
# Lab: PowerShell Script Execution Visibility (4103/4104)

## Objective
Confirm visibility of PowerShell activity without gpedit.

## Generate Data
Run:
```powershell
Get-Service
Get-Process
Write-Output "Hello from Pavithra"

SPL

index=wineventlog (EventID=4103 OR EventID=4104)
| table _time, Account_Name, ScriptBlockText
| sort - _time


4104 shows the actual script block text.
In real SOC work, hunt for patterns like -enc, IEX, FromBase64String, WebClient/Invoke-WebRequest


---

### `splunk_queries/failed_logins.spl`
```spl
index=wineventlog (EventID=4625 OR EventCode=4625)
| stats count by Account_Name, Source_Network_Address, Workstation_Name
| sort - count

splunk_queries/successful_logins.spl

index=wineventlog (EventID=4624 OR EventCode=4624)
| stats count by Account_Name, Logon_Type, Workstation_Name


splunk_queries/powershell_execution.spl

index=wineventlog (EventID=4103 OR EventID=4104)
| table _time, Account_Name, ScriptBlockText

4) Uploaded screenshots

Event Viewer (PowerShell → Operational)

Splunk search for 4625

Splunk search for 4103/4104



## Splunk Windows Event Log User Field Extraction Setup

This document summarizes the steps taken to configure Splunk for extracting user account information from Windows Security event logs, troubleshooting SID translation, and considerations on domain joining.

---

## Splunk Inputs Configuration for SID Translation

In `inputs.conf` on the Splunk Universal Forwarder or Indexer:

[WinEventLog://Security]
disabled = 0
evt_resolve_ad_obj = 1
use_old_eventlog_api = 1

evt_dc_name = your_domain_controller_name # Optional: specify DC(Domain controller
) if needed


- Save and restart Splunk forwarder after editing.

---

## Troubleshooting SID Translation Issues

- Use `splunk btool inputs list --debug` to confirm settings are applied.
- Check `splunkd.log` for errors: look for messages about binding to domain controllers, SID translation failures, or "NOT_TRANSLATED".
- Common error example indicates: `Failed to get domain controller name with DsGetDcName: (1355)`.
- This means the host cannot reach or discover a domain controller.

---

## Domain Joining Requirements

- SID translation requires the forwarding host to be joined to an Active Directory domain.
- Verify domain join status:
  - Windows System > About should show domain membership.
  - Local Account means not joined to any domain.
- If not domain joined:
  - SID translation will fail with “NOT_TRANSLATED” in logs.
  - Join the host to the domain to enable SID resolution.

---

## Risks of Joining a Domain (Considerations)

- Domain joining centralizes control and may restrict local settings.
- Increased security risk if device or credentials are compromised.
- Adds IT management overhead.
- For personal or standalone systems, domain join may not be necessary or desired.

---

## Final Notes

- If domain join is not possible or desired, user names may remain untranslated.
- Alternative approaches: configuring local accounts or logging with user-friendly names if possible.
- Always test configuration and monitor logs to verify correct field extraction.

---

*This README serves as a quick reference guide to configure and troubleshoot Windows Security logs and user field extraction in Splunk.*
