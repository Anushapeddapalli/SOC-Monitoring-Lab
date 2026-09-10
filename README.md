##🔐 SOC Monitoring & Threat Detection Lab

A hands-on Security Operations Center (SOC) monitoring and threat detection lab built using **Wazuh, OpenCanary, Sysmon, and Windows security telemetry**.

The project demonstrates centralized security monitoring, honeypot-based detection, Windows endpoint telemetry collection, file integrity monitoring, security event analysis, and investigation through a custom Wazuh dashboard.

---

## 🎯 Project Objective

The objective of this project is to build a centralized SOC monitoring environment where security events from different sources can be collected, analyzed, detected, and investigated from a single Wazuh platform.

The lab combines:

- 🪤 OpenCanary honeypot monitoring
- 🖥️ Windows endpoint monitoring
- 🔍 Sysmon telemetry
- 📁 File Integrity Monitoring (FIM)
- ⚡ PowerShell Script Block Logging
- 🛡️ Windows Defender telemetry
- 🚨 Wazuh alert detection
- 📦 Wazuh Archives for broader event visibility
- 📊 Custom SOC dashboard

---

## 🏗️ Architecture


                         ┌─────────────────┐
                         │   Kali Linux    │
                         │ Attacker / Test │
                         └────────┬────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
          ┌─────────────────┐          ┌─────────────────┐
          │   OpenCanary    │          │  Windows 11     │
          │    Honeypot     │          │    Endpoint     │
          └────────┬────────┘          └────────┬────────┘
                   │                            │
                   │ Logs                       │ Wazuh Agent
                   │                            │
                   └─────────────┬──────────────┘
                                 ▼
                       ┌───────────────────┐
                       │  Wazuh Manager    │
                       │ Detection &       │
                       │ Analysis           │
                       └─────────┬─────────┘
                                 │
                       ┌─────────▼─────────┐
                       │  Wazuh Indexer    │
                       │ Event Storage     │
                       └─────────┬─────────┘
                                 │
                       ┌─────────▼─────────┐
                       │ Wazuh Dashboard   │
                       │ SOC Monitoring    │
                       └───────────────────┘
🛠️ Technologies Used
Technology	Purpose
Wazuh	Security monitoring, event analysis and detection
OpenCanary	Honeypot and service interaction detection
Sysmon	Windows system and network telemetry
Windows 11	Monitored endpoint
Kali Linux	Security testing / attacker simulation
PowerShell	Endpoint activity and Script Block Logging
Windows Defender	Endpoint security telemetry
VirtualBox	Isolated virtual lab environment
🔎 Monitoring Components
🪤 OpenCanary

OpenCanary was configured as a honeypot with simulated services:

FTP — Port 2121
SSH — Port 2222
Telnet — Port 2323
HTTP — Port 8888

Interactions with these services generate OpenCanary logs which are collected by Wazuh and processed using custom detection rules.

🖥️ Windows Endpoint

The Windows endpoint sends security telemetry to the Wazuh Manager through the Wazuh Agent.

Collected sources include:

Windows Security
System
Application
Windows Defender
PowerShell
Sysmon
🔍 Sysmon

Sysmon provides detailed Windows telemetry.

The project specifically verified telemetry including:

Event ID 3 — Network Connection
Event ID 22 — DNS Query
Other Sysmon events collected through Wazuh Archives
📁 File Integrity Monitoring

Wazuh FIM monitors selected Windows directories for file changes.

Verified FIM activity includes:

File modification
File creation
File deletion
⚡ PowerShell Monitoring

PowerShell Script Block Logging was enabled to collect PowerShell operational events.

Event ID 4104 was verified in Wazuh Archives.

🛡️ Windows Defender

Windows Defender operational events are collected by the Wazuh Agent.

A Defender Quick Scan was performed and the resulting telemetry was successfully received by Wazuh.

🚨 Alerts vs Archives

An important part of this project is the distinction between alerts and archives.

Wazuh Alerts
Event / Log
     ↓
Wazuh Rule Engine
     ↓
Rule Match
     ↓
Alert Generated

The wazuh-alerts-* index contains events that triggered Wazuh detection rules.

Wazuh Archives
Event / Log
     ↓
Wazuh
     ↓
Archive

The wazuh-archives-* index provides broader event visibility, including telemetry that may not have generated an alert.

This allows the SOC analyst to use:

Alerts → Detection

Archives → Investigation / Telemetry

📊 SOC Dashboard

The project includes a custom Wazuh SOC dashboard containing:

Total Alerts
High/Critical Alerts
Alert Severity Distribution
Alert Trend
Detection Types
OpenCanary Activity
Windows Security Alerts
Windows Telemetry Sources
Sysmon Activity
PowerShell Activity
FIM Activity
Windows Defender Activity
Windows Event Activity

The dashboard also supports filtering for investigation, including agent and OpenCanary activity.

🧪 Attack & Detection Testing

Security testing was performed from Kali Linux against the isolated lab environment.

Verified OpenCanary scenarios include:

HTTP interaction
SSH connection
FTP login attempt
Telnet connection

Windows telemetry was also verified through controlled testing of:

DNS queries
Network connections
File changes
PowerShell activity
Windows Defender scanning
Windows security events
🚨 Custom Wazuh Detection Rules

Custom rules were created for OpenCanary activity.

Detection	Rule ID
OpenCanary SSH connection	100501
OpenCanary FTP login attempt	100502
OpenCanary Telnet connection	100503
OpenCanary HTTP connection	Custom OpenCanary HTTP rule

These rules allow OpenCanary events to become Wazuh alerts for centralized SOC monitoring.

📸 Screenshots

Screenshots of the following will be added to this repository:

SOC dashboard
OpenCanary activity
Wazuh alerts
Windows telemetry
Sysmon events
FIM activity
PowerShell events
Windows Defender events
🔄 Detection Flow
OpenCanary Detection
Kali Linux
    ↓
Connection to Honeypot Service
    ↓
OpenCanary
    ↓
OpenCanary Log
    ↓
Wazuh Manager
    ↓
Custom Detection Rule
    ↓
Wazuh Alert
    ↓
Wazuh Dashboard
Windows Telemetry Flow
Windows 11
    ↓
Wazuh Agent
    ↓
Windows Event Sources
    ├── Security
    ├── System
    ├── Application
    ├── Sysmon
    ├── PowerShell
    ├── Defender
    └── FIM
          ↓
    Wazuh Manager
          ↓
    ┌─────┴─────┐
    ↓           ↓
Archives      Rules
    ↓           ↓
Telemetry    Alerts
    └─────┬─────┘
          ↓
   Wazuh Dashboard
🔐 Security & Lab Disclaimer

This project was developed in an isolated virtualized lab environment for educational and defensive security monitoring purposes.

Testing was performed only against systems controlled by the project author.

No unauthorized systems or services were targeted.

Sensitive credentials, passwords, API keys, and other secrets should never be committed to this repository.

🚀 Future Improvements

Potential future improvements include:

Additional Wazuh detection rules
More Windows security telemetry
Automated incident response
Additional controlled attack simulations
Threat intelligence integration
Improved investigation workflows
Additional SOC dashboard visualizations
👩‍💻 Author

Anusha Peddapalli

Cybersecurity | SOC | Threat Detection | Security Monitoring
