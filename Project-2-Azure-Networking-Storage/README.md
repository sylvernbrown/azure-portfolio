# Project 2: Azure Networking and Storage

![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![VNet](https://img.shields.io/badge/VNet-Configured-blue?style=for-the-badge)
![Storage](https://img.shields.io/badge/Storage-Encrypted-green?style=for-the-badge)
![ARM](https://img.shields.io/badge/ARM_Templates-Deployed-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

## Overview

This project covers deploying a segmented Azure Virtual Network with two subnets, securing traffic with Network Security Groups, deploying an encrypted storage account with private endpoint access, and automating the entire deployment using ARM templates exported from the portal.

This is part of a 5-project Azure portfolio built toward the **AZ-104: Microsoft Azure Administrator** certification.

---

## Business Scenario

A company needs to:
- Isolate web-facing and database resources into separate network segments
- Restrict storage access to internal database subnet only (no public internet)
- Enforce encryption on all stored data
- Automate resource deployment for repeatability using Infrastructure-as-Code

---

## Skills Demonstrated

| Skill | Tool/Service |
|---|---|
| Network segmentation | Azure Virtual Network, Subnets |
| Traffic filtering | Network Security Groups (NSG) |
| Secure storage | Storage Account, Private Endpoint |
| Data encryption | Microsoft-managed keys |
| Infrastructure as Code | ARM Templates, Template Specs |
| Secure access testing | SAS Tokens, AzCopy, Storage Explorer |

---

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                  Azure VNet (10.0.0.0/16)            │
│                                                      │
│  ┌────────────────────────┐  ┌──────────────────┐    │
│  │   Web Subnet           │  │  Database Subnet  │    │
│  │   10.0.1.0/27          │  │  10.0.0.0/27      │    │
│  │                        │  │                   │    │
│  │  NSG attached:         │  │  Service Endpoint │    │
│  │  ✅ Allow HTTPS (443)  │  │  Microsoft.Storage│    │
│  │  ❌ Block all other    │  │                   │    │
│  │     inbound            │  │  ┌─────────────┐  │    │
│  └────────────────────────┘  │  │  Storage    │  │    │
│                              │  │  Account    │  │    │
│                              │  │  (Private   │  │    │
│                              │  │  Endpoint)  │  │    │
│                              │  └─────────────┘  │    │
│                              └──────────────────┘    │
└──────────────────────────────────────────────────────┘

ARM Template Specs → Redeployable via Template Specs Gallery
```

---

## Step-by-Step Walkthrough

### Step 1 — Create the Virtual Network

1. Navigate to **Azure Portal > Virtual Networks > Create**
2. Add two subnets:

**Database Subnet:**
- Name: `Database`
- Address: `10.0.0.0/27`
- Enable **Private Subnet**
- Enable **Service Endpoints**: `Microsoft.Storage`

**Web Subnet:**
- Name: `Web`
- Address: `10.0.1.0/27`

3. Review and create

> **Why /27?** A /27 gives 32 addresses (30 usable after Azure reserves 5). Right-sized for isolated workloads — not wasteful, not restrictive.

> **Service Endpoints vs. Private Endpoints:** Service Endpoints extend the VNet identity to Azure services over the Microsoft backbone — simpler but the storage account still has a public IP. Private Endpoints give the storage account a private IP *inside* your VNet — stronger isolation. We use Private Endpoint for storage here.

---

### Step 2 — Deploy and Configure the NSG

1. Navigate to **Network Security Groups > Create**
2. Associate the NSG with the **Web subnet**:
   - NSG > **Subnets > Associate > Web**
3. Add inbound rules:

| Priority | Name | Port | Action |
|---|---|---|---|
| 100 | Allow-HTTPS | 443 | Allow |
| 200 | Block-All-Inbound | * | Deny |

> **Default rules cannot be modified** — but they can be overridden with a lower priority number. Default rules: `AllowVnetInBound` (65000), `AllowAzureLoadBalancerInBound` (65001), `DenyAllInBound` (65500).

> If SSH or RDP is needed for management, add it at a higher priority (lower number) than the block-all rule.

---

### Step 3 — Create and Configure the Storage Account

1. Navigate to **Storage Accounts > Create**
2. Configure:
   - **Replication:** LRS (locally redundant) for dev/test; GRS for production
   - **Advanced:** Set permitted scope to *"From storage accounts that have a private endpoint"*
   - **Networking:** Disable public access
   - **Private Endpoint:** Add one → select the **Database subnet**
   - **Encryption:** Microsoft-managed keys (default)

> **LRS vs. GRS:**
> - LRS = 3 copies in one datacenter. Cheap. No regional failover.
> - GRS = 6 copies across 2 regions. More expensive. Survives regional outages.

---

### Step 4 — Export ARM Templates and Deploy via Template Specs

For both the VNet and Storage Account:

1. Go to the resource > **Automation > Export Template > Download**
2. Navigate to **Template Specs > Import Template**
3. Upload the `template.json` from the download
4. Click the **⋯ menu** on your new Template Spec > **Deploy**

This makes the infrastructure **repeatable and version-controlled** — a core IaC practice.

---

### Step 5 — Test Access with SAS Token

**Generate a SAS token:**
1. Storage Account > **Security + Networking > Shared Access Signature**
2. Select all Allowed Resource Types > **Generate SAS and connection string**

**Test with Azure Storage Explorer:**
1. Download [Azure Storage Explorer](https://azure.microsoft.com/en-us/products/storage/storage-explorer/)
2. Connect > **Shared Access Signature (SAS) URI**
3. Paste the Connection String
4. Create a blob container and upload/download a test file

**Test with AzCopy:**
```bash
# Upload a file
azcopy copy 'local-file.txt' 'https://<storage-account>.blob.core.windows.net/<container>/local-file.txt<SAS-token>'

# Download a file
azcopy copy 'https://<storage-account>.blob.core.windows.net/<container>/local-file.txt<SAS-token>' 'downloaded-file.txt'
```

> Replace `<SAS-token>` with the token generated in the portal (starts with `?sv=`).

---

## Key Concepts

**Network Segmentation** — Separating web and database traffic into different subnets limits blast radius if one layer is compromised. A web server should never have direct database-level network access.

**Private Endpoint vs. Service Endpoint** — Service Endpoints route traffic over Microsoft's backbone but storage still has a public IP. Private Endpoints assign a private IP from your VNet — fully air-gapped from the public internet.

**ARM Templates** — JSON-based Infrastructure-as-Code for Azure. Exported directly from deployed resources, they enable repeatable, consistent deployments and are the foundation for automation pipelines.

**SAS Tokens** — Time-limited, scoped credentials for storage access. Best practice: generate with minimum required permissions and shortest viable expiry.

---

## Resources

- [Azure Virtual Network Documentation](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)
- [Network Security Groups](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Azure Storage Overview](https://learn.microsoft.com/en-us/azure/storage/common/storage-introduction)
- [Private Endpoints for Storage](https://learn.microsoft.com/en-us/azure/storage/common/storage-private-endpoints)
- [ARM Template Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/overview)
- [AzCopy Download](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10)

---

## Part of the AZ-104 Portfolio

| # | Project | Topics |
|---|---|---|
| 1 | Azure Compute and Identity Management | VMs, RBAC, Key Vault, Policy, Cost Management |
| **2** | **Azure Networking and Storage** ← *you are here* | VNets, NSGs, Storage, ARM Templates |
| 3 | Monitoring, Backup, and Recovery | Azure Monitor, Backup, Site Recovery, Log Analytics |
| 4 | Entra ID Integration and Identity Management | Entra ID, App Registration, Conditional Access, MFA |
| 5 | Azure App Service Deployment and Scaling | App Service, Slots, Autoscaling, CI/CD |
