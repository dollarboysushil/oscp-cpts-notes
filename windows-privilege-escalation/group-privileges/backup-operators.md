# Backup Operators

#### **Overview**

The **Backup Operators** group is a built-in group in Windows that grants members the ability to back up and restore files, even if they do not have permission to access the files under normal circumstances. This privilege makes the group particularly powerful for potential abuse in **privilege escalation** scenarios, as **Backup Operators** can access sensitive files like the **SAM** (Security Account Manager) database and system files, which can lead to gaining higher-level access or even **SYSTEM** privileges.

#### **Key Privileges of Backup Operators**

Members of the **Backup Operators** group have two key privileges:

1. **SeBackupPrivilege**:
   * Allows users to **bypass file system permissions** to back up files. This means a Backup Operator can read files that they normally do not have permissions to access.
2. **SeRestorePrivilege**:
   * Allows users to **restore files** to any location on the file system, including protected or sensitive locations. This also allows modifying files that would otherwise be restricted.

## **Exploiting the Backup Operators Group for Privilege Escalation**

## **Backup and Extract the SAM Database**

One of the primary ways to exploit **Backup Operators** is by accessing and extracting the **SAM** (Security Account Manager) database. The **SAM** database contains password hashes for local user accounts, including **Administrator** and **SYSTEM**.

**Steps to Exploit:**

1.  **Backup the SAM, SYSTEM, and SECURITY Hives:**

    Use the `reg save` command to back up these registry hives, which contain critical security information (including password hashes).

    ```bash
    reg save hklm\sam c:\temp\sam
    reg save hklm\system c:\temp\system
    reg save hklm\security c:\temp\security
    ```

    The `reg save` command can be used because **Backup Operators** have the privilege to read files they normally wouldn't have access to.
2.  **Extract Password Hashes Using Tools**:

    Once you've backed up the hives, you can copy them to your attacker machine and use tools like **mimikatz** or **John the Ripper** to extract password hashes from the **SAM** and crack them.

    Example using **mimikatz**:

    ```bash
    sekurlsa::samdump::local c:\temp\sam c:\temp\system
    ```

    \
    or from linux<br>

    Using **secretsdump.py**:

    ```bash
    dollarboysushil@kali$ secretsdump.py -ntds ntds.dit -system system.back LOCAL
    ```
3.  **Crack the Hashes or Use Pass-the-Hash**:

    If the hashes are crackable, you can attempt to crack them and log in with a privileged account. Alternatively, you can use the **pass-the-hash** technique to impersonate a privileged user (e.g., **Administrator**) without knowing the password.

