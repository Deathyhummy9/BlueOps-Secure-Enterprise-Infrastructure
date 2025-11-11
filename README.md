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
