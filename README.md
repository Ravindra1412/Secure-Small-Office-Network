# Secure Small Office Network Using VLANs, Inter-VLAN Routing, DHCP, DNS and ACL-Based Access Control

[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco%20Packet%20Tracer-v8.0%2B%20%7C%20v9.0-blue.svg)](https://www.netacad.com/)
[![Course](https://img.shields.io/badge/Course-BCSE308P%20Computer%20Networks%20Lab-orange.svg)](https://vit.ac.in)
[![Topology](https://img.shields.io/badge/Topology-Router--on--a--Stick%20(802.1Q)-green.svg)]()
[![Security](https://img.shields.io/badge/Security-Extended%20ACL%20%2B%20Port%20Security-red.svg)]()

---

## 1. Project Overview

This project designs, implements, and rigorously tests a segmented, secure small office local area network (LAN) modeled inside **Cisco Packet Tracer**. The network establishes departmental isolation across three distinct corporate units—**Human Resources (HR)**, **Information Technology (IT)**, and **Sales**—while maintaining controlled inter-VLAN routing, automated host configuration, internal DNS name resolution, an intranet HTTP portal, and Layer 2 / Layer 3 access control mechanisms.

The entire infrastructure operates through a centralized Cisco 2911 router and a Catalyst 2960 switch using an **802.1Q Router-on-a-Stick** design.

---

## 2. Problem Statement

A growing small business operating on a traditional "flat" (unsegmented) Layer 2 switch network encounters severe operational and security challenges:
1. **Unrestricted Inter-Department Visibility:** Workstations in Sales can freely snoop, probe, or access confidential employee files and servers located within the HR department.
2. **Excessive Broadcast Overhead:** ARP broadcasts, DHCP discovery traffic, and link-layer floods propagate across the entire office, degrading overall network performance.
3. **Manual Addressing Burden:** Manually maintaining static IP addresses across multi-department endpoints is error-prone, leading to address collisions and administrative overhead.
4. **Physical Port Vulnerabilities:** Any rogue visitor or unapproved personal laptop plugged into an unused wall jack gains full visibility into the company network without physical port authentication.

---

## 3. Objectives

- **Logical Network Segmentation:** Partition departmental traffic into independent virtual broadcast domains: **VLAN 10 (HR)**, **VLAN 20 (IT)**, and **VLAN 30 (Sales)**.
- **Inter-VLAN Routing:** Implement 802.1Q trunking and router-on-a-stick subinterfaces on a Cisco 2911 router to route between authorized subnets over a single physical link.
- **Automated IP Management:** Configure router-based dynamic IP allocation via DHCP pools, preserving a reserved block (`.1` to `.20`) for servers and default gateways.
- **Internal Network Services:** Deploy a central company server providing:
  - **DNS Service:** Resolves `company.local` to `192.168.10.10`.
  - **HTTP Intranet Portal:** Hosts an internal webpage displaying company announcements and department shortcuts.
- **Layer 3 Access Control:** Enforce an extended Access Control List (**`SALES_RESTRICTION`**) inbound on the Sales subinterface to permit Sales access to the company server while strictly denying Sales access to HR workstations.
- **Layer 2 Edge Protection:** Configure switch port security on all end-user access ports (Fa0/1 to Fa0/9) using **sticky MAC learning**, a maximum limit of 1 MAC address per port, and violation mode set to **shutdown**.
- **Systematic Verification & Troubleshooting:** Document real failure modes (mispatched trunk and access cabling) encountered during deployment and prove policy enforcement with packet traces and match counters.

---

## 4. Features Implemented

- [x] **802.1Q Departmental VLAN Segmentation** (VLAN 10, 20, 30).
- [x] **Router-on-a-Stick (ROAS)** using subinterfaces `Gi0/0.10`, `Gi0/0.20`, and `Gi0/0.30`.
- [x] **Cisco IOS DHCP Server** with 3 distinct pools, default gateway propagation, and DNS pointer.
- [x] **Centralized Server Application Services** (HTTP Intranet & DNS `company.local` A-record).
- [x] **Extended Access Control List (ACL)** filtering directional IP traffic.
- [x] **Switch Port Security** with persistent sticky MAC learning and auto-shutdown on violation.
- [x] **Diagnostic Fault Isolation** methodology documenting Layer 1/Layer 2 cabling recovery.

---

## 5. Network Topology and Architecture

The architecture utilizes a star-tree hierarchical topology connecting nine end-user workstations (3 per department) and one centralized company server through a Catalyst 2960 switch to a Cisco 2911 router.

```
                               +----------------------------+
                               |     Cisco 2911 Router      |
                               |            (R1)            |
                               +--------------+-------------+
                                              |
                                              | 802.1Q Trunk Link
                                              | (Carries VLANs 1, 10, 20, 30)
                                              | R1 Gi0/0 <---> SW1 Gi0/1
                                              |
                               +--------------+-------------+
                               |    Catalyst 2960 Switch    |
                               |           (SW1)            |
                               +----+---------+--------+----+
                                    |         |        |    \
        +---------------------------+         |        |     +-------------------------+
        |                                     |        |                               |
        v [Fa0/1 - Fa0/3]                     v [Fa0/4 - Fa0/6]                                v [Fa0/7 - Fa0/9]               v [Fa0/10]
+-------------------+                 +-------------------+  +-------------------+     +-----------------------+
|   HR Department   |                 |   IT Department   |  | Sales Department  |     |    Company-Server     |
|      VLAN 10      |                 |      VLAN 20      |  |      VLAN 30      |     |  VLAN 10 (192.168.10.10)|
|  192.168.10.0/24  |                 |  192.168.20.0/24  |  |  192.168.30.0/24  |     |  DNS + HTTP Portal    |
| (HR-PC1..HR-PC3)  |                 | (IT-PC1..IT-PC3)  |  | (SALES-PC1..PC3)  |     +-----------------------+
+-------------------+                 +-------------------+  +-------------------+
```

---

## 6. Hardware and Software Requirements

| Component | Specification / Model | Role in Network |
| :--- | :--- | :--- |
| **Simulation Platform** | Cisco Packet Tracer v8.0, v8.2, or v9.0 | Simulation runtime environment |
| **Router (R1)** | Cisco 2911 Integrated Services Router (ISR) | Default gateway, inter-VLAN routing, DHCP, Extended ACL |
| **Switch (SW1)** | Cisco Catalyst 2960-24TT | Layer 2 switching, VLAN trunking, access ports, port security |
| **Workstations** | 9 × Generic PC-PT | Endpoints for HR (3), IT (3), and Sales (3) |
| **Server** | 1 × Generic Server-PT (`Company-Server`) | Intranet web portal (HTTP) and local DNS server |
| **Cabling** | Copper Straight-Through Ethernet | Physical interconnections |

---

## 7. VLAN and IP Addressing Table

| Department / Node | Device Name | Switch Port | VLAN ID | Network Subnet | Default Gateway | DNS Server | IP Assignment Method |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **HR Workstation 1** | `HR-PC1` | `Fa0/1` | 10 | `192.168.10.0/24` | `192.168.10.1` | `192.168.10.10` | Dynamic (DHCP: `192.168.10.22`) |
| **HR Workstation 2** | `HR-PC2` | `Fa0/2` | 10 | `192.168.10.0/24` | `192.168.10.1` | `192.168.10.10` | Dynamic (DHCP: `192.168.10.21`) |
| **HR Workstation 3** | `HR-PC3` | `Fa0/3` | 10 | `192.168.10.0/24` | `192.168.10.1` | `192.168.10.10` | Dynamic (DHCP: `192.168.10.23`) |
| **IT Workstation 1** | `IT-PC1` | `Fa0/4` | 20 | `192.168.20.0/24` | `192.168.20.1` | `192.168.10.10` | Dynamic (DHCP: `192.168.20.21`) |
| **IT Workstation 2** | `IT-PC2` | `Fa0/5` | 20 | `192.168.20.0/24` | `192.168.20.1` | `192.168.10.10` | Dynamic (DHCP: `192.168.20.23`) |
| **IT Workstation 3** | `IT-PC3` | `Fa0/6` | 20 | `192.168.20.0/24` | `192.168.20.1` | `192.168.10.10` | Dynamic (DHCP: `192.168.20.22`) |
| **Sales Workstation 1** | `SALES-PC1`| `Fa0/7` | 30 | `192.168.30.0/24` | `192.168.30.1` | `192.168.10.10` | Dynamic (DHCP: `192.168.30.23`) |
| **Sales Workstation 2** | `SALES-PC2`| `Fa0/8` | 30 | `192.168.30.0/24` | `192.168.30.1` | `192.168.10.10` | Dynamic (DHCP: `192.168.30.21`) |
| **Sales Workstation 3** | `SALES-PC3`| `Fa0/9` | 30 | `192.168.30.0/24` | `192.168.30.1` | `192.168.10.10` | Dynamic (DHCP: `192.168.30.22`) |
| **Company Server** | `Company-Server`| `Fa0/10` | 10 | `192.168.10.0/24` | `192.168.10.1` | `192.168.10.10` | Static (`192.168.10.10`) |
| **Router Subinterfaces**| `R1 Gi0/0.10` | - | 10 | `192.168.10.0/24` | - | - | Static (`192.168.10.1`) |
| | `R1 Gi0/0.20` | - | 20 | `192.168.20.0/24` | - | - | Static (`192.168.20.1`) |
| | `R1 Gi0/0.30` | - | 30 | `192.168.30.0/24` | - | - | Static (`192.168.30.1`) |
| **Trunk Link** | `SW1 Gi0/1` <-> `R1 Gi0/0` | - | 1,10,20,30 | - | - | - | 802.1Q encapsulation |

---

## 8. Configuration Overview

### A. Switch Configuration (Catalyst 2960 - SW1)
1. **VLAN Segmentation:** Created VLANs 10, 20, and 30 with names `HR`, `IT`, and `SALES`.
2. **Access Port Grouping:** Assigned `Fa0/1-3` and `Fa0/10` to VLAN 10; `Fa0/4-6` to VLAN 20; `Fa0/7-9` to VLAN 30.
3. **Trunk Port:** Set `Gi0/1` to trunk mode with `switchport nonegotiate` to transport all VLAN tags to R1.
4. **Port Security:** Configured `switchport port-security`, `mac-address sticky`, `maximum 1`, and `violation shutdown` on `Fa0/1-Fa0/9`.

### B. Router Configuration (Cisco 2911 - R1)
1. **Physical Interface:** `GigabitEthernet0/0` brought up without an IP address.
2. **Subinterfaces:** Configured dot1Q encapsulation:
   - `Gi0/0.10`: `encapsulation dot1Q 10`, `ip address 192.168.10.1 255.255.255.0`
   - `Gi0/0.20`: `encapsulation dot1Q 20`, `ip address 192.168.20.1 255.255.255.0`
   - `Gi0/0.30`: `encapsulation dot1Q 30`, `ip address 192.168.30.1 255.255.255.0`
3. **DHCP Services:** Reserved `.1` to `.20` across each subnet via `ip dhcp excluded-address`. Defined pools `HR_POOL`, `IT_POOL`, and `SALES_POOL` delivering network prefixes, default-router IPs, and DNS server `192.168.10.10`.
4. **Extended Access List (`SALES_RESTRICTION`):**
   ```cisco
   ip access-list extended SALES_RESTRICTION
    10 permit ip 192.168.30.0 0.0.0.255 host 192.168.10.10
    20 deny   ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
    30 permit ip any any
   ```
   Applied inbound on subinterface `Gi0/0.30`:
   ```cisco
   interface GigabitEthernet0/0.30
    ip access-group SALES_RESTRICTION in
   ```

### C. Server Application Services (`Company-Server`)
- **Static Settings:** `192.168.10.10/24`, Gateway: `192.168.10.1`, DNS: `192.168.10.10`.
- **HTTP Service:** Enabled on port 80 hosting `index.html` ("ABC COMPANY - Internal Network Portal").
- **DNS Service:** Enabled with an **A-Record** mapping `company.local` to `192.168.10.10`.

---

## 9. Step-by-Step Instructions to Open and Run the Simulation

1. **Prerequisites:** Ensure **Cisco Packet Tracer** (version 8.0 or newer) is installed.
2. **Download / Clone:**
   ```bash
   git clone https://github.com/Ravindra1412/Secure-Small-Office-Network.git
   cd Secure-Small-Office-Network
   ```
3. **Open Simulation:**
   - Launch Cisco Packet Tracer.
   - Click `File` -> `Open` and navigate to `Simulation/CNproject.pkt`.
4. **Initialize Convergence:**
   - Allow 30 seconds for STP link lights to transition from amber to green.
   - (Optional shortcut: Click the "Fast Forward Time" button in the bottom control panel twice).
5. **Inspect Devices:**
   - Double-click switch `SW1` or router `R1`, open the **CLI** tab, and enter `enable` mode.
   - Run verification commands (see Section 10).

---

## 10. Verification and Testing Instructions

Execute the following test sequences directly within the Packet Tracer environment:

### Test 1: Verify Switch VLAN & Trunk Status
- On **SW1 CLI**:
  ```ios
  SW1# show vlan brief
  SW1# show interfaces trunk
  ```
- **Expected:** VLANs 10, 20, 30 active with corresponding ports; Gi0/1 trunking 802.1Q for VLANs 1, 10, 20, 30.

### Test 2: Verify DHCP Address Leases
- On **R1 CLI**:
  ```ios
  R1# show ip dhcp binding
  ```
- **Expected:** Exactly 9 dynamic leases bound to client MAC addresses in the `.21`+ range.

### Test 3: Verify Client IP Configuration
- Click **HR-PC3** -> **Desktop** -> **Command Prompt**:
  ```cmd
  ipconfig
  ```
- **Expected:** IPv4: `192.168.10.23`, Gateway: `192.168.10.1`, DNS: `192.168.10.10`.

### Test 4: Verify Inter-VLAN Routing
- On **HR-PC1 Command Prompt**:
  ```cmd
  ping 192.168.20.21
  ```
- **Expected:** 4/4 replies received, TTL=127 (confirming one routed hop through R1).

### Test 5: Verify DNS Resolution and HTTP Intranet Portal
- On **IT-PC1 Command Prompt**:
  ```cmd
  ping company.local
  ```
  - **Expected:** Resolves to `192.168.10.10` with 4 replies.
- Open **Desktop** -> **Web Browser**, enter `http://company.local` and click **Go**:
  - **Expected:** Company portal webpage loads successfully.

### Test 6: Verify ACL Policy Enforcement (Sales to Server vs Sales to HR)
- On **SALES-PC1 Command Prompt**:
  1. Test server access:
     ```cmd
     ping 192.168.10.10
     ```
     - **Expected:** 4/4 replies received (Permitted by ACL Rule 10).
  2. Test access to HR workstation:
     ```cmd
     ping 192.168.10.21
     ```
     - **Expected:** `Reply from 192.168.30.1: Destination host unreachable.` (Blocked by ACL Rule 20).
- Check counters on **R1 CLI**:
  ```ios
  R1# show ip access-lists SALES_RESTRICTION
  ```
  - **Expected:** Match count incremented on rule 10 (permit) and rule 20 (deny).

### Test 7: Verify Switch Port Security
- On **SW1 CLI**:
  ```ios
  SW1# show port-security
  SW1# show port-security address
  ```
- **Expected:** Fa0/1 to Fa0/9 listed with MaxSecureAddr=1, CurrentAddr=1, ViolationMode=Shutdown. Active sticky MAC entries recorded.

---

## 11. Test Results: Expected vs. Actual Observed Evidence

The following results represent **actual, verified test data** captured directly from the simulation:

| Scenario / Metric | Expected Outcome | Actual Observed Status | Evidence / Recorded Output |
| :--- | :--- | :--- | :--- |
| **Gateway Reachability** | 4/4 ICMP replies from default gateway | **PASS** | 4/4 replies; RTT < 1ms; TTL=255 |
| **Inter-VLAN Routing (HR to IT)** | 4/4 replies across router hop | **PASS** | 4/4 replies; RTT 0-1ms; TTL=127 |
| **DNS Resolution (`company.local`)** | Name resolves to `192.168.10.10` | **PASS** | Resolved to `192.168.10.10`; 4/4 replies |
| **HTTP Intranet Portal** | Webpage displays in browser | **PASS** | Title: "ABC COMPANY - Internal Network Portal" |
| **Sales to Server Access** | Permitted by ACL line 10 | **PASS** | 4/4 replies; line 10 hit counter = 5 matches |
| **Sales to HR Host Access** | Denied by ACL line 20 | **BLOCKED (PASS)** | "Destination host unreachable" from `192.168.30.1`; line 20 hit counter = 4 matches |
| **DHCP Dynamic Allocation** | 9 PCs obtain leases from `.21`+ | **PASS** | 9 active bindings displayed on R1; 0 collisions |
| **Switch Port Security** | Strict 1-MAC limit with sticky learning | **CONFIGURED** | Ports Fa0/1-Fa0/9 enabled; sticky MACs verified |
| **Cabling Fault Isolation** | Diagnosing physical mispatches | **RESOLVED** | 100% loss reduced to 0% loss (mean RTT 4.75 ms) after re-patching trunk and server |

> **Note on Baseline Comparison:** No numerical performance metrics (such as throughput or packet latency under stress) were gathered for an unsegmented network. The baseline comparison is feature-based: the proposed system provides 3 logical broadcast domains, automated DHCP addressing, and Layer 2/3 access controls that do not exist on a flat switch network.

---

## 12. Screenshots and Documentation Evidence

All evidence screenshots are stored in the `Screenshots/` directory:

| Filename | Description | Corresponding Lab Scenario |
| :--- | :--- | :--- |
| [`01_Network_Topology.png`](Screenshots/01_Network_Topology.png) | Complete Cisco Packet Tracer visual topology showing all devices and subnets | Network Architecture & Deployment |
| [`02_SW1_VLAN_Configuration.png`](Screenshots/02_SW1_VLAN_Configuration.png) | Catalyst 2960 `show vlan brief` showing VLAN 10, 20, 30 port assignments | Layer 2 Segmentation |
| [`03_R1_DHCP_Bindings.png`](Screenshots/03_R1_DHCP_Bindings.png) | Cisco 2911 `show ip dhcp binding` demonstrating 9 active client leases | Automated IP Configuration |
| [`04_Host_IP_Configuration.png`](Screenshots/04_Host_IP_Configuration.png) | HR-PC3 `ipconfig` output displaying IP `192.168.10.23` and DNS `192.168.10.10` | Host Network Settings |
| [`05_Web_Portal_HTTP_Company_Local.png`](Screenshots/05_Web_Portal_HTTP_Company_Local.png) | Packet Tracer desktop web browser accessing `http://company.local` | Application Layer Services |
| [`06_InterVLAN_Ping_Test.png`](Screenshots/06_InterVLAN_Ping_Test.png) | ICMP reachability from HR-PC1 to IT-PC1 (`192.168.20.21`) with TTL=127 | Inter-VLAN Routing |
| [`07_DNS_Resolution_and_ACL_Block.png`](Screenshots/07_DNS_Resolution_and_ACL_Block.png) | Ping resolving `company.local` and ICMP destination unreachable to HR-PC2 | DNS & ACL Enforcement |
| [`08_R1_ACL_Hit_Counters.png`](Screenshots/08_R1_ACL_Hit_Counters.png) | `show ip access-lists` displaying matching packet counters for lines 10, 20, 30 | Access Control Verification |
| [`09_SW1_Port_Security_Status.png`](Screenshots/09_SW1_Port_Security_Status.png) | `show port-security` and `show port-security interface fa0/1` showing sticky MAC | Physical Port Security |
| [`10_Architecture_Block_Diagram.png`](Screenshots/10_Architecture_Block_Diagram.png) | Architectural block diagram detailing interfaces, subnets, and gateways | Design Specification |
| [`11_ACL_Decision_Flowchart.png`](Screenshots/11_ACL_Decision_Flowchart.png) | Flowchart illustrating top-to-bottom evaluation logic of `SALES_RESTRICTION` | Security Policy Logic |

---

## 13. Known Limitations and Security Considerations

1. **Simulation Boundaries:** The network is simulated in Cisco Packet Tracer. Physical hardware idiosyncrasies, cable attenuation, and real ASIC switching delays were not evaluated.
2. **Single Points of Failure (SPOF):** The network relies on a single router (R1), a single switch (SW1), and a single server link. A failure on interface `Gi0/0` halts all inter-VLAN and server communications.
3. **No WAN / Internet Uplink:** The office LAN is completely isolated without an ISP gateway, NAT/PAT translation, or public DNS root servers.
4. **Unencrypted Application Traffic:** Internal web services utilize plaintext **HTTP (port 80)**. While Packet Tracer supports HTTPS, TLS certificates were not deployed.
5. **Broad ACL Permit Rule:** The extended ACL permits **all IP protocols** (`permit ip 192.168.30.0 ... host 192.168.10.10`) from Sales to the company server, rather than restricting access strictly to HTTP (port 80) and DNS (port 53). A least-privilege enhancement would refine this rule.
6. **Bidirectional Investigation Note:** In earlier testing, reverse connectivity from HR-PC1 to Sales-PC1 failed. Because the ACL is applied inbound on `Gi0/0.30`, it should not block HR-initiated requests. This remains an item for deeper packet inspection.

---

## 14. Project File Structure

```
Secure-Small-Office-Network/
│
├── README.md                                 # Master project documentation
│
├── Simulation/
│   └── CNproject.pkt                         # Original Cisco Packet Tracer simulation
│
├── Report/
│   ├── CN_Report_Captioned.pdf               # Verified final 27-page lab report (PDF)
│   ├── CN_Report_Captioned.docx              # Final captioned report (DOCX)
│   ├── CN_Report_Original.docx               # Preserved untouched original report
│   └── CN_Report.docx                        # Working report copy
│
├── Presentation/
│   └── Secure_Small_Office_Network_Presentation.pptx # Verified 10-slide project presentation
│
├── Screenshots/                              # Genuine experimental evidence
│   ├── 01_Network_Topology.png
│   ├── 02_SW1_VLAN_Configuration.png
│   ├── 03_R1_DHCP_Bindings.png
│   ├── 04_Host_IP_Configuration.png
│   ├── 05_Web_Portal_HTTP_Company_Local.png
│   ├── 06_InterVLAN_Ping_Test.png
│   ├── 07_DNS_Resolution_and_ACL_Block.png
│   ├── 08_R1_ACL_Hit_Counters.png
│   ├── 09_SW1_Port_Security_Status.png
│   ├── 10_Architecture_Block_Diagram.png
│   └── 11_ACL_Decision_Flowchart.png
│
├── Configurations/                           # Device configuration exports
│   ├── SW1_Switch_Configuration.cfg          # Switch VLAN, trunk, and port security commands
│   ├── R1_Router_Configuration.cfg           # Router subinterfaces, DHCP, and ACL commands
│   └── Server_Services_Configuration.txt     # DNS and HTTP application services profile
│
├── Documentation/                            # Engineering supporting documentation
│   ├── Addressing_and_VLAN_Table.md          # IP addressing and switch port mapping
│   ├── Test_Plan_and_Results.md              # Test matrix, metrics, and comparisons
│   └── Verification_and_Troubleshooting_Guide.md # Step-by-step evaluation guide
│
└── Submission/                               # Prepared archives for VTOP upload
```

---

## 15. Team Members

| Registration Number | Student Name | Degree & Specialization | Institution |
| :--- | :--- | :--- | :--- |
| **24BCI0104** | **Ravindra Swami** | B.Tech. Computer Science and Engineering | School of Computer Science and Engineering (SCOPE), VIT |
| **24BKT0101** | **Jash Chordiya** | B.Tech. Computer Science and Engineering | School of Computer Science and Engineering (SCOPE), VIT |
| **24BAI0012** | **Pratyush Kumar Sahoo** | B.Tech. Computer Science and Engineering | School of Computer Science and Engineering (SCOPE), VIT |

---

## 16. References and Acknowledgements

1. **IEEE Std 802.1Q-2018:** *IEEE Standard for Local and Metropolitan Area Networks—Bridges and Bridged Networks*, IEEE, 2018.
2. **RFC 2131:** R. Droms, *"Dynamic Host Configuration Protocol"*, IETF Network Working Group, March 1997.
3. **RFC 1035:** P. Mockapetris, *"Domain Names - Implementation and Specification"*, IETF, November 1987.
4. **RFC 1918:** Y. Rekhter et al., *"Address Allocation for Private Internets"*, IETF, February 1996.
5. **RFC 9110:** R. Fielding et al., *"HTTP Semantics"*, IETF, June 2022.
6. **CCNA 200-301 Official Cert Guide:** W. Odom, Cisco Press, 2019.
7. **Catalyst 2960 and IOS Configuration Guides:** Cisco Systems, Inc.
8. **Computer Networking: A Top-Down Approach (8th Ed.):** J. F. Kurose and K. W. Ross, Pearson, 2021.
9. **Cisco Networking Academy:** Packet Tracer simulation environment ([netacad.com](https://www.netacad.com/)).
10. Special thanks to our course faculty, **Prof. SREETHAR S**, for guidance throughout the **BCSE308P Computer Networks Lab** course.
