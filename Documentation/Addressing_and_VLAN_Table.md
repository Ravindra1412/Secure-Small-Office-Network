# Addressing & VLAN Plan

This document details the complete Layer 2 and Layer 3 segmentation scheme for the Secure Small Office Network project.

## 1. VLAN Allocation Table

| VLAN ID | VLAN Name | Purpose / Department | Subnet Network | Subnet Mask | Usable Host Range | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **10** | `HR` | Human Resources Department | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.1` - `192.168.10.254` | `192.168.10.1` (R1 Gi0/0.10) |
| **20** | `IT` | Information Technology | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.1` - `192.168.20.254` | `192.168.20.1` (R1 Gi0/0.20) |
| **30** | `SALES` | Sales & Marketing | `192.168.30.0/24` | `255.255.255.0` | `192.168.30.1` - `192.168.30.254` | `192.168.30.1` (R1 Gi0/0.30) |
| **1** | `default` | Native VLAN / Management | - | - | - | - |

---

## 2. IP Addressing & Switch Port Assignment Table

| Device Name | Device Model | Connected Port | VLAN | IP Assignment | IP Address | Subnet Mask | Default Gateway | DNS Server |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **R1** | Cisco 2911 ISR | Gi0/0 | Trunk | Static | - | - | - | - |
| **R1 (Gi0/0.10)**| Subinterface | Gi0/0.10 | 10 | Static | `192.168.10.1` | `255.255.255.0` | - | - |
| **R1 (Gi0/0.20)**| Subinterface | Gi0/0.20 | 20 | Static | `192.168.20.1` | `255.255.255.0` | - | - |
| **R1 (Gi0/0.30)**| Subinterface | Gi0/0.30 | 30 | Static | `192.168.30.1` | `255.255.255.0` | - | - |
| **SW1** | Catalyst 2960 | Gi0/1 | Trunk (802.1Q) | - | - | - | - | - |
| **HR-PC1** | PC-PT | Fa0/1 | 10 | DHCP | `192.168.10.22` | `255.255.255.0` | `192.168.10.1` | `192.168.10.10` |
| **HR-PC2** | PC-PT | Fa0/2 | 10 | DHCP | `192.168.10.21` | `255.255.255.0` | `192.168.10.1` | `192.168.10.10` |
| **HR-PC3** | PC-PT | Fa0/3 | 10 | DHCP | `192.168.10.23` | `255.255.255.0` | `192.168.10.1` | `192.168.10.10` |
| **IT-PC1** | PC-PT | Fa0/4 | 20 | DHCP | `192.168.20.21` | `255.255.255.0` | `192.168.20.1` | `192.168.10.10` |
| **IT-PC2** | PC-PT | Fa0/5 | 20 | DHCP | `192.168.20.23` | `255.255.255.0` | `192.168.20.1` | `192.168.10.10` |
| **IT-PC3** | PC-PT | Fa0/6 | 20 | DHCP | `192.168.20.22` | `255.255.255.0` | `192.168.20.1` | `192.168.10.10` |
| **SALES-PC1** | PC-PT | Fa0/7 | 30 | DHCP | `192.168.30.23` | `255.255.255.0` | `192.168.30.1` | `192.168.10.10` |
| **SALES-PC2** | PC-PT | Fa0/8 | 30 | DHCP | `192.168.30.21` | `255.255.255.0` | `192.168.30.1` | `192.168.10.10` |
| **SALES-PC3** | PC-PT | Fa0/9 | 30 | DHCP | `192.168.30.22` | `255.255.255.0` | `192.168.30.1` | `192.168.10.10` |
| **Company-Server**| Server-PT | Fa0/10 | 10 | Static | `192.168.10.10` | `255.255.255.0` | `192.168.10.1` | `192.168.10.10` |

---

## 3. DHCP Pool & Reservation Summary

To prevent IP collisions with statically addressed infrastructure:
- **Excluded Addresses:**
  - `192.168.10.1` to `192.168.10.20`
  - `192.168.20.1` to `192.168.20.20`
  - `192.168.30.1` to `192.168.30.20`
- **Leasable Client Dynamic Range:** `.21` through `.254` across all three subnets.
- **Default Router Option:** Configured to the respective router subinterface `.1` address.
- **DNS Server Option:** Configured to `192.168.10.10` across all three DHCP pools.
