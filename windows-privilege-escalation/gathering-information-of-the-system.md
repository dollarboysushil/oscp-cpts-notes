# Gathering Information of the System

## 1. Network & System Information

* **Network Configuration:**
  * `ipconfig /all` → View detailed network interface configurations (IP, DNS, etc.).
  * `arp -a` → Display ARP cache (shows local network devices).
  * `route print` → View the system's routing table.
* **Service Information:**
  * `tasklist /svc` → List all running processes along with their services.
  * `netstat -ano` → Display active TCP/UDP connections and listening ports with process IDs.
* **System Info:**
  * `systeminfo` → Get a comprehensive overview of the system (OS version, architecture, hotfixes, etc.).
  * `wmic product get name` → List installed software via the command line.
    * `Get-WmiObject -Class Win32_Product | select Name, Version` → List installed software via PowerShell.

## **2. User & Privilege Enumeration**

* **Current User & Privileges:**
  * `whoami /priv` → List current user privileges.
  * `whoami /groups` → List group memberships for the current user.
  * `net user` → Get a list of all user accounts.
  * `query user` → Display logged-in users on the system.
* **Groups & Password Policies:**
  * `net localgroup` → List all local groups.
  * `net localgroup "Backup Operators"` → List users in the Backup Operators group.
  * `net accounts` → View password policies and other account-related configurations.

## **3. Security Tools & Configuration**

* **Windows Defender:**
  * `Get-MpComputerStatus` → Check the status of Windows Defender (active, signatures, etc.).
* **AppLocker:**
  * `Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections` → List effective AppLocker rules.
  * `Test-AppLockerPolicy -Path C:\Windows\System32\cmd.exe -User Everyone` → Test if a specific executable (cmd.exe) can be run for a specific user.

## **4. Named Pipes & Permission Enumeration**

* **Listing Named Pipes:**
  * `pipelist.exe /accepteula` → List all named pipes on the system.
* **Access Rights to Named Pipes:**
  * `accesschk.exe /accepteula \\.\Pipe\lsass -v` → Check permissions for a specific named pipe (e.g., LSASS pipe).
  * `accesschk.exe /accepteula -w \\.\Pipe\SQLLocal\SQLEXPRESS01 -v` → Check write access for a SQL pipe.

## **5. Environment Variables & Other Useful Commands**

* **View Environment Variables:**
  * `set` → Display environment variables for the current session.
