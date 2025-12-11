<div align="center"
  
# **Network Documentation: BKUP01 Integration**

  </div>
  
#### **1. High-Level Overview**
This document details the configuration for integrating the new backup server, **BKUP01**, into the existing **SERVERS (VLAN 80)** network segment, which is hosted on a Proxmox VE hypervisor cluster.

|Component|Description|
|------|----|
|**Project**|New Backup Server Deployment|
|**VM Name**|BKUP01|
|**Hypervisor**|Proxmox VE|
|**VLAN**|80 (SERVERS)|
|**Domain**|ict.margielos.uk|
|**Purpose**|Dedicated Backup Server|

#### 2. IP Addressing and VLAN Details
The SERVERS network uses the $192.168.80.0/24$ subnet. The Proxmox host's bridge (vmbr0) is configured to handle traffic for VLAN 80. The new VM, BKUP01, is assigned a static IP address within this subnet.💡 VLAN and SubnetVLAN ID: 80VLAN Name: SERVERSSubnet: $192.168.80.0/24$Gateway: $192.168.80.
