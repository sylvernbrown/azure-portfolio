# Project 1: Azure Compute and Identity Management

![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Windows Server](https://img.shields.io/badge/Windows_Server_2022-Deployed-blue?style=for-the-badge)
![Key Vault](https://img.shields.io/badge/Key_Vault-Configured-green?style=for-the-badge)
![RBAC](https://img.shields.io/badge/RBAC-Assigned-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

## Overview

This project covers deploying a Windows Server 2022 virtual machine on Azure, configuring secure RDP access with a Network Security Group, setting a static private IP, creating an Azure Key Vault using the RBAC permission model, storing secrets, assigning role-based access to an Entra ID security group, initializing and formatting a data disk inside the live VM, building an Azure Policy initiative for governance, and setting up Cost Management budgets with threshold alerts.

This is part of a 5-project Azure portfolio built toward the **AZ-104: Microsoft Azure Administrator** certification.

---

## Business Scenario

A company is standing up its first Azure environment and needs:
- A Windows Server VM accessible only via RDP (no public SSH exposure)
- Secrets stored securely in Key Vault — not hardcoded in config files
- Role-based access controlled through Entra ID groups, not individual user assignments
- A governance policy that enforces tagging and restricts deployable resource types
- Spending alerts before the bill runs away

---

## Skills Demonstrated

| Skill | Tool/Service |
|---|---|
| Windows Server VM deployment | Azure Virtual Machines |
| RDP-only access control | Network Security Groups |
| Static private IP assignment | Virtual Network / NIC IP Config |
| Secrets management | Azure Key Vault |
| Identity-based access control | Microsoft Entra ID + RBAC |
| Disk initialization and formatting | Windows Server Disk Management |
| Governance enforcement | Azure Policy Initiative |
| Cloud spend monitoring | Cost Management + Budgets |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│  Resource Group: rg-vm-lab                              │
│                                                         │
│  ┌─────────────────────┐   ┌─────────────────────────┐ │
│  │  vm-win-lab01        │   │  kv-vm-1 (Key Vault)    │ │
│  │  Windows Server 2022 │   │  RBAC permission model  │ │
│  │  Standard_D2s_v3     │   │  Secret: Adminuser      │ │
│  │  Static IP: 10.0.0.4 │   └─────────────────────────┘ │
│  └──────────┬──────────┘             ▲                  │
│             │                        │                  │
│  ┌──────────▼──────────┐   ┌─────────┴───────────────┐ │
│  │  vm-win-lab01-nsg    │   │  Entra ID: KV-Admins    │ │
│  │  Inbound: RDP (3389) │   │  Group → Key Vault      │ │
│  │  only                │   │  Administrator role     │ │
│  └─────────────────────┘   └─────────────────────────┘ │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Azure Policy: VM Governance Initiative          │   │
│  │  - Inherit tag from subscription if missing      │   │
│  │  - Inherit tag from resource group if missing    │   │
│  │  - Allowed resource types: VMs only              │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Cost Management: VM_LAB_Budget ($20/mo)         │   │
│  │  Alerts at 50% / 80% / 100%                      │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Walkthrough

### Step 1 — Create the Windows Server VM

1. Navigate to **Azure Portal > Virtual Machines > + Create > Azure virtual machine**
2. On the **Basics** tab configure:
   - **Subscription / Resource Group:** Create new — `rg-vm-lab`
   - **VM Name:** `vm-win-lab01`
   - **Region:** West US
   - **Image:** Windows Server 2022 Datacenter: Azure Edition
   - **Size:** Standard_D2s_v3 (2 vCPUs, 8 GiB RAM)

![Create VM - Basics tab with Windows Server 2022 selected](Screenshot%202026-05-30%20at%208.49.05%20PM.png)
*Basics tab — VM name, region, and image set before moving to size selection*

![VM size selection panel](Screenshot%202026-05-30%20at%208.51.32%20PM.png)
*Size selection panel — Standard_D2s_v3 provides the right balance of compute for a lab VM*

![Basics tab with vm-win-lab01 name and Windows Server 2022 image confirmed](Screenshot%202026-05-30%20at%208.51.44%20PM.png)
*Basics tab with name and image locked in*

3. Scroll down to **Administrator account** — set a username and strong password
4. Under **Inbound port rules**, select **Allow selected ports** → choose **RDP (3389)**

![Administrator account section with RDP (3389) inbound port rule selected](Screenshot%202026-05-30%20at%208.52.23%20PM.png)
*Admin credentials configured. RDP is the only inbound port — no SSH, no HTTP exposure*

![Inbound port dropdown showing only RDP (3389) checked](Screenshot%202026-05-30%20at%208.54.50%20PM.png)
*Confirming RDP (3389) is the sole allowed inbound port — HTTP and SSH left unchecked*

---

### Step 2 — Add a Data Disk

1. Click **Next: Disks**
2. Under **Data disks**, click **Create and attach a new disk**
3. Set size to **32 GiB**, disk type **Standard SSD**

![Disks tab showing OS disk and data disk configuration](Screenshot%202026-05-30%20at%208.52.46%20PM.png)
*Disks tab — OS disk defaults to Standard SSD; data disk section ready for attachment*

![Select disk size dialog with 32 GB Standard SSD selected](Screenshot%202026-05-30%20at%208.53.06%20PM.png)
*32 GiB Standard SSD — appropriate size for a lab data volume*

---

### Step 3 — Configure Networking

1. Click **Next: Networking**
2. Azure auto-creates a VNet (`vm-win-lab01-vnet`), subnet, and public IP
3. Confirm the **NSG** is set to restrict inbound to RDP only

![Networking tab showing VNet, subnet, public IP, and NSG configuration](Screenshot%202026-05-30%20at%208.54.36%20PM.png)
*Networking tab — Azure auto-provisions the VNet, NSG, and public IP. NSG will enforce RDP-only inbound access*

4. Click **Review + Create** → **Create**

---

### Step 4 — Deployment

Azure deploys all associated resources simultaneously: the VM, NIC, VNet, NSG, public IP, and data disk.

![Deployment in progress showing all resources being created](Screenshot%202026-05-30%20at%208.58.47%20PM.png)
*Deployment in progress — vm-win-lab01, NIC, VNet, NSG, public IP, and data disk all provisioning in parallel*

![Deployment complete](Screenshot%202026-05-30%20at%208.59.44%20PM.png)
*Deployment complete — all resources created successfully under resource group rg-vm-lab*

---

### Step 5 — Set a Static Private IP

By default, Azure assigns private IPs dynamically. Setting a static IP ensures consistent internal addressing — important for DNS records, firewall rules, and service configurations that reference the IP directly.

1. Navigate to **vm-win-lab01 > Networking > Network settings**
2. Click the NIC name (`vm-win-lab01524`)

![Network interface IP configuration link](Screenshot%202026-05-30%20at%209.01.01%20PM.png)
*Navigating to the NIC IP configuration to change allocation from dynamic to static*

3. Click **ipconfig1** → set **Private IP address allocation** to **Static**
4. Enter `10.0.0.4`
5. Click **Save**

![Edit IP configuration showing Static allocation set to 10.0.0.4](Screenshot%202026-05-30%20at%209.02.07%20PM.png)
*Private IP changed to Static at 10.0.0.4. Public IP (20.245.108.149) remains associated for RDP access*

> **Why static?** Dynamic IPs can change on VM restart. If NSG rules, DNS entries, or monitoring configurations reference the IP, a change breaks them. Static IPs eliminate that risk.

---

### Step 6 — Create the Key Vault

Key Vault centralizes secrets management. Storing credentials in Key Vault instead of config files means they're audited, versioned, and access-controlled through RBAC.

1. Navigate to **Key vaults > + Create**

![Key vaults page — empty, ready to create first vault](Screenshot%202026-05-30%20at%209.03.40%20PM.png)
*Starting from an empty Key Vaults list*

2. Configure:
   - **Name:** `kv-vm-1`
   - **Resource Group:** `rg-vm-lab`
   - **Region:** West US
   - **Pricing tier:** Standard

![Create Key Vault Basics tab: kv-vm-1, rg-vm-lab, West US, Standard tier](Screenshot%202026-05-30%20at%209.04.28%20PM.png)
*Key Vault basics — name, resource group, and region match the VM deployment*

3. On the **Access configuration** tab, select **Azure role-based access control (Recommended)**

![Permission model showing Azure RBAC selected](Screenshot%202026-05-30%20at%209.05.06%20PM.png)
*RBAC permission model selected — access is managed through Entra ID role assignments rather than legacy access policies*

4. Review + Create

---

### Step 7 — Create an Entra ID Security Group

Instead of assigning Key Vault access to individual users, we use an Entra ID security group. Add users to the group — not to the resource.

1. Navigate to **Microsoft Entra ID** (left sidebar)

![Azure portal left nav with Microsoft Entra ID highlighted](Screenshot%202026-05-30%20at%209.05.31%20PM.png)
*Microsoft Entra ID in the portal navigation*

2. Click **+ Add > Group**

![Entra ID with Add dropdown showing Group option](Screenshot%202026-05-30%20at%209.07.31%20PM.png)
*Add dropdown — selecting Group to create a new security group*

3. Configure:
   - **Group type:** Security
   - **Group name:** `KV-Admins`
   - **Description:** Admin Key Vault

![New group form: Security type, KV-Admins name, Admin Key Vault description](Screenshot%202026-05-30%20at%209.08.07%20PM.png)
*Security group named KV-Admins — will receive the Key Vault Administrator role*

4. Click **No members selected** → search for and add your user account

![Select members panel with Sylver Brown selected](Screenshot%202026-05-30%20at%209.08.15%20PM.png)
*Selecting Sylver Brown as the initial group member*

![Members selection confirming Sylver Brown checked](Screenshot%202026-05-30%20at%209.08.33%20PM.png)
*Member confirmed — Sylver Brown added to KV-Admins*

5. Create the group

---

### Step 8 — Assign Key Vault RBAC Role to the Group

1. Navigate to **kv-vm-1 > Access control (IAM)**

![kv-vm-1 left nav with Access control (IAM) selected](Screenshot%202026-05-30%20at%209.09.43%20PM.png)
*Key Vault IAM blade — where role assignments are managed*

2. Click **+ Add > Add role assignment**

![Add role assignment button](Screenshot%202026-05-30%20at%209.10.17%20PM.png)
*Initiating a new role assignment on the Key Vault*

3. Search for and select **Key Vault Administrator**

![Role list showing Key Vault Administrator](Screenshot%202026-05-30%20at%209.11.00%20PM.png)
*Key Vault Administrator grants full data plane access — read, write, and manage secrets, keys, and certificates*

4. On the **Members** tab, assign to the **KV-Admins** group → Review + assign

![Add role assignment Members tab with KV-Admins group selected](Screenshot%202026-05-30%20at%209.11.32%20PM.png)
*KV-Admins security group assigned — any member of this group inherits the Key Vault Administrator permission*

---

### Step 9 — Store a Secret in Key Vault

1. Navigate to **kv-vm-1 > Objects > Secrets**

![kv-vm-1 left nav with Secrets selected under Objects](Screenshot%202026-05-30%20at%209.13.12%20PM.png)
*Navigating to Secrets within the Key Vault*

2. Click **+ Generate/Import** and configure:
   - **Name:** `Adminuser`
   - **Secret value:** (VM admin password)
   - **Set expiration date:** enabled

![Create a secret: Name = Adminuser, secret value masked, expiration enabled](Screenshot%202026-05-30%20at%209.17.12%20PM.png)
*VM admin credentials stored as a Key Vault secret — versioned, audited, and never exposed in plaintext*

> **Why this matters:** Rotating the password means updating one secret. Any app or admin retrieving it via the Key Vault API gets the current version automatically — no config file changes needed.

---

### Step 10 — Assign VM Contributor Role via Entra ID Group

Assign the KV-Admins group to the VM itself, so the same identity group manages both secrets and the compute resource.

1. Navigate to **vm-win-lab01 > Access control (IAM)**

![vm-win-lab01 left nav with Access control (IAM) selected](Screenshot%202026-05-30%20at%209.24.38%20PM.png)
*VM IAM blade — assigning group-based RBAC to the virtual machine*

2. Click **+ Add > Add role assignment**

![Add role assignment button on VM IAM page](Screenshot%202026-05-30%20at%209.24.53%20PM.png)
*Initiating RBAC assignment on the VM resource*

3. Select **Virtual Machine Contributor**

![Role list showing Virtual Machine Contributor](Screenshot%202026-05-30%20at%209.25.38%20PM.png)
*Virtual Machine Contributor — manage the VM without access to the underlying network or storage resources*

4. Assign to **KV-Admins** group → Review + assign

![Add role assignment Members tab showing KV-Admins assigned](Screenshot%202026-05-30%20at%209.25.59%20PM.png)
*KV-Admins group assigned Virtual Machine Contributor on vm-win-lab01*

**Verify group membership:**

Navigate to **Entra ID > Groups > KV-Admins > Members**

![KV-Admins Members page confirming Sylver Brown as group member](Screenshot%202026-05-30%20at%209.27.23%20PM.png)
*Group membership confirmed — Sylver Brown inherits both Key Vault Administrator and VM Contributor through KV-Admins*

---

### Step 11 — Connect to the VM via RDP

1. Navigate to **vm-win-lab01 > Connect > Connect**
2. Select **Native RDP** → **Download RDP file**
3. Enter credentials: username `Adminuser`, password from Key Vault secret

![vm-win-lab01 Connect page: Native RDP, public IP 20.245.108.149, port 3389, username Adminuser](Screenshot%202026-05-30%20at%209.28.52%20PM.png)
*Connect page — public IP 20.245.108.149 on port 3389, RDP file ready to download*

---

### Step 12 — Initialize and Format the Data Disk

The 32 GiB data disk arrives as raw, uninitialized storage. Windows won't see it until we initialize and format it inside the VM.

1. Inside the VM, right-click the **Start** menu → **Computer Management**

![Windows Server right-click Start menu with Computer Management highlighted](Screenshot%202026-05-30%20at%209.30.24%20PM.png)
*Computer Management — the Windows tool for disk initialization and partitioning*

2. Navigate to **Storage > Disk Management**
3. Right-click **Disk 2** (31.98 GB, Unallocated) → **New Simple Volume...**

![Disk Management: Disk 2 unallocated with New Simple Volume context menu](Screenshot%202026-05-30%20at%209.32.29%20PM.png)
*Disk 2 shows as unallocated — this is the Azure data disk. New Simple Volume Wizard launches on right-click*

4. Assign drive letter **F:**

![New Simple Volume Wizard — Assign Drive Letter F](Screenshot%202026-05-30%20at%209.32.47%20PM.png)
*Drive letter F: assigned*

5. Format as **NTFS**, quick format → Finish

![Format Partition: NTFS file system, quick format selected](Screenshot%202026-05-30%20at%209.32.58%20PM.png)
*NTFS formatting — standard for Windows Server data volumes*

6. Verify the volume appears in File Explorer

![File Explorer showing New Volume (F:) under This PC](Screenshot%202026-05-30%20at%209.33.24%20PM.png)
*New Volume (F:) visible in File Explorer — the Azure data disk is initialized, formatted, and ready for use*

> **On-prem parallel:** Initializing an Azure data disk in Windows Server Disk Management is identical to initializing a new SAN LUN or physical disk on any Windows Server. Azure handles provisioning the disk to the VM; Windows handles the filesystem layer.

---

### Step 13 — Create an Azure Policy Initiative

An initiative bundles multiple policy definitions into a single assignable object for governance at scale.

1. Navigate to **All services > Policy > Definitions > + Initiative definition**
2. **Basics** tab:
   - **Name:** `VM Governance Initiative`
   - **Category:** Governance (create new)

![Initiative definition Basics: VM Governance Initiative, Governance category](Screenshot%202026-05-30%20at%2010.41.08%20PM.png)
*Initiative basics — name and category defined*

3. **Policies** tab → **+ Add policy definition(s)**

![Policies tab with Add policy definition(s) button](Screenshot%202026-05-30%20at%2010.41.33%20PM.png)
*Ready to add policy definitions to the initiative*

4. Search **"inherit"** → select:
   - **Inherit a tag from the subscription if missing**
   - **Inherit a tag from the resource group if missing**

![Tag inheritance policies selected](Screenshot%202026-05-30%20at%2010.42.00%20PM.png)
*Tag inheritance policies ensure resources automatically inherit tags from parent scopes*

5. Search **"allowed resource"** → select **Allowed resource types**

![Allowed resource types policy selected](Screenshot%202026-05-30%20at%2010.42.39%20PM.png)
*Allowed resource types restricts deployments to the approved list only*

6. **Initiative parameters** tab → create parameter:
   - **Name:** `ResourceTag`
   - **Type:** String
   - **Allowed Values:** `["Environment"]`

![Create initiative parameter: ResourceTag, String, allowed value Environment](Screenshot%202026-05-30%20at%2010.45.16%20PM.png)
*ResourceTag parameter — passed to the tag inheritance policies at assignment time*

7. **Policy parameters** tab → map tag policies to `ResourceTag`, set allowed resource types to `Microsoft.Compute/virtualMachines`

![Policy parameters mapping tag policies to ResourceTag, allowed types to VMs](Screenshot%202026-05-30%20at%2011.04.20%20PM.png)
*Policy parameters wired — tag inheritance uses ResourceTag; allowed resource types locked to VMs only*

8. Review + Create

---

### Step 14 — Configure Cost Management Budget

1. Navigate to **Cost Management + Billing > Budgets**

![Cost Management Budgets page — empty, ready to create](Screenshot%202026-05-30%20at%2010.49.01%20PM.png)
*Starting from an empty Budgets list*

2. Click **+ Add** and configure:
   - **Name:** `VM_LAB_Budget`
   - **Reset period:** Monthly
   - **Amount:** $20

![Create budget: VM_LAB_Budget, Monthly, $20, expiration April 2028](Screenshot%202026-05-30%20at%2010.49.40%20PM.png)
*$20/month budget — conservative for a lab, designed to catch any runaway costs immediately*

3. **Set alerts** — configure three thresholds:
   - 50% ($10) — early warning
   - 80% ($16) — action threshold
   - 100% ($20) — budget reached

![Set alerts: 50%, 80%, 100% thresholds with email to sylver.brown@outlook.com](Screenshot%202026-05-30%20at%2010.50.24%20PM.png)
*Three alert thresholds configured with email notification — no surprise bills*

4. Verify spend in **Cost analysis**

![Cost analysis showing accumulated cost vs. monthly budget](Screenshot%202026-05-30%20at%2010.51.41%20PM.png)
*Cost analysis — full visibility into resource spend tracked against the $20/month budget*

---

## Key Concepts

**RBAC vs. Classic Access Policies** — Key Vault supports two permission models. Azure RBAC (recommended) integrates with Entra ID and uses standard role assignments. The legacy access policies model is being retired. Always choose RBAC for new deployments.

**Security Groups over Individual Users** — Assigning roles to Entra ID groups means onboarding/offboarding is a group membership change, not a series of role reassignments across every resource. One group update propagates everywhere.

**Static Private IP** — Set static when the VM hosts a service other resources connect to by IP, NSG rules reference the IP, or you need predictable addressing for documentation.

**Disk Initialization in Windows** — Azure data disks arrive uninitialized. The Azure portal handles provisioning the disk to the VM; Windows Disk Management handles the filesystem layer — same workflow as initializing a new SAN LUN on any Windows Server.

**Policy Initiatives vs. Individual Policies** — An initiative is a single assignable object that enforces multiple policies simultaneously. Update the initiative once — all assignments inherit the change.

**Cost Management Alerts ≠ Spending Limits** — Budget alerts notify you; they don't stop charges. Use alerts as a trigger for manual review or automated runbooks.

---

## Resources

- [Azure Virtual Machines Documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/)
- [Azure Key Vault Overview](https://learn.microsoft.com/en-us/azure/key-vault/general/overview)
- [Azure RBAC Documentation](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)
- [Microsoft Entra ID Groups](https://learn.microsoft.com/en-us/entra/fundamentals/groups-view-azure-portal)
- [Azure Policy Overview](https://learn.microsoft.com/en-us/azure/governance/policy/overview)
- [Cost Management + Billing](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-acm-create-budgets)

---

## Part of the AZ-104 Portfolio

| # | Project | Topics |
|---|---|---|
| **1** | **Azure Compute and Identity Management** ← *you are here* | VMs, RBAC, Key Vault, Entra ID, Policy, Cost Management |
| 2 | Azure Networking and Storage | VNets, NSGs, Storage, Private Endpoints, ARM Templates |
| 3 | Monitoring, Backup, and Recovery | Azure Monitor, Log Analytics, Backup, Site Recovery |
| 4 | Entra ID Integration and Identity Management | App Registration, Conditional Access, MFA, SSPR |
| 5 | Azure App Service Deployment and Scaling | App Service, Slots, Autoscaling, GitHub Actions CI/CD |
