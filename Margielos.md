<div align="center"> 
  
# Network Documentation
## The Margielos Project
245 Smith Avenue, Winnipeg,   
HR - 2046678234     
info@margielos.uk
  

</div>


**Margielos** is a small e-commerce clothing company headquartered in Winnipeg, Manitoba.
The business operates from a **single site** (no branches), with approximately **15 users** across:

<div align="center">
  
## _By_ 
## *Azureusers (Group 5)*
</div>

## 1. Introduction & Scope
### 1.1 Organization Overview
* Sales & Customer Service
* Warehouse
* HR & Management
* IT
* Marketing & E-Commerce
* Production Floor / Operations
* Finance
* Guest / Visitor access

The environment is built on:
* Cisco routers and switches (R1, R2, SW1, SW2)
* Cisco ASA firewalls (ASA01, ASA02)
* Windows and Linux virtual machines hosted on Proxmox.
* 
The goal is to simulate a realistic “small production network” for an online retail company, while staying within
the constraints of an NSA630 capstone lab.
### 1.2 Project Objectives
The primary objective of this project is to design and implement a small but realistic enterprise LAN/WAN for
Margielos that:
* **Meets or exceeds the NSA630 capstone rubric,** especially in:
  * Network design and redundancy
  * Core services integration
  * Security hardening and documentation quality
* Provides **VLAN separation** for all major departments (Sales/CS, Warehouse, HR/Management,      IT, Marketing/E-Com, Production Floor, Finance, Guest).
* Delivers **centralized identity and core services:**
  * Active Directory Domain Services
  * DNS
  * DHCP with failover between two domain controllers
    (Details of AD/DNS/DHCP implementation are documented in the server team’s section; this
    document focuses on networking.)
* Implements **redundant default gateways** using HSRP across two routers (R1/R2) and dual      core switches (SW1/SW2), avoiding single points of failure at Layer 3.
* Provides secure **Internet access** for internal users through ASA firewalls and NAT to the MITT Capstone network.
* Delivers **staff and guest Wi-Fi** mapped to the appropriate VLANs, with guest users isolated from internal resources.
* Enforces **network security controls** including:
  * Inter-VLAN segmentation with ACLs
  * Management-plane protection (SSH-only, restricted management VLAN)
  * Layer-2 hardening (port-security, BPDU Guard, DHCP snooping, Dynamic ARP Inspection, blackhole VLAN for unused ports)
* Enables **demonstrable features** for the final presentation:
  * HSRP failover
  * VLAN routing and DHCP
  * Internet access through the ASA firewalls
  * ACL-based access restrictions (e.g., guest and production networks)
  * Selected hardening features (e.g., port-security, DHCP snooping)
### 1.3 Scope – Networking Focus
This document covers the **networking portion** of the Margielos capstone only:
* Physical and logical topology (routers, switches, firewalls, links)
* VLAN design and IP addressing
* Inter-VLAN routing and gateway redundancy
* NAT and Internet edge integration with ASA01/ASA02
* ACLs and security policy at Layer 3
* Switch security and Layer-2 protections
* Management access strategy (INFRA_MGMT and remote access)
The following are **intentionally kept brief** here and are documented in detail by other team members:
* Active Directory forest/domain design
* DNS/DHCP configuration and failover
* File, backup, and mail services
* Detailed server-side security (GPOs, file permissions, backup policies)
### 1.4 Assumptions & Constraints
* The solution is deployed in the **MITT NSA630 lab environment** with:
  * Limited physical hardware (two routers, two core switches, two ASAs, a small Proxmox cluster)
  * A shared **Capstone** uplink for Internet connectivity
* Only a **single physical site** is implemented; any branch or multi-site design is out of scope and treated as future work.
* Public IP addressing is not directly assigned on internal devices; Internet access is provided via the Capstone network and ASA NAT.
* Design choices favour:
  * **Clarity and demo-ability** for the capstone presentation
  * **Alignment with the NSA630 rubric** over full enterprise complexity

This documentation aims to be clear, technically accurate, and detailed enough that another network administrator could understand and maintain the Margielos environment without access to the original student team.

## 2. Network Topology & Design
### 2.1 Objective
Describe how the Margielos network is physically and logically structured using:
  * Two Cisco ISR 4321 routers (R1, R2)
  * Two Cisco 2650 switches (SW1, SW2) acting as a collapsed core
  * Two Cisco ASA 5506-X firewalls (ASA01, ASA02)
  * A small set of virtualized servers and wireless access points

