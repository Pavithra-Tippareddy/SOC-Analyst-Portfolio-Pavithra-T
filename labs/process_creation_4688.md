# Lab: Process Creation (4688)

## Objective
See every new process and its command line (useful for malware & LOLBins).

## How I enabled it
```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1 /f

## How I generated logs
notepad.exe
cmd.exe /c whoami
powershell -c "Write-Output '4688 test'"

##SPL
index=wineventlog (EventID=4688 OR EventCode=4688)
| table _time, host, Account_Name, New_Process_Name, Parent_Process_Name, Process_Command_Line

##Findings

Saw parents like explorer.exe → cmd.exe → whoami.

Command line visibility confirmed.
