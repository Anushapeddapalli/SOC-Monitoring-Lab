# PowerShell Script Block Logging

PowerShell Script Block Logging is enabled to collect PowerShell execution activity from the Windows endpoint.

## Enable Script Block Logging

Open PowerShell as Administrator and run:

```powershell
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force | Out-Null

Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Name EnableScriptBlockLogging -Value 1
```

## Verify the Configuration

```powershell
Get-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
```

The expected value is:

```text
EnableScriptBlockLogging : 1
```

## Event Channel

PowerShell events are collected from:

```text
Microsoft-Windows-PowerShell/Operational
```

Script Block Logging uses Event ID:

```text
4104
```

## Generate a Test Event

Run:

```powershell
Write-Output "WAZUH POWERSHELL TEST $(Get-Date)"
```

## Verify Locally

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-PowerShell/Operational'
    Id=4104
} -MaxEvents 1
```

## Investigate in Wazuh

Use the following query in Wazuh Archives:

```text
agent.id:001 AND data.win.system.eventID:4104
```

To view all PowerShell events:

```text
agent.id:001 AND data.win.system.channel:"Microsoft-Windows-PowerShell/Operational"
```

## Relevant Event Information

Event ID 4104 can provide information such as:

- Script block content
- Timestamp
- User information
- Host information
- PowerShell activity

This telemetry can be correlated with Sysmon, FIM, Windows Security, and Windows Defender events during investigation.