The design aims to provide a practical, redundant “small production network” for a single-site e-commerce
company, while staying aligned with the NSA630 design and documentation rubric items.
### 2.2 Logical Topology Overview
At a high level, the network uses a collapsed core with Layer 2 switching and Layer 3 routing on the edge routers:
  * **Core switching – SW1 & SW2 (Cisco 2650)**
      * Operate as Layer 2 switches for all user VLANs.
      * Connected together using an **LACP EtherChannel (Port-Channel1)** built from Fa0/1– Fa0/2 on both switches.
      * The EtherChannel is an 802.1Q trunk carrying VLANs 10,20,30,40,50,60,70,80,85,90 with
**native VLAN 999 (BLACKHOLE).**
  * **Routers – R1 & R2 (Cisco 4321)**
       * Provide **inter-VLAN routing** and default gateways for all VLANs using **router-on-a-stick** subinterfaces.
       * Each user VLAN has:
          * HSRP virtual IP .1 as the default gateway
          * R1 IP .2 on Gi0/1.x
          * R2 IP .3 on Gi0/0/1.x
       * **R1 → SW1:**
          * R1 Gi0/1 is an 802.1Q trunk to SW1 Gi0/1, carrying all VLANs.
       * **R2 → SW2:**
          * R2 Gi0/0/1 is an 802.1Q trunk to SW2 Gi0/1, carrying the same VLANs.
  * **Firewalls – ASA01 & ASA02 (Cisco ASA 5506-X)**
      * **ASA01 (primary edge)**
          * Gi1/2 – **inside:** 192.168.254.2/30, connected to R1 Gi0/0 (192.168.254.1/30).
          * Gi1/1 – **outside:** DHCP from the MITT Capstone network (ip address dhcp setroute).
      * **ASA02 (redundant edge)**
          * Gi1/2 – **inside:** 192.168.254.6/30, connected to R2 Gi0/0/0 (192.168.254.5/30).
          * Gi1/1 – **outside:** DHCP from Capstone, similar to ASA01.
      
      * **Internal VLANs / Subnets (Layer 3 view)**
         * 10 – SALES_CS – 192.168.10.0/24
         * 20 – WAREHOUSE – 192.168.20.0/24
         * 30 – HR_MGMT – 192.168.30.0/24
         * 40 – IT – 192.168.40.0/24
         * 50 – MARKETING_ECOM – 192.168.50.0/24
         * 60 – PROD_FLOOR – 192.168.60.0/24
         * 70 – INFRA_MGMT – 192.168.70.0/24
         * 80 – SERVERS – 192.168.80.0/24
         * 85 – FINANCE – 192.168.85.0/24
         * 90 – GUEST – 192.168.90.0/24
         * 999 – BLACKHOLE/native – 192.168.199.0/24 (no end hosts)

Routing between all these VLANs is handled by R1/R2 subinterfaces with HSRP, as detailed in **Section 4 – Gateway Redundancy** and **Section 5 – Inter-VLAN Routing & Default Route.**

### 2.3 Logical Topology Diagram
#### Figure 2.1 – Logical Network Topology
<img width="2165" height="1020" alt="image" src="https://github.com/user-attachments/assets/3da2e281-04ab-431f-960b-156193c4220a" />


This figure should show:
   * R1/R2 connected to SW1/SW2 via 802.1Q trunks
   * SW1/SW2 connected by Port-Channel1
   * ASA01/ASA02 between the routers and the Capstone network
   * VLANs (10–90, 999) and their roles
   * Server VLAN (80) with DC01, DC02, and Proxmox host(s)

### 2.4 Physical Topology Overview

