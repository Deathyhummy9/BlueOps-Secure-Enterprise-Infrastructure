#  BlueOps: Secure Enterprise Infrastructure Lab 

# Overview 
A hands-on, enterprise-level IT infrastructure built entirely in **VirtualBox**.  
It replicates a secure multi-VLAN corporate network, complete with **pfSense firewalling**, **Windows Server Active Directory**, and a **Windows 11 domain workstation**.

🧠 Objectives
- Simulate a realistic enterprise network as a Network / Systems Admin would manage.
- Gain practical skills in VLAN segmentation, firewall configuration, DNS, DHCP, and AD.
- Gain practical skills in Siem and zabbix monitoring
- gain practical skills in Vpn usgae and PowerShell automation
- Prepare for Network+ / CCNA / SysAdmin roles through hands-on experience.


🧱 BlueOps – Phase 1: Virtualization & Base Network Setup

Phase 1 lays the foundation of the **BlueOps: Secure Enterprise Infrastructure** project.  
This phase focuses on building the virtual environment in **VirtualBox**, deploying the core VMs, and confirming that **pfSense** is reachable as the network gateway for your lab.

---

###  Goals

- Establish a controlled, enterprise-style virtualization environment  
- Deploy and configure **pfSense** as the central firewall/router  
- Create **internal networks** to simulate VLANs  
- Verify that client systems can access pfSense’s web GUI  
- Prepare the base for multi-VLAN expansion in **Phase 2**

---

###  Step 1 – Create Virtual Machines in VirtualBox

| VM Name | Purpose | OS Type | CPU / RAM / Disk | Network Adapters | Notes |
|----------|----------|---------|------------------|------------------|--------|
| **BlueOps-pfSense** | Core firewall + router | Other / FreeBSD | 2 CPU / 4 GB RAM / 30 GB VDI | <ul><li>Adapter 1 → **NAT** (WAN)</li><li>Adapter 2 → **Internal Network – LAN** (192.168.5.1/24) </li> <li>Adapter 3 → **Internal Network – Vlan10Net** </li> <li>Adapter 4 → **Internal Network – Vlan20Net**  </li> </ul> | Central routing point |
| **BlueOps-DC-Server** | Domain Controller (Windows Server 2022) | Windows Server 2022 (64-bit) | 2 CPU / 4 GB RAM / 50 GB disk | Adapter 1 → **Internal Network – Lvlan10** | Static IP `192.168.10.5` |
| **BlueOps-Client01** | Windows 11 Pro domain workstation | Windows 11 Pro (64-bit) | 2 CPU / 4 GB RAM / 40 GB disk | Adapter 1 → **Internal Network – vlan10** | Must be Pro edition to join domain | ip 192.168.10.10
| **BlueOps-Mint-Admin** | Linux Mint Admin VM for pfSense GUI access | Linux Mint 21.2 | 1 CPU / 2 GB RAM / 20 GB disk | Adapter 1 → **Internal Network – LAN** | Used to configure pfSense via browser | ip 192.168.5.19

> 🔸 Later in **Phase 2**, additional internal networks (`VLAN10Net`, `VLAN20Net`, `VLAN30Net`) will be added for segmented subnets.

---

###  Step 2 – Install pfSense

1. Download the **pfSense 2.7.1 ISO** from Netgate.  
2. In VirtualBox → **Storage**, mount the ISO to BlueOps-pfSense.  
3. Boot → choose **Install pfSense** → accept defaults.  
4. After reboot, choose **Option 2 – Set Interface IPs**:
   - **WAN:** auto-assigned via DHCP (NAT)  
   - **LAN:** `192.168.5.1/24`  
