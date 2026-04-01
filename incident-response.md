# 🚨 Incident Response Playbook

This file documents the automated SOAR playbook configured in Microsoft Sentinel for incident response workflows.

---

## 📋 Playbook Overview

| Field | Details |
|---|---|
| **Playbook Name** | Sentinel-Incident-Email-Alert |
| **Trigger** | Microsoft Sentinel Incident |
| **Action** | Send email notification via Office 365 |
| **Status** | Active |

---

## 🔧 How It Works

When Microsoft Sentinel creates or updates an incident, this Logic App playbook is automatically triggered. It extracts key incident details and sends a formatted email alert to the SOC analyst.

```
Sentinel Incident Created
        │
        ▼
Logic App Triggered (HTTP Trigger)
        │
        ▼
Parse Incident Entity (Sentinel connector)
        │
        ▼
Send Email via Office 365
        │
        ▼
Incident Notification Delivered
```

---

## 📧 Email Notification Contents

The automated email includes the following fields extracted from the incident:

- **Incident Title** — Name of the triggered analytics rule
- **Severity** — High / Medium / Low / Informational
- **Status** — New / Active / Closed
- **Incident URL** — Direct link to the incident in Sentinel portal
- **Description** — Rule description and triggered condition
- **Time Created** — Timestamp of incident creation

---

## ⚙️ Logic App Configuration

### Step 1 — Trigger
- **Connector:** Microsoft Sentinel
- **Trigger type:** When a response to a Microsoft Sentinel alert is triggered

### Step 2 — Get Incident Details
- **Action:** Microsoft Sentinel — Get incident
- **Input:** Incident ARM ID from trigger body

### Step 3 — Send Email
- **Connector:** Office 365 Outlook
- **Action:** Send an email (V2)
- **To:** SOC analyst email address
- **Subject:** `[Sentinel Alert] {{Incident Title}} — {{Severity}}`
- **Body:** Formatted HTML with all incident fields

---

## 🔗 Automation Rule

An Automation Rule in Sentinel is configured to trigger this playbook automatically:

| Setting | Value |
|---|---|
| **Rule Name** | Auto-Notify on High Severity |
| **Trigger** | When incident is created |
| **Condition** | Severity equals High OR Medium |
| **Action** | Run playbook → Sentinel-Incident-Email-Alert |

---

## 📸 Screenshots

Screenshots of the Logic App configuration and triggered email alerts are available in `docs/screenshots/`.