Physically, the network is built in a single rack / lab pod with:
  * **Routers**
      * R1 (ISR 4321) mounted near the top of the rack, cabled:
        * Gi0/1 → SW1 Gi0/1 (trunk)
        * Gi0/0 → ASA01 Gi1/2 (inside transit)
      * R2 (ISR 4321) cabled:
        * Gi0/0/1 → SW2 Gi0/1 (trunk)
        * Gi0/0/0 → ASA02 Gi1/2 (inside transit)

  * **Core Switches (Cisco 2650)**
       * SW1
           * Gi0/1 – trunk to R1
           * Fa0/1–2 – LACP member links to SW2 Fa0/1–2 (Port-Channel1)
           * Fa0/3 – access port in VLAN 80 for DC01 (AD/DNS/DHCP).
           * Fa0/4 – access port in VLAN 80 for a Proxmox host (backup server and other VMs).
           * Fa0/5–8 – access ports in VLAN 10 for Sales/CS PCs and AP1.
           * Fa0/9–10 – access ports in VLAN 30 for HR_MGMT PCs.
           * Fa0/11–12 – access ports in VLAN 40 for IT workstations.
           * Fa0/13–16 – VLAN 999 (BLACKHOLE) for unused ports, administratively shut.
       * SW2
           * Gi0/1 – trunk to R2 (description uplink to R2 G0/0/1).
           * Fa0/1–2 – LACP member links to SW1 Fa0/1–2 (Port-Channel1).
           * Fa0/3 – access port in VLAN 80 for DC02 (AD/DNS/DHCP).
           * Fa0/4 – access port in VLAN 90 for AP2 / Guest Wi-Fi.
           * Additional access ports allocated per VLAN for user PCs as needed (Warehouse,
             Prod_Floor, Finance, etc.).

 * **Firewalls (ASA 5506-X)**
      * ASA01
          
           * Gi1/2 (inside) → R1 Gi0/0 on 192.168.254.0/30
           * Gi1/1 (outside) → Capstone T-port (DHCP, default route learned upstream)
      
      * ASA02
           * Gi1/2 (inside) → R2 Gi0/0/0 on 192.168.254.4/30
           * Gi1/1 (outside) → second Capstone T-port
 
  * **Servers & Virtualization**

     * All servers live in **VLAN 80 (SERVERS, 192.168.80.0/24)** and are connected via switch access ports:

        * **DC01** (AD/DNS/DHCP) on SW1 Fa0/3
        * **DC02** (AD/DNS/DHCP) on SW2 Fa0/3
        * A **Proxmox host** on SW1 Fa0/4 providing VMs such as the backup server; the original on-premise mail server was replaced by **Microsoft 365** and no longer requires a dedicated VM.
### 2.5 Physical Topology Diagram
#### Figure 2.2 – Physical Network Topology
<img width="1782" height="1021" alt="image" src="https://github.com/user-attachments/assets/9420d8f8-ea48-4192-b422-a72512697896" />

This figure should show:
   * Physical locations of R1, R2, SW1, SW2, ASA01, ASA02
   * Lab computers / Proxmox host connections to VLAN 80
   * AP1 cabled to SW1 Fa0/5 (VLAN 10) and AP2 to SW2 Fa0/4 (VLAN 90)
   * Capstone T-ports connected to ASA01/ASA02 outside interfaces
### 2.6 Wireless Integration (Logical Placement)
Wireless access is intentionally simple and VLAN-based:
  
   * **AP1 – Staff Wi-Fi (on SW1)**
        * Connected to SW1 Fa0/5 as an **access port in VLAN 10 (SALES_CS).**
        * Provides Staff/Corp SSID mapped to VLAN 10, giving the same access as wired clients           in that VLAN (subject to routed ACLs).

  * **AP2 – Guest Wi-Fi (on SW2)**
    * Connected to SW2 Fa0/4 as **an access port in VLAN 90 (GUEST).**
    * Provides Guest SSID mapped to VLAN 90, with Internet-only access enforced by the              GUEST_IN ACL and firewall/NAT design (described in Section 7 and Section 6).

Both APs are standalone (no central WLC), which is appropriate for a 15-user single-site deployment and still satisfies the **WLAN** expectations of the rubric.

## 3. IP Addressing & Core Services
### 3.1 Objective
Define a clear IPv4 addressing scheme and summarize how core network services (DNS and DHCP) are
integrated into the Margielos network.
This section focuses on:
   * How VLANs and subnets are structured.
   * How default gateway and server IPs follow a consistent pattern.
   * How DHCP and DNS are provided centrally from the SERVERS VLAN using DHCP relay (ip      helper-address) on the routers.

