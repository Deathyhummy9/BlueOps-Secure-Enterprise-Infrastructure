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

- # 🧱 Phase 1 – Virtualization & Base Network Setup
- Phase 1 lays the foundation for the entire **Secure Enterprise Infrastructure** lab.

In this phase, I:
- Built the virtual environment in **VirtualBox**
- Created internal networks to simulate VLANs
- Deployed core VMs:
  - pfSense firewall
  - Windows Server 2022 (future Domain Controller)
  - Windows client (Windows 11 Pro) for domain join
  - linux mint to configure pfSense on the web gui
 
1. Virtual Machines Creation
   -
  - Name:'Blueops-pfSense', os: other / Free BSD,  CPU: 2, Ram:4 GB, Disk 30GB(VDI)
  - Adapter 1 : Nat, Adapter 2: attach to "Internal Network", Namke-"VLAN10Net"(192.168.10.0/24) role "Admin/server LAN"
  - Adapter 3 " Internal Network "- Name VLAN20Net role:user network (192.168.20.0/24)
  - Adapter 4: "Internal Network"-Name VLAN30Net role:Guests Network


  -Name BlueOps-Client01, OsType:windows 11(64-bit) pro (It has to be pro to join domain)  cpu:2,RAM 4GB, Disk 40GB Adapter 1 attach to internal Network - VLAN10Net ip:192.168.5.1

 -Name BlueOps-DC-Server, OsType:Windows Server 2022 CPU:2, RAM: 4gb disk:50GB ip 192.168.10.10 Adapter 1 :Internal   Network VLAN10Net




3 .pfSense Console Setup:
    After booting pfSense: Just run through the basic set up and make the ip for lan 192.168.5.1
    go on the linux vm make sure its on the same interal net work  and establish Vlans

    
#  BlueOps – Phase 2: VLAN Segmentation & pfSense Interface Configuration

Phase 2 expands the base lab into a segmented enterprise-style network using **pfSense**.  
Here we:
- Create multiple LAN segments (Admin / User / Guest)
- Assign gateways and IP ranges
- Enable DHCP for each subnet
- Verify internal routing and Internet connectivity    
  
 Goals
---
Simulate enterprise-grade network separation  
 Establish pfSense as the central router for all VLANs  
 Provide DHCP + NAT to each subnet  
 Prepare for firewall rule enforcement in Phase 3  

 1. Establish Vlan Interfaces
 - Start with making Vlan 10 with proierty tag 10 and partent em2 repeat adding 10 to each tag and adding 1 to the parent
   ip ranges
| VLAN10_Admin | 192.168.10.100 – 192.168.10.199 | Admin / Server static IPs reserved below .100 |
| VLAN20_Users | 192.168.20.100 – 192.168.20.199 | Typical user workstations |
| VLAN30_Guests | 192.168.30.100 – 192.168.30.199 | Guest Wi-Fi style subnet |

