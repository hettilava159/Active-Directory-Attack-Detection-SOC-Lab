# ⚔️ PowerShell Execution Attack (T1059.001)

---

## 🎯 Objective

Simulate malicious PowerShell command execution on a Windows system and detect it using Sysmon and Splunk.

This attack represents adversary behavior where PowerShell is used for execution of commands or payloads.

---

## 🧠 MITRE ATT&CK Mapping

- Technique: T1059.001  
- Name: Command and Scripting Interpreter – PowerShell  

---

## 🛠️ Tools Used

- PowerShell  
- Sysmon  
- Splunk SIEM  

---

## ⚙️ Attack Execution

Run the following command on the Windows machine:

```powershell
powershell -ExecutionPolicy Bypass -Command "Get-Process"
```

---

## 🧩 Steps Performed

1. Login to Windows 10 victim machine  
2. Open Command Prompt or PowerShell  
3. Execute PowerShell command with bypass policy  
4. Observe execution of command  

---

## 📊 Telemetry Generated

### Sysmon Logs

- Event ID 1 → Process Creation  
- Image → powershell.exe  
- CommandLine → ExecutionPolicy Bypass  

### Windows Logs

- PowerShell activity logs  

---

## 🔍 Detection in Splunk

### Basic Detection Query

```spl
index=sysmon EventCode=1
| search Image="*powershell.exe"
```

---

### Advanced Detection

```spl
index=sysmon EventCode=1
| search CommandLine="*ExecutionPolicy*Bypass*"
| stats count by Computer, User, CommandLine
```

---

## 🧠 Detection Logic

- Detect execution of PowerShell process  
- Identify suspicious flags like:
  - ExecutionPolicy Bypass  
  - Hidden execution  
- Monitor abnormal command usage  

---

## ⚠️ False Positives

- Legitimate admin scripts  
- IT automation tools  

### Mitigation

- Filter known admin accounts  
- Monitor unusual frequency  

---

## 📸 Screenshots Required

- PowerShell command execution  
- Sysmon Event Viewer logs  
- Splunk detection results  

Paths:

- screenshots/attacks/powershell-execution.png  
- screenshots/detections/powershell-splunk.png  

---

## 🚀 Outcome

- Successfully simulated PowerShell execution  
- Captured logs via Sysmon  
- Detected activity using Splunk  

---
