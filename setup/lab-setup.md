# Lab Setup

## Network
All machines configured on same network with static IP.

## Active Directory
Install AD:
Install-WindowsFeature AD-Domain-Services

Create domain:
Install-ADDSForest -DomainName "hettilava.local"

## Sysmon
sysmon64.exe -i sysmonconfig.xml

## Splunk
Install Splunk and enable port 9997

## Forwarder
splunk add forward-server 192.168.56.10:9997

## Validation
index=wineventlog
