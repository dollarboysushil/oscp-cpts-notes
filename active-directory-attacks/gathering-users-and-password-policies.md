# Gathering Users & Password Policies

## Gathering Users

#### User Enumeration Techniques for Active Directory Attacks

Here are various methods to enumerate users from an Active Directory (AD) environment using different tools.

1. **enum4linux**
   *   **Command:**

       ```bash
       enum4linux -U {DC-IP}
       ```
   * **Description:** Enumerates users from the target Domain Controller (DC) using SMB.
2. **RPCClient**
   *   **Command:**

       ```bash
       rpcclient -U "" -N {DC-IP}
       ```
   *   **Followed by:**

       ```bash
       rpcclient$> enumdomusers
       ```
   * **Description:** Uses the Windows RPC protocol to list domain users. The command starts an RPCClient session and then executes `enumdomusers` to retrieve user details.
3. **CrackMapExec**
   *   **Command:**

       ```bash
       crackmapexec smb {DC-IP} --users
       ```
   * **Description:** Enumerates users via the SMB protocol. Useful for checking user existence across a range of IPs or for one specific DC.
4. **LDAPSearch**
   *   **Command:**

       ```bash
       ldapsearch -h {DC-IP} -x -b "DC=MARVEL,DC=LOCAL" -s sub "(&(objectclass=user))"
       ```
   * **Description:** Performs LDAP enumeration to retrieve all users from the Active Directory, specifying the base DN and search scope.
5. **WindapSearch**
   *   **Command:**

       ```bash
       ./windapsearch.py --dc-ip {DC-IP} -u "" -U
       ```
   * **Description:** Uses WindapSearch to enumerate users via LDAP, providing a quick way to find users on the DC.
6. **Kerbrute**
   *   **Command:**

       ```bash
       kerbrute userenum -d marvel.local --dc {DC-IP} /opt/seclists/usernames/xato-net-10-million-usernames.txt
       ```
   * **Description:** Enumerates valid usernames via Kerberos, useful for finding valid accounts by trying different usernames.
7. **RID-Brute with CrackMapExec**
   *   **Command:**

       ```bash
       crackmapexec smb {DC-IP} -u 'guest' -p '' --rid-brute
       ```
   * **Description:** Performs RID brute-forcing to identify user accounts by enumerating Security Identifiers (SIDs).

These techniques help gather user information from an AD environment, which is essential for subsequent attacks like password spraying or privilege escalation.

## Enumerating Password Policies

### Enumerating & Retrieving Password Policies - Credentialed ⭐

With valid domain credentials, password policies can be obtained remotely using tools like `crackmapexec` or `rpcclient`.

1. **CrackMapExec**
   *   **Command:**

       ```bash
       crackmapexec smb {DC-IP} -u sushil -p poudel --pass-pol
       ```
   * **Description:** Retrieves domain password policy with a valid username and password.
2. **rpcclient**
   *   **Command:**

       ```bash
       rpcclient -U "username" {DC-IP}
       ```
   *   **Followed by:**

       ```bash
       rpcclient$> getdompwinfo
       ```
   * **Description:** Lists the password policy information using valid credentials.

### Enumerating Password Policies - SMB NULL Sessions ⭐

SMB NULL sessions allow an unauthenticated attacker to retrieve information from the domain, such as a list of users, groups, and password policies.

1. **rpcclient**
   *   **Command:**

       ```bash
       rpcclient -U "" -N {DC-IP}
       ```
   *   **Followed by:**

       ```bash
       rpcclient$> querydominfo
       rpcclient$> getdompwinfo
       ```
   * **Description:** Uses `querydominfo` to confirm NULL session access and `getdompwinfo` to retrieve the password policy.
2. **enum4linux**
   *   **Command:**

       ```bash
       enum4linux -P {DC-IP}
       ```
   * **Description:** Retrieves the password policy from a domain controller using SMB NULL sessions.
3. **enum4linux-ng**
   *   **Command:**

       ```bash
       enum4linux-ng -P {DC-IP} -oA output
       ```
   * **Description:** A Python rewrite of `enum4linux` with additional features like exporting the output.

### Enumerating Null Sessions - from Windows

Performing a NULL session attack from a Windows machine is less common, but still possible.

*   **Command:**

    ```bash
    net use \\{DC-NAME}\ipc$ "" /u:""
    ```
* **Description:** Establishes a NULL session to the domain controller.

If we are authenticated to domain joined windows host, then we can use command\
&#x20;`net accounts` \
to retrieve password policy.

### Enumerating Password Policies - LDAP Anonymous Bind

Anonymous LDAP binds can also be used to gather password policies without credentials.

1. **ldapsearch**
   *   **Command:**

       ```bash
       ldapsearch -h {DC-IP} -x -b "DC=marvel,DC=local" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
       ```
   * **Description:** Searches LDAP anonymously to extract password policy details.
