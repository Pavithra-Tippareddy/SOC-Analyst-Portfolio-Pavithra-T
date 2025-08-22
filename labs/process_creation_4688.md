# Lab: Process Creation (4688)

## 🎯 Objective
Track every new process and its command line. This is useful for detecting malware and Living-off-the-Land Binaries (LOLBins).

---

## 🛠️ How I Enabled It
```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1 /f


📝 How I Generated Logs
# notepad.exe
# cmd.exe /c whoami
# powershell -c "Write-Output '4688 test'"

🔎 SPL Query
index=wineventlog (EventID=4688 OR EventCode=4688)
| table _time, host, Account_Name, New_Process_Name, Parent_Process_Name, Process_Command_Line


📊 Findings
# Saw explorer.exe → cmd.exe → whoami
# Command line visibility confirmed ✅

## Note: In my lab, Parent_Process_Name didn’t show up. This may be due to missing Sysmon or event log enrichment. But New_Process_Name and Process_Command_Line worked fine


```markdown
## 📸 Evidence
![Process Creation Log in Splunk](/screenshots/4688_process_creation.png)