Detailed AD/DNS/DHCP configuration is documented in the server team’s section; this document only covers the networking view.
### 3.2 Addressing Conventions
The IPv4 addressing scheme for Margielos uses one /24 subnet per VLAN with a consistent, easy--to-remember pattern:
    
  * **VLAN subnets:**
     * 10 – SALES_CS – 192.168.10.0/24
     * 20 – WAREHOUSE – 192.168.20.0/24
     * 30 – HR_MGMT – 192.168.30.0/24
     * 40 – IT – 192.168.40.0/24
     * 50 – MARKETING_ECOM – 192.168.50.0/24
     * 60 – PROD_FLOOR – 192.168.60.0/24
     * 70 – INFRA_MGMT – 192.168.70.0/24
     * 80 – SERVERS – 192.168.80.0/24
     * 85 – FINANCE – 192.168.85.0/24
     * 90 – GUEST – 192.168.90.0/24
     * 999 – BLACKHOLE/native – 192.168.199.0/24 (no end hosts)

   * **Default gateway pattern (per VLAN):**
     * .1 – HSRP virtual IP (gateway)
     * .2 – R1 subinterface IP
     * .3 – R2 subinterface IP
  
   * **Server addressing (VLAN 80 – SERVERS):**
     * .11–.19 range reserved for key infrastructure servers:
          * DC01: 192.168.80.11
          * DC02: 192.168.80.12
          * Proxmox host: 192.168.80.13
          * Backup server VM: 192.168.80.14

   * **DHCP pool ranges:**
     * For most VLANs, DHCP scope range: .100–.200
     * For Guest VLAN 90, DHCP scope range: .50–.200 (to allow more guest clients without            clashing with infrastructure patterns)

Static IPs (gateways, servers, network devices) are chosen outside of the DHCP ranges to avoid conflicts and keep the layout predictable.

### 3.3 Core Servers & Roles (Networking View)
All core servers are located in **VLAN 80 – SERVERS (192.168.80.0/24):**
|**Server**|**IP Address**|**Role (high level)**|
|----|----|----|
|**DC01**| 192.168.80.11| AD DS, DNS, DHCP|
|**DC02**| 192.168.80.12| AD DS, DNS, DHCP (failover partner to DC01)|
|**Proxmox** |192.168.80.13 |Virtualization host (e.g., backup server VM)|
|**Backup VM**| 192.168.80.14| Backup services (VM-level / file-level, as needed)|

Email services were originally planned to run on-prem, but in the final design **mail is hosted on Microsoft 365**, using the margielos.uk domain. This reduces complexity on the LAN side; no internal mail server IP is required.

### 3.4 DNS & AD Namespace
The Active Directory and internal DNS namespace is:
     
  * **AD / DNS domain:** ict.margielos.uk

Both DC01 and DC02 host:
   * AD DS for user/computer authentication.
   * Internal DNS zones for ict.margielos.uk (and any necessary forward/reverse zones).

From the network perspective:
   * All client VLANs use the HSRP gateway (.1) as their default gateway.
   * DNS servers handed out by DHCP (from DC01/DC02) are 192.168.80.11 and 192.168.80.12, so
internal name resolution always stays on the LAN and does not depend on external DNS for local
services.

### 2. Network Devices and Servers/Services
#### 2.1 Core Network Devices
|**Device**|**Model/Type**|**Role**	|**Interfaces**|**Notes**|
|---|---|---|---|---|
|R1 / R2|	Cisco ISR Router (2900 & 4300)|	Redundant edge routers|	Gig0/0 → ISP uplink, Gig0/1 → Core SW	|HSRP configured for default gateway redundancy|
|SW-A / SW-B	|Cisco Catalyst 2960|	Core switches|	24x FastEthernet, 2x Gig uplinks	|LACP trunk between SW-A and SW-B, VLAN trunking enabled|
|WLC2504	|Cisco Wireless LAN Controller	|Centralized AP management	|4x Gig ports	|Manages AP1 and AP2, guest VLAN isolated|
|AP1 / AP2	|tp-link AP|	Wireless access	|1x Gig uplink	|AP1 → Sales floor, AP2 → Guest/Production coverage|

#### 2.2 Server Infrastructure
|Server	|Hostname	|Role	|VLAN	|IP Address|
|---|---|---|---|---|
|DC01|	Domain Controller	|Active Directory, DNS, DHCP	|Servers VLAN (80)	|192.168.80.2|
|DC02	|Backup Domain Controller|	Redundancy, DNS secondary	|Servers VLAN (80)	|192.168.80.3|
|FileSrv01	|File Server	|Shared storage	|Servers VLAN (80)|	192.168.80.20|
|InvSrv01	|Inventory Server	|Warehouse DB	|Servers VLAN (80)|192.168.80.30| 

