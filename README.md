# Active Directory & File Services Homelab Documentation

The objective of this project was to deploy a fully functional Active Directory Domain Services (AD DS) environment, join a Windows 11 client machine to the domain, configure structured Organizational Units (OUs) and Role-Based Access Control (RBAC) groups, and establish a centralized File Server utilizing both Share and NTFS permission hierarchies.

## Project Overview

* Built a complete Windows Server enterprise lab using VMware Workstation Pro.
* Deployed Active Directory Domain Services (AD DS).
* Configured DNS and static IP addressing.
* Joined a Windows 11 client to the domain.
* Created an enterprise Organizational Unit (OU) structure.
* Implemented Role-Based Access Control (RBAC) using Active Directory Security Groups.
* Configured a centralized File Server with Share and NTFS permissions.
* Verified authentication, access control, and file permissions from a domain-joined client.

---

# Phase 1 – Virtualization Platform Selection

### Hypervisors Evaluated

* Evaluated **VMware Workstation Pro**

  * Selected as the primary hypervisor.
  * Used snapshots for recovery.
  * Installed VMware Tools for improved guest integration.
  * Enterprise-like environment similar to VMware ESXi.

* Evaluated **Oracle VM VirtualBox**

  * Considered as an open-source alternative.
  * Reviewed GUI and deployment process.

* Reviewed

  * Microsoft Hyper-V
  * Proxmox VE
  * Compared enterprise virtualization options.

---

# Phase 2 – Windows Server Installation

### Server Deployment

* Downloaded Windows Server 2025 Evaluation ISO.
* Created a new virtual machine.
* Configured:

  * 40 GB Virtual Disk
  * 2 CPU Cores
  * 4 GB RAM

### Operating System Installation

* Installed:

  * Windows Server 2025 Standard (Desktop Experience)

### VMware Tools

* Installed VMware Tools.
* Enabled:

  * Full-screen support
  * Automatic display resizing
  * Improved VM performance

### Hostname Configuration

* Renamed the server from the default hostname to:

  **FILESERVER01**

* Verified hostname using:

```cmd
hostname
```

---

# Phase 3 – Network Configuration

### Static IP Configuration

* Configured a static IPv4 address.
* Assigned:

  * Static IP Address
  * Subnet Mask
  * Default Gateway
* Configured Preferred DNS Server:

  * **127.0.0.1**
* Verified configuration using:

```cmd
ipconfig /all
```

---

# Phase 4 – Active Directory Deployment

### Installed Active Directory Domain Services

* Opened Server Manager.
* Added:

  * Active Directory Domain Services (AD DS)
* Installed required management tools.

### Domain Controller Promotion

* Promoted the server to a Domain Controller.
* Created a new forest.

### Root Domain

* **eastcharmer.local**

### Configuration

* Created DSRM password.
* Completed prerequisite checks.
* Installed AD DS.
* Rebooted the server.

---

# Phase 5 – Active Directory Organization

## Organizational Unit Structure

```
eastcharmer.local

└── USA
    ├── Users
    │
    ├── IT
    ├── HR
    ├── Finance
    │
    └── Computers
```

### Organizational Units Created

* USA
* Users
* Computers
* IT
* HR
* Finance

---

## User Accounts

### Administrative Account

* Created an IT Administrator account.
* Added account to:

  * Domain Admins

### Standard Users

Created users for:

* HR
* IT
* Finance

Configured:

* Password Never Expires (Lab Environment)

---

## Security Groups

Created Global Security Groups:

* IT
* HR
* Finance

Added department users to their respective security groups.

---

# Phase 6 – Windows 11 Client Deployment

### Virtual Machine Configuration

Created Windows 11 Professional VM.

Configured:

* 70 GB Virtual Disk
* 4 GB RAM
* TPM Enabled

Installed:

* VMware Tools

---

## Network Configuration

Configured Preferred DNS Server to the IP address of:

**FILESERVER01**

Verified connectivity:

```cmd
ping FILESERVER01
```

---

## Domain Join

* Renamed workstation to:

  * CLIENT01
* Joined:

  * eastcharmer.local
* Authenticated using Domain Admin credentials.
* Restarted the workstation.
* Logged in using domain user credentials.

---

## Computer Object Management

Moved CLIENT01 into:

```
USA
└── Computers
```

Added an asset description for administrative tracking.

---

# Phase 7 – File Server Deployment

## Folder Structure

```
C:\CompanyData

├── HR
├── IT
├── Finance
└── Public
```

Shared Folder:

```
\\FILESERVER01\CompanyData
```

---

## Share Permissions

Configured:

* Everyone

  * Full Control

Purpose:

* Allow NTFS permissions to control access.

---

# Phase 8 – NTFS Permission Configuration

For each department folder:

### HR Folder

* Disabled inheritance.
* Converted inherited permissions into explicit permissions.
* Removed unnecessary default groups.
* Granted:

  * HR Security Group

    * Modify

---

### IT Folder

* Disabled inheritance.
* Removed unnecessary permissions.
* Granted:

  * IT Security Group

    * Modify

---

### Finance Folder

* Disabled inheritance.
* Removed unnecessary permissions.
* Granted:

  * Finance Security Group

    * Modify

---

### Public Folder

Configured:

* Domain Users

  * Read

* IT Security Group

  * Modify

---

# Permission Summary

| Folder  | Share Permission        | NTFS Permission                  | Effective Access                             |
| ------- | ----------------------- | -------------------------------- | -------------------------------------------- |
| HR      | Everyone – Full Control | HR – Modify                      | HR users can read/write. Others denied.      |
| IT      | Everyone – Full Control | IT – Modify                      | IT users can read/write. Others denied.      |
| Finance | Everyone – Full Control | Finance – Modify                 | Finance users can read/write. Others denied. |
| Public  | Everyone – Full Control | Domain Users – Read, IT – Modify | Everyone can read. IT can manage files.      |

---

# Phase 9 – Validation & Testing

### HR User Testing

Logged into:

* CLIENT01

Using:

* HR domain account

Verified:

* Connected to:

```
\\FILESERVER01\CompanyData
```

Successfully:

* Opened HR folder.
* Created files.
* Modified files.
* Saved documents.

Confirmed:

* Access denied to:

  * IT
  * Finance

Verified:

* Public folder provided read-only access.
* Unable to create files in the Public directory as a standard user.

---

# Technologies Used

* VMware Workstation Pro
* Windows Server 2025
* Windows 11 Pro
* Active Directory Domain Services (AD DS)
* DNS
* Organizational Units (OUs)
* Active Directory Users and Computers (ADUC)
* Domain Controller
* Role-Based Access Control (RBAC)
* SMB File Sharing
* NTFS Permissions
* Security Groups
* Windows Networking
* UNC Paths
* Command Prompt (ipconfig, ping, hostname)


