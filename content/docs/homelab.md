---
title: "Homelab Setup - Hardware Focus"
weight: 30
description: "Inventory of the hybrid x86/ARM compute cluster and managed network fabric."
draft: false
---

# Hardware Infrastructure Inventory

This document details the physical assets powering the homelab environment. The infrastructure utilizes a hybrid architecture combining **x86-64** high-performance compute nodes with **ARM64** low-power edge devices to support virtualization, simulation, and continuous network services.

## 1. System Architecture Overview
The environment is designed as a heterogeneous cluster. It prioritizes a balance between raw compute power for virtualization (Type-1 Hypervisor) and energy efficiency for always-on network services.

* **Primary Architecture:** x86-64 (Intel Core)
* **Secondary Architecture:** ARM64 (Rockchip)
* **Operational Goal:** High-performance virtualization with continuous low-power monitoring.

---

## 2. Compute Resources

### Primary Compute Node
**Device:** HP ProDesk 600 G6 SFF

* **Processor:** Intel Core i5-10500 (6 Cores / 12 Threads @ 3.10 GHz Base)
* **Memory:** 32GB DDR4
* **Storage:** 1TB NVMe SSD
* **Form Factor:** Small Form Factor (SFF)

> **Hardware Capability:**
> This node serves as the heavy lifter for the lab. It is configured as a high-performance x86 workstation adapted for **headless virtualization**. The specifications support Type-1 hypervisor deployment (Proxmox VE), multi-tenant virtual machine hosting, and resource-intensive simulation environments.

### Secondary Edge Node
**Device:** Orange Pi 3b

* **Processor:** Rockchip RK3566 (Quad-Core ARM Cortex-A55)
* **Memory:** LPDDR4
* **Connectivity:** Gigabit Ethernet, Wi-Fi 5, Bluetooth 5.0

> **Hardware Capability:**
> A low-power ARM-based Single Board Computer (SBC). Designed for continuous uptime (`24/7`), it is utilized for lightweight networking services, out-of-band management, and independent environment monitoring separate from the main cluster.

---

## 3. Network Infrastructure

### Managed Switching
**Device:** TP-Link TL-SG108E

* **Type:** 8-Port Gigabit Smart Managed Switch
* **Management:** Layer 2

> **Hardware Capability:**
> Supports essential Layer 2 management features including **Port Mirroring**, **QoS**, and **IGMP Snooping**. The hardware is **802.1Q VLAN capable**, enabling future implementation of traffic segmentation and network isolation (e.g., separating IoT traffic from Server traffic).

### WAN Gateway & Access Layer
**Devices:** Arris SB8200 & TP-Link Archer AX55

* **Modem:** DOCSIS 3.1 Cable Modem
* **Router:** AX3000 Wi-Fi 6 Router
* **Throughput:** Gigabit WAN support

> **Hardware Capability:**
> Provides the physical interface to the Wide Area Network (WAN) and local wireless connectivity. The router handles standard NAT, DHCP, and port forwarding functionalities required for external access management and VPN ingress.