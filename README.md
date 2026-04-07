# 🛡️ Active Directory Attack Detection SOC Lab

## 🎯 Project Overview

This project demonstrates the simulation and detection of real-world Active Directory (AD) attack techniques within a controlled lab environment.

The primary goal is to replicate attacker behavior across different stages of the attack lifecycle and build detection mechanisms using Splunk SIEM and Sysmon telemetry.

Unlike basic lab setups, this project focuses on:
- Realistic attack simulation aligned with MITRE ATT&CK
- Deep log analysis (Windows Security Logs + Sysmon)
- Detection engineering using Splunk SPL
- SOC-style investigation and correlation

---

## 🏗️ Lab Architecture

The lab environment simulates a small enterprise network with centralized logging.

| System | Role | IP Address |
|--------|------|-----------|
| Kali Linux | Attacker Machine | 192.168.56.250 |
| Windows 10 | Domain User (Victim) | 192.168.56.100 |
| Windows Server | Domain Controller | 192.168.56.7 |
| Ubuntu Server | Splunk SIEM | 192.168.56.10 |

**Domain:** `hettilava.local`

### 🔑 Architecture Highlights
- Centralized log collection via Splunk
- Endpoint telemetry using Sysmon
- Domain-based authentication monitoring
- Segregated attacker and victim machines

![Architecture diagram](/screenshot/architecture.png)

---

## ⚔️ Attack Scenarios & MITRE Mapping

This lab simulates a multi-stage attack chain:

| Stage | Attack | MITRE ID | Description |
|------|--------|----------|------------|
| Initial Access | Brute Force | T1110 | Password guessing using Hydra |
| Execution | PowerShell | T1059.001 | Command execution via PowerShell |
| Persistence | Account Creation | T1136.001 | Unauthorized user creation |
| Command & Control | Reverse Shell | - | Remote access via PowerShell |

---

## 🔍 Detection Engineering Approach

Detection logic is built using a combination of:

### 📌 Log Sources
- Windows Security Logs (Event IDs: 4624, 4625, 4720)
- Sysmon Logs (Event IDs: 1 – Process Creation, 3 – Network Connections)

### 📌 Methodology
- Behavioral analysis instead of static signatures
- Threshold-based anomaly detection
- Correlation across authentication, process, and network events

### 📌 Example Detection Flow
1. Multiple failed logins (Event ID 4625)
2. Followed by successful login (Event ID 4624)
3. Followed by PowerShell execution (Sysmon Event ID 1)

➡️ Indicates potential account compromise

---

## 🧪 Lab Setup

The lab environment was manually configured to simulate a real enterprise AD infrastructure.

### Key Components:
- Active Directory Domain Controller
- Domain-joined Windows endpoint
- Sysmon for endpoint telemetry
- Splunk SIEM for centralized logging

📂 Detailed setup guide:  
👉 `setup/lab-setup.md`

---

## 📊 Key Outcomes

- Successfully simulated multiple AD attack techniques
- Built Splunk detection queries for each attack phase
- Correlated logs across multiple systems
- Identified attacker behavior patterns
- Reduced false positives through filtering and tuning

---

## ⚠️ Challenges & Solutions

| Challenge | Resolution |
|----------|-----------|
| Sysmon logs not forwarding | Fixed forwarder configuration |
| Splunk ingestion issues | Verified port 9997 and inputs |
| Time synchronization mismatch | Aligned system time across machines |
| Excess noise in logs | Applied filtering and thresholds |

---

## 📸 Evidence & Screenshots

The repository includes:
- Attack execution screenshots
- Splunk detection results
- Log evidence from Sysmon and Windows logs

📂 Available in:
- `screenshots/attacks/`
- `screenshots/detections/`
- `screenshots/setup/`

---

## 🧠 Skills Demonstrated

- SIEM Engineering (Splunk)
- Detection Engineering & Log Correlation
- Active Directory Security
- Threat Analysis & MITRE ATT&CK Mapping
- Endpoint Monitoring (Sysmon)

---

## 🚀 Future Improvements

- Build Splunk dashboards for visualization
- Integrate Sigma rules for detection standardization
- Add advanced attack scenarios (Credential Dumping, Lateral Movement)
- Implement automated alerting

---

## 🔐 Disclaimer

This project was conducted in a controlled lab environment for educational and research purposes only. No real systems were targeted.

---

## ⭐ Why This Project Matters

This project reflects practical SOC skills including:
- Attack simulation
- Detection engineering
- Log analysis
- Security monitoring

It demonstrates the ability to think like both an attacker and a defender — a key requirement for cybersecurity roles.

---
