# User Privileges

## **Basic Privilege Enumeration**

*   **Check current user privileges:**

    * `whoami /priv` → Lists all privileges assigned to the current user. This is crucial for identifying rights that could lead to privilege escalation (e.g., SeBackupPrivilege, SeImpersonatePrivilege).

    **Common Privileges to Look for:**

    * `SeBackupPrivilege` → Allows the user to bypass file security to perform backups.
    * `SeRestorePrivilege` → Allows the user to overwrite system files, useful for file replacement attacks.
    * `SeTakeOwnershipPrivilege` → Allows taking ownership of any object, which can lead to control over important files or processes.
    * `SeImpersonatePrivilege` → Allows impersonation of a token, often leading to privilege escalation (common in token impersonation attacks).
    * `SeDebugPrivilege` → Allows debugging processes, typically restricted to admins. Can be used to inject code into privileged processes.





