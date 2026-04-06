# PowerShell Execution (T1059.001)

## Command
powershell -ExecutionPolicy Bypass -Command "Get-Process"

## Detection
index=sysmon EventCode=1
| search Image="*powershell.exe"
