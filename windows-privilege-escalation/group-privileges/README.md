# Group Privileges

#### **Overview**

Windows groups are collections of user accounts that share the same security permissions and rights. Each group has specific privileges that determine what actions members of that group can perform on a system. Some groups have extensive privileges that can be leveraged for privilege escalation.

Understanding group privileges is crucial for enumerating potential attack vectors in **privilege escalation** scenarios. Some groups, like **Administrators**, **Backup Operators**, and **Remote Desktop Users**, can provide direct or indirect paths to gaining **SYSTEM** or administrative privileges.

## **Common Privileged Groups in Windows**

### **1. Administrators Group**

The **Administrators** group has the highest level of access on a Windows machine, including full control over all system files, settings, and user accounts. Members of this group can:

* Execute any command with **SYSTEM** privileges.
* Install and modify software, hardware drivers, and security settings.
* Manage user accounts and change passwords.
* Take ownership of files and directories.

**Key Commands:**

* `net localgroup administrators` → List all members of the **Administrators** group.

**Privilege Escalation Path:**

* If you can add yourself to the **Administrators** group or impersonate a member, you can gain full control of the system.

***

### **2. Backup Operators Group**

The **Backup Operators** group has special privileges that allow members to back up and restore files, even if they don't have explicit permissions to access those files. Members of this group can:

* Bypass **NTFS** permissions and access files for backup and restoration.
* **Backup files** from any directory.
* **Restore files** to any directory.
* Log on locally to a machine.

**Key Commands:**

* `net localgroup "Backup Operators"` → List all members of the **Backup Operators** group.

**Privilege Escalation Path:**

* Backup Operators can read or overwrite sensitive files, including the **SAM** (Security Account Manager) database, which stores user password hashes. By extracting hashes, they can perform **pass-the-hash** attacks or crack the hashes to gain higher-level access.

***

### **3. Remote Desktop Users Group**

The **Remote Desktop Users** group grants the ability to log in to a system via **Remote Desktop Protocol (RDP)**. While not a highly privileged group, remote access can facilitate further exploitation, such as:

* Running commands remotely.
* Enumerating the system and potentially exploiting local vulnerabilities to escalate privileges.

**Key Commands:**

* `net localgroup "Remote Desktop Users"` → List all members of the **Remote Desktop Users** group.

**Privilege Escalation Path:**

* **Remote Desktop Users** can connect to the system via RDP, and if they can exploit a local privilege escalation vulnerability (such as **SeImpersonatePrivilege**), they can elevate to **SYSTEM** or **Administrator**.

***

### **4. Power Users Group**

In earlier versions of Windows (pre-Windows Vista), the **Power Users** group had elevated privileges similar to the **Administrators** group. However, in modern versions of Windows, the **Power Users** group has significantly reduced capabilities. Members of this group can:

* Install programs.
* Modify system settings to a limited extent.
* Manage some services and user accounts.

**Key Commands:**

* `net localgroup "Power Users"` → List all members of the **Power Users** group.

**Privilege Escalation Path:**

* While this group has been deprecated, on older systems or misconfigured environments, being a member of this group may still provide a path for privilege escalation by installing malicious software or exploiting vulnerable system configurations.

***

### **5. Users Group**

The **Users** group contains standard user accounts with limited privileges. Members can:

* Access their own files and certain shared system resources.
* Run applications that do not require elevated privileges.
* Access shared folders if permissions allow.

**Key Commands:**

* `net localgroup users` → List all members of the **Users** group.

**Privilege Escalation Path:**

* Users in this group typically do not have elevated privileges, but privilege escalation is possible if there are misconfigurations or vulnerable software that can be exploited to gain administrative rights.

***

### **6. Guests Group**

The **Guests** group is designed for temporary users with minimal permissions. Members can:

* Log on with a temporary profile.
* Access basic resources on the system.

**Key Commands:**

* `net localgroup guests` → List all members of the **Guests** group.

**Privilege Escalation Path:**

* Members of this group have very limited rights, but if they can exploit a vulnerability (such as a privilege escalation bug or improperly configured permissions), they may be able to escalate to a more privileged account.

***

#### **Other Special Groups**

### **7. Distributed COM Users Group**

Members of this group can launch, activate, and use Distributed Component Object Model (DCOM) objects remotely. While not immediately powerful, if DCOM is misconfigured, it can be used in **DCOM/NTLM reflection** attacks for privilege escalation.

**Key Commands:**

* `net localgroup "Distributed COM Users"` → List all members of the **Distributed COM Users** group.

***

### **8. Hyper-V Administrators Group**

Members of this group have administrative privileges to manage Hyper-V virtual machines (VMs). They can:

* Control virtual machine configurations.
* Start or stop VMs.
* Access virtual hard drives (VHDs) containing sensitive data.

**Key Commands:**

* `net localgroup "Hyper-V Administrators"` → List all members of the **Hyper-V Administrators** group.

## **Enumeration of Group Privileges**

Enumerating group memberships and privileges is crucial for assessing the attack surface in a Windows environment.

**Useful Commands for Enumeration:**

*   **List All Users in a Group**:

    ```bash
    net localgroup <groupname>
    ```
*   **List All Groups**:

    ```bash
    net localgroup
    ```
*   **Check User Group Membership**:

    ```bash
    whoami /groups
    ```
*   **Check Privileges of the Current User**:

    ```bash
    whoami /priv
    ```
