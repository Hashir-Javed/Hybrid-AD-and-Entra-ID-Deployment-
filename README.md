# 🌐Hybrid AD & Entra ID Deployment


## Project Overview
This project demonstrates the setup and management of a hybrid IT environment that combines on-premises infrastructure with cloud services. It focuses on identity and access management, including user account provisioning, group management, authentication, and endpoint administration. The goal is to build and maintain a secure and scalable environment that reflects how many modern small-to-medium businesses manage users, devices, and resources across both on-premises and cloud platforms.

---


## Network Diagram
![diagram](Images/diagram.png)

---


## Technologies Used
* **Hypervisor:** VMware Workstation Pro (Configured with isolated NAT networking)
* **Operating Systems:** Windows Server 2022 (Standard Evaluation), Windows 11 Pro
* **Directory & Identity Services:** Active Directory Domain Services (AD DS), Microsoft Entra ID (Azure AD)
* **Identity Synchronization:** Microsoft Entra Connect
* **Network Services:** DNS (Domain Name System)
* **Administration & Automation:** Windows PowerShell, Group Policy Management, Active Directory Users and Computers (ADUC)

---

## Automated User and Infrastructure Management
To simulate a scalable corporate environment and avoid inefficient manual data entry, a custom **PowerShell script** was executed on the Domain Controller (CoreDC1). This script programmatically built the complete nested Organizational Unit (OU) hierarchy and bulk-provisioned department-specific user accounts with standard corporate attributes.

### 1. Hierarchy Strategy
The hierarchy is structured to separate administrative controls cleanly. The root container **CoreTech-Corporate** isolates lab assets from default Windows service objects, splitting them into:
* **Groups**: Dedicated for security and distribution group scoping.
* **Workstations**: Enclosure for corporate endpoint placement.
* **Users**: Nested departmental OUs (**IT**, **Sales**, **Finance**) managing targeted employee access profiles.

### 2. The Configuration Script
The following automated configuration script was executed via PowerShell ISE with Administrative privileges:

```powershell
# Define the main root folder path
$MainOU = "OU=CoreTech-Corporate,DC=coretech,DC=lab"
$Password = ConvertTo-SecureString "Password321" -AsPlainText -Force

# 1. Create the base sub-OUs
New-ADOrganizationalUnit -Name "Groups" -Path $MainOU
New-ADOrganizationalUnit -Name "Workstations" -Path $MainOU
New-ADOrganizationalUnit -Name "Users" -Path $MainOU

# Define the path to the newly created Users OU
$UsersOU = "OU=Users,$MainOU"

# 2. Create the Department OUs inside the Users OU
New-ADOrganizationalUnit -Name "IT" -Path $UsersOU
New-ADOrganizationalUnit -Name "Sales" -Path $UsersOU
New-ADOrganizationalUnit -Name "Finance" -Path $UsersOU

# 3. Create the Users in their respective department OU folders

# James Blake in IT
New-ADUser -Name "James Blake" -SamAccountName "jblake" -UserPrincipalName "jblake@coretech.lab" -Path "OU=IT,$UsersOU" -AccountPassword $Password -ChangePasswordAtLogon $true -Enabled $true -Department "IT"

# Jon Jones in Sales
New-ADUser -Name "Jon Jones" -SamAccountName "jjones" -UserPrincipalName "jjones@coretech.lab" -Path "OU=Sales,$UsersOU" -AccountPassword $Password -ChangePasswordAtLogon $true -Enabled $true -Department "Sales"

# Alex Wong in Finance
New-ADUser -Name "Alex Wong" -SamAccountName "awong" -UserPrincipalName "awong@coretech.lab" -Path "OU=Finance,$UsersOU" -AccountPassword $Password -ChangePasswordAtLogon $true -Enabled $true -Department "Finance"
```

### 3. Configuration Results
Upon execution, the script successfully:
* Built the multi-tiered OU architecture.
* Populated the **Department** attribute for all objects to ensure granular directory tracking.
* Enforced standard security compliance by setting initial secure passwords and checking the **ChangePasswordAtLogon** flag for all new hires.

