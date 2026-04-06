# Splunk Queries

## Brute Force
index=wineventlog EventCode=4625
| stats count by Account_Name
| where count > 10

## Account Creation
index=wineventlog EventCode=4720

## PowerShell
index=sysmon EventCode=1

## Reverse Shell
index=sysmon EventCode=3
