# Lab: Failed & Successful Login Detection

## Objective
Detect failed (4625) and successful (4624) login attempts.

## How I Generated Data
- Failed login: `runas /user:.\Administrator notepad` with wrong password (3 times).
- Successful login: Same command but correct password.

## Event Viewer Evidence
- 4625: Failed login attempts logged.
- 4624: Successful login logged.

## SPL Queries
```spl
index=wineventlog EventCode=4625
| stats count by Account_Name, Workstation_Name, Source_Network_Address

index=wineventlog EventCode=4624
| stats count by Account_Name, Logon_Type, Workstation_Name


**Findings**

4625 events clearly show failed logins.
4624 confirms successful authentication.
Together, these can be used to detect brute-force attempts.

