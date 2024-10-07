# Print Operators

#### **Overview**

The **Print Operators** group in Windows is designed for users who need the ability to manage printers and print jobs. Members of this group can perform tasks like configuring printers, managing print queues, and performing print server tasks. While these permissions are focused on printing functionalities, they can potentially be exploited for privilege escalation on a Windows system.

#### **Privilege Escalation via Print Operators Group and Capcom.sys Driver**

The **Print Operators** group is a highly privileged group in Windows that grants its members several significant permissions, including:

* **`SeLoadDriverPrivilege`**: Allows members to load and manage system drivers.
* The ability to manage, create, share, and delete printers connected to a Domain Controller.
* The ability to log on locally to a Domain Controller and shut it down.

Given these privileges, members of this group can load system drivers, enabling them to exploit the system further.

#### **Using Capcom.sys for Privilege Escalation**

The **Capcom.sys** driver is a well-known driver that allows users to execute shell code with system privileges. This driver can be particularly useful for escalating privileges in a Windows environment.

1. **Download the Capcom.sys Driver**:
   * The Capcom.sys driver can be downloaded from the following GitHub repository:
     * [Capcom-Rootkit - Capcom.sys](https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys)
   * Additionally, you can find useful tools such as `LoadDriver.exe` and `ExploitCapcom.exe` in the following repository:
     * [SeLoadDriverPrivilege - Josh Morrison](https://github.com/JoshMorrison99/SeLoadDriverPrivilege)
2. **Create a Malicious Executable**:
   * Using Metasploit, create a malicious executable (e.g., `rev.exe`) that will provide a reverse shell when executed. This executable will be run with elevated privileges after loading the Capcom.sys driver.
3. **Load the Capcom.sys Driver**:
   *   Use the `LoadDriver.exe` tool to load the Capcom.sys driver. The command syntax is as follows:

       ```powershell
       .\LoadDriver.exe System\CurrentControlSet\MyService C:\Users\Test\Capcom.sys
       ```
   * Upon successful execution, this command should return `NTSTATUS: 00000000, WinError: 0`. If it does not, check the location of `Capcom.sys` or ensure that you are executing `LoadDriver.exe` from the correct directory.
4. **Execute the Malicious Executable**:
   *   After successfully loading the driver, use `ExploitCapcom.exe` to execute your malicious executable with elevated privileges:

       ```powershell
       .\ExploitCapcom.exe C:\Windows\Place\to\reverseshell\rev.exe
       ```
   * This command runs the `rev.exe` file with system privileges, providing the attacker with a reverse shell.

#### **Conclusion**

By leveraging the permissions granted to members of the **Print Operators** group, especially the ability to load drivers, an attacker can use the **Capcom.sys** driver to execute malicious code with system privileges. Understanding these techniques is crucial for securing Windows environments and preventing unauthorized access.
