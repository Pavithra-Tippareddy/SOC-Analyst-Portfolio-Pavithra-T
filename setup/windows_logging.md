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

