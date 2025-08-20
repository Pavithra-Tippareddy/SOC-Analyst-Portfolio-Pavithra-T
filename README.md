A hands-on learning log showing my SOC Level‑1 skills with small, reproducible labs.


🔹 About Me :

I am learning SOC Analyst skills step by step. This repo is my portfolio where I document labs, detections, and Splunk queries I’ve practiced.



🔹 Lab Setup

OS: Windows 11 with WSL (Ubuntu & Kali)

SIEM: Splunk Free (running locally)

Data Source: Windows Event Viewer (Security, PowerShell, System logs)

Goal: Learn to detect failed logins, privilege escalation, and suspicious PowerShell activity

🔹 Labs

1. Failed Login Detection

Event ID: 4625

How I generated: Wrong password attempts using runas

Splunk Query: 
index=wineventlog EventCode=4625


2. Successful Login Detection
Event ID: 4624

How I generated: Logged in with correct credentials

Splunk Query:

index=wineventlog EventCode=4624


What I saw: Failed login events in Event Viewer + Splunk


3. PowerShell Execution
   
Event IDs: 4103 (Modules), 4104 (Scripts)

How generated: Ran sample PowerShell commands

Splunk Query:

index=wineventlog EventCode=4104
| stats count by ScriptBlockText


🔹 SPL Queries Library

All queries are stored in /splunk_queries/.


🔹 Incident Writeups

Example files:

/writeups/failed_login_analysis.md
/writeups/powershell_attack_detection.md

Each includes:
What happened
How it was detected
Splunk query used
SOC analyst response



🔹 Cheat Sheet

I don’t memorize everything — I keep a list:

Reset WSL password
Enable Windows Event IDs
Common Splunk queries