### 3 Addressing Documentation
|**Department**|**VLAN ID**|**Subnet**|**Gateway(HSRP VIP)**|**DHCP Range (clients)**|**Devices**|**Purpose**|
|----|----|----|----|---|----|---|
|SALES_CS|10 |192.168.10.0/24|192.168.10.1|.100-.200|PCs: 18-20, POS: 3 units|Sales & customer service,POS VLANS nested under Sales for cashier terminals|
|WAREHOUSE|20|192.168.20.0/24|192.168.20.1|.100-.200 |PCs: 29-30|Inventory & Warehouse|
|HR_MGMT|30|192.168.30.0/24|192.168.30.1|no DHCP |PCs: 24-25|HR &management(sensitive)|
|IT |40|192.168.40.0/24 |192.168.40.1 |.100-.200 |PCs: 26-27|IT workstations /admin |
|MARKETING_ECOM |50|192.168.50.0/24 |192.168.50.1|.100-.200 |PCs: 23-24|Marketing & ecommerce|
|PROD_FLOOR|60 |192.168.60.0/24 |192.168.60.1 |.100-.200 |PCs: 31-32| Production devices |
|INFRA_MGMT|70 |192.168.70.0/24 |192.168.70.1 |no DHCP | |Management SVIs, restricted access|
|SERVERS|80 |192.168.80.0/24 |192.168.80.1 |static | |App/DB/Infrastructure VMs|
|DMZ (Optional, will apply if we host it internally)|85|192.168.85.0/24| 192.168.85.1| static| |Public-facing web/reverse-proxy (If we host the webserver internally)|
|Finance |120|192.168.120.0/24|192.168.120.1|.100-.200|PC: 33| Restricted VLAN|
|GUEST|90 |19168.90.0/24 |192.168.90.1|.100-.200|Laptop, Smartphone| Internet-only guests|
|BLACKHOLE|999 |192.168.199.0/24 |192.168.199.1 |none | |Native/unused VLAN|

### 5. Organizational Structure, Users & Permissions – Day 1 Deliverable
#### 5.1. Organizational Unit (OU) Structure
|**Domain**|**OUs**|**Sub-OUs**|
|---|---|---|
|Margielos.local|Sales_and_CustomerService|SalesFloor, CustomerServiceDesk|
|  |Inventory_and_Warehouse|InventoryControl, WarehouseOperations|
|   |Management_and_HR|Management, HumanResources|
|  |IT_Department|ITSupport, ITAdministration|
|  |Marketing_and_Ecommerce |DigitalMarketing, EcommerceManagement|
|  |Production|DesignTeam, ProductionFloor|
#### 5.2 User List – Names, JOB TITLEs, Emails
##### Sales & Customer Service (3 Users)
|**NAME**|**JOB TITLE**|**EMAIL**|
|---|----|----|
|Barry Allen|Sales Representative|barry.allen@margielos.uk|
|Kara Danvers|Sales Associate|kara.danvers@margielos.uk|
|Cisco Ramon|Customer Service Agent|cisco.ramon@margielos.uk|
##### Inventory & Warehouse (2 Users)
|**NAME**|**JOB TITLE**|**EMAIL**|
|---|----|----|
|Winn Schott|Inventory Controller|winn.schott@margielos.uk|
|James Olsen|Warehouse Technician|james.olsen@margielos.uk|
##### Management & HR (2 Users)
|**NAME**|**JOB TITLE**|**EMAIL**|
|---|---|---|
|Oliver Queen|Operations Manager|oliver.queen@margielos.uk|
|Laurel Lance|HR Specialist|laurel.lance@margielos.uk|
##### IT Department (2 Users)
|**NAME**|**JOB TITLE**|**EMAIL**|
|----|---|---|
|Ray Palmer|IT Support Technician|ray.palmer@margielos.uk|
|Gideon AI|IT Administrator|it.admin@margielos.uk|
##### Marketing & E-commerce (2 Users)
|**NAME**|**JOB TITLE**|**EMAIL**|
|----|---|----|
|Felicity Smoak|Digital Marketing Coordinator|felicity.smoak@margielos.uk|
|Iris West|E-commerce Manager|iris.west@margielos.uk|
##### Production (2 Users)
|**NAME**|**JOB TITLE**|**EMAIL**|
|----|----|-----|
|Nia Nal|Product Designer|nia.nal@margielos.uk|
|Jefferson Pierce|Production Worker|jefferson.pierce@margielos.uk|
#### 5.3 UserNAME & EMAIL Naming Convention
|**Rule**|**Format**|**Example**|
|----|----|----|
|Standard Users|firstNAME.lastNAME|john.doe@margielos.uk|
|IT Admin|Special account|it.admin@margielos.uk|
#### 5.4 Required Security Groups
|**Security Group**|**Purpose**|
|----|----|
|SalesUsers|Access to Sales shared data & sales applications|
|InventoryUsers|Access to inventory systems & databases|
|HRConfidential|Restricted HR files & private employee data|
|ProductionUsers|Access to production-related designs & documents|
|ITAdmins|Full access for administration and IT operations|
|MarketingUsers|Access to marketing assets & digital materials|
#### 5.5 File Shares & Permission Structure
|**File Share NAME**|**Group with Access**  |**Purpose**|
|----|----|----|
|HRConfidential|HRConfidential|Secure HR records and documents|
|SalesData|SalesUsers|Sales reports, customer info, product data|
|InventoryDB|InventoryUsers|Inventory database and stock files|
|ProductionDesigns|ProductionUsers|Design files and production workflows|
|MarketingAssets|MarketingUsers|Graphics, campaigns, brandiing files|
### 6. Server & Services Lead
 
