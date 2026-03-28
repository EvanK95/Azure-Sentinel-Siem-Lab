# 🛡️ Azure Sentinel SIEM Lab

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Microsoft%20Azure-0078D4?style=for-the-badge&logo=microsoftazure)
![SIEM](https://img.shields.io/badge/SIEM-Microsoft%20Sentinel-00B4D8?style=for-the-badge)
![KQL](https://img.shields.io/badge/Language-KQL-orange?style=for-the-badge)

> Building a cloud-native Security Operations Center (SOC) environment using Microsoft Sentinel to simulate real-world threat detection and incident response workflows.

---

## 📋 Project Overview

This lab demonstrates the setup and configuration of a fully functional SIEM environment in Azure. The goal is to simulate enterprise-level security monitoring, including log ingestion from multiple sources, custom KQL detection rules, and automated incident creation.

## 🎯 Objectives

- ✅ Deploy a Microsoft Sentinel workspace connected to a Windows VM log source
- ✅ Configure Azure Monitor Agent (AMA) via Data Collection Rules
- ✅ Write custom KQL detection rules for brute force attack patterns
- ✅ Create Analytics Rules that automatically generate Incidents
- ✅ Simulate a brute force attack and detect it in real-time
- ⬜ Build incident response playbooks using Azure Logic Apps (SOAR)
- ⬜ Add additional KQL detection rules (privilege escalation, new user creation)
- ⬜ Create custom SOC dashboard (Workbook)

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Sentinel | SIEM / SOAR platform |
| Azure Log Analytics | Log ingestion & querying |
| KQL (Kusto Query Language) | Custom detection rules |
| Azure Monitor Agent (AMA) | Log collection from VM |
| Windows Server 2022 | Log source (attack simulation) |
| Microsoft Defender Portal | Incident management |

## 🏗️ Lab Architecture

```
Windows Server 2022 VM
        │
        ▼
Azure Monitor Agent (AMA)
        │
        ▼
Log Analytics Workspace (sentinel-lab-workspace)
        │
        ▼
Microsoft Sentinel
        │
   ┌────┴────┐
   ▼         ▼
KQL Rules  Analytics Rules
              │
              ▼
         INCIDENTS
```

## 📁 Repository Structure

```
azure-sentinel-siem-lab/
├── README.md
├── docs/
│   └── screenshots/          # Step-by-step documentation screenshots
├── kql-queries/
│   └── detection-rules.md    # Coming soon
└── playbooks/
    └── incident-response.md  # Coming soon
```

---

## 🚀 Lab Walkthrough

### Phase 1 — Azure Infrastructure Setup

**Step 1: Azure Subscription & Resource Group**

Created a dedicated Resource Group `sentinel-lab-rg` in West Europe to contain all lab resources.

![Resource Group](docs/screenshots/01-log-analytics-deployment-complete.png)

---

**Step 2: Log Analytics Workspace**

Deployed `sentinel-lab-workspace` — the data store where all logs are ingested and queried by Sentinel.

![Log Analytics Workspace](docs/screenshots/02-sentinel-overview-dashboard.png)

---

**Step 3: Microsoft Sentinel Deployment**

Enabled Microsoft Sentinel on top of the Log Analytics Workspace. This is the SIEM layer that provides threat detection, incident management, and SOAR capabilities.

![Sentinel Overview](docs/screenshots/02-sentinel-overview-dashboard.png)

---

### Phase 2 — Data Collection

**Step 4: Windows Security Events Connector**

Installed the **Windows Security Events** solution from the Sentinel Content Hub. This includes:
- 2 Data Connectors
- 20 Analytics Rule templates
- 2 Workbooks
- 50 Hunting Queries

![Content Hub](docs/screenshots/05-windows-security-events-installed.png)

---

**Step 5: Windows Server VM Deployment**

Deployed a Windows Server 2022 Datacenter VM (`sentinel-windows-vm`) in Azure to act as a log source.

- **Size:** Standard B2ts v2
- **Region:** West Europe
- **Auto-shutdown:** 11:00 PM (cost control)

![VM Overview](docs/screenshots/09-vm-overview.png)

---

**Step 6: Data Collection Rule (Azure Monitor Agent)**

Created a Data Collection Rule to automatically install the Azure Monitor Agent on the VM and stream **All Security Events** to the Log Analytics Workspace.

```
Windows VM → Azure Monitor Agent → Log Analytics → Sentinel
```

![Data Collection Rule](docs/screenshots/11-dcr-resources-vm-selected.png)

---

### Phase 3 — Threat Detection

**Step 7: RDP Connection & Attack Simulation**

Connected to the VM via RDP and simulated a brute force attack by generating multiple failed login attempts using `runas /user:fakeuser cmd`.

![Event Viewer](docs/screenshots/15-vm-failed-logon-events-4625.png)

Windows Security Event **ID 4625** (Failed Logon) captured in Event Viewer.

---

**Step 8: KQL Query — Failed Logon Detection**

Ran the first KQL query in Microsoft Sentinel to detect failed logon events (Event ID 4625):

```kql
SecurityEvent
| where EventID == 4625
| project TimeGenerated, Account, Computer, EventID, Activity
| order by TimeGenerated desc
```

![KQL Results](docs/screenshots/17-kql-failed-logon-results.png)

Successfully retrieved **4 failed logon events** from the `sentinel-window\fakeuser` account.

---

**Step 9: Analytics Rule — Brute Force Detection**

Created a custom scheduled Analytics Rule to automatically detect brute force attacks:

```kql
SecurityEvent
| where EventID == 4625
| summarize FailedAttempts = count() by Account, Computer, bin(TimeGenerated, 5m)
| where FailedAttempts >= 5
```

**Rule Configuration:**
- **Name:** Brute Force - Multiple Failed Logons
- **Severity:** High
- **MITRE ATT&CK:** Credential Access / T1110 - Brute Force
- **Frequency:** Every 5 minutes
- **Lookup period:** Last 5 minutes
- **Threshold:** > 0 results

![Analytics Rule](docs/screenshots/25-analytics-rule-active.png)

---

### Phase 4 — Incident Response

**Step 10: Brute Force Attack Simulation**

Simulated a realistic brute force attack by running 20+ failed login attempts in rapid succession using a batch command on the Windows VM.

![Brute Force Simulation](docs/screenshots/26-vm-brute-force-simulation.png)

---

**Step 11: Incident Auto-Generated**

Microsoft Sentinel automatically detected the attack and created a **High severity Incident** in the Microsoft Defender portal.

![Incident Detected](docs/screenshots/28-incident-brute-force-detected.png)

---

**Step 12: Incident Investigation**

Investigated the incident details showing:
- **Account targeted:** `sentinel-window\fakeuser`
- **Failed attempts detected:** 24 within 5 minutes
- **Detection time:** Real-time (within 5 minutes of attack)
- **MITRE tactic:** Credential Access

![Incident Details](docs/screenshots/29-incident-details-brute-force.png)

---

## ✅ Progress Checklist

**Phase 1 — Infrastructure Setup**
- [x] Azure subscription & resource group (`sentinel-lab-rg`)
- [x] Log Analytics Workspace deployment (`sentinel-lab-workspace`)
- [x] Microsoft Sentinel enablement

**Phase 2 — Data Collection**
- [x] Windows Security Events connector installation (Content Hub)
- [x] Windows Server 2022 VM deployment (`sentinel-windows-vm`)
- [x] Azure Monitor Agent via Data Collection Rule
- [x] All Security Events ingestion confirmed

**Phase 3 — Threat Detection**
- [x] KQL detection query for Event ID 4625 (Failed Logon)
- [x] Custom Analytics Rule — Brute Force detection (5+ attempts / 5 min)
- [x] MITRE ATT&CK mapping (T1110 — Brute Force)

**Phase 4 — Incident Response**
- [x] Brute force attack simulation (20+ failed logins)
- [x] Real-time incident generation & investigation (24 failed attempts detected)

**Phase 5 — Automation (SOAR)**
- [ ] Azure Logic App playbook — automated incident response
- [ ] Auto-notification on High severity incidents

**Phase 6 — Additional Detection Rules**
- [ ] Successful login after multiple failures (potential successful brute force)
- [ ] New user account creation detection
- [ ] Privilege escalation detection

**Phase 7 — SOC Dashboard**
- [ ] Custom Workbook with failed login trends
- [ ] Top targeted accounts visualization
- [ ] Attack timeline dashboard

**Phase 8 — Final Documentation**
- [ ] Architecture diagram
- [ ] Complete KQL queries documentation
- [ ] Final write-up & lessons learned

## 🔑 Key KQL Queries

### Failed Logon Detection
```kql
SecurityEvent
| where EventID == 4625
| project TimeGenerated, Account, Computer, EventID, Activity
| order by TimeGenerated desc
```

### Brute Force Detection (5+ attempts in 5 minutes)
```kql
SecurityEvent
| where EventID == 4625
| summarize FailedAttempts = count() by Account, Computer, bin(TimeGenerated, 5m)
| where FailedAttempts >= 5
```

---

## 📌 What I Learned

- How to deploy and configure Microsoft Sentinel from scratch
- How Azure Monitor Agent (AMA) collects and forwards logs
- How to write KQL queries for security event analysis
- How Analytics Rules work to automate threat detection
- The complete SOC workflow: log ingestion → detection → incident

---

*Part of my cybersecurity portfolio — [evank95.github.io/My-CV-Page](https://evank95.github.io/My-CV-Page/)*
