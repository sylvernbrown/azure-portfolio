# Project 5: Azure App Service Deployment and Scaling

![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![App Service](https://img.shields.io/badge/App_Service-Deployed-blue?style=for-the-badge)
![Autoscaling](https://img.shields.io/badge/Autoscaling-Configured-green?style=for-the-badge)
![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub_Actions-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

## Overview

This project covers deploying a web application with Azure App Service, setting up a staging slot for zero-downtime deployments, configuring autoscaling rules based on CPU demand, automating backups, and monitoring application health through Azure Monitor metrics and alerts.

This is part of a 5-project Azure portfolio built toward the **AZ-104: Microsoft Azure Administrator** certification.

---

## Business Scenario

A company wants to host their web application on Azure App Service with:
- A safe testing environment (staging slot) before promoting to production
- Automatic scale-out during traffic spikes to maintain performance
- Scheduled backups for disaster recovery
- Proactive alerting when performance degrades

---

## Skills Demonstrated

| Skill | Tool/Service |
|---|---|
| PaaS web application hosting | Azure App Service |
| Zero-downtime deployments | Deployment Slots (Staging → Production swap) |
| CI/CD pipeline | GitHub Actions |
| Auto-scaling | App Service Plan Scale Out rules |
| Backup and restore | App Service Backups |
| Application monitoring | Azure Monitor Metrics and Alerts |

---

## Architecture

```
┌────────────────────────────────────────────────────────────┐
│                  GitHub Repository                         │
│                                                            │
│  main branch ──► GitHub Actions ──► Production Slot       │
│  staging branch ─────────────────► Staging Slot           │
│                                          │                 │
│                                     Swap ▼                 │
│                              ┌──────────────────┐         │
│                              │  App Service      │         │
│                              │                   │         │
│                              │  Production       │         │
│                              │  Slot             │         │
│                              └──────────────────┘         │
│                                                            │
│  ┌────────────────────────────────────────────────────┐   │
│  │  App Service Plan (Scale Out)                      │   │
│  │  CPU > 70% for 10 min → +1 instance               │   │
│  │  CPU < 30% for 10 min → -1 instance               │   │
│  └────────────────────────────────────────────────────┘   │
│                                                            │
│  ┌───────────────────────┐  ┌──────────────────────────┐  │
│  │  Azure Monitor Alerts │  │  App Service Backups     │  │
│  │  (CPU, Response Time) │  │  (Daily, to Storage)     │  │
│  └───────────────────────┘  └──────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Walkthrough

### Step 1 — Create the App Service

1. Navigate to **Azure Portal > App Services > + Create**
2. Configure:
   - **Subscription** and **Resource Group**
   - **Name:** must be globally unique (becomes `yourname.azurewebsites.net`)
   - **Runtime Stack:** choose your language (e.g., Node.js 20 LTS, Python 3.12, .NET 8)
   - **Region:** same as other project resources
   - **Pricing Tier:** Standard S1 or higher (required for deployment slots and autoscaling)
3. Review and Create

> **Pricing tier matters:** Free and Shared tiers don't support deployment slots or autoscaling. Standard (S1+) is the minimum for production-like features.

---

### Step 2 — Deploy the Application via GitHub Actions

1. Push your application code to a GitHub repository
2. In the App Service portal: **Deployment Center > GitHub**
3. Authorize Azure to access your GitHub account
4. Select your repository and branch
5. Azure will auto-generate a GitHub Actions workflow file (`.github/workflows/`) and commit it to your repo
6. Every push to the selected branch triggers an automatic deployment

**Verify deployment:**
- Navigate to `https://<your-app-name>.azurewebsites.net`
- App should be live

> **Alternative:** Use **Azure DevOps Pipelines** if your organization uses Azure DevOps. GitHub Actions is simpler for personal portfolio projects.

---

### Step 3 — Set Up Deployment Slots

1. Navigate to App Service > **Deployment Slots > + Add Slot**
2. Name: `staging`
3. **Clone settings from:** Production (inherits config)
4. Deploy a new version to the staging slot:
   - Point a separate GitHub branch to the staging slot via Deployment Center
5. Test the staging version at `https://<app-name>-staging.azurewebsites.net`
6. When ready: **Swap** → promotes staging to production with zero downtime

> **Why slots?** A slot swap is near-instantaneous and reversible. If the new version breaks in production, swap back. This eliminates deployment risk for end users.

**Slot-specific settings:**
- Some app settings (connection strings, environment variables) should be marked as **Slot settings** so they don't swap with the deployment — for example, a production database connection should stay in production regardless of which code is deployed.

---

### Step 4 — Configure Autoscaling

1. Navigate to **App Service Plan > Scale Out (App Service Plan)**
2. Select **Custom autoscale**
3. Add a **Scale Out rule:**
   - Metric: CPU Percentage
   - Operator: Greater than
   - Threshold: 70%
   - Duration: 10 minutes
   - Action: Increase count by 1
4. Add a **Scale In rule:**
   - Metric: CPU Percentage
   - Operator: Less than
   - Threshold: 30%
   - Duration: 10 minutes
   - Action: Decrease count by 1
5. Set **instance limits:** Minimum 1, Maximum 5, Default 1
6. Save

> **Scale-in cooldown:** Set a cooldown period (e.g., 5 minutes) between scale-in actions to prevent thrashing — rapidly adding and removing instances as load fluctuates near the threshold.

---

### Step 5 — Configure Backup and Restore

1. Navigate to App Service > **Backups > Configure**
2. Prerequisites:
   - Standard tier or higher
   - A linked **Azure Storage Account** and container
3. Configure:
   - **Scheduled backups:** Daily at a consistent time
   - **Retention:** 7–30 days
   - **Include database:** if your app has a connected Azure SQL or MySQL database
4. Run a **Backup Now** to confirm it completes successfully
5. Test restore:
   - Backups > select a backup > **Restore**
   - Restore to the same app or a new App Service to verify integrity

---

### Step 6 — Monitor and Configure Alerts

**View metrics:**
1. App Service > **Monitoring > Metrics**
2. Add metrics to the chart:
   - **CPU Time** — total CPU consumed
   - **Average Response Time** — latency indicator
   - **Http 5xx** — server-side error rate
   - **Requests** — traffic volume

**Create an alert:**
1. App Service > **Monitoring > Alerts > + New Alert Rule**
2. Configure:
   - **Signal:** Average Response Time
   - **Condition:** Greater than 2 seconds
   - **Action Group:** email notification
   - **Alert name:** High Response Time
3. Save

> Combine **autoscaling** (fixes the problem automatically) with **alerting** (notifies you it happened) for a complete observability strategy.

---

## Key Concepts

**PaaS vs. IaaS** — In Project 1, you managed a VM (IaaS) — OS patches, security, runtime all your responsibility. App Service (PaaS) abstracts the OS and runtime. You manage only the application code and configuration. Faster to deploy, less operational overhead.

**Deployment Slots** — Separate instances of the App Service sharing the same App Service Plan. Swapping is atomic and reversible. Standard enterprise practice for blue/green and canary deployments.

**Autoscaling vs. Scale Up** — Autoscaling (scale out) adds more instances horizontally. Scale up increases the size of the existing instance (more CPU/RAM). Autoscaling is more resilient and cost-efficient for variable load.

**GitHub Actions for Azure** — Azure publishes official GitHub Actions for deploying to App Service, Functions, Container Apps, and more. Worth learning — it's how most modern Azure deployments are automated.

---

## Resources

- [Azure App Service Documentation](https://learn.microsoft.com/en-us/azure/app-service/overview)
- [Deployment Slots](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots)
- [Autoscale in Azure](https://learn.microsoft.com/en-us/azure/azure-monitor/autoscale/autoscale-overview)
- [GitHub Actions for Azure](https://learn.microsoft.com/en-us/azure/app-service/deploy-github-actions)
- [App Service Backups](https://learn.microsoft.com/en-us/azure/app-service/manage-backup)
- [App Service Monitoring](https://learn.microsoft.com/en-us/azure/app-service/troubleshoot-diagnostic-logs)

---

## Part of the AZ-104 Portfolio

| # | Project | Topics |
|---|---|---|
| 1 | Azure Compute and Identity Management | VMs, RBAC, Key Vault, Policy, Cost Management |
| 2 | Azure Networking and Storage | VNets, NSGs, Storage, ARM Templates |
| 3 | Monitoring, Backup, and Recovery | Azure Monitor, Backup, Site Recovery, Log Analytics |
| 4 | Entra ID Integration and Identity Management | Entra ID, App Registration, Conditional Access, MFA |
| **5** | **Azure App Service Deployment and Scaling** ← *you are here* | App Service, Slots, Autoscaling, CI/CD |