#### 6.1 Full Server List + Purpose
* **DC1 – Primary Domain Controller**  
   Handles Active Directory Domain Services (AD DS) and DNS. Central authority for authentication and directory lookups.
* **DC2 – Secondary Domain Controller**  
    Provides redundancy for AD DS and DNS. Ensures high availability if DC1 fails.
* **DNS Server**  
    Resolves internal hostnames to IP addresses. Critical for domain operations and service discovery.
* **DHCP Server**  
    Assigns IP addresses dynamically to client devices. Simplifies network management.
* **File Server**  
    Stores departmental files and shared resources. Backbone for collaboration.
* **Web Server (IIS)**  
    Hosts TrendyThreads’ demo e‑commerce site. Used by Marketing & E‑commerce.
* **Mail Server (Exchange)**  
    Provides internal email services for testing and communication.
* **Backup Server**  
    Runs backup tools to snapshot and protect all major servers. Ensures disaster recovery.
 
#### 6.2 Roles & Features Installed
*  **DC1/DC2** → Active Directory Domain Services (AD DS), DNS role  
* **DHCP Server** → DHCP role  
* **File Server** → File Services role  
* **Web Server** → IIS role  
* **Mail Server** → Microsoft Exchange Server role  
* **Backup Server** → Backup software (e.g., Windows Server Backup, Veeam)

#### 6.3 File Share Layout Planning
 
|**Share Name**         |**Used By**           | **Purpose**                          |
|---------------------|------------------|----------------------------------|
| HR_Confidential     | HR Department    | Store sensitive HR documents     |
| Sales_Data          | Sales Department | Store customer and sales records |
| Inventory_DB        | Inventory Team   | Store stock and warehouse data   |
| Production_Designs  | Production Team  | Store design files and blueprints|
| Marketing_Assets    | Marketing Team   | Store images, videos, and ads    |
| Shared              | All Departments  | General collaboration folder     |

#### 6.4 Basic Service Dependency Map
* **DHCP → DNS** (DHCP leases must register with DNS for name resolution)  
* **File Server → AD Groups** (permissions tied to security groups defined by Identity Lead)  
* **Web Server → Domain + Network** (requires DNS resolution and AD authentication)
* **Backup Server → All Servers** (backs up DCs, File Server, Web Server, Mail Server)

#### 6.5 Department Service Requirements Overview
 
| **Department**            | **Services Used**                          |
|------------------------|----------------------------------------|
| Sales                  | Sales_Data share, Email                |
| HR                     | HR_Confidential share, Email           |
| Inventory & Warehouse  | Inventory_DB share                     |
| Production             | Production_Designs share               |
| Marketing & E‑commerce | Marketing_Assets share, Web Server     |
| IT                     | All services (admin, support, backups) |
| Management             | Shared folder, Email                   |
 
#### 6.6 Additional Details
##### 1. Web Server Choice
We’re using **IIS (Internet Information Services)** on Windows Server.  
* Integrates seamlessly with Active Directory.
* Supports ASP.NET and static content.
* Easy to manage via GUI and PowerShell.
##### 2. File Server Choice
We’re using **Windows Server with File Services role**.  
* Native NTFS + AD group integration.  
* Supports SMB protocol for secure file sharing.  
*Simple departmental share configuration.
 
##### 3. Exchange Server Location
Installed on a **dedicated Windows Server**, separate from DC1/DC2.  
* Avoids resource competition with domain controllers.
* Improves performance and troubleshooting.
 
##### 4. DNS & DHCP Placement
* **DNS** → Installed on **both DC1 and DC2** for redundancy.  
* **DHCP** → Installed on **DC1 only**.  
* Ensures high availability for DNS, while DHCP remains simple to manage.
  
### 7. **Backup Server (BKUP01) Integration**

  </div>
  
#### **7.1 High-Level Overview**
This document details the configuration for integrating the new backup server, **BKUP01**, into the existing **SERVERS (VLAN 80)** network segment, which is hosted on a Proxmox VE hypervisor cluster.

