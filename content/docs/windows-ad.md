---
title: "Windows Server AD on Proxmox"
weight: 80
description: "Step-by-step guide to deploying a Windows Server 2022 Domain Controller on Proxmox VE using VirtIO drivers."
draft: false
---

# Setting up Windows Server AD on Proxmox

Deploying Windows Server on Proxmox VE requires specific configuration to run efficiently. Windows does not natively support the Linux KVM virtualization architecture; therefore, **VirtIO drivers** must be injected during installation to ensure proper storage and network performance.

> **Why VirtIO?**
> Without VirtIO drivers, Windows must use emulated IDE/SATA storage and Intel E1000 networking, which results in significant performance degradation. VirtIO allows the guest OS to "speak" directly to the hypervisor.

## Phase A: The Downloads

Before beginning the VM creation, ensure the following ISO files are uploaded to your Proxmox storage (`local` > **ISO Images** > **Upload**).

1.  **Windows Server 2022 Evaluation**
    * **Source:** [Microsoft Evaluation Center](https://www.microsoft.com/evalcenter/evaluate-windows-server-2022)
    * **Edition:** 64-bit ISO.
2.  **VirtIO Driver ISO**
    * **Source:** [Fedora People (Stable)](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso)
    * **Contents:** Required paravirtualized storage and network drivers.

## Phase B: Virtual Machine Configuration

Use the following specifications to balance performance with resource conservation.

### 1. General & OS
* **Name:** `DC01` (Standard naming convention).
* **Type:** Microsoft Windows.
* **Version:** 11/2022.

### 2. System
* **Graphics Card:** `Default` or `SPICE` (SPICE offers smoother console performance).
* **Qemu Agent:** `Checked` (Crucial for IP reporting and graceful shutdowns).

### 3. Disks (Storage)
* **Bus/Device:** `SCSI` (Best performance).
* **Storage:** `local-lvm` (or preferred storage).
* **Disk Size:** `60 GB` (Min: 30GB for OS; 60GB allows for logs/updates).
* **Cache:** `Write back` (Safe for labs; improves speed).
* **Discard:** `Checked` (TRIM support to save SSD space).

### 4. CPU & Memory
* **Cores:** `2` (DC tasks are generally low-load).
* **Type:** `Host` (Passes through host CPU features).
* **Memory:**
    * **Minimum:** `2048 MB`
    * **Recommended:** `4096 MB` (Server Core) or `6144 MB` (Desktop Experience).

### 5. Network
* **Model:** `VirtIO` (Paravirtualized).
* **Warning:** Do not use Intel E1000; VirtIO supports 10Gbps link speeds.

## Phase C: The "Two-CD" Trick

To install the OS on a SCSI drive, the installer requires access to the drivers simultaneously.

1.  Navigate to **VM** > **Hardware**.
2.  Verify the **Windows ISO** is attached to the primary CD drive.
3.  Click **Add** > **CD/DVD Drive**.
4.  Select the `virtio-win.iso`.

> **Result:** You must have **two** CD drives attached before booting: one containing the OS installer, and one containing the drivers.

## Phase D: Installation & Driver Loading

1.  Start the VM and open the **Console**.
2.  Select **Windows Server 2022 Standard Desktop Experience**.
3.  Proceed to the screen: **"Where do you want to install Windows?"**
    * *Note:* The list will be empty because Windows cannot see the SCSI drive yet.
4.  Click **Load Driver** > **Browse**.
5.  Navigate to the storage driver:
    ```text
    CD Drive (VirtIO) > vioscsi > 2k22 > amd64
    ```
6.  Click **OK/Next**. The 60GB drive will appear.

> **Pro Tip:** You can repeat the "Load Driver" step now for the network driver to ensure connectivity immediately after boot. Navigate to: `NetKVM > 2k22 > amd64`

## Phase E: Post-Install Configuration

### 1. Guest Tools
Once logged into Windows:
1.  Open **File Explorer**.
2.  Open the **VirtIO CD drive**.
3.  Run `virtio-win-guest-tools.exe`.
4.  *Result:* This installs the QEMU agent and fixes screen resolution and mouse latency.

### 2. Promote to Domain Controller
1.  Open **Server Manager**.
2.  Go to **Manage** > **Add Roles and Features** > **Active Directory Domain Services**.
3.  Complete the installation.
4.  Click the **Yellow Flag** notification icon in the top right.
5.  Select **Promote this server to a domain controller**.
6.  Select **Add a new forest** (e.g., `homelab.local` or `ad.yourname.com`).