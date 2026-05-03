Home Lab Project — Part 05
# 🔐 Account Lockout & Audit Policy on Windows Server 2022

![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-blue?style=flat-square&logo=windows)
![Active Directory](https://img.shields.io/badge/Active%20Directory-Integrated-green?style=flat-square)
![Security](https://img.shields.io/badge/Security-Audit%20Policy-red?style=flat-square)
![PowerShell](https://img.shields.io/badge/PowerShell-Automation-blue?style=flat-square&logo=powershell)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat-square)

> **Home Lab Project — Part 5**
> Implementing comprehensive security auditing, account lockout policies, event log investigation, and security incident reporting on Windows Server 2022 — simulating real enterprise IT security operations and compliance monitoring.

---

## 📑 Table of Contents

- [Lab Environment](#lab-environment)
- [Objectives](#objectives)
- [What is Audit Policy?](#what-is-audit-policy)
- [Baseline Assessment](#baseline-assessment)
- [Audit Policy Configuration](#audit-policy-configuration)
- [Account Lockout GPO](#account-lockout-gpo)
- [Security Incident Simulation](#security-incident-simulation)
- [Event Log Investigation](#event-log-investigation)
- [Account Unlock Process](#account-unlock-process)
- [Security Report Generation](#security-report-generation)
- [Event ID Reference](#event-id-reference)
- [Helpdesk Workflow](#helpdesk-workflow)
- [Architecture](#architecture)
- [Skills Demonstrated](#skills-demonstrated)
- [Related Projects](#related-projects)

---

## 🏗️ Lab Environment

| Component | Details |
|---|---|
| **Hypervisor** | Oracle VirtualBox |
| **Domain Controller** | WIN-6ARV79FMB48 — Windows Server 2022 |
| **Client Machine** | CPC-NSimon-001 — Windows 10 Pro |
| **Domain** | mylab.local |
| **Server IP** | 192.168.1.200 |
| **Test User** | nsimon (Neil Marvin Simon) |

---

## 🎯 Objectives

- [x] Perform baseline audit policy assessment
- [x] Enable comprehensive Security Audit Policies
- [x] Create Account Lockout & Audit GPO
- [x] Simulate a brute force / account lockout scenario
- [x] Investigate Event ID 4625 (Failed Login) in Security Logs
- [x] Investigate Event ID 4740 (Account Lockout) in Security Logs
- [x] Unlock locked account via PowerShell
- [x] Generate automated Security Incident Report
- [x] Document the complete helpdesk resolution workflow

---

## 📖 What is Audit Policy?

**Audit Policy** controls what security events Windows records in the Security Event Log. Think of it as a security camera system — it records everything that happens so IT admins can investigate incidents, detect threats, and prove compliance.

### Why Audit Policies matter in real companies:

| Industry | Compliance Requirement |
|---|---|
| **Healthcare** | HIPAA requires audit logs of all access to patient data |
| **Finance** | SOX requires tracking of all financial system access |
| **Government** | FISMA requires comprehensive security event logging |
| **All companies** | Incident response requires logs to investigate breaches |

### Event Types Tracked:

- **Success** — Records when an action was allowed (e.g., successful login)
- **Failure** — Records when an action was denied (e.g., wrong password)
- **Success and Failure** — Records both (most comprehensive)

---

## 📊 Baseline Assessment

Before making changes, the current audit policy was assessed:

```powershell
auditpol /get /category:*
```

### Baseline Findings

| Category | Before |
|---|---|
| Logon | Success only |
| Account Lockout | Success only |
| Security Group Management | Success only |
| User Account Management | Success only |
| Sensitive Privilege Use | No Auditing |
| Process Creation | No Auditing |
| File System | No Auditing |

---

## 🔧 Audit Policy Configuration

Enhanced audit policies were enabled to capture both Success and Failure events:

```powershell
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable
auditpol /set /subcategory:"Sensitive Privilege Use" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
auditpol /set /subcategory:"Audit Policy Change" /success:enable /failure:enable
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
auditpol /set /subcategory:"File System" /success:enable /failure:enable
```

### After Configuration

| Category | After |
|---|---|
| Logon | ✅ Success and Failure |
| Account Lockout | ✅ Success and Failure |
| Security Group Management | ✅ Success and Failure |
| User Account Management | ✅ Success and Failure |
| Sensitive Privilege Use | ✅ Success and Failure |
| Process Creation | ✅ Success and Failure |
| File System | ✅ Success and Failure |

---

## 🔒 Account Lockout GPO

A dedicated GPO was created for domain-wide security settings:

```powershell
# Create GPO
New-GPO -Name "Security Policy - Account Lockout and Audit" `
  -Comment "Domain-wide account lockout and security audit settings"

# Link to domain
New-GPLink -Name "Security Policy - Account Lockout and Audit" `
  -Target "DC=mylab,DC=local"
```

### GPO Details

| Setting | Value |
|---|---|
| **GPO Name** | Security Policy - Account Lockout and Audit |
| **Status** | AllSettingsEnabled |
| **Linked To** | DC=mylab,DC=local (domain-wide) |
| **Lockout Threshold** | 5 failed attempts |
| **Lockout Duration** | 30 minutes |
| **Observation Window** | 30 minutes |

---

## 🎭 Security Incident Simulation

A brute force attack was simulated by intentionally entering wrong passwords for nsimon:

1. Attempted login as `mylab\nsimon` with wrong password
2. Repeated 6 times to trigger lockout
3. Account locked with message: *"The referenced account is currently locked out and may not be logged on to."*

**Result:** Account successfully locked after exceeding the lockout threshold ✅

---

## 🔍 Event Log Investigation

### Query Failed Login Attempts (Event ID 4625)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 10 | Select-Object TimeCreated, Message | Format-List
```

### Query Account Lockout Events (Event ID 4740)

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4740
} -MaxEvents 5 | Select-Object TimeCreated, Message | Format-List
```

### Findings from Event Logs

| Event ID | Time | Account | Source IP | Finding |
|---|---|---|---|---|
| 4625 | 4/21/2026 | Administrator | 192.168.1.9 | ⚠️ Failed admin login from unknown device |
| 4625 | 4/20/2026 | Admin | 192.168.1.8 | ⚠️ Non-existent account attempt |
| 4625 | 4/20/2026 | jdicay | 192.168.1.9 | ⚠️ Unauthorized RDP attempt |
| 4740 | 5/2/2026 | nsimon | CPC-NSIMON-001 | 🔒 Account locked out (simulated) |

> **Security Note:** The logs revealed multiple suspicious login attempts from IP 192.168.1.9 targeting Administrator and other accounts — this pattern is typical of a brute force attack and would trigger an investigation in a real environment.

---

## 🔓 Account Unlock Process

After investigating logs and confirming the lockout was legitimate, the account was unlocked:

```powershell
# Check lockout status
Get-ADUser -Identity "nsimon" -Properties LockedOut, BadLogonCount, LastBadPasswordAttempt |
  Select-Object Name, LockedOut, BadLogonCount, LastBadPasswordAttempt
```

**Before Unlock:**
```
Name             LockedOut  BadLogonCount  LastBadPasswordAttempt
----             ---------  -------------  ----------------------
Neil Marvin Simon  True         5          5/2/2026 8:49:23 PM
```

```powershell
# Unlock the account
Unlock-ADAccount -Identity "nsimon"

# Verify
Get-ADUser -Identity "nsimon" -Properties LockedOut | Select-Object Name, LockedOut
```

**After Unlock:**
```
Name              LockedOut
----              ---------
Neil Marvin Simon   False
```

---

## 📋 Security Report Generation

An automated security report was generated using PowerShell:

```powershell
$report = @"
========================================
SECURITY INCIDENT REPORT
Generated: $(Get-Date)
Domain: mylab.local
========================================
FAILED LOGIN ATTEMPTS (Last 24 hours)
"@

$failedLogins = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
    StartTime = (Get-Date).AddHours(-24)
} -ErrorAction SilentlyContinue

$report += "`nTotal Failed Attempts: $($failedLogins.Count)"

$lockouts = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4740
    StartTime = (Get-Date).AddDays(-7)
} -ErrorAction SilentlyContinue

$report += "`nTotal Lockouts (Last 7 days): $($lockouts.Count)"
$report | Out-File -FilePath "C:\Users\Administrator\Desktop\SecurityReport.txt"
```

### Report Output
```
========================================
SECURITY INCIDENT REPORT
Generated: 05/02/2026 20:57:45
Domain: mylab.local
========================================
FAILED LOGIN ATTEMPTS (Last 24 hours): 0
Total Lockouts (Last 7 days): 1
```

---

## 📚 Event ID Reference

| Event ID | Description | Severity |
|---|---|---|
| **4624** | Successful login | Info |
| **4625** | Failed login attempt | ⚠️ Warning |
| **4740** | Account locked out | 🔴 High |
| **4767** | Account unlocked | Info |
| **4728** | User added to security group | ⚠️ Warning |
| **4732** | User added to local group | ⚠️ Warning |
| **4756** | User added to universal group | ⚠️ Warning |
| **4648** | Logon with explicit credentials | ⚠️ Warning |
| **4719** | Audit policy changed | 🔴 High |
| **4720** | User account created | Info |
| **4726** | User account deleted | 🔴 High |

---

## 🎯 Helpdesk Workflow

This is the standard helpdesk process for account lockout incidents:

```
1. User calls helpdesk
   "I can't log in to my computer!"
           ↓
2. IT checks Event Log (Event ID 4740)
   Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4740}
           ↓
3. IT analyzes failed attempts (Event ID 4625)
   - How many attempts?
   - What source IP?
   - Was it the user or a hacker?
           ↓
4. IT verifies user identity
   "Can you confirm your employee ID and department?"
           ↓
5. IT unlocks account
   Unlock-ADAccount -Identity "username"
           ↓
6. IT advises user
   "Please change your password immediately"
           ↓
7. IT documents the incident
   Save report to SecurityReport.txt
           ↓
8. Ticket closed ✅
```

---

## 📐 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       mylab.local Domain                         │
│                                                                 │
│  ┌──────────────────────────┐   ┌────────────────────────────┐  │
│  │   WinServer2022          │   │   CPC-NSimon-001           │  │
│  │   Domain Controller      │   │   Windows 10 Pro           │  │
│  │                          │   │                            │  │
│  │   Security Event Log:    │   │   nsimon enters wrong      │  │
│  │   ✅ 4625 — Failed Login │◄──│   password 5+ times        │  │
│  │   ✅ 4740 — Locked Out   │   │                            │  │
│  │   ✅ 4767 — Unlocked     │   │   Account Locked! 🔒       │  │
│  │                          │   │                            │  │
│  │   Audit Policies:        │   │   IT Admin investigates    │  │
│  │   ✅ Logon S&F           │   │   event logs on server     │  │
│  │   ✅ Lockout S&F         │   │                            │  │
│  │   ✅ Privilege Use S&F   │   │   Account Unlocked ✅      │  │
│  │   ✅ Process Creation    │   │                            │  │
│  │                          │   │   IP: 192.168.1.10         │  │
│  │   IP: 192.168.1.200      │   └────────────────────────────┘  │
│  └──────────────────────────┘                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 Skills Demonstrated

- Windows Security Audit Policy Configuration
- Account Lockout Policy Implementation
- Security Event Log Analysis
- Event ID Investigation & Interpretation
- Brute Force Attack Simulation & Detection
- Account Unlock via PowerShell
- Security Incident Report Generation
- GPO Security Policy Deployment
- Helpdesk Incident Resolution Workflow
- Compliance & Security Documentation
- PowerShell Security Automation

---

## 🚀 Future Improvements

- [ ] Configure email alerts when account lockouts occur
- [ ] Set up Windows Event Forwarding to centralize logs
- [ ] Create scheduled task to auto-generate daily security reports
- [ ] Implement Microsoft Sentinel for advanced SIEM capabilities
- [ ] Configure audit logs for file access on FileServer
- [ ] Set up PowerShell script to auto-notify users when their account is locked

---

## 🔗 Related Projects

| Project | Description |
|---|---|
| **Part 1** | [DNS & DHCP Server Setup](https://yuzuki007.github.io/DNS-DHCP-Server-Setup-on-Windows-Server-2022/) |
| **Part 2** | [Windows 10 Client & AD Integration](https://yuzuki007.github.io/Windows-10-Client-Setup-Active-Directory-Integration/) |
| **Part 3** | Group Policy Objects (GPOs) |
| **Part 4** | File Server with NTFS Permissions |
| **Part 5** | Account Lockout & Audit Policy ← You are here |

---

## 👤 Author

**Neil Marvin Simon**
Home Lab Project — Part 5 | Windows Server 2022 | Security Audit | Account Lockout
*Built for IT Portfolio purposes*