|Component|Description|
|------|----|
|**Project**|New Backup Server Deployment|
|**VM Name**|BKUP01|
|**Hypervisor**|Proxmox VE|
|**VLAN**|80 (SERVERS)|
|**Domain**|ict.margielos.uk|
|**Purpose**|Dedicated Backup Server|


#### 7.2 IP Addressing and VLAN Details
The SERVERS network uses the $192.168.80.0/24$ subnet. The Proxmox host's bridge (vmbr0) is configured to handle traffic for VLAN 80. The new VM, BKUP01, is assigned a static IP address within this subnet.
#### VLAN and Subnet
* **VLAN ID:** 80
* **VLAN Name:** SERVERS
* **Subnet:** 192.168.80.0/24
* **Gateway:** 192.168.80.1

#### 7.3 Virtualization Details
The VM is configured to ensure optimal network and disk performance by leveraging Proxmox's paravirtualized drivers.

|Setting|Value|Rationale|
|-----|-----|-----|
|Proxmox Bridge|vmbr0|Host interface connected to the physical switch.|
|VM Network Model|VirtIO (paravirtualized)|Required for high-speed network I/O.|
|VLAN Tag|80|Tags traffic with the correct VLAN ID on vmbr0.|
|SCSI Controller|VirtIO SCSI|Required for high-speed disk I/O.|

#### 7.4 BKUP01 Server Properties Summary
This table outlines the physical/virtual specifications, network configuration, and operating system details for the BKUP01 backup server.

|Property|Value|Note|
|---|----|----|
|VM Name|BKUP01|New Backup Server|
|Hypervisor|Proxmox VE (PVE)|Host System|
|Operating System|Windows Server 2022|Evaluation or Full Version|
|CPU (Cores/Sockets)|4 Cores / 1 SocketConfigured in Proxmox VM settings.|
|Memory (RAM)|8 GiB (8192 MiB)|A solid baseline for Windows Server 2022.|
|Boot Disk Size|100 GB|Initial disk size for OS installation.|
|SCSI Controller|VirtIO SCSI|Crucial for optimized disk performance.|
|Network Model|VirtIO (Paravirtualized)|Crucial for optimized network performance.|
|Qemu Agent|Enabled|Allows host-guest communication for monitoring/shutdowns.|
|BIOS Type|OVMF (UEFI) or SeaBIOS\Recommended: OVMF (Requires EFI Disk).|

#### 7.5 Post-Installation Configuration (BKUP01)
Once Windows Server 2022 is installed on the BKUP01 VM and you have logged in, these are the critical steps to integrate it properly into your ict.margielos.uk domain and the SERVERS network.
* **(i) Install VirtIO and Qemu Guest Agent**
Before proceeding with networking, ensure the VM has the optimized drivers and communication agent installed.
  * **Open File Explorer** and navigate to the mounted VirtIO ISO drive (e.g., D: or E:).
  * **Install the **VirtIO drivers** using the installer provided on the ISO.
  * Install the **Qemu Guest Agent.** This service allows Proxmox to accurately monitor the VM's resource usage, send clean shutdown commands, and report the correct IP address in the Proxmox UI.
* **(ii) Configure Static IP Addressing**
The VM must be configured with a static IP address to ensure reliable connectivity and DNS resolution.
  * **Open Network Connections** (e.g., via ncpa.cpl).
  * Right-click the network adapter and select Properties.
  * Select Internet Protocol Version 4 (TCP/IPv4) and click Properties.
  * Select Use the following IP address and enter the details:

|Field|Value|
|----|----|
|**IP Address**|192.168.80.14|
|**Subnet Mask**|255.255.255.0|
|**Default Gateway**|192.168.80.1|
|**Preferred DNS Server**|192.168.80.11 (DC01)|
|**Alternate DNS Server**|192.168.80.12 (DC02)|
  * **(iii) Change Server Name and Join Domain**
This step registers the server with your domain and sets the correct hostname.
 * **Open System Properties** (e.g., via sysdm.cpl).
 * In the **Computer Name** tab, click **Change...**
 * **Computer name:** Enter  **BKUP01**.
 * **Member of:** Select **Domain** and enter the domain name: **ict.margielos.uk.**.
 * Click **OK.** You will be prompted for credentials (use a domain admin account, like ict\administrator).
 * You will receive a welcome message, and the server will prompt you to **Restart** the VM.
 * After the restart, you should be able to log in using domain credentials (e.g., ICT\YourAdminUsername).
