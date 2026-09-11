# ⚙️ Lab Setup Guide

This guide explains how to reproduce the SOC monitoring lab using VirtualBox, Ubuntu, Wazuh, OpenCanary, Windows 11, Wazuh Agent, Sysmon, PowerShell logging, and Wazuh FIM.

---

## Prerequisites

Before starting, prepare:

- Oracle VM VirtualBox
- Ubuntu VM
- Windows 11 VM
- Kali Linux VM
- Internet connectivity during installation
- Sufficient RAM and storage for the virtual machines

---

## 1. Create the Virtual Machines

Create three virtual machines in VirtualBox:

```text
Kali Linux
    ↓
Security testing / attacker simulation

Ubuntu
    ↓
Wazuh Manager
Wazuh Indexer
Wazuh Dashboard
OpenCanary

Windows 11
    ↓
Wazuh Agent
Sysmon
Windows Defender
PowerShell logging
FIM
```

Configure the virtual machines so that they can communicate with each other on the lab network.

---

## 2. Install Wazuh on Ubuntu

Start the Ubuntu VM and update the package list:

```bash
sudo apt update
```

Download the Wazuh installation script:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
```

Run the all-in-one installation:

```bash
sudo bash ./wazuh-install.sh -a -i
```

This installs the following Wazuh components:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

Verify the services after installation:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

Open the Wazuh Dashboard using the IP address of the Ubuntu system.

---

## 3. Install the Wazuh Agent on Windows

Download and install the official Wazuh Agent on the Windows 11 VM.

Open the Wazuh Agent configuration file:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Configure the Wazuh Manager connection:

```xml
<client>
    <server>
        <address>WAZUH_MANAGER_IP</address>
        <port>1514</port>
        <protocol>tcp</protocol>
    </server>
</client>
```

Replace `WAZUH_MANAGER_IP` with the IP address of the Ubuntu Wazuh Manager.

Restart the Wazuh Agent service.

Verify that the Windows endpoint appears as an active agent in the Wazuh Dashboard or from the Wazuh Manager.

---

## 4. Configure Windows Event Collection

Open the Wazuh Agent configuration file:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Add the required Windows Event Channels:

```xml
<localfile>
    <location>Application</location>
    <log_format>eventchannel</log_format>
</localfile>

<localfile>
    <location>Security</location>
    <log_format>eventchannel</log_format>
</localfile>

<localfile>
    <location>System</location>
    <log_format>eventchannel</log_format>
</localfile>

<localfile>
    <location>Microsoft-Windows-Windows Defender/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>

<localfile>
    <location>Microsoft-Windows-PowerShell/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>

<localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>
```

The Security Event Channel was also configured with exclusions for several high-volume event IDs.

Restart the Wazuh Agent after modifying the configuration.

---

## 5. Install Sysmon

Download Sysmon from the official Microsoft Sysinternals distribution.

Extract the files to:

```text
C:\Sysmon
```

Open PowerShell as Administrator:

```powershell
cd C:\Sysmon
```

Install Sysmon:

```powershell
.\Sysmon64.exe -accepteula -i
```

Apply the Sysmon configuration:

```powershell
.\Sysmon64.exe -c sysmonconfig.xml
```

Verify that Sysmon is running.

The project specifically verified:

```text
Event ID 3  → Network Connection
Event ID 22 → DNS Query
```

Additional Sysmon events were also collected through Wazuh Archives.

---

## 6. Enable PowerShell Script Block Logging

Open PowerShell as Administrator.

Create the Script Block Logging registry key:

```powershell
New-Item `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
  -Force | Out-Null
```

Enable Script Block Logging:

```powershell
Set-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
  -Name EnableScriptBlockLogging `
  -Value 1
```

Verify the configuration:

```powershell
Get-ItemProperty `
  "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
```

Generate controlled PowerShell activity:

```powershell
Write-Output "WAZUH POWERSHELL TEST $(Get-Date)"
```

Verify the resulting PowerShell telemetry in Wazuh Archives.

The project verified:

```text
Event ID 4104 → PowerShell Script Block Logging
```

---

## 7. Configure Wazuh File Integrity Monitoring

Open the Wazuh Agent configuration file:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Configure the directories to monitor:

```xml
<syscheck>
    <directories check_all="yes" realtime="yes">C:\Wazuh-FIM-Test</directories>
    <directories check_all="yes" realtime="yes">C:\Users</directories>
    <directories check_all="yes" realtime="yes">C:\ProgramData</directories>

    <disabled>no</disabled>
</syscheck>
```

Create the test directory:

```powershell
New-Item -ItemType Directory -Path "C:\Wazuh-FIM-Test" -Force
```

Restart the Wazuh Agent.

Test real-time monitoring by modifying a file:

```powershell
"WAZUH FIM REALTIME TEST $(Get-Date)" |
Out-File -FilePath "C:\Wazuh-FIM-Test\realtime-test.txt" -Append
```

