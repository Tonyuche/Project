<div align="center"> 
  
# Network Documentation
## Trendy Designs
245 Smith Avenue, Winnipeg,   
HR - 2046678234     
info@trendydesigns.ca
  

</div>


**Trendy Designs** is a new Cloth manufacturing and sales company seeking a whole IT Infrastructure deployment for its operations.
Trendy Designs currently has a total of 25 employees who carry out their daily work activities using their PCs and POS machines.

<div align="center">
  
### Network Documentation By *AzureCloud Solutions Ltd.*
</div>

### Physical Topology
<img width="1916" height="802" alt="image" src="https://github.com/user-attachments/assets/9bdc3fba-a208-4318-83d2-e4a07e70a425" />

###  Logical Topology
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/31c1eaa8-8701-4408-bc89-44aaae9f31b8" />

<img width="650" height="950" alt="Logical Draw" src="https://github.com/user-attachments/assets/40cbb3cc-1828-4b8b-a225-fb25d2b00825" />
|**VLAN ID**|**Name**|**Subnet / Mask**|**DHCP Range (Example)**|**Purpose**|
|------|------|-----|------|-------|
|10|USERS|192.168.10.0/24|192.168.10.100 - .200|"Employee PCs, Laptops, Printers"
20,POS,192.168.20.0/24,Static IPs / Short DHCP,"POS Machines, Payment Devices (Highly Secured)"
30,SERVERS,192.168.30.0/24,Static IPs,"Servers (File, Inventory, Domain Controller)"
99,MANAGEMENT,192.168.99.0/24,Static IPs,"Network Devices (Switches, APs, Router/Firewall)"
100,GUEST_WIFI,192.168.100.0/24,192.168.100.10 - .50,Isolated Guest Internet Access





