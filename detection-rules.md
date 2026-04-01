# 🔍 KQL Detection Rules

This file documents all custom KQL queries used in the Azure Sentinel SIEM Lab for threat detection and security monitoring.

---

## 📋 Table of Contents

- [Failed Login Attempts](#failed-login-attempts)
- [Brute Force Detection](#brute-force-detection)
- [Successful Login After Multiple Failures](#successful-login-after-multiple-failures)
- [Logon Outside Business Hours](#logon-outside-business-hours)
- [New User Account Created](#new-user-account-created)

---

## Failed Login Attempts

**Purpose:** Detect any failed login attempts against Windows VMs ingested via AMA.

```kql
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(1h)
| project TimeGenerated, Account, Computer, IpAddress, LogonTypeName
| order by TimeGenerated desc
```

**Trigger:** Any failed logon (Event ID 4625)  
**Severity:** Low  

---

## Brute Force Detection

**Purpose:** Identify potential brute force attacks by flagging accounts with more than 5 failed logins within 10 minutes.

```kql
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(10m)
| summarize FailedAttempts = count() by Account, IpAddress, Computer
| where FailedAttempts > 5
| order by FailedAttempts desc
```

**Trigger:** 5+ failed logins in 10 minutes from same IP  
**Severity:** High  

---

## Successful Login After Multiple Failures

**Purpose:** Detect a successful login (4624) that follows multiple failed attempts (4625) — a classic indicator of a successful brute force.

```kql
let FailedLogins = SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(1h)
| summarize FailCount = count() by Account, IpAddress
| where FailCount >= 3;
SecurityEvent
| where EventID == 4624
| where TimeGenerated > ago(1h)
| join kind=inner FailedLogins on Account
| project TimeGenerated, Account, IpAddress, Computer, FailCount
```

**Trigger:** Successful login preceded by 3+ failures from same account  
**Severity:** High  

---

## Logon Outside Business Hours

**Purpose:** Flag successful logins that occur outside of standard working hours (08:00–18:00).

```kql
SecurityEvent
| where EventID == 4624
| where TimeGenerated > ago(24h)
| extend Hour = datetime_part("Hour", TimeGenerated)
| where Hour < 8 or Hour > 18
| project TimeGenerated, Account, Computer, IpAddress, LogonTypeName
| order by TimeGenerated desc
```

**Trigger:** Successful logon before 08:00 or after 18:00  
**Severity:** Medium  

---

## New User Account Created

**Purpose:** Alert when a new local user account is created — useful for detecting persistence techniques.

```kql
SecurityEvent
| where EventID == 4720
| where TimeGenerated > ago(24h)
| project TimeGenerated, Account, Computer, SubjectAccount = SubjectUserName
| order by TimeGenerated desc
```

**Trigger:** Event ID 4720 (user account created)  
**Severity:** Medium  
