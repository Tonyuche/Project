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

#### 5. Post-Installation Configuration (BKUP01)
Once Windows Server 2022 is installed on the BKUP01 VM and you have logged in, these are the critical steps to integrate it properly into your ict.margielos.uk domain and the SERVERS network.
* **Install VirtIO and Qemu Guest Agent**
Before proceeding with networking, ensure the VM has the optimized drivers and communication agent installed.
  * **Open File Explorer** and navigate to the mounted VirtIO ISO drive (e.g., D: or E:).
  * **Install the **VirtIO drivers** using the installer provided on the ISO.
  * Install the **Qemu Guest Agent.** This service allows Proxmox to accurately monitor the VM's resource usage, send clean shutdown commands, and report the correct IP address in the Proxmox UI.
* **Configure Static IP Addressing**
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
  * **Change Server Name and Join Domain**
This step registers the server with your domain and sets the correct hostname.Open System Properties (e.g., via sysdm.cpl).In the Computer Name tab, click Change...Computer name: Enter BKUP01.Member of: Select Domain and enter the domain name: ict.margielos.uk.Click OK. You will be prompted for credentials (use a domain admin account, like ict\administrator).You will receive a welcome message, and the server will prompt you to Restart the VM.After the restart, you should be able to log in using domain credentials (e.g., ICT\YourAdminUsername).4. Basic System ConfigurationPerform these administrative tasks for a complete setup.Windows Updates: Install all necessary patches and updates.Time Zone: Set the correct time zone for the server.Firewall: Configure the Windows Firewall to allow necessary traffic (e.g., backup application ports, ICMP for monitoring, RDP).Remote Desktop (RDP): Ensure RDP is enabled so you can manage the server remotely without using the Proxmox console.5. Storage Preparation (If Applicable)If you have allocated additional virtual disks for backup storage (separate from the boot disk):Open Disk Management (e.g., via diskmgmt.msc).The new disk(s) should appear as Unallocated.Initialize the disk (GPT is recommended for modern systems).Create a New Simple Volume on the disk, format it with NTFS or ReFS (ReFS is often preferred for large backup volumes), and assign a drive letter (e.g., B: for Backups).
