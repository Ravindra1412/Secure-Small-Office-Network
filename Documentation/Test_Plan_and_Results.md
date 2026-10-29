# Test Plan, Verification Matrix, and Experimental Results

This document records the systematic verification plan, commands executed, observed outcomes, and metrics for the Secure Small Office Network project.

---

## 1. Test Verification Matrix

| Test ID | Test Scenario | Source Device | Target Destination | Command / Tool | Expected Result | Observed Status | Evidence / Metric |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **T01** | Gateway Reachability | HR-PC1 (`192.168.10.22`) | Default Gateway (`192.168.10.1`) | `ping 192.168.10.1` | 4/4 ICMP Echo Replies | **PASS** | 4/4 replies; RTT < 1ms; TTL=255 |
| **T02** | Inter-VLAN Routing | HR-PC1 (`192.168.10.22`) | IT-PC1 (`192.168.20.21`) | `ping 192.168.20.21` | Routed reachability across VLANs | **PASS** | 4/4 replies; RTT 0-1ms; TTL=127 (decremented by 1 router hop) |
| **T03** | Server Reachability | HR-PC1 (`192.168.10.22`) | Server (`192.168.10.10`) | `ping 192.168.10.10` | Switched reachability inside VLAN 10 | **PASS** | 4/4 replies; TTL=128 (Layer 2 switched) |
| **T04** | DNS Resolution | Host PC | `company.local` | `ping company.local` | Name resolves to `192.168.10.10` | **PASS** | Successfully resolved to `192.168.10.10`; 4/4 replies received |
| **T05** | Intranet Web Portal | IT-PC1 (`192.168.20.21`) | `http://company.local` | Packet Tracer Web Browser | HTTP Portal loads successfully | **PASS** | Portal rendered: "ABC COMPANY - Internal Network Portal" |
| **T06** | Sales to Server (ACL Permit) | SALES-PC1 (`192.168.30.23`) | Server (`192.168.10.10`) | `ping 192.168.10.10` | Permitted by ACL rule 10 | **PASS** | 4/4 replies; ACL rule 10 counter incremented (5 matches) |
| **T07** | Sales to HR (ACL Deny) | SALES-PC1 (`192.168.30.23`) | HR-PC2 (`192.168.10.21`) | `ping 192.168.10.21` | Dropped by ACL rule 20 | **BLOCKED (PASS)** | "Destination host unreachable" from `192.168.30.1`; 100% loss; ACL rule 20 counter incremented (4 matches) |
| **T08** | DHCP Allocation | Router R1 | All 9 Host PCs | `show ip dhcp binding` | 9 automatic IP leases | **PASS** | 9 active leases displayed from `.21` upward; 0 collisions |
| **T09** | Switch Port Security | Switch SW1 | Ports Fa0/1 - Fa0/9 | `show port-security` | Enabled, max 1 MAC, sticky | **CONFIGURED (PASS)** | Secure-up status; sticky MACs recorded; violation mode shutdown |
| **T10** | Fault Isolation & Cable Fix | Company-Server | Gateway (`192.168.10.1`) | `ping 192.168.10.1` | Diagnostic isolation of mispatched port | **VERIFIED** | 100% loss corrected to 0% loss (mean RTT 4.75 ms) after re-patching to Fa0/10 |

---

## 2. Experimental Data: Baseline vs. Proposed Implementation

| Evaluation Dimension | Baseline (Conceptual Flat Office LAN) | Proposed Segmented Architecture | Technical Justification / Evidence |
| :--- | :--- | :--- | :--- |
| **Broadcast Domains** | 1 unified broadcast domain (all PCs and servers flood together) | 3 logical broadcast domains (VLAN 10, VLAN 20, VLAN 30) | Switch separation reduces ARP broadcast storms and improves security boundaries. |
| **Inter-Department Routing** | Direct Layer 2 unrestricted reachability | Controlled Layer 3 routing via 802.1Q trunk on R1 Gi0/0 | Packets must traverse router subinterfaces and inspection policies. |
| **IP Address Management** | Manual static assignment (high risk of duplicates/misconfiguration) | Automated Cisco IOS DHCP daemon with pools and exclusions | 9 leases issued automatically from `.21` upward; `.1` - `.20` protected. |
| **Name Resolution Service** | IP-only memorization or decentralized hosts files | Centralized DNS server resolving `company.local` | Verified through command prompt: `ping company.local` -> `192.168.10.10`. |
| **Access Control (HR Isolation)**| None (Sales can directly snoop or access HR hosts) | Extended ACL (`SALES_RESTRICTION`) on Gi0/0.30 inbound | Sales ping to HR host blocked (Destination host unreachable); deny counter: 4 matches. |
| **Access to Shared Server** | Unrestricted / Unfiltered | Controlled permit rule (rule 10) allows Sales to reach `192.168.10.10` | Sales reaches server web/DNS while blocked from all other HR hosts. |
| **Physical Port Security** | Promiscuous (any rogue device or laptop can connect) | Sticky MAC port security on access ports Fa0/1 - Fa0/9 | Only 1 authorized MAC per port; unauthorized frame triggers port shutdown. |
