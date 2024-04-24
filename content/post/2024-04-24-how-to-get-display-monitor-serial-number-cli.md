---
date: "2024-04-24T00:00:00Z"
title: How to get display monitor serial number from PowerShell
---

This is a quick one, I had to get the SN of my external screens recently and I thought I should share the powershell command I used for this task. This works only on your local machine, for AD nodes there are [other options out there on the internet](https://community.spiceworks.com/t/cmd-command-get-monitor-serial-number-help-me/333757).

```powershell
Get-WmiObject WmiMonitorID -Namespace root\wmi |Select-Object @{l="Manufacturer";e={[System.Text.Encoding]::ASCII.GetString($_.ManufacturerName)}},@{l="Model";e={[System.Text.Encoding]::ASCII.GetString($_.UserFriendlyName)}},@{l="SerialNumber";e={[System.Text.Encoding]::ASCII.GetString($_.SerialNumberID)}}
```

This should return something like this:
```powershell
PS C:\Users\costin> Get-WmiObject WmiMonitorID -Namespace root\wmi |Select-Object @{l="Manufacturer";e={[System.Text.Encoding]::ASCII.GetString($_.ManufacturerName)}},@{l="Model";e={[System.Text.Encoding]::ASCII.GetString($_.UserFriendlyName)}},@{l="SerialNumber";e={[System.Text.Encoding]::ASCII.GetString($_.SerialNumberID)}}

Manufacturer     Model         SerialNumber
------------     -----         ------------
HPN HP E243i 6CM90XXXX
HPN HP E223 3CQXXXXX

```
