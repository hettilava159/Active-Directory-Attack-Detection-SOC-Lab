# T1059.001 - PowerShell Execution (Atomic Red Team)

## 📌 Overview
This attack simulates adversary behavior using PowerShell for command execution, aligned with MITRE ATT&CK technique **T1059.001 (PowerShell)**.

PowerShell is commonly abused by attackers for:
- Fileless malware execution
- Downloading remote payloads
- Privilege escalation
- Persistence mechanisms

---

## 🧪 Lab Setup

| Component | IP Address |
|----------|-----------|
| Attacker (Kali Linux) | 192.168.56.250 |
| Victim (Windows 10) | 192.168.56.100 |
| Domain Controller | 192.168.56.7 |
| Splunk Server | 192.168.56.101 |

---

## ⚙️ Tools Used
- Atomic Red Team
- Sysmon
- Splunk SIEM

---

## 🚀 Attack Execution

### Step 1: Install Atomic Red Team

```powershell
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics
```

---

### Step 2: Import Module

```powershell
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1"
```

---

### Step 3: Execute Attack

```powershell
Invoke-AtomicTest T1059.001 -TestNumbers 1
```

---

## 🔍 Observed Behavior

The following suspicious PowerShell activities were observed:
- Execution with `-ExecutionPolicy Bypass`
- Use of `IEX (Invoke-Expression)`
- Remote script download via `DownloadString`
- Encoded command execution

---

## 📊 Log Source

### Sysmon Logs
- **Event ID 1** → Process Creation
- Key fields:
  - Image: powershell.exe
  - CommandLine: Suspicious arguments
  - ParentImage: Process spawning PowerShell

---

## 🔎 Splunk Detection Query

```spl
index=endpoint host=192.168.56.100
| rex field=_raw "<EventID>(?<EventID>\d+)</EventID>"
| where EventID=1
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]*)</Data>"
| rex field=_raw "<Data Name='CommandLine'>(?<CommandLine>[^<]*)</Data>"
| rex field=_raw "<Data Name='User'>(?<User>[^<]*)</Data>"
| rex field=_raw "<Data Name='Computer'>(?<Computer>[^<]*)</Data>"
| rex field=_raw "<Data Name='ParentImage'>(?<ParentImage>[^<]*)</Data>"
| where like(Image,"%powershell.exe%") OR like(Image,"%pwsh.exe%")
| regex CommandLine="(?i)(EncodedCommand|IEX|DownloadString|Bypass)"
| table _time, Computer, EventID, Image, ParentImage, User, CommandLine
| sort - _time
```

---

## 📈 Detection Logic

This detection identifies suspicious PowerShell execution based on:
- Encoded or obfuscated commands
- Download and execution patterns
- Execution policy bypass attempts

---

## 🛡️ Mitigation

- Restrict PowerShell execution policies
- Enable PowerShell logging (Script Block Logging)
- Monitor suspicious parent-child processes
- Use endpoint detection tools

---

## 🧠 MITRE ATT&CK Mapping

| Tactic | Technique |
|------|---------|
| Execution | T1059.001 - PowerShell |

---

## 📸 Evidence

Include screenshots of:
- Splunk detection results
![Hydra Attack Output](/screenshot/attacks/T1059.001-PowerShell/Splunk_detection_results.png)
  
- Atomic Red Team execution
![Hydra Attack Output](/screenshot/attacks/T1059.001-PowerShell/Atomic_Red_Team_execution.png)

---

## ✅ Conclusion

The simulation successfully demonstrated how attackers leverage PowerShell for execution.  
Detection was achieved using Sysmon logs and Splunk queries, identifying suspicious command-line patterns and execution behavior.
