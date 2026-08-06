# Windows-HomeLab

Enterprise Active Directory & File Services Homelab Documentation
Executive Overview
This documentation outlines the complete step-by-step implementation of a virtualized Windows Server infrastructure. The objective of this project was to deploy a fully functional Active Directory Domain Services (AD DS) environment, join a Windows 11 client machine to the domain, configure structured Organizational Units (OUs) and Role-Based Access Control (RBAC) groups, and establish a centralized File Server utilizing both Share and NTFS permission hierarchies.


Phase 1: Virtualization Platform Evaluation & Hypervisor Selection
Before provisioning virtual machines, I evaluated the primary hypervisor options to determine the best environment for hosting a Windows enterprise lab.
VMware Workstation Pro: Selected as the primary platform due to its seamless native guest drivers, robust performance tools, snapshot capabilities, and enterprise parity with ESXi environments.
Oracle VM VirtualBox: Evaluated as a secondary lightweight alternative, preferred for its open-source license model and straightforward GUI layout.
Microsoft Hyper-V & Proxmox VE: Analyzed for enterprise context; Hyper-V as a native Windows feature and Proxmox VE as a bare-metal Type-1 hypervisor.
Phase 2: Base Operating System Installation & Initial Provisioning
I acquired the Windows Server 2025 Evaluation ISO from the Microsoft Evaluation Center and provisioned the virtual machine.
Server VM Provisioning
Virtual Machine Creation: Created a new VM with standard lab hardware specifications (40 GB Virtual Hard Disk, 2 CPU Cores, 4 GB RAM).
OS Deployment: Mounted the Windows Server 2025 ISO, booted the VM, and explicitly selected Windows Server 2025 Standard (Desktop Experience) to ensure the Graphical User Interface (GUI) was installed rather than Server Core.
Guest Additions: Installed VMware Tools via the Virtual Machine menu, mounted the virtual CD drive in This PC, ran setup.exe, and rebooted to enable full-screen auto-resolution and enhanced guest performance.
Hostname Standardization: To eliminate default generic hostnames (e.g., WIN-XXXXX), I navigated to Advanced System Settings > Computer Name and renamed the host to FILESERVER01 prior to AD DS installation.
DOS
:: Verification of hostname modification
hostname
:: Output: FILESERVER01

Phase 3: Active Directory Domain Services (AD DS) & Network Configuration
To establish the enterprise identity boundary, I prepared the server's network configuration and promoted it to a Domain Controller.
Static IP & DNS Assignment
Domain Controllers require static IP addressing to prevent dynamic lease changes from severing client connections.
I executed ipconfig /all to identify the hypervisor-assigned gateway and subnet.
Navigated to Network & Internet > Ethernet > IP Assignment and converted the connection from DHCP to Manual (IPv4).
Assigned a static IP address, subnet mask, and default gateway.
Configured the Preferred DNS Server to 127.0.0.1 (loopback), ensuring the server references its own local DNS service.
DOS
:: Network Verification post-static assignment
ipconfig /all

AD DS Role Installation & Forest Promotion
Opened Server Manager > Add Roles and Features.
Selected Role-based or feature-based installation, targeted FILESERVER01, and checked Active Directory Domain Services along with the required management tools.
Completed the role installation and clicked Promote this server to a domain controller.
Selected Add a new forest and established the Root Domain Name as mylestech.local.
Set the Directory Services Restore Mode (DSRM) password and proceeded through the prerequisite checks.
Clicked Install, causing the server to automatically reboot upon completion and initialize Active Directory.
Phase 4: Active Directory Hierarchy & Identity Management
Following the promotion, I structured Active Directory to reflect a typical enterprise organizational model using Active Directory Users and Computers (dsa.msc).
mylestech.local (Domain Root)
└── USA (Top-Level OU)
    ├── Users (Sub-OU)
    │   ├── IT (Sub-OU)
    │   ├── HR (Sub-OU)
    │   └── Accounting / Finance (Sub-OU)
    └── Computers (Sub-OU)

