# Lab: Nmap Reconnaissance on Ubuntu VM

## Objective
Performed network scanning using Nmap from Kali (attacker) against Ubuntu (victim) to identify host availability, open ports, services, and OS details.

---

## Steps Performed

### Step 1: Identify Target IP
On Ubuntu VM:

ip addr show
Victim IP identified: 192.168.30.128

## Step 2: Host Discovery (Ping Scan)
From Kali:
nmap -sn 192.168.30.128

Result: 
# Host is up (responsive).

## Step 3: Port Scan (TCP SYN)
nmap -sS 192.168.30.128

Result:
# Host is alive.
# All 1000 scanned TCP ports returned as closed

## Step 4: Service Detection
nmap -sV 192.168.30.128

Result:
# No services detected (all ports closed).
# Confirms no active applications listening on common ports.

## Step 5: OS Detection
sudo nmap -O 192.168.30.128

Result:
# Host detected as Linux kernel (2.6.x – 5.x range).
# OS detection limited due to lack of open ports.

## Findings
* Victim host (192.168.30.128) is reachable.

* All scanned TCP ports closed (likely due to default firewall rules or no services running).

* OS fingerprinting successful despite no open ports.

* This is a realistic scenario: analysts often see closed-port hosts in reconnaissance, which still helps with threat assessment.

## SOC Analyst Notes
* Attackers use Nmap to probe networks for weaknesses.
* Even when ports are closed, defenders can detect the scan attempts in firewall/SIEM logs.
* In production, security teams monitor for repeated SYN scans as signs of reconnaissance.



