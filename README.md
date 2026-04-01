# 🛡️ Azure Sentinel SIEM Lab

![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Microsoft%20Azure-0078D4?style=for-the-badge&logo=microsoftazure)
![SIEM](https://img.shields.io/badge/SIEM-Microsoft%20Sentinel-00B4D8?style=for-the-badge)
![KQL](https://img.shields.io/badge/Language-KQL-orange?style=for-the-badge)

> Building a cloud-native Security Operations Center (SOC) environment using Microsoft Sentinel to simulate real-world threat detection and incident response workflows.

---

## 📋 Project Overview

This lab demonstrates the setup and configuration of a fully functional SIEM environment in Azure. The goal is to simulate enterprise-level security monitoring, including log ingestion from multiple sources, custom KQL detection rules, automated incident creation, SOAR playbooks with email alerting, and a live SOC dashboard.

## 🎯 Objectives

- ✅ Deploy a Microsoft Sentinel workspace connected to a Windows VM log source
- ✅ Configure Azure Monitor Agent (AMA) via Data Collection Rules
- ✅ Write custom KQL detection rules for brute force attack patterns
- ✅ Create Analytics Rules that automatically generate Incidents
- ✅ Simulate a brute force attack and detect it in real-time
- ✅ Build incident response playbook using Azure Logic Apps (SOAR)
- ✅ Configure automated email alerting via Outlook when incidents are detected
- ✅ Add additional KQL detection rules (privilege escalation, new user creation)
- ✅ Create custom SOC dashboard (Workbook)

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Sentinel | SIEM / SOAR platform |
| Azure Log Analytics | Log ingestion & querying |
| KQL (Kusto Query Language) | Custom detection rules |
| Azure Monitor Agent (AMA) | Log collection from VM |
| Azure Logic Apps | Automated incident response (SOAR) |
| Outlook / Office 365 | Email alert delivery |
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
   ┌────┴────────────────────┐
   ▼                         ▼
KQL Analytics Rules     Automation Rule
        │                    │
        ▼                    ▼
    INCIDENTS          Logic App Playbook
                             │
                             ▼
                    📧 Email Notification
                    (Outlook - High Importance)
```

## 📁 Repository Structure

```
azure-sentinel-siem-lab/
├── README.md
├── docs/
│   └── screenshots/          # Step-by-step documentation screenshots
├── kql-queries/
│   └── detection-rules.md    # All KQL detection rules
└── playbooks/
    └── incident-response.md  # SOAR playbook documentation
```

---

## 🚀 Lab Walkthrough

### Phase 1 — Azure Infrastructure Setup

**Step 1: Log Analytics Workspace Deployment**

Deployed `sentinel-lab-workspace` as the central data store where all logs are ingested and queried. Resource Group `sentinel-lab-rg` created in West Europe.

![Log Analytics Deployment Complete](docs/screenshots/01-log-analytics-deployment-complete.png)

---

**Step 2: Microsoft Sentinel Enabled**

Enabled Microsoft Sentinel on top of the Log Analytics Workspace. Overview shows clean state — 0 incidents, 0 data connectors, ready to configure.

![Sentinel Overview Dashboard](docs/screenshots/02-sentinel-overview-dashboard.png)

---

**Step 3: Data Connectors — Empty State**

Confirmed the Data Connectors page starts with 0 connected sources before configuration.

![Data Connectors Empty](docs/screenshots/03-data-connectors-empty.png)

---

### Phase 2 — Data Collection

**Step 4: Windows Security Events Connector**

Located the **Windows Security Events** solution in the Content Hub. Includes 2 data connectors, 20 analytics rule templates, 2 workbooks, and 50 hunting queries.

![Content Hub - Not Installed](docs/screenshots/04-content-hub-windows-security-events.png)

![Content Hub - Installed](docs/screenshots/05-windows-security-events-installed.png)

---

**Step 5: Windows Server VM Deployment**

Deployed `sentinel-windows-vm` — Windows Server 2022 Datacenter, Standard B2ts v2, West Europe, with RDP enabled and auto-shutdown at 11:00 PM.

![VM Review and Create](docs/screenshots/06-vm-review-create.png)

![VM Deployment Complete](docs/screenshots/07-vm-deployment-complete.png)

All lab resources visible in the All Resources view: workspace, VM, network interfaces, storage.

![All Resources Overview](docs/screenshots/08-all-resources-overview.png)

![VM Overview - Running](docs/screenshots/09-vm-overview.png)

---

**Step 6: Data Collection Rule (DCR)**

Created `windows-security-events-dcr` to automatically install Azure Monitor Agent on the VM and stream **All Security Events** to the workspace.

![DCR Creation](docs/screenshots/10-data-collection-rule-create.png)

![DCR - VM Selected](docs/screenshots/11-dcr-resources-vm-selected.png)

![DCR Created - Connected](docs/screenshots/12-dcr-created.png)

---

### Phase 3 — Threat Detection

**Step 7: RDP Connection & Log Verification**

Connected to the VM via RDP. Server Manager Local Server confirms connection to `sentinel-window` (IP 51.124.185.72).

![RDP Connected - Server Manager](docs/screenshots/13-vm-rdp-connected.png)

Windows Event Viewer Security log showing EventID 4688 (Process Creation) — confirms security auditing is active.

![Event Viewer - Security Log](docs/screenshots/14-vm-event-viewer-security.png)

EventID 4625 (Audit Failure) and EventID 4724 visible alongside CMD prompt showing failed `net user labadmin wrongpassword123` attempts.

![Event Viewer - 4625 Audit Failure](docs/screenshots/14-vm-event-viewer-security-events.png)

EventID 4625 detail — `runas /user:fakeuser cmd` generating failed logon events.

![Event Viewer - runas fakeuser](docs/screenshots/15-vm-failed-logon-events-4625.png)

---

**Step 8: KQL — Logs Flowing into Sentinel**

Confirmed SecurityEvent table populated — 39 results (1,000 results shown) from `sentinel-window` in the last 24 hours.

![Sentinel Logs Flowing](docs/screenshots/16-sentinel-security-events-flowing.png)

**Verification KQL:**
```kql
SecurityEvent
| where EventID == 4625
| project TimeGenerated, Account, Computer, EventID, Activity
| order by TimeGenerated desc
```

4 EventID 4625 results confirmed from `sentinel-window\fakeuser`.

![KQL Failed Logon Results](docs/screenshots/17-kql-failed-logon-results.png)

---

**Step 9: Analytics Rule — Brute Force Detection**

Defender Analytics starts empty — 0 active rules.

![Defender Analytics - 0 Rules](docs/screenshots/18-defender-portal-analytics.png)

New Scheduled rule created using the Analytics rule wizard:

![Analytics Rule Wizard - Empty Form](docs/screenshots/19-analytics-rule-wizard.png)

**Rule: Brute Force - Multiple Failed Logons** configured with name, description, High severity, MITRE ATT&CK T1110.

![Analytics Rule - General Settings](docs/screenshots/19-analytics-rule-general.png)

KQL query and scheduling configured — runs every 5 minutes, looks back 5 minutes, threshold > 0:

```kql
SecurityEvent
| where EventID == 4625
| summarize FailedAttempts = count() by Account, Computer, bin(TimeGenerated, 5m)
| where FailedAttempts >= 5
```

![Analytics Rule - KQL and Scheduling](docs/screenshots/20-analytics-rule-scheduling.png)

![Analytics Rule - Incident Settings Disabled](docs/screenshots/21-analytics-rule-incident-settings.png)

![Analytics Rule - Alert Grouping Enabled](docs/screenshots/22-analytics-rule-incident-settings.png)

![Analytics Rule - Automated Response](docs/screenshots/23-analytics-rule-automated-response.png)

Review confirms all settings — Brute Force, High, T1110, 5min frequency, Alert grouping Enabled.

![Analytics Rule - Review and Create](docs/screenshots/24-analytics-rule-review.png)

Rule saved — **1 Active rule** confirmed. Toast: "Analytics rule 'Brute Force - Multiple Failed Logons' saved successfully."

![Analytics - 1 Active Rule](docs/screenshots/25-analytics-rule-active.png)

---

### Phase 4 — Incident Response

**Step 10: Brute Force Attack Simulation**

Simulated brute force with repeated `runas /user:fakeuser cmd` failing — "The user name or password is incorrect" in a continuous loop.

```cmd
for /L %i in (1,1,20) do runas /user:fakeuser cmd
```

![VM - Brute Force Simulation](docs/screenshots/26-vm-brute-force-simulation.png)

Sentinel confirmed **55 EventID 4625 events** from `sentinel-window\fakeuser`.

![Sentinel Logs - 55 Events](docs/screenshots/27-sentinel-failed-logons-55-events.png)

---

**Step 11: Incident Auto-Generated**

Microsoft Sentinel automatically created a **High severity Incident** — "Brute Force - Multiple Failed Logons".

![Incidents - Brute Force Detected](docs/screenshots/28-incident-brute-force-detected.png)

Incident details: Account `sentinel-window\fakeuser`, **24 FailedAttempts**, Credential Access category.

![Incident Details - fakeuser 24 Attempts](docs/screenshots/29-incident-details-brute-force.png)

---

### Phase 5 — Automation (SOAR)

**Step 12: Logic App Deployment**

Deployed `sentinel-brute-force-playbook` Logic App in `sentinel-lab-rg`, West Europe.

![Logic App Deployment Complete](docs/screenshots/30-logic-app-deployment-complete.png)

Logic App Overview — 0 triggers, 0 actions, Stateful workflow, Enabled.

![Logic App Overview](docs/screenshots/31-logic-app-overview.png)

---

**Step 13: Logic App Designer — Trigger & Actions**

Empty designer — "Add a trigger" ready for configuration.

![Logic App Designer Empty](docs/screenshots/32-logic-app-designer-empty.png)

Microsoft Sentinel incident trigger added — connected to `live.com#kotsvagg83@gmail.com` via OAuth.

![Sentinel Trigger Connected](docs/screenshots/33-logic-app-sentinel-connected.png)

Create connection panel — OAuth authentication, Default Directory tenant.

![Create Connection - OAuth](docs/screenshots/33-logic-app-sentinel-connection.png)

Gmail (SOC-ALERT) connection attempted first — `kotsvagg83@gmail.com` connected.

> **Note:** Gmail connector is blocked by Microsoft policy when combined with Sentinel (`GmailConnectorPolicyViolation`). Switched to Outlook.com.

![Gmail Connection Attempt](docs/screenshots/34-logic-app-gmail-connection.png)

Gmail email body configured with dynamic Sentinel fields (Incident Title, Severity, Status, Created Time UTC).

![Gmail Email Body - Dynamic Fields](docs/screenshots/37-logic-app-complete-email-body.png)

Final configuration switched to **Outlook Send an email (V2)** — `kotsvagg83@hotmail.com`, dynamic fields, connected to Outlook.com.

![Outlook Email Configured](docs/screenshots/37-logic-app-outlook-email-configured.png)

---

**Step 14: Playbook Saved**

Logic App saved successfully. Notification: "Successfully saved workflow 'sentinel-brute-force-playbook'". Final flow: `[Microsoft Sentinel incident] → [Send an email (V2)]`.

![Logic App Saved Successfully](docs/screenshots/38-logic-app-saved-successfully.png)

---

**Step 15: Sentinel Permissions & Automation Rule**

Automation page — 0 rules, 1 enabled playbook.

![Automation Page](docs/screenshots/39-automation-page.png)

Granted Sentinel permissions to run playbooks by selecting `sentinel-lab-rg`.

![Manage Permissions - sentinel-lab-rg](docs/screenshots/40-sentinel-permissions-playbook.png)

Created Automation Rule **"Brute Force - Run Playbook"**:
- Trigger: When incident is created
- Condition: Analytic rule name Contains "Brute Force - Multiple Failed Logons"
- Action: Run playbook → `sentinel-brute-force-playbook`
- Expiration: Indefinite, Order: 1

![Automation Rule Configured](docs/screenshots/41-automation-rule-configured.png)

Automation Rule active — 1 rule, 1 enabled rule, 1 enabled playbook confirmed.

![Automation Rule Active](docs/screenshots/42-automation-rule-active.png)

---

**Step 16: End-to-End Validation**

Second brute force simulation to test the full automation chain:

```cmd
for /L %i in (1,1,20) do runas /user:fakeuser cmd
```

![VM - Brute Force Simulation 2](docs/screenshots/43-vm-brute-force-simulation-2.png)

Logic App Run history — **2 successful runs** (9:28 PM and 9:23 PM, both Succeeded).

![Logic App - 2 Successful Runs](docs/screenshots/46-logic-app-run-succeeded.png)

Automated email received in Outlook inbox — High importance, all dynamic fields populated:
- **Subject:** 🚨 Sentinel Alert: Brute Force Incident Detected
- **Incident Title:** Brute Force - Multiple Failed Logons
- **Severity:** High
- **Status:** New
- **Time:** 2026-03-30T18:26:14.8833333Z
- **Message:** "Please investigate immediately in Microsoft Sentinel."

![Email Alert Received](docs/screenshots/47-email-alert-with-dynamic-data.png)

**Full SOAR chain validated:** Attack → Sentinel detection → Incident created → Automation Rule triggered → Logic App executed → Email delivered ✅

---

### Phase 6 — Additional KQL Detection Rules

**Step 17: Three Additional Analytics Rules**

---

**Rule 1 — Successful Login After Multiple Failures**

Detects a successful logon (EventID 4624) from an account that previously generated 3+ failed logons (EventID 4625) within 1 hour. High-fidelity indicator of a completed brute force attack.

```kql
let FailedLogins = SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(1h)
| summarize FailedCount = count() by Account, IpAddress = IpAddress
| where FailedCount >= 3;
SecurityEvent
| where EventID == 4624
| where TimeGenerated > ago(1h)
| join kind=inner FailedLogins on Account
| project TimeGenerated, Account, IpAddress, FailedCount, Computer
| order by TimeGenerated desc
```

- **Severity:** High | **MITRE:** Credential Access / T1110
- **Frequency:** Every 5 min | **Lookup:** Last 1 hour

Analytics dashboard showing 2 active rules — Rule 1 selected with full details panel visible.

![Rule 1 - Successful Login After Multiple Failures](docs/screenshots/48-rule1-successful-login-after-failures.png)

---

**Rule 2 — New User Account Created**

Detects creation of a new local user account (EventID 4720). Common persistence technique after initial compromise.

```kql
SecurityEvent
| where EventID == 4720
| project TimeGenerated, Account, Computer, EventID
| order by TimeGenerated desc
```

- **Severity:** Medium | **MITRE:** Persistence / T1136 - Create Account
- **Frequency:** Every 5 min | **Lookup:** Last 1 hour

Analytics dashboard showing 3 active rules — Rule 2 saved with toast notification "Analytics rule 'New User Account Created' saved successfully."

![Rule 2 - New User Account Created](docs/screenshots/49-rule2-new-user-account-created.png)

---

**Rule 3 — User Added to Admin Group**

Detects when a user is added to the local Administrators group (EventID 4732). Privilege escalation indicator.

```kql
SecurityEvent
| where EventID == 4732
| project TimeGenerated, Account, Computer, EventID
| order by TimeGenerated desc
```

- **Severity:** High | **MITRE:** Privilege Escalation / T1078 - Valid Accounts
- **Frequency:** Every 5 min | **Lookup:** Last 1 hour

> **Note:** Rule 3 screenshot was not captured separately — it is visible as the 3rd active rule in the Analytics list (screenshot 49 shows 3 active rules).

---

**Step 18: Attack Simulation — All Rules**

**Stage 1 — Brute force** (`net use` against localhost, triggering Brute Force rule):

```cmd
for /L %i in (1,1,10) do net use \\localhost\IPC$ /user:testuser wrongpassword 2>nul
```

![Attack Simulation - net use loop (testuser)](docs/screenshots/51-brute-force-incidents-triggered.png)

**Stages 2 & 3 — New user + privilege escalation:**

```cmd
net user hacker P@ssword123! /add
net localgroup Administrators hacker /add
```

Full attack chain visible — brute force loop (`labadmin` + `LabAdmin@2025!`), followed by `net user hacker /add` and `net localgroup Administrators hacker /add`.

![Attack Simulation - Full Attack Chain](docs/screenshots/55-all-three-incidents-active.png)

---

**Step 19: Incident Investigation**

Brute Force alert details — query results show 8 items: `administrator` (60 failed attempts), `user` (15), `scans` (6), confirming mass authentication attempts detected.

![Brute Force Alert Details](docs/screenshots/52-brute-force-alert-details.png)

All three incident types generated — **User Added to Admin Group** (High, Privilege escalation), **New User Account Created** (Medium, Persistence), **Brute Force - Multiple Failed Logons** (High, Credential access).

![All Three Incident Types Active](docs/screenshots/56-attack-simulation-commands.png)

Also confirmed: Brute Force incidents list showing multiple detections over 1 week (Incident IDs 1 and 2).

![Brute Force Incidents List](docs/screenshots/51-brute-force-incidents-triggered__1_.png)

> **Note on Rule 1:** The `net use` simulation did not produce matching EventID 4625 + 4624 pairs for the same account under NTLM. The rule logic is valid and would fire correctly in a real network brute-force scenario.

---

### Phase 7 — SOC Dashboard

**Step 20: Custom Workbook — SOC Dashboard**

Built **"SOC Dashboard - Azure Sentinel Lab"** with 4 live visualization panels via Sentinel Workbooks JSON editor:

| Panel | Type | What it shows |
|---|---|---|
| Incidents by Severity | Donut chart | High (19) vs Medium (6) — Total 25 |
| Incidents Over Time | Time chart | Attack spike timeline Mar 29–30 |
| Top 10 Failed Login Accounts | Bar chart | `sentinel-window\administrator` — 605 attempts |
| Recent Incidents | Table | Latest incidents: "User Added to Admin Group", High, New |

![SOC Dashboard - Azure Sentinel Lab](docs/screenshots/57-soc-dashboard-saved.png)

---

## ✅ Progress Checklist

**Phase 1 — Infrastructure Setup**
- [x] Azure subscription & resource group (`sentinel-lab-rg`)
- [x] Log Analytics Workspace (`sentinel-lab-workspace`)
- [x] Microsoft Sentinel enablement

**Phase 2 — Data Collection**
- [x] Windows Security Events connector (Content Hub)
- [x] Windows Server 2022 VM (`sentinel-windows-vm`)
- [x] Azure Monitor Agent via Data Collection Rule
- [x] All Security Events ingestion confirmed

**Phase 3 — Threat Detection**
- [x] KQL query for EventID 4625 (Failed Logon)
- [x] Analytics Rule — Brute Force (5+ attempts / 5 min)
- [x] MITRE ATT&CK mapping (T1110)

**Phase 4 — Incident Response**
- [x] Brute force simulation (20+ failed logins)
- [x] Real-time incident generation & investigation

**Phase 5 — Automation (SOAR)**
- [x] Azure Logic App deployed (`sentinel-brute-force-playbook`)
- [x] Microsoft Sentinel incident trigger (OAuth)
- [x] Outlook email action with dynamic incident fields
- [x] Sentinel permissions granted (`sentinel-lab-rg`)
- [x] Automation Rule: "Brute Force - Run Playbook"
- [x] End-to-end validated: attack → incident → email ✅

**Phase 6 — Additional Detection Rules**
- [x] Successful Login After Multiple Failures (T1110)
- [x] New User Account Created (T1136)
- [x] User Added to Admin Group (T1078)
- [x] All rules tested with live VM simulation

**Phase 7 — SOC Dashboard**
- [x] Incidents by Severity (donut chart)
- [x] Incidents Over Time (time chart)
- [x] Top 10 Failed Login Accounts (bar chart)
- [x] Recent Incidents (table)
- [x] Saved as "SOC Dashboard - Azure Sentinel Lab"

---

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

### Successful Login After Multiple Failures
```kql
let FailedLogins = SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(1h)
| summarize FailedCount = count() by Account, IpAddress = IpAddress
| where FailedCount >= 3;
SecurityEvent
| where EventID == 4624
| where TimeGenerated > ago(1h)
| join kind=inner FailedLogins on Account
| project TimeGenerated, Account, IpAddress, FailedCount, Computer
| order by TimeGenerated desc
```

### New User Account Created
```kql
SecurityEvent
| where EventID == 4720
| project TimeGenerated, Account, Computer, EventID
| order by TimeGenerated desc
```

### User Added to Admin Group
```kql
SecurityEvent
| where EventID == 4732
| project TimeGenerated, Account, Computer, EventID
| order by TimeGenerated desc
```

---

## 🗺️ MITRE ATT&CK Coverage

| Tactic | Technique | Detection Rule |
|---|---|---|
| Credential Access | T1110 - Brute Force | Brute Force - Multiple Failed Logons |
| Credential Access | T1110 - Brute Force | Successful Login After Multiple Failures |
| Persistence | T1136 - Create Account | New User Account Created |
| Privilege Escalation | T1078 - Valid Accounts | User Added to Admin Group |

---

## 📌 What I Learned

- How to deploy and configure Microsoft Sentinel from scratch in Azure
- How Azure Monitor Agent (AMA) collects and forwards Windows Security Events
- How to write KQL queries for real-time security event analysis
- How Analytics Rules automate threat detection and incident creation
- How to build SOAR playbooks with Azure Logic Apps for automated response
- How to integrate Sentinel with Outlook to deliver email alerts with dynamic incident data
- How to simulate real attack scenarios (brute force, persistence, privilege escalation) and verify detection
- How to build live SOC dashboards using Sentinel Workbooks
- The complete SOC workflow: log ingestion → KQL detection → incident → SOAR → email alert

---

*Part of my cybersecurity portfolio — [evank95.github.io/My-CV-Page](https://evank95.github.io/My-CV-Page/)*
