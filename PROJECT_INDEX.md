# Azure Portfolio Project Index
*Last updated: 2026-05-30 | Session memory file — read this at the start of every session*

---

## Who This Is For

**Nick (Sylver N. Brown)** — IST Engineer at Dataserv (K-12 district IT admin, Cleveland OH). B.S. Finance, Penn State, GPA 3.7, Dec 2024. Relocating to Chicago July 2026. Targeting hybrid sysadmin / MSP sysadmin roles.

**The gap being closed:** Strong on-prem background (AD, Palo Alto, GPO, Chromebook MDM, PowerShell) but no Azure infrastructure experience. This portfolio closes that gap for AZ-104 and job callbacks.

**Email:** nicholasbrownbusiness@gmail.com

---

## Portfolio Goal

5 Azure projects built in a weekend → documented on GitHub → published to personal website + LinkedIn → updated resume → more callbacks for hybrid sysadmin / MSP sysadmin roles.

**Workflow per project:** Build in Azure → drop screenshots into project folder → I read screenshots, map to lab steps → embed in README → push to GitHub → post LinkedIn writeup.

---

## Files in This Folder

| File | Status | Notes |
|---|---|---|
| `index.html` | ✅ Updated | Portfolio website — 5 Azure project cards added above existing content. GitHub URL placeholders: `YOUR_GITHUB_REPO_URL_1` through `_5` need replacing once repos are live. |
| `Sylver_Brown_Final.docx` | ✅ Complete | One-page resume. Added AZ-104 cert + 12 Azure services to skills + Azure Portfolio section (5 project one-liners). Removed: CCNA, VoIP skills line, weak Dataserv bullet, duplicate UPMC bullet. |
| `PROJECT_INDEX.md` | ✅ This file | Session memory |

---

## Projects

### Project 1 — Azure Compute and Identity Management
**Folder:** `Project-1-Azure-Compute-Identity-Management/`  
**README status:** ⚠️ IN PROGRESS — needs screenshot embedding  
**What Nick actually built:** Windows Server 2022 VM (not Linux). RDP access. Static private IP. Key Vault. RBAC. Entra ID security group. Azure Policy. Cost Management budget.  
**Screenshots:** 51 total in folder (all timestamped 2026-05-30 8:49 PM – 11:04 PM)

Screenshot mapping completed so far (6/51):
| Timestamp | What it shows | Lab Step |
|---|---|---|
| 8:49:05 PM | Create VM Basics tab — Windows Server 2022, West US, Standard_D2s_v3 | Step 1: Create VM |
| 8:51:32 PM | VM size selection panel | Step 1: Size selection |
| 8:51:44 PM | Basics tab — vm-win-lab01 name, Windows Server 2022 image confirmed | Step 1: Name/image |
| 8:52:23 PM | Admin account section + RDP (3389) inbound port rule selected | Step 1: Admin + inbound ports |
| 8:52:46 PM | Disks tab — OS disk + data disk section | Step 2: Add data disk |
| 8:53:06 PM | Select disk size dialog — 32 GB Standard SSD selected | Step 2: Disk size |

**Remaining screenshots to map:** 8:53:51 PM through 11:04:20 PM (45 screenshots)  
**Expected lab steps in remaining screenshots:**
- Networking tab (VNet, subnet, NSG, public IP)
- Static private IP assignment
- Review + Create / deployment in progress / deployment complete
- Key Vault creation
- RBAC role assignments (VM Contributor, Key Vault Administrator)
- Entra ID security group creation
- Secrets creation in Key Vault
- RDP connection to VM
- Windows Server Manager — disk initialization (GPT, NTFS)
- Azure Policy — initiative assignment
- Cost Management — budget creation with alerts

---

### Project 2 — Azure Networking and Storage
**Folder:** `Project-2-Azure-Networking-Storage/`  
**README status:** ✅ Complete (no screenshots yet)  
**Topics:** VNets, NSGs, Storage Accounts, Private Endpoints, ARM Templates

---

### Project 3 — Monitoring, Backup, and Recovery
**Folder:** `Project-3-Monitoring-Backup-Recovery/`  
**README status:** ✅ Complete (no screenshots yet)  
**Topics:** Azure Monitor, Log Analytics, KQL, Backup Vault, Site Recovery

---

### Project 4 — Entra ID Integration and Identity Management
**Folder:** `Project-4-Entra-ID-Identity-Management/`  
**README status:** ✅ Complete (no screenshots yet)  
**Topics:** Entra ID, App Registration, Conditional Access, MFA, SSPR

---

### Project 5 — Azure App Service Deployment and Scaling
**Folder:** `Project-5-Azure-App-Service-Deployment-Scaling/`  
**README status:** ✅ Complete (no screenshots yet)  
**Topics:** App Service, Deployment Slots, Autoscaling, GitHub Actions CI/CD, Azure Monitor Alerts

---

## Resume Changes Made (Sylver_Brown_Final.docx)

**Added:**
- AZ-104: Microsoft Azure Administrator (In Progress) in Certifications
- Azure skills line: `Azure (VMs, VNets, NSGs, Key Vault, RBAC, Entra ID, Monitor, Log Analytics, App Service, Backup, Site Recovery, Cost Management)`
- Azure Portfolio section with 5 one-liner project bullets

**Removed:**
- CCNA In Progress (postponed until new role)
- VoIP & Telephony skills line
- Weak "Participated in major network..." Dataserv bullet
- Duplicate UPMC variance bullet
- UPMC SQL bullet

**Spacing reductions:** Bullet spacing 30→15pt, section headers 100→60pt before, margins 720→560 DXA top/bottom

---

## Website (index.html) Changes Made

- Added 5 Azure project cards (blue-tinted) to AZ-104 Portfolio section above existing labs
- Added Azure skills chips to skills section
- Added AZ-104 cert row
- Added hero badge
- GitHub URL placeholders still need replacing: `YOUR_GITHUB_REPO_URL_1` through `_5`

---

## Pending Tasks

- [ ] **PRIMARY:** Finish reading remaining 45 Project 1 screenshots, map to lab steps, rewrite README with all screenshots embedded
- [ ] Add screenshots to Projects 2–5 READMEs as Nick drops them
- [ ] Replace GitHub URL placeholders in index.html once repos are live
- [ ] LinkedIn posts with AZ-104 keywords
- [ ] Cover letter template (mentioned, not started)

---

## Key Decisions / Preferences

- README style: badges at top, business scenario, skills table, ASCII architecture diagram, step-by-step walkthrough with screenshots, key concepts, resources, portfolio table at bottom
- Screenshots embedded as: `![Description](Screenshot%202026-05-30%20at%20X.XX.XX%20PM.png)` with italicized caption below
- Resume: one page, hard constraint
- Writing style: direct, no fluff, ATS-optimized keywords
- Don't ask clarifying questions via UI widget — ask in plain text if needed
