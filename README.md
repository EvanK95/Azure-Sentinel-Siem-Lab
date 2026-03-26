# 🛡️ Azure Sentinel SIEM Lab

![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Microsoft%20Azure-0078D4?style=for-the-badge&logo=microsoftazure)
![SIEM](https://img.shields.io/badge/SIEM-Microsoft%20Sentinel-00B4D8?style=for-the-badge)

> Building a cloud-native Security Operations Center (SOC) environment using Microsoft Sentinel to simulate real-world threat detection and incident response workflows.

---

## 📋 Project Overview

This lab demonstrates the setup and configuration of a fully functional SIEM environment in Azure. The goal is to simulate enterprise-level security monitoring, including log ingestion from multiple sources, custom detection rules, and automated incident response.

## 🎯 Objectives

- Deploy a Microsoft Sentinel workspace connected to multiple log sources
- Write custom KQL (Kusto Query Language) detection rules for common attack patterns
- Build incident response playbooks using Azure Logic Apps (SOAR)
- Integrate Microsoft Defender for Endpoint as a data connector
- Create custom dashboards for SOC analyst workflows
- Simulate and detect attacks (brute force, suspicious logins, lateral movement)

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Sentinel | SIEM / SOAR platform |
| Azure Log Analytics | Log ingestion & querying |
| KQL | Custom detection rules |
| Azure Logic Apps | Automated incident response |
| Microsoft Defender for Endpoint | Endpoint telemetry |
| Azure Monitor | Metrics and alerting |

## 📁 Repository Structure

```
azure-sentinel-siem-lab/
├── README.md
├── docs/
│   └── architecture-diagram.png      # Coming soon
├── kql-queries/
│   └── detection-rules.md            # Coming soon
├── playbooks/
│   └── incident-response.md          # Coming soon
└── screenshots/
    └── ...                           # Coming soon
```

## 🚧 Progress

- [ ] Azure environment setup & Sentinel workspace deployment
- [ ] Data connectors configuration (Defender, Azure AD, Syslog)
- [ ] Custom KQL detection rules
- [ ] Logic Apps playbooks (automated response)
- [ ] Custom SOC dashboard
- [ ] Attack simulation & detection walkthrough
- [ ] Final documentation & write-up

## 📌 Status: In Progress

This project is actively being built. Documentation, screenshots, and configuration files will be added as each phase is completed.

---

*Part of my cybersecurity portfolio — [evank95.github.io/my-personal-website](https://evank95.github.io/my-personal-website/)*
