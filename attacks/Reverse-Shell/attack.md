# Reverse Shell

## Command
powershell reverse shell payload

## Detection
index=sysmon EventCode=3
| stats count by DestinationIp, DestinationPort