* **(iv) Basic System Configuration**
Perform these administrative tasks for a complete setup.
  * **Windows Updates:** Install all necessary patches and updates.
  * **Time Zone:** Set the correct time zone for the server.
  * **Firewall:** Configure the Windows Firewall to allow necessary traffic (e.g., backup application ports, ICMP for monitoring, RDP).
  * **Remote Desktop (RDP):** Ensure RDP is enabled so you can manage the server remotely without using the Proxmox console.
* **(v) Storage Preparation (If Applicable)** If you have allocated additional virtual disks for backup storage (separate from the boot disk):
    * **Open Disk Management** (e.g., via diskmgmt.msc).
    * The new disk(s) should appear as Unallocated.
    * **Initialize** the disk (GPT is recommended for modern systems).
    * **Create a New Simple Volume** on the disk, format it with NTFS or ReFS (ReFS is often preferred for large backup volumes), and assign a drive letter (e.g., B: for Backups).

 ### 8. Management & Security
* **INFRA_MGMT (VLAN 70)**
  * Devices: PC01 (admin workstation), router/firewall interfaces, switch SVIs
  * Purpose: Restricted access for IT staff only
* **Security Features**
  * ACLs applied to POS VLANs (only allowed to reach Inventory Server + Internet)
  * HSRP for router redundancy
  * LACP trunk between core switches for loop prevention
  * Guest VLAN isolated via WLC with Internet-only access
#### 8.1 Secure Credential Storage
The method used to securely store all administrative login credentials (Firewall, Switch, APs, Server Admin accounts, ERP passwords) is:
* **Method:** A centralized, business-grade, encrypted *Password Manager* (e.g., Bitwarden Teams or 1Password Business) is implemented.
* **Access:** Access to the Password Manager requires a unique master password for each administrator user, combined with MFA (Multi-Factor Authentication).
* **Repository:** The actual passwords are not stored in the GitHub documentation repository. Only this methodology and the necessary unprivileged user account names are documented.
* **Principle:** Credentials are encrypted in transit and at rest using strong standards like AES-256 and are only decrypted locally on authorized administrator workstations.
#### 8.2 Management
- Full Control Panel access
- No operational restrictions
- Access to **all** shared drives:
  - I:, W:, T:, M:, E:, P:, F:, S:, C:, H:

#### 8.3 Human Resources
- Removable storage blocked
- Auto-lock after **8 minutes**
- Drive: **H:** `\\DC01\HR`
- Protected environment for confidential data

### 9. Shared Drive Architecture

#### 9.1 Folder Structure
Located on the file server (DC01):


#### 9.2 Drive Mapping Table

| Department            | Drive Letter | Path                              |
|----------------------|-------------|-----------------------------------|
| Inventory            | I:          | \\DC01\Inventory                  |
| Warehouse            | W:          | \\DC01\Warehouse                  |
| IT                   | T:          | \\DC01\IT                         |
| Marketing            | M:          | \\DC01\Marketing                  |
| E-Commerce           | E:          | \\DC01\Ecommerce                  |
| Production Design    | P:          | \\DC01\Production_Design          |
| Production Floor     | F:          | \\DC01\Production_Floor           |
| Sales                | S:          | \\DC01\Sales                      |
| Customer Service     | C:          | \\DC01\CustomerService            |
| HR                   | H:          | \\DC01\HR                         |
| Management           | ALL         | All shares                        |

#### 9.3 Permissions Outcome
- **Department Groups:** Modify permissions to their respective shares
- **Management:** Full Control to all department shares
- **IT Admins:** Full Control everywhere
- **Everyone:** Removed from share access

---

#### 9.4 Additional Security Measures
These solutions enhance protection beyond GPO policies:

* **BitLocker**
  Full-disk encryption on all laptops and workstations.
* **LAPS** (Local Administrator Password Solution)
  Unique rotating local admin passwords on every workstation.
* **AppLocker**
  Restricts executable, script, and installer execution to approved items only.
* **Shadow Copies**
  Enabled on all department shares for quick file recovery.

#### 9.5 Final Outcome
The completed GPO deployment provides:

- A **secure and segmented domain** across margielos.uk and ict.margielos.uk  
- Strict departmental separation with mapped drives and permissions  
- Consistent workstation security across all 2–3 workstations per department  
- Hardened systems protected from misuse or unauthorized changes  
- Centralized file access with reliable permission boundaries  
- A professional, enterprise-grade configuration suitable for production  

This infrastructure supports operational efficiency, data integrity, and strong security posture across all departments at Margielos.




