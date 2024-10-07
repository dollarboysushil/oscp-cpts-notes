# Server Operators

#### **Overview**

The **Server Operators** group is a built-in security group in Windows Server environments. Members of this group are granted specific administrative privileges that allow them to perform server-related tasks without having full administrative rights. This group is primarily designed for delegated server management.

#### **Key Privileges of Server Operators**

Members of the **Server Operators** group have the following privileges:

1. **Start and Stop Services**:
   * They can start, stop, and pause services on the server, which is crucial for server maintenance and troubleshooting.
2. **Manage Shared Resources**:
   * Server Operators can create, modify, and delete shared folders and manage printer shares, allowing them to administer shared resources effectively.
3. **Backup and Restore Operations**:
   * Members can back up files and restore files from backup, making it easier to manage data recovery processes.
4. **Log on Locally**:
   * Members have the ability to log on locally to the server, which allows them to directly manage the server through its console.
5. **Manage Local Users and Groups**:
   * They can add or remove users from local groups and manage local accounts, which is important for user management tasks.

#### **Limitations**

While the **Server Operators** group has significant privileges, it does not have the same level of access as the **Domain Admins** group. Notably, Server Operators cannot:

* **Manage Active Directory**: They do not have permissions to modify Active Directory objects or group memberships outside of local server settings.
* **Modify System Settings**: Critical system configurations that affect the entire domain or security policies are beyond their reach.
