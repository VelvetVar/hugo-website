---
title: "Enterprise Active Directory Homelab"
weight: 1
date: 2023-10-27
draft: false
bookFlatSection: false
---

# Enterprise Active Directory Homelab

## Project Overview

| Attribute | Details |
| :--- | :--- |
| **Version** | 1.2 |
| **Status** | In Progress |
| **Infrastructure** | HP ProDesk 600 G6 (Proxmox VE)|
| **Objective** | Deploy a functional Active Directory environment to simulate Helpdesk, SysAdmin, and Security operations. |

---

## 1. Directory Services Setup (AD DS)

### Environment Configuration

* **Server:** Windows Server 2019/2022 (Hostname: `DC01`)
* **Client:** Windows 10/11 Enterprise
* **Domain:** `homelab.local`

### Organizational Structure (OUs)

To facilitate granular Group Policy inheritance, the following hierarchical structure was implemented:

* **Root:** `homelab.local`
    * **_Corp** (Parent OU for all organizational assets)
        * **HR** (Contains departmental users, e.g., *John HRDoe*)
        * **IT** (Administrative accounts)
        * **Sales** (Departmental assets)

### User & Group Management

* **Standard User Identity:**
    * **Name:** John HRDoe
    * **Username:** `JHRDoe`
    * **Location:** `_Corp/HR`
* **Security Groups:**
    * `HR_Files_RW`: Created for Read/Write permission management within the HR scope.
* **Domain Join Procedure:**
    1.  Configure Client VM DNS to point to the Domain Controller IP.
    2.  Join Client to `homelab.local` domain.

---

## 2. File Services & Storage

**Goal:** Establish a "Company Public" drive accessible to all authenticated employees.

### Permission Strategy

We utilized the **"Open Share, Restricted Security"** model to ensure ease of mapping while maintaining strict security via NTFS.

#### Share Permissions
* **Path:** `C:\Public`
* **Access:** `Everyone` -> **Full Control**

#### NTFS (Security) Permissions
* **Access:** `Domain Users` -> **Modify**

> **Result:** The network share is discoverable ("Open Door"), but the underlying file system restricts access strictly to authenticated domain users ("Locked Vault").

---

## 3. Group Policy Management (GPO)

All policies were linked to the **_Corp** OU to ensure inheritance across all sub-departments (HR, IT, Sales).

### Policy A: Automatic Drive Mapping

**Objective:** Automatically map the Public folder to the `Z:` drive for all users.

* **Configuration Path:** `User Configuration > Preferences > Drive Maps`

#### Troubleshooting Record

| Issue | Resolution |
| :--- | :--- |
| **Drive failed to appear** | **Fix 1:** Manually input the UNC path (`\\ServerIP\Public`) instead of browsing for it.<br>**Fix 2:** Enabled *"Run in logged-on user's security context"* in the Common tab of the policy. |

### Policy B: Corporate Wallpaper

**Objective:** Enforce a standard desktop background across the organization.

* **Configuration Path:** `User Config > Admin Templates > Desktop > Desktop Wallpaper`

#### Troubleshooting Record

| Issue | Root Cause | Resolution |
| :--- | :--- | :--- |
| **Wallpaper failed to update** | Path was set to local `C:\...`, causing the client to search its own local drive. | Changed path to UNC format: `\\192.168.x.x\Public\background.jpg`. <br> *Note: Using the IP address mitigates potential DNS latency issues.* |

---

## 4. Troubleshooting Knowledge Base (KB)

The following common issues were encountered and resolved during deployment.

| Symptom | Root Cause | Resolution |
| :--- | :--- | :--- |
| **"Network Path Not Found"** | DNS Failure | Client VM was utilizing the Router's DNS. Changed Client Network Adapter IPv4 DNS settings to point **only** to the Domain Controller's Static IP. |
| **GPO Not Applying (Linking)** | Linking Error | Confirmed GPO was linked to the **User's OU** (`_Corp` or `HR`), rather than the Computer's OU. |
| **GPO Not Applying (Caching)** | Caching Latency | Executed `gpupdate /force` via command line and performed a full **Sign Out / Sign In** cycle to refresh user token. |