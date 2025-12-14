<div align="center"> 
  
# Network Documentation
## The Margielos Project
245 Smith Avenue, Winnipeg,   
HR - 2046678234     
info@margielos.uk
  

</div>


**Margielos Clothings** is a new Cloth manufacturing and sales company seeking a whole IT Infrastructure deployment for its operations.  
 Margielos Clothings currently has a total of 15 employees who carry out their daily work activities using their PCs.

<div align="center">
  
## _By_ 
## *Azureusers (Group 5)*
 ## Network Documentation
</div>

### 1. Network Topology

#### 1.1 Physical Topology
<img width="1782" height="1021" alt="image" src="https://github.com/user-attachments/assets/9420d8f8-ea48-4192-b422-a72512697896" />


* **Focus:** A centralized, rack-mounted approach for core equipment with structured cabling to endpoints.
* **Core Equipment Location:** A dedicated, locked server closet or data cabinet is recommended for the Firewall/Router, Managed Switch, and Server.
* **Wiring:** All permanent devices (PCs, Access Points) must be connected using **Cat6 Ethernet cable** running through a Patch Panel in the closet, connecting to the Managed Switch.
* **Endpoints:**
  *  PCs devices are wired for maximum reliability and speed.
  *  Wireless Access Points (APs) are ceiling-mounted and PoE-powered from the switch to provide office-wide $\text{Wi-Fi}$ coverage.

#### 1.2 Logical Topology
<img width="2165" height="1020" alt="image" src="https://github.com/user-attachments/assets/3da2e281-04ab-431f-960b-156193c4220a" />

#### 1.3 Network Topology Summary ###
 * Two routers (R1, R2) each connected redundantly to two core switches (SW1, SW2).
 * SW1 <-> SW2: 2x parallel links aggregated with LACP (Port-Channel) to avoid STP flapping.
 * R1 and R2 present router-on-a-stick subinterfaces (802.1Q trunk) for VLANs.
 * HSRP for default-gateway high-availability across VLANs.
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

This infrastructure supports operational efficiency, data integrity, and strong security posture across all departments at Margielos UK.
# Contributing Guidelines

Thanks for your interest in improving Margielos Atelier. Because this project was created for a capstone project and designed to be simple, contributions are welcome but please follow the guidelines below.

## Setting up your environment

1. Install Node.js 18+ and npm.
2. Install dependencies with `npm install`.
3. Run the development server with `npm run dev` and visit `http://localhost:3000`.

## Coding Standards

- Use TypeScript for all `.tsx` files.
- Use functional components and React hooks.
- Tailwind CSS is the styling framework; avoid inline styles.
- Keep components small and focused. Reusable UI pieces should live in `app/components/`.

## Branching & Pull Requests

- Create a descriptive feature branch (e.g. `feature/add-wishlist`).
- Commit messages should be concise and present‑tense.
- Before opening a Pull Request, ensure `npm run build` and `npm run lint` pass without errors.
- Provide context and screenshots in the PR description.

## Adding Products

Product data is stored statically in `data/products.ts`. To add a product:

1. Define a new `Product` object in the `products` array. Make sure the `id` is unique.
2. Provide `slug`, `name`, `description`, `price`, `category`, and image URLs.
3. If you add new categories, update the navigation links in `app/page.tsx`.

## Future Ideas

While the current version uses localStorage for the cart and static data for products, future improvements could include:

- Connecting a real back‑end (e.g. via Prisma or MongoDB).
- Implementing user authentication for wishlists or order history.
- Adding payment processing with Stripe or PayPal.
- Internationalization and accessibility improvements.

Contributions addressing any of these ideas are welcome. Please open an issue first to discuss large changes.