5. Verify LAN gateway by pinging from pfSense console:
   ```bash
   ping 8.8.8.8      # optional Internet check
   ping 192.168.5.1  # confirm LAN gateway


### Step 3 – Access pfSense Web GUI

Boot the Linux Mint Admin VM attached to the LAN.

In terminal:

ifconfig


Confirm it received an IP (e.g. 192.168.5.10) or assign one manually.

Open a browser → https://192.168.5.1

Default login:

Username: admin

Password: pfsense

Complete the setup wizard (skip WAN change)

###Step 4 – Verify Connectivity

From the Linux Mint terminal
ping 192.168.5.1        # pfSense gateway
ping 8.8.8.8            # verify internet

# 🧱 BlueOps – Phase 2: VLAN Segmentation & pfSense Interface Configuration

Phase 2 expands the **BlueOps Secure Enterprise Infrastructure** by introducing **VLAN segmentation**.  
This phase converts the single LAN from Phase 1 into a multi-network environment that mirrors a real-world enterprise, where servers, users, and guests are separated by virtual LANs routed through **pfSense**.

---

### 🎯 Goals

- Implement **VLAN-based network segmentation** on pfSense  
- Assign subnets and gateways for Admin, User, and Guest networks  
- Enable **DHCP** and **routing** for each VLAN  
- Verify end-to-end connectivity between VLANs and Internet access  
- Prepare the environment for **firewall rule enforcement** in Phase 3

- ###  Step 1 – Create VLAN Interfaces in pfSense

1. In pfSense, go to **Interfaces → Assignments → VLANs**.  
2. Click **Add** and configure:
   - **Parent Interface:** `em2` (LAN adapter)  
   - **VLAN Tag:** `10`  → Description: `VLAN10_Admin`  
   - **VLAN Tag:** `20`  → Description: `VLAN20_Users`  
   - **VLAN Tag:** `30`  → Description: `VLAN30_Guests`
3. Return to **Interfaces → Assignments**, add each VLAN as a new interface, and enable them.  
4. Rename for clarity:
   - OPT1 → VLAN10_Admin/servers  
   - OPT2 → VLAN20_Users  
   - OPT3 → VLAN30_Guests  

---

###  Step 2 – Assign IPs + Enable DHCP

For each VLAN interface:

| Interface | Static IP / Mask | DHCP Enabled | DHCP Scope |
|------------|-----------------|---------------|-------------|
| **LAN** | `192.168.5.1/24` |  | `192.168.5.50 – 192.168.5.150` |
| **VLAN10_Admin** | `192.168.10.1/24` |  | `192.168.10.100 – 192.168.10.199` |
| **VLAN20_Users** | `192.168.20.1/24` |  | `192.168.20.100 – 192.168.20.199` |
| **VLAN30_Guests** | `192.168.30.1/24` |  | `192.168.30.100 – 192.168.30.199` |

>  *Tip:* Reserve `.5` – `.50` in VLAN10 for servers and `.10` – `.20` for testing workstations.

---

###  Step 3 – Connect VMs to VLANs

1. In VirtualBox, attach:  
   - **BlueOps-DC-Server → Internal Network → VLAN10Net**  
   - **BlueOps-Client01 → Internal Network → VLAN10Net** (for now; future VLAN20 VM will be added later)  
   - **BlueOps-Mint-Admin → Internal Network → LAN**  
2. Reboot each VM and confirm IP assignment via DHCP or static address.  

---

###  Step 4 – Verify Routing & Connectivity

From each network, confirm reachability:

### Linux Mint (Admin Network)
ping 192.168.5.1       # pfSense LAN gateway
ping 8.8.8.8           # Internet reach

# 🔒 BlueOps – Phase 3: Firewall Policy & Access Control

Phase 3 transitions the BlueOps network from open routing to a **controlled, security-enforced environment**.  
Here, I designed and implemented **pfSense firewall rules** to define which VLANs can communicate — ensuring isolation, least-privilege access, and safe Internet routing.

---

### 🎯 Goals

- Apply **principle of least privilege** across all VLANs  
- Configure **firewall rules** per interface (LAN, VLAN10, VLAN20, VLAN30)  
- Allow required services (DNS, AD, RDP) and block unnecessary traffic  
- Maintain Internet access while limiting lateral movement  
- Validate rule enforcement through controlled connectivity tests

---

### ⚙️ Network Context

| Interface | Purpose | Subnet | Gateway | Notes |
|------------|----------|---------|----------|--------|
| **LAN** | Management & pfSense GUI access | `192.168.5.0/24` | `192.168.5.1` | Used by Admin (Linux Mint) |
| **VLAN10_Admin** | Domain Controller + Admin Systems | `192.168.10.0/24` | `192.168.10.1` | Contains Windows Server 2022 |
| **VLAN20_Users** | Standard Workstations | `192.168.20.0/24` | `192.168.20.1` | Office users needing DNS/AD access |
| **VLAN30_Guests** | Guest Wi-Fi / Isolated Network | `192.168.30.0/24` | `192.168.30.1` | Internet-only, no internal access |

---

###  Step 1 – Review Default Rules

- pfSense defaults to **allow all LAN → any** traffic.  
- Newly created VLAN interfaces have **no rules** (all traffic blocked).  
- Objective: explicitly define what each network can reach.

---

###  Step 2 – Create VLAN10 (Admin) Rules

**Purpose:** Allow full management capability and AD services.

| Action | Protocol | Source | Destination | Ports | Description |
|--------|-----------|---------|--------------|--------|--------------|
|  Pass | Any | `192.168.10.0/24` | Any | Any | Full access for Admin VLAN |
|  Pass | ICMP | `192.168.10.0/24` | Any | * | Allow ping/troubleshooting |
|  Block | Any | `192.168.10.0/24` | `192.168.30.0/24` | * | Prevent Admin ↔ Guest traffic |

>  This network mimics IT staff with elevated access — needed for managing servers, AD, and pfSense itself.

---

###  Step 3 – Create VLAN20 (User) Rules

**Purpose:** Allow users to authenticate to AD/DNS but restrict them from reaching admin systems.

| Action | Protocol | Source | Destination | Ports | Description |
|--------|-----------|---------|--------------|--------|--------------|
|  Pass | TCP/UDP | `192.168.20.0/24` | `192.168.10.5` | 53, 389, 445, 3389 | Allow DNS, LDAP, SMB, RDP to DC |
|  Pass | Any | `192.168.20.0/24` | Any | 80, 443 | Allow web access |
|  Block | Any | `192.168.20.0/24` | `192.168.5.0/24` | * | Deny management-LAN access |
|  Pass | ICMP | `192.168.20.0/24` | Any | * | Allow ping (diagnostics) |
|  Block | Any | `192.168.20.0/24` | `192.168.30.0/24` | * | Deny user-guest traffic |

---

###  Step 4 – Create VLAN30 (Guest) Rules

**Purpose:** Give Internet-only access — no LAN or VLAN reachability.

| Action | Protocol | Source | Destination | Ports | Description |
|--------|-----------|---------|--------------|--------|--------------|
|  Pass | TCP/UDP | `192.168.30.0/24` | Any | 80, 443 | Allow Internet web browsing |
|  Block | Any | `192.168.30.0/24` | `192.168.10.0/24` | * | Block access to Admin VLAN |
|  Block | Any | `192.168.30.0/24` | `192.168.20.0/24` | * | Block access to Users VLAN |
|  Block | Any | `192.168.30.0/24` | `192.168.5.0/24` | * | Block access to pfSense Mgmt LAN |

---

###  Step 5 – NAT & Outbound Access

Ensure all internal networks can reach the Internet:

1. Go to **Firewall → NAT → Outbound**  
2. Select **Automatic Outbound NAT** (default) or add manual rules:  
   - Source Networks: `192.168.5.0/24`, `192.168.10.0/24`, `192.168.20.0/24`, `192.168.30.0/24`  
   - Translation Interface: **WAN**

---

### 🧪 Step 6 – Validate Rule Enforcement

Run connectivity checks:

#### From VLAN10 (Server)

ping 192.168.10.1
ping 8.8.8.8
tracert 192.168.20.10    # should succeed
tracert 192.168.30.10    # should fail
From VLAN20 (User)

Copy code
ping 192.168.10.5         # should succeed (to DC)
ping 192.168.5.1          # should fail (mgmt LAN)
From VLAN30 (Guest)

Copy code
ping 8.8.8.8              # works
ping 192.168.10.5         # blocked
ping 192.168.5.1          # blocked
