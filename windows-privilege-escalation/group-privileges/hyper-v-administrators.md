# Hyper-V Administrators

#### **Hyper-V Administrators Group and Domain Controller Security Risks**

The **Hyper-V Administrators** group possesses comprehensive access to all Hyper-V features, granting its members significant control over virtualized environments. This level of access poses critical security implications, particularly when it comes to virtualized Domain Controllers (DCs).

**Key Points:**

1. **Full Access to Hyper-V Features**:
   * Members of the Hyper-V Administrators group can manage all aspects of Hyper-V, including the ability to create, modify, and delete virtual machines. This includes virtualized Domain Controllers.
2. **Virtualization of Domain Controllers**:
   * In environments where Domain Controllers are virtualized, the implications of having Hyper-V Administrator privileges are profound. These administrators effectively have the power to control the Domain Controller as if they were Domain Admins.
3. **Cloning Domain Controllers**:
   * A Hyper-V Administrator can easily create a clone of a live Domain Controller. This process involves taking a snapshot or creating a copy of the virtual machine hosting the Domain Controller, which can be done with minimal oversight.
4. **Mounting Virtual Disks**:
   * Once a clone is created, the administrator can mount the virtual disk of the cloned Domain Controller offline. This allows them to access sensitive files without the usual security measures in place.
5. **Extracting NTDS.dit**:
   * The **NTDS.dit** file is the Active Directory database file that contains all user accounts, group memberships, and password hashes within the domain. By accessing this file, an administrator could extract NTLM password hashes for all users in the domain.
6. **Potential for Privilege Escalation**:
   * With access to NTLM hashes, an attacker could perform offline attacks to crack passwords, potentially gaining access to higher-privileged accounts within the domain.
