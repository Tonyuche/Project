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
The SERVERS network uses the $192.168.80.0/24$ subnet. The Proxmox host's bridge (vmbr0) is configured to handle traffic for VLAN 80. The new VM, BKUP01, is assigned a static IP address within this subnet.
#### VLAN and Subnet
* **VLAN ID:** 80
* **VLAN Name:** SERVERS
* **Subnet:** 192.168.80.0/24
* **Gateway:** 192.168.80.1

#### 3. Virtualization Details
The VM is configured to ensure optimal network and disk performance by leveraging Proxmox's paravirtualized drivers.

|Setting|Value|Rationale|
|-----|-----|-----|
|Proxmox Bridge|vmbr0|Host interface connected to the physical switch.|
|VM Network Model|VirtIO (paravirtualized)|Required for high-speed network I/O.|
|VLAN Tag|80|Tags traffic with the correct VLAN ID on vmbr0.|
|SCSI Controller|VirtIO SCSI|Required for high-speed disk I/O.|

#### 4. BKUP01 Server Properties Summary
This table outlines the physical/virtual specifications, network configuration, and operating system details for the BKUP01 backup server.

|Property|Value|Note
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