OU & Account Creation
Organizational Units: Created a top-level OU named USA, followed by nested sub-OUs for Users and Computers. Within Users, I established departmental OUs (IT, HR, Finance).
Administrative Account: Created an IT administrative account within the IT OU. Added this account to the Domain Admins Security Group via the Member Of tab to facilitate domain join actions.
Standard User Accounts: Created standard departmental accounts (e.g., HR User) with options configured to Password never expires for lab persistence.
Security Groups: Created Global Security Groups matching each department (IT, HR, Finance) inside Active Directory and populated them with their respective departmental user accounts.
Phase 5: Client VM Provisioning & Active Directory Domain Join
To test authentication and remote policies, I provisioned a Windows 11 Professional client machine.
Windows 11 Client Setup
Downloaded the Windows 11 ISO and created a virtual machine configured with a 70 GB Virtual Disk, 4 GB RAM, and TPM encryption enabled.
Selected Windows 11 Pro during setup to ensure domain join support (excluding Windows Home edition).
Installed VMware Tools on the client machine post-install.
Network Alignment & Domain Join
Navigated to Network & Internet > Ethernet on the Windows 11 client and set the Preferred DNS Server manually to the IP address of FILESERVER01 (mylestech.local).
Opened Command Prompt and verified network connectivity and name resolution:
DOS
ping 192.168.x.x  :: (IP of FILESERVER01)

Opened System Properties (sysdm.cpl) > Computer Name tab, clicked Change, set the computer name to CLIENT01, and selected the Domain radio button.
Entered mylestech.local as the domain and authenticated using the created Domain Admin credentials.
Received the "Welcome to the mylestech.local domain" prompt and restarted the machine.
Logged in to CLIENT01 using standard domain user credentials (mylestech\username).
Asset Management & OU Staging
Returned to FILESERVER01 and opened Active Directory Users and Computers.
Located CLIENT01 under the default Computers container, right-clicked, and selected Move to place it into USA > Computers OU for administrative tracking.
Updated the object properties with a clear asset description.
Phase 6: Centralized File Server & Dual-Layer Permission Matrix
I configured a secure, centralized shared directory structure using the intersection of SMB Share Permissions and NTFS File System Permissions.
C:\
└── CompanyData (Shared Path: \\FILESERVER01\CompanyData)
    ├── HR
    ├── IT
    ├── Finance
    └── Public

Directory & Share Provisioning
Created a root directory named C:\CompanyData on the server.
Created nested folders: HR, IT, Finance, and Public.
Right-clicked C:\CompanyData, navigated to Properties > Sharing > Advanced Sharing, checked Share this folder, and accessed Permissions.
Set Share Permissions for Everyone to Full Control.
Architectural Rationale: Applying broad Share Permissions allows granular access control to be strictly governed by NTFS permissions, following Microsoft best practices.
NTFS Permission Hardening & Inheritance Explicit Conversion
To secure departmental directories (e.g., C:\CompanyData\HR):
Right-clicked C:\CompanyData\HR > Properties > Security tab > Advanced.
Clicked Disable Inheritance and selected "Convert inherited permissions into explicit permissions on this object".
Removed broad group access (e.g., standard Users) from the Access Control List (ACL), keeping only SYSTEM and Administrators.
Added the Active Directory Security Group HR to the explicit ACL and granted Modify permissions.
Applied identical access control policies across all directories:
IT Directory: IT Group granted Modify; explicit inheritance disabled.
Finance Directory: Finance Group granted Modify; explicit inheritance disabled.
Public Directory: Domain Users granted Read permissions; IT Group granted Modify permissions.
Access Enforcement Matrix
Directory
Share Permissions
NTFS Permissions
Effective Access
\HR
Everyone: Full Control
HR Group: Modify

Unauthorized Groups: Removed
HR members can read/write/modify; non-HR domain users are explicitly denied.
\IT
Everyone: Full Control
IT Group: Modify

Unauthorized Groups: Removed
IT members can read/write/modify; other users are explicitly denied.
\Finance
Everyone: Full Control
Finance Group: Modify

Unauthorized Groups: Removed
Finance members can read/write/modify; other users are explicitly denied.
\Public
Everyone: Full Control
Domain Users: Read

IT Group: Modify
All domain users can read files; IT can manage and modify contents.

Phase 7: Verification & Client Access Validation
To verify effective permissions and operational capabilities:
Logged into CLIENT01 as an HR User.
Accessed the file share via UNC Path: \\FILESERVER01\CompanyData.
Successfully navigated into \HR and created/edited test documents.
Attempted to access \Finance and \IT directories, confirming access was denied as expected due to NTFS security settings.
Accessed \Public, confirming read-only functionality and restriction from creating new files within the root directory.