Verify the resulting FIM event in Wazuh.

The project verified:

- File creation
- File modification
- File deletion

---

## 8. Install OpenCanary on Ubuntu

Update the Ubuntu package list:

```bash
sudo apt update
```

Install the required dependencies:

```bash
sudo apt install -y python3-venv python3-pip python3-dev libssl-dev libpcap-dev
```

Create a Python virtual environment:

```bash
python3 -m venv ~/opencanary-env
```

Activate the virtual environment:

```bash
source ~/opencanary-env/bin/activate
```

Install OpenCanary:

```bash
pip install opencanary
```

Generate the initial OpenCanary configuration:

```bash
opencanaryd --copyconfig
```

The configuration file is created at:

```text
/etc/opencanaryd/opencanary.conf
```

---

## 9. Configure OpenCanary Services

Edit the OpenCanary configuration:

```text
/etc/opencanaryd/opencanary.conf
```

Enable the following simulated services:

```text
FTP     → 2121
SSH     → 2222
Telnet  → 2323
HTTP    → 8888
```

Enable HTTP redirect request logging:

```text
http.log_redirect_request → true
```

OpenCanary writes its events to:

```text
/var/tmp/opencanary.log
```

Start OpenCanary:

```bash
sudo -E ~/opencanary-env/bin/opencanaryd --start
```

Verify that the configured honeypot services are running.

---

## 10. Connect OpenCanary to Wazuh

On the Ubuntu Wazuh Manager, open:

```text
/var/ossec/etc/ossec.conf
```

Add the OpenCanary log as a JSON log source:

```xml
<localfile>
    <location>/var/tmp/opencanary.log</location>
    <log_format>json</log_format>
</localfile>
```

Create custom Wazuh rules for OpenCanary activity.

The verified rules include:

```text
100501 → OpenCanary SSH connection
100502 → OpenCanary FTP login attempt
100503 → OpenCanary Telnet connection
```

An OpenCanary HTTP detection was also verified during testing.

Restart the Wazuh Manager after modifying its configuration.

---

## 11. Enable Wazuh Archives

On Ubuntu, open:

```text
/var/ossec/etc/ossec.conf
```

Ensure the following options are enabled:

```xml
<logall>yes</logall>
<logall_json>yes</logall_json>
```

Next, open:

```text
/etc/filebeat/filebeat.yml
```

Enable archive indexing:

```yaml
archives:
  enabled: true
```

Restart Filebeat:

```bash
sudo systemctl restart filebeat
```

The archive index pattern used in the Wazuh Dashboard is:

```text
wazuh-archives-*
```

---

## 12. Configure the Wazuh Dashboard

Open the Wazuh Dashboard.

Create an index pattern for Wazuh Archives:

```text
wazuh-archives-*
```

Use the following field as the time field:

```text
timestamp
```

The archive index can then be used to investigate events that may not have generated Wazuh alerts.

---

## 13. Test OpenCanary

From Kali Linux, perform controlled connections against the OpenCanary services.

### HTTP

Connect to:

```text
http://WAZUH_UBUNTU_IP:8888/
```

### SSH

Connect to:

```text
WAZUH_UBUNTU_IP:2222
```

### FTP

Connect to:

```text
WAZUH_UBUNTU_IP:2121
```

### Telnet

Connect to:

```text
WAZUH_UBUNTU_IP:2323
```

Replace `WAZUH_UBUNTU_IP` with the Ubuntu system's lab IP address.

Verify that the interactions appear in Wazuh.

---

## 14. Test Windows Telemetry

Generate controlled activity on the Windows endpoint.

### DNS Query

Run:

```powershell
Resolve-DnsName example.com
```

Verify the Sysmon DNS Query event in Wazuh Archives.

### Network Connection

Generate a controlled network connection and verify the Sysmon Network Connection event.

The project verified:

```text
Event ID 3 → Network Connection
```

### File Integrity Monitoring

Modify a file inside:

```text
C:\Wazuh-FIM-Test
```

Verify the resulting FIM event.

### PowerShell

Run a PowerShell command and verify:

```text
Event ID 4104
```

in Wazuh Archives.

### Windows Defender

Run:

```powershell
Start-MpScan -ScanType QuickScan
```

Verify the resulting Windows Defender telemetry in Wazuh.

---

## 15. Verify the Complete Pipeline

The completed monitoring pipeline should operate as follows:

```text
Kali Linux / Windows 11
          ↓
      Test Activity
          ↓
     Event Generation
          ↓
    Wazuh Collection
          ↓
   ┌──────┴───────┐
   ↓              ↓
Archives         Rules
   ↓              ↓
Telemetry        Alerts
   └──────┬───────┘
          ↓
   Wazuh Dashboard
```

The completed lab provides centralized visibility into:

- OpenCanary activity
- Windows Security events
- Sysmon telemetry
- File Integrity Monitoring
- PowerShell activity
- Windows Defender telemetry

---

