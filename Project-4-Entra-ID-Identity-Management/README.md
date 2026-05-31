# Project 4: Entra ID Integration and Identity Management

![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Entra ID](https://img.shields.io/badge/Entra_ID-Configured-blue?style=for-the-badge)
![MFA](https://img.shields.io/badge/MFA-Enabled-green?style=for-the-badge)
![Conditional Access](https://img.shields.io/badge/Conditional_Access-Active-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

## Overview

This project demonstrates enterprise identity management using Microsoft Entra ID (formerly Azure Active Directory): creating and organizing users and groups, registering an application with scoped API permissions, enforcing conditional access policies, enabling MFA, and auditing sign-in activity through logs.

This is part of a 5-project Azure portfolio built toward the **AZ-104: Microsoft Azure Administrator** certification.

---

## Business Scenario

A company wants to integrate their web application with Entra ID for authentication. Requirements:
- Manage users and groups with differentiated access levels
- Register the application so it can authenticate against Entra ID
- Enforce MFA for all users accessing the application
- Restrict access based on location or device compliance
- Audit who is logging in and flag failed authentication attempts

---

## Skills Demonstrated

| Skill | Tool/Service |
|---|---|
| User and group management | Microsoft Entra ID |
| Application identity | App Registrations, Client Secrets |
| API permission scoping | Microsoft Graph permissions |
| Access control enforcement | Conditional Access Policies |
| Multi-factor authentication | Entra ID MFA |
| Security auditing | Sign-in logs |

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    Microsoft Entra ID                        │
│                                                              │
│  ┌─────────────────────┐     ┌────────────────────────────┐  │
│  │  Users & Groups     │     │  App Registration          │  │
│  │                     │     │                            │  │
│  │  Admin Group ──┐    │     │  - Client ID               │  │
│  │  Dev Group   ──┼────┼────►│  - Client Secret           │  │
│  │  Reader Group──┘    │     │  - API Permissions         │  │
│  └─────────────────────┘     │    (Microsoft Graph)       │  │
│                              └────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Conditional Access Policy                           │    │
│  │                                                      │    │
│  │  IF: User accesses the app                          │    │
│  │  AND: Location is outside trusted range             │    │
│  │  THEN: Require MFA                                  │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  Sign-In Logs → Audit trail of all auth attempts     │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Walkthrough

### Step 1 — Create Users and Groups

1. Navigate to **Entra ID > Users > + Create User**
2. Create users with distinct roles:

| Display Name | Role Purpose |
|---|---|
| Nick Admin | Full administrative access |
| Nick Developer | Application-level access |
| Nick Reader | Read-only access |

3. Navigate to **Groups > + New Group**
   - Type: Security
   - Add the relevant users to each group
   - Assign group owners

> **Security Groups vs. Microsoft 365 Groups:** Security groups are used for access control and RBAC assignments. M365 groups are for collaboration (Teams, SharePoint). Use Security groups for identity and access management.

---

### Step 2 — Register an Application in Entra ID

1. Navigate to **App Registrations > + New Registration**
2. Configure:
   - **Name:** your application name
   - **Supported account types:** Accounts in this organizational directory only
   - **Redirect URI:** `https://your-app-url/auth/callback` (Web)
3. After registration, note the **Application (client) ID** and **Directory (tenant) ID**

**Add API Permissions:**
1. App Registration > **API Permissions > + Add a permission > Microsoft Graph**
2. Select **Delegated permissions > User.Read**
3. Click **Grant admin consent**

**Create a Client Secret:**
1. App Registration > **Certificates & Secrets > + New Client Secret**
2. Set expiry (recommend 12 months max)
3. **Copy the secret value immediately** — it will not be shown again

> **Client Secret vs. Certificate:** Client secrets are simpler but carry risk if leaked. Certificates (stored in Key Vault) are the production best practice. For this lab, a secret is sufficient.

---

### Step 3 — Assign Users and Groups to the Application

1. Navigate to **Enterprise Applications** > select your registered app
2. Go to **Users and Groups > + Add User/Group**
3. Assign each group created in Step 1 with appropriate roles

This ensures only members of the assigned groups can authenticate to the application.

---

### Step 4 — Configure Conditional Access

1. Navigate to **Entra ID > Security > Conditional Access > + New Policy**
2. Configure:

**Assignments:**
- **Users:** All users (or target specific groups)
- **Cloud apps:** Select your registered application
- **Conditions:** Add location-based or device-compliance conditions as needed

**Access Controls:**
- **Grant:** Require multi-factor authentication

3. Set policy state to **On** (use **Report-only** first to preview impact without enforcing)
4. Save the policy

> **Report-only mode** is a best practice before enabling enforcement — it logs what *would* have been blocked without actually blocking anyone. Use it to validate scope before going live.

---

### Step 5 — Enable Multi-Factor Authentication

1. Navigate to **Entra ID > Users > Per-User MFA** (legacy method)
   - Select target users > **Enable**
2. **Preferred approach:** Let Conditional Access drive MFA enforcement (Step 4) rather than per-user MFA, which is legacy and less flexible

**Test MFA flow:**
1. Open an InPrivate/Incognito browser session
2. Sign in as one of the test users
3. Verify MFA prompt appears (Authenticator app, SMS, or phone call)

---

### Step 6 — Review Sign-In Logs

1. Navigate to **Entra ID > Sign-ins**
2. Filter by user, application, or date range
3. Review:
   - Successful authentications
   - Failed attempts (wrong password, MFA failures)
   - Location and IP of sign-in attempts
   - Whether Conditional Access policies applied

> Sign-in logs retain 30 days by default. For longer retention, export to a **Log Analytics Workspace** or **Storage Account**.

---

## Key Concepts

**App Registration vs. Enterprise Application** — App Registration is where you *define* the application identity (client ID, secret, permissions). Enterprise Application is the *service principal* — the instance of that app in your tenant, where you assign users and configure SSO.

**Conditional Access** — Policy-based access control evaluated at sign-in time. More flexible than per-user MFA: you can require MFA only from specific locations, only for specific apps, or only on non-compliant devices.

**Microsoft Graph** — The unified API for Microsoft 365, Entra ID, and Azure services. `User.Read` is a basic delegated permission allowing an app to read the signed-in user's profile. Always scope permissions to minimum required.

**Least Privilege in Identity** — Assign users to groups with the minimum role needed. Don't assign Global Administrator unless absolutely necessary — use scoped roles like Application Administrator or User Administrator.

---

## Resources

- [Microsoft Entra ID Documentation](https://learn.microsoft.com/en-us/entra/identity/)
- [App Registration Overview](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app)
- [Conditional Access Documentation](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview)
- [Microsoft Graph Permissions Reference](https://learn.microsoft.com/en-us/graph/permissions-reference)
- [Entra ID Sign-In Logs](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-sign-ins)

---

## Part of the AZ-104 Portfolio

| # | Project | Topics |
|---|---|---|
| 1 | Azure Compute and Identity Management | VMs, RBAC, Key Vault, Policy, Cost Management |
| 2 | Azure Networking and Storage | VNets, NSGs, Storage, ARM Templates |
| 3 | Monitoring, Backup, and Recovery | Azure Monitor, Backup, Site Recovery, Log Analytics |
| **4** | **Entra ID Integration and Identity Management** ← *you are here* | Entra ID, App Registration, Conditional Access, MFA |
| 5 | Azure App Service Deployment and Scaling | App Service, Slots, Autoscaling, CI/CD |
