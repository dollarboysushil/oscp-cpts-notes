# Credentialed Enumeration From Windows

## **Active Directory Credentialed Enumeration (Windows)**

### **1️⃣ PowerShell & Active Directory Module**

PowerShell’s built-in `ActiveDirectory` module provides essential tools for querying and managing Active Directory objects.

#### **Loading the Module**

* `Get-Module` → Lists available PowerShell modules
* `Import-Module ActiveDirectory` → Loads the Active Directory module if not already imported

#### **Domain & User Enumeration**

* `Get-ADDomain` → Displays general domain information
* `Get-ADTrust -Filter *` → Checks for domain trust relationships
* `Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName` → Identifies Kerberoastable accounts
* `Get-ADGroup -Filter * | Select-Object Name` → Lists all AD groups
* `Get-ADGroup -Identity "Administrators"` → Retrieves details of a specific group
* `Get-ADGroupMember -Identity "Domain Admins"` → Lists members of a specified group

### **2️⃣ PowerView**

[PowerView](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) is a robust PowerShell tool used for Active Directory reconnaissance.

#### **Basic Enumeration Commands**

* `Get-Domain` → Retrieves details about the Active Directory domain
* `Get-DomainUser` → Lists users in the domain
* `Get-DomainComputer` → Enumerates domain-joined computers
* `Get-DomainController` → Identifies domain controllers
* `Get-DomainGroup` → Lists all domain groups
* `Get-DomainOU` → Enumerates Organizational Units (OUs)
* `Find-InterestingDomainAcl` → Detects modification rights in domain ACLs

#### **Privilege & Access Enumeration**

* `Find-DomainUserLocation` → Identifies machines where users are logged in
* `Find-DomainShare` → Discovers accessible file shares in the domain
* `Find-InterestingDomainShareFile` → Searches for potentially sensitive files in shares
* `Test-AdminAccess` → Verifies administrative privileges on a remote machine

#### **Trust & Group Membership Analysis**

* `Get-DomainTrust` → Retrieves domain trust relationships
* `Get-ForestTrust` → Identifies forest-level trust relationships
* `Get-DomainForeignUser` → Lists users who belong to groups outside their primary domain
* `Get-DomainForeignGroupMember` → Finds groups with members from outside domains
* `Get-DomainTrustMapping` → Maps domain trust relationships

### **3️⃣ SharpView**

SharpView is a .NET alternative to PowerView, offering similar enumeration functionality in a compiled format.

* `.\SharpView.exe Get-DomainUser -Identity "john.doe"` → Retrieves details about a specific user

### **4️⃣ BloodHound & SharpHound**

BloodHound is a powerful tool used for visualizing Active Directory attack paths. The `SharpHound.exe` collector gathers necessary information.

* `.\SharpHound.exe -c All --zipfilename DataCollection` → Executes full AD enumeration

Once collected, transfer the results to an attacker-controlled machine and analyze them in BloodHound.

### **5️⃣ Snaffler - File Scavenging Tool**

[Snaffler](https://github.com/SnaffCon/Snaffler) automates the discovery of sensitive files within an Active Directory environment.

* `Snaffler.exe -s -d corp.local -o findings.log -v data` → Runs a deep scan for valuable files

### **6️⃣ Additional Recon Tools & Methods**

#### **General System Enumeration**

* `Get-NetLocalGroup` → Lists local groups on a machine
* `Get-NetLocalGroupMember` → Enumerates members of a local group
* `Get-NetShare` → Identifies accessible shared directories
* `Get-NetSession` → Retrieves active session details on a machine

#### **Service Principal Name (SPN) Enumeration**

* `Get-DomainSPNTicket` → Requests Kerberos tickets for Service Principal Name (SPN) accounts (potential Kerberoasting targets)

By leveraging these techniques, an attacker or pentester can gather valuable information about an Active Directory environment while minimizing detection. 🚀
