# Verification & Troubleshooting Guide

This guide provides step-by-step instructions for reproducing, inspecting, and defending the Secure Small Office Network project during lab evaluation or viva voce.

---

## 1. Opening and Initializing the Simulation

1. **Launch Cisco Packet Tracer** (v8.0+ or v9.0+).
2. **Open the project file:** File -> Open -> Select `CNproject.pkt` (located in `Simulation/CNproject.pkt`).
3. **Allow Network Convergence:**
   - Wait 30–45 seconds for Cisco Spanning Tree Protocol (STP) to transition all switch ports from blocking/listening to forwarding (all amber link lights should turn green).
   - Alternatively, click the **Fast Forward Time** button (bottom left toolbar) twice to bypass convergence delays immediately.

---

## 2. Command-Line Verification Checklist

### A. Switch Verification (SW1)
Click on **SW1**, switch to the **CLI** tab, press Enter, and execute:

```ios
SW1> enable
SW1# show vlan brief
```
- **Check:** Verify VLAN 10 (`HR`), VLAN 20 (`IT`), and VLAN 30 (`SALES`) are active and assigned to their respective FastEthernet access ports:
  - Fa0/1 - Fa0/3, Fa0/10 in VLAN 10.
  - Fa0/4 - Fa0/6 in VLAN 20.
  - Fa0/7 - Fa0/9 in VLAN 30.

```ios
SW1# show interfaces trunk
```
- **Check:** Port `Gig0/1` should show mode `on`, encapsulation `802.1q`, status `trunking`, native VLAN `1`, with VLANs `1,10,20,30` allowed and active.

```ios
SW1# show port-security
SW1# show port-security address
```
- **Check:** Fa0/1 through Fa0/9 should have MaxSecureAddr = 1, Violation Action = Shutdown. The address table should show learned `SecureSticky` MACs for active host ports.

---

### B. Router Verification (R1)
Click on **R1**, switch to the **CLI** tab, press Enter, and execute:

```ios
R1> enable
R1# show ip interface brief
```
- **Check:** Verify that `GigabitEthernet0/0` is unassigned but `up/up`, and subinterfaces `Gi0/0.10` (`192.168.10.1`), `Gi0/0.20` (`192.168.20.1`), and `Gi0/0.30` (`192.168.30.1`) are all `up/up`.

```ios
R1# show ip dhcp binding
```
- **Check:** 9 active leases should be displayed (3 for HR in `192.168.10.21-23`, 3 for IT in `192.168.20.21-23`, and 3 for Sales in `192.168.30.21-23`).

```ios
R1# show ip access-lists SALES_RESTRICTION
```
- **Check:** Verify the extended ACL structure and note the packet match counters:
  - Line 10: `permit ip 192.168.30.0 0.0.0.255 host 192.168.10.10`
  - Line 20: `deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255`
  - Line 30: `permit ip any any`

---

## 3. Host Connectivity & Application Services Tests

### A. Automatic IP Configuration
1. Click any PC (e.g., **HR-PC3**), navigate to **Desktop** -> **Command Prompt**.
2. Run `ipconfig`:
   - IPv4 Address should be in `192.168.10.21-23`.
   - Subnet Mask: `255.255.255.0`.
   - Default Gateway: `192.168.10.1`.
   - DNS Server: `192.168.10.10`.

### B. DNS and Web Portal Verification
1. On any workstation (e.g., **IT-PC1**), open **Command Prompt** and type:
   ```cmd
   ping company.local
   ```
   - Host resolves `company.local` to `192.168.10.10` and receives 4 replies (TTL=127).
2. Open **Desktop** -> **Web Browser**, enter URL `http://company.local` and press **Go**:
   - The webpage renders with title: **"ABC COMPANY - Secure Small Office Network | Internal Portal"**.

### C. Access Control List Enforcement
1. Open **SALES-PC1** -> **Command Prompt**:
2. Test access to shared Company-Server:
   ```cmd
   ping 192.168.10.10
   ```
   - **Result:** 4/4 replies received (Permitted by rule 10).
3. Test access to HR workstation (`192.168.10.21`):
   ```cmd
   ping 192.168.10.21
   ```
   - **Result:** `Reply from 192.168.30.1: Destination host unreachable.` (Blocked by rule 20).
4. Switch back to **R1 CLI** and re-run `show ip access-lists` to observe the deny match counter increment by 4.

---

## 4. Real Faults Encountered & Troubleshooting History

During initial implementation in the laboratory, two critical cabling faults were detected and isolated systematically:

1. **Trunk Port Mispatching (R1 Gi0/0 cabled to SW1 Fa0/11 instead of Gi0/1):**
   - *Symptom:* Server ping to default gateway `192.168.10.1` reported 100% loss (0 received out of 4).
   - *Diagnosis:* `show ip interface brief` on SW1 revealed port `Fa0/11` was `up/up` while trunk port `Gi0/1` was `down/down`. R1 had been physically connected to an access port in default VLAN 1 rather than the designated Gigabit trunk port.
   - *Resolution:* Removed the patch cable and reconnected R1 Gi0/0 directly to SW1 Gi0/1. `show interfaces trunk` confirmed 802.1Q trunking became operational.

2. **Server Access Port Mispatching (Company-Server cabled to Fa0/11 instead of Fa0/10):**
   - *Symptom:* Server still unable to reach gateway `192.168.10.1`.
   - *Diagnosis:* Hovering over the switch revealed the server link was on Fa0/11 (assigned to default VLAN 1), isolating the server from VLAN 10.
   - *Resolution:* Re-patched the server cable into Fa0/10 (configured for VLAN 10). Immediate subsequent ping yielded 4/4 replies with 0% loss (average RTT: 4.75 ms).
