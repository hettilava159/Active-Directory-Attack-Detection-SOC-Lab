# Account Creation (T1136.001)

## Command
net user attacker Pass@123 /add

## Detection
index=wineventlog EventCode=4720
