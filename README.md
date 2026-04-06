# Active Directory Attack Detection SOC Lab

## ?? Objective
Simulate Active Directory attacks and detect them using Splunk SIEM and Sysmon.

---

## ??? Lab Architecture

| System | Role | IP |
|--------|------|----|
| Kali Linux | Attacker | 192.168.56.250 |
| Windows 10 | Victim | 192.168.56.100 |
| Windows Server | Domain Controller | 192.168.56.7 |
| Ubuntu Server | Splunk | 192.168.56.10 |

Domain: hettilava.local

---

##  Attack Chain
- T1110: Brute Force
- T1059.001: PowerShell Execution
- T1136.001: Account Creation
- Reverse Shell

---

##  Detection Strategy
- Windows Security Logs
- Sysmon Logs
- Splunk Correlation

---

##  Setup
See: setup/lab-setup.md

---

##  Skills
SIEM | Detection Engineering | AD Security | Threat Analysis
