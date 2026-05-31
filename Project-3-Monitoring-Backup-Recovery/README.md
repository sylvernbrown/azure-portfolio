# Project 3: Monitoring, Backup, and Recovery

![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Azure Monitor](https://img.shields.io/badge/Azure_Monitor-Active-blue?style=for-the-badge)
![Backup](https://img.shields.io/badge/Backup-Configured-green?style=for-the-badge)
![Site Recovery](https://img.shields.io/badge/Site_Recovery-Enabled-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

## Overview

This project demonstrates setting up end-to-end observability and resilience for Azure infrastructure: enabling VM Insights, configuring performance alerts, automating backups with recovery policies, implementing cross-region disaster recovery with Azure Site Recovery, and writing KQL queries to analyze infrastructure logs.

This builds on the VM deployed in **Project 1** and is part of a 5-project Azure portfolio built toward the **AZ-104: Microsoft Azure Administrator** certification.

---

## Business Scenario

A company needs to ensure their Azure infrastructure is observable and recoverable. Requirements:
- Alert on high CPU usage before it impacts end users
- Automatically back up VMs daily with a defined retention window
- Fail over to a secondary region if the primary region goes down
- Query logs to identify performance trends and anomalies

---

## Skills Demonstrated

| Skill | Tool/Service |
|---|---|
| Performance monitoring | Azure Monitor, VM Insights |
| Alert configuration | Action Groups, Alert Rules |
| Automated backups | Recovery Services Vault, Backup Policies |
| Disaster recovery | Azure Site Recovery |
| Log analysis | Log Analytics Workspace, KQL |
| Encryption verification | Recovery Services Vault properties |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Primary Region                           │
│                                                             │
│  ┌──────────┐    Metrics/Logs    ┌────────────────────────┐ │
│  │    VM    │ ─────────────────► │   Log Analytics        │ │
│  │(Project 1│                   │   Workspace             │ │
│  │   VM)    │                   └────────────────────────┘ │
│  └────┬─────┘                            │                  │
│       │                                  ▼                  │
│       │ Backup             ┌──────────────────────────┐     │
│       ▼                   │   Azure Monitor Alerts    │     │
│  ┌──────────────────┐     │   (CPU > 80% → Email)    │     │
│  │ Recovery Services│     └──────────────────────────┘     │
│  │ Vault            │                                       │
│  │ (Daily backups)  │                                       │
│  └──────────────────┘                                       │
│       │ Replication                                         │
│       ▼                                                     │
│ ┌─────────────────────┐                                     │
│ │   Secondary Region  │                                     │
│ │   (Site Recovery    │                                     │
│ │    failover target) │                                     │
│ └─────────────────────┘                                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Walkthrough

### Step 1 — Enable VM Insights and Configure Alerts

**Enable VM Insights:**
1. Navigate to the VM (from Project 1) > **Monitoring > Insights**
2. Select **Azure Monitor > Enable VM Insights > Enable > Configure**
   - This deploys the Azure Monitor Agent and enables performance collection

**Configure a CPU Alert:**
1. VM > **Monitoring > Alerts > View + set up**
2. Switch on **High CPU** alert
3. Set threshold: **80%**
4. Create an **Action Group** with email notification

> **Why 80%?** Alerting below the saturation point (100%) gives time to respond before users are impacted. 80% is a common production threshold.

---

### Step 2 — Configure Azure Backup

1. Navigate to **Recovery Services Vault > Create**
   - Place in the same region as the VM
2. Go to the vault > **Backup > Azure Virtual Machine**
3. Create or select a **Backup Policy**:
   - Frequency: Daily
   - Retention: 7 days (adjust per business requirements)
   - Enable **encryption at rest** (verify in vault Properties)
4. Select the VM and **Enable Backup**
5. Run a **Backup Now** to perform a manual backup and verify it completes successfully

**Verify backup status:** Recovery Services Vault > **Backup Items > Azure Virtual Machine**

---

### Step 3 — Implement Azure Site Recovery

1. Navigate to the VM > **Disaster Recovery**
2. Click **Enable Replication** and select a **secondary region**
3. Review replication settings:
   - Replication policy: defines RPO (Recovery Point Objective) and app-consistent snapshot frequency
   - Target resource group and VNet in secondary region
4. After replication is established, run **Test Failover** to validate DR works without impacting production
5. Monitor replication health: Recovery Services Vault > **Site Recovery > Replicated Items**

> **RTO vs. RPO:**
> - **RPO (Recovery Point Objective):** Maximum acceptable data loss — how old can the restore point be?
> - **RTO (Recovery Time Objective):** Maximum acceptable downtime — how long can recovery take?
> Site Recovery typically achieves RPO of minutes and RTO of under an hour for VMs.

---

### Step 4 — Query Logs with KQL in Log Analytics

1. Navigate to **Log Analytics Workspace > Logs**
2. Run queries to analyze VM performance:

**CPU usage over time:**
```kql
Perf
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 5m), Computer
| render timechart
```

**Available memory trend:**
```kql
Perf
| where ObjectName == "Memory"
| where CounterName == "Available MBytes"
| summarize AvgMemMB = avg(CounterValue) by bin(TimeGenerated, 5m), Computer
| render timechart
```

**Failed sign-in attempts:**
```kql
SecurityEvent
| where EventID == 4625
| summarize FailedLogins = count() by Account, Computer, bin(TimeGenerated, 1h)
| order by FailedLogins desc
```

3. Pin query results as charts to a shared **Azure Dashboard** for ongoing visibility

---

### Step 5 — Verify Backup Encryption

1. Recovery Services Vault > **Properties**
   - Confirm **Encryption** section shows Microsoft-managed keys or customer-managed keys
2. All backup traffic uses **TLS** in transit by default — no additional configuration required

---

## Key Concepts

**VM Insights vs. Basic Metrics** — Basic metrics (CPU, disk) are collected by default. VM Insights adds the Azure Monitor Agent for richer data: process-level metrics, dependency maps, and performance baselines.

**Recovery Services Vault** — A single vault handles both **Azure Backup** and **Azure Site Recovery**. Backup = restore from snapshot. Site Recovery = replicate and failover the live VM to another region.

**KQL (Kusto Query Language)** — Azure's log query language. Used across Azure Monitor, Microsoft Sentinel, and Application Insights. Understanding KQL is a high-value skill for sysadmins moving into cloud operations.

**Action Groups** — Reusable notification configurations (email, SMS, webhook, ITSM). Attach them to multiple alert rules to avoid duplicating notification setup.

---

## Resources

- [Azure Monitor Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/overview)
- [VM Insights Overview](https://learn.microsoft.com/en-us/azure/azure-monitor/vm/vminsights-overview)
- [Azure Backup Documentation](https://learn.microsoft.com/en-us/azure/backup/backup-overview)
- [Azure Site Recovery](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview)
- [KQL Reference](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [Log Analytics Workspace](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview)

---

## Part of the AZ-104 Portfolio

| # | Project | Topics |
|---|---|---|
| 1 | Azure Compute and Identity Management | VMs, RBAC, Key Vault, Policy, Cost Management |
| 2 | Azure Networking and Storage | VNets, NSGs, Storage, ARM Templates |
| **3** | **Monitoring, Backup, and Recovery** ← *you are here* | Azure Monitor, Backup, Site Recovery, Log Analytics |
| 4 | Entra ID Integration and Identity Management | Entra ID, App Registration, Conditional Access, MFA |
| 5 | Azure App Service Deployment and Scaling | App Service, Slots, Autoscaling, CI/CD |
