<div align="center"> 
  
# Network Documentation
## Trendy Threads
245 Smith Avenue, Winnipeg,   
HR - 2046678234     
info@trendythreads.ca
  

</div>


**Trendy Designs** is a new Cloth manufacturing and sales company seeking a whole IT Infrastructure deployment for its operations.
Trendy Designs currently has a total of 25 employees who carry out their daily work activities using their PCs and POS machines.

<div align="center">
  
## Network Documentation By *AzureCloud Solutions Ltd.*
</div>

### 1. Network Topology

#### 1.1 Physical Topology
<img width="1916" height="802" alt="image" src="https://github.com/user-attachments/assets/9bdc3fba-a208-4318-83d2-e4a07e70a425" />
* **Focus:** A centralized, rack-mounted approach for core equipment with structured cabling to endpoints.
* **Core Equipment Location:** A dedicated, locked server closet or data cabinet is recommended for the Firewall/Router, Managed Switch, and Server.
* **Wiring:** All permanent devices (PCs, POS terminals, Printers, Access Points) must be connected using **Cat6 Ethernet cable** running through a Patch Panel in the closet, connecting to the Managed Switch.
* **Endpoints:**
  *  PCs and POS devices are wired for maximum reliability and speed.
  *  Wireless Access Points (APs) are ceiling-mounted and PoE-powered from the switch to provide office-wide $\text{Wi-Fi}$ coverage.

#### 1.2 Logical Topology
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/31c1eaa8-8701-4408-bc89-44aaae9f31b8" />

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
|WLC2504	|Cisco Wireless LAN Controller	|Centralized AP management	|4x Gig ports	|Manages AP0 and AP1, guest VLAN isolated|
|AP0 / AP1	|Cisco Lightweight AP|	Wireless access	|1x Gig uplink	|AP0 → Sales floor, AP1 → Guest/Production coverage|

#### 2.2 Server Infrastructure
|Server	|Hostname	|Role	|VLAN	|IP Address|
|---|---|---|---|---|
|DC01|	Domain Controller	|Active Directory, DNS, DHCP	|Servers VLAN (30)	|192.168.80.2|
|DC02	|Backup Domain Controller|	Redundancy, DNS secondary	|Servers VLAN (30)	|192.168.80.3|
|FileSrv01	|File Server	|Shared storage	|Servers VLAN (30)|	192.168.30.20|
|InvSrv01	|Inventory Server	|POS + Warehouse DB	|Servers VLAN (30)|192.168.30.30|

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
|GUEST|90 |192.168.90.0/24 |192.168.90.1|.100-.200|Laptop, Smartphone| Internet-only guests|
|BLACKHOLE|999 |192.168.199.0/24 |192.168.199.1 |none | |Native/unused VLAN|

### 4. Devices Configurations (Summary)
The full configuration files will be stored securely, but key configurations are summarized here:
* **Firewall/Router Configuration**
  * **Default Deny:** The firewall is configured to block all incoming traffic (WAN to LAN) by default.
  * **Inter-VLAN Routing:** Rules are applied to block VLAN 20 (POS) and VLAN 100(Guest) from accessing VLAN 30 (Servers) entirely.
     **Security:** Web filtering and IPS are enabled.
* **Core Switch Configuration**
  * **VLAN Tagging:** All uplink ports (to Firewall/Router and APs) are configured as Trunk Ports carrying VLANs 10, 20, 30, 99, 100.
  * **Access Ports:** Ports are statically assigned to the required VLAN:
    * PCs ports: Access VLAN 10
    * POS ports: Access VLAN 10
    * PoE: Enabled on ports connecting to APs and potentially VoIP phones.
**Wireless Access Point Configuration**
  * **SSID Mapping:**
      * **Trendy-Data** ->VLAN 10 (WPA2-Enterprise authentication via DC-Server)
      * **Trendy-WLAN** ->VLAN 99 (WPA2-PSK for employee mobile)
      * **Trendy-Guest** ->VLAN 90 (Captive Portal enabled)
### 5. Management & Security
* **INFRA_MGMT (VLAN 70)**
  * Devices: PC01 (admin workstation), router/firewall interfaces, switch SVIs
  * Purpose: Restricted access for IT staff only
* **Security Features**
  * ACLs applied to POS VLANs (only allowed to reach Inventory Server + Internet)
  * HSRP for router redundancy
  * LACP trunk between core switches for loop prevention
  * Guest VLAN isolated via WLC with Internet-only access
#### 5.1 Secure Credential Storage
The method used to securely store all administrative login credentials (Firewall, Switch, APs, Server Admin accounts, ERP passwords) is:
* **Method:** A centralized, business-grade, encrypted *Password Manager* (e.g., Bitwarden Teams or 1Password Business) is implemented.
* **Access:** Access to the Password Manager requires a unique master password for each administrator user, combined with MFA (Multi-Factor Authentication).
* **Repository:** The actual passwords are not stored in the GitHub documentation repository. Only this methodology and the necessary unprivileged user account names are documented.
* **Principle:** Credentials are encrypted in transit and at rest using strong standards like AES-256 and are only decrypted locally on authorized administrator workstations.

**IP convention for DCHP/DNS**
* .1 = HSRP/VRRP virtual gateway
* .2 = R1 subinterface
* .3 = R2 subinterface
* .10 - .19 = Reserved for servers/infrastructure reservations
* .100–.200 → DHCP pool for clients




