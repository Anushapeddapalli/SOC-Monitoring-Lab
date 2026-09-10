## 🧪 Security Testing & Validation

This document records the security testing performed against the isolated SOC lab and the telemetry observed in Wazuh.

All testing was performed against systems within the controlled virtual lab environment.

---

## 1. OpenCanary Service Interaction Testing

OpenCanary was tested from the Kali Linux system by interacting with the simulated services exposed by the honeypot.

### HTTP

A request was sent to the OpenCanary HTTP service.

```bash
curl -i http://WAZUH_UBUNTU_IP:8888/
```

**Expected result:**

The request should generate an OpenCanary HTTP event.

**Observed result:**

The HTTP interaction was recorded by OpenCanary and successfully processed by Wazuh.

---

### SSH

A connection was attempted against the OpenCanary SSH service:

```bash
ssh -p 2222 testuser@WAZUH_UBUNTU_IP
```

**Expected result:**

The connection attempt should generate an OpenCanary SSH event.

**Observed result:**

The SSH connection attempt was recorded by OpenCanary and generated a Wazuh alert.

---

### FTP

An FTP connection was attempted against port `2121`.

```text
FTP → WAZUH_UBUNTU_IP:2121
```

**Expected result:**

The connection/login attempt should generate an OpenCanary FTP event.

**Observed result:**

The FTP login attempt was recorded and generated a Wazuh alert.

---

### Telnet

A Telnet connection was attempted against port `2323`.

```text
Telnet → WAZUH_UBUNTU_IP:2323
```

**Expected result:**

The connection should generate an OpenCanary Telnet event.

**Observed result:**

The Telnet interaction was recorded by OpenCanary and generated a Wazuh alert.

---

## 2. Sysmon Network Telemetry

Sysmon was used to provide additional Windows endpoint telemetry.

### Event ID 3 — Network Connection

A controlled network connection was generated from the Windows endpoint.

Example test:

```powershell
Test-NetConnection 8.8.8.8 -Port 443
```

**Expected result:**

Sysmon should record a Network Connection event.

**Observed result:**

Sysmon Event ID `3` was received by the Wazuh Agent and was visible in Wazuh Archives.

---

## 3. Sysmon DNS Telemetry

### Event ID 22 — DNS Query

A DNS query was generated on the Windows endpoint:

```powershell
Resolve-DnsName example.com
```

**Expected result:**

Sysmon should generate a DNS Query event.

**Observed result:**

Sysmon Event ID `22` was successfully received by Wazuh and was visible in Wazuh Archives.

The event included DNS query information such as the queried domain and originating process.

---

## 4. File Integrity Monitoring

Wazuh File Integrity Monitoring was tested using the dedicated directory:

```text
C:\Wazuh-FIM-Test
```

A file was created and modified using:

```powershell
"WAZUH FIM REALTIME TEST $(Get-Date)" |
Out-File -FilePath "C:\Wazuh-FIM-Test\realtime-test.txt" -Append
```

**Expected result:**

The file change should be detected by Wazuh FIM.

**Observed result:**

The FIM event was received in Wazuh with real-time monitoring information.

The following FIM activities were verified:

- File creation
- File modification
- File deletion

The events included file paths and integrity-related information such as hashes and modification details.

---

## 5. PowerShell Activity

PowerShell Script Block Logging was enabled on the Windows endpoint.

A controlled PowerShell command was executed:

```powershell
Write-Output "WAZUH POWERSHELL TEST $(Get-Date)"
```

**Expected result:**

PowerShell Script Block Logging should generate an operational event.

**Observed result:**

PowerShell Event ID `4104` was successfully collected by the Wazuh Agent and observed in Wazuh Archives.

---

## 6. Windows Security Events

Windows Security auditing was enabled and collected by the Wazuh Agent.

A successful Windows logon event was verified.

### Event ID 4624

**Observed result:**

Windows Security Event ID `4624` was successfully received by Wazuh.

This confirmed that Windows Security Event Channel telemetry was reaching the monitoring platform.

---

## 7. Windows Defender Telemetry

Windows Defender monitoring was tested using a controlled Quick Scan.

The scan was initiated with:

```powershell
Start-MpScan -ScanType QuickScan
```

**Expected result:**

Windows Defender should generate operational telemetry describing the scan.

**Observed result:**

Defender Event ID `1001` was successfully received by Wazuh.

The event identified the scan as a completed Quick Scan.

---

## 8. Wazuh Alert Validation

The tests were used to validate two different types of security data in Wazuh.

### Alert-Generating Events

OpenCanary interactions were processed through custom Wazuh detection rules and generated alerts.

Examples include:

- OpenCanary SSH connection
- OpenCanary FTP login attempt
- OpenCanary Telnet connection
- OpenCanary HTTP connection

### Archived Telemetry

Several Windows telemetry sources were successfully collected in Wazuh Archives even when they were not necessarily converted into alerts.

Examples include:

- Sysmon Event ID 3
- Sysmon Event ID 22
- PowerShell Event ID 4104
- Windows Defender events
- Windows Security events
- FIM events

---

## 9. Validation Summary

| Test | Source | Result |
|---|---|---|
| HTTP interaction | OpenCanary | Alert generated |
| SSH connection | OpenCanary | Alert generated |
| FTP login attempt | OpenCanary | Alert generated |
| Telnet connection | OpenCanary | Alert generated |
| Network connection | Sysmon Event 3 | Telemetry received |
| DNS query | Sysmon Event 22 | Telemetry received |
| File creation/modification/deletion | Wazuh FIM | Events received |
| PowerShell activity | PowerShell Event 4104 | Telemetry received |
| Windows logon | Security Event 4624 | Telemetry received |
| Quick Scan | Windows Defender Event 1001 | Telemetry received |

---

## 🔎 Investigation Workflow

The validated monitoring workflow is:

```text
Controlled Test Activity
          ↓
Event Generated
          ↓
Wazuh Agent / OpenCanary
          ↓
Wazuh Manager
          ↓
    ┌─────┴─────┐
    ↓           ↓
 Alerts      Archives
    ↓           ↓
Detection   Investigation
    └─────┬─────┘
          ↓
   Wazuh Dashboard
```

The testing demonstrates that the lab can collect endpoint and honeypot telemetry centrally and provide both detection and investigation data through the Wazuh Dashboard.
