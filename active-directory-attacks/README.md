# Active Directory Attacks

## Some Popular Tools

| **Tool**          | **Purpose**                                                                                    | **Features**                                                                                             |
| ----------------- | ---------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **BloodHound**    | Reveals attack paths in AD using graph theory.                                                 | Maps potential attack paths, visualizes high-value targets, identifies misconfigurations.                |
| **PowerView**     | PowerShell tool for enumerating AD environments.                                               | Enumerates users, groups, computers, ACLs, SPNs, and shares; identifies group memberships.               |
| **Mimikatz**      | Extracts plaintext passwords, hashes, PINs, and Kerberos tickets from memory.                  | Supports Pass-the-Hash, Pass-the-Ticket, Golden Ticket attacks, and extracts credentials from LSASS.     |
| **Impacket**      | Python library for network protocol manipulation, used for remote code execution and AD tasks. | Tools for dumping secrets (`secretsdump.py`), remote execution (`wmiexec.py`), and interacting with SMB. |
| **Responder**     | Captures credentials by poisoning LLMNR, NBT-NS, and MDNS requests.                            | Acts as a rogue server, captures NTLM hashes, and can be used for SMB relay attacks.                     |
| **CrackMapExec**  | Multifunctional tool for enumeration, exploitation, and post-exploitation in AD.               | Checks for SMB shares, password policies, performs password spraying, and executes remote commands.      |
| **Cobalt Strike** | Commercial penetration testing tool with C2 and post-exploitation capabilities.                | Supports lateral movement, credential harvesting, integrates with Mimikatz, and provides C2 framework.   |
| **Rubeus**        | C# tool for Kerberos interaction and abuse.                                                    | Performs Kerberos ticket requests, Pass-the-Ticket, Overpass-the-Hash, and Kerberoasting.                |
| **SharpHound**    | Data collection component of BloodHound, written in C#.                                        | Gathers information about AD objects via LDAP, SMB, DCOM, and supports stealthy data collection.         |
| **ADRecon**       | Reconnaissance tool that generates detailed AD reports.                                        | Enumerates domain controllers, trusts, user accounts, groups, and group memberships.                     |
| **Nishang**       | Collection of PowerShell scripts for exploitation, post-exploitation, and reconnaissance.      | Performs AD enumeration, privilege escalation, lateral movement, and generates reverse shells.           |
| **Kerbrute**      | Tool for brute-forcing Kerberos logins and enumerating valid usernames.                        | Performs user enumeration and password spraying via Kerberos.                                            |
| **Inveigh**       | PowerShell-based tool for network protocol poisoning and credential capture.                   | Acts like Responder to capture NTLMv1/v2 hashes over LLMNR/NetBIOS.                                      |
| **rpcinfo**       | Displays information about RPC services running on a remote machine.                           | Enumerates available RPC services, helping identify remote accessible functionalities.                   |
| **rpcclient**     | Command-line tool for interacting with Windows RPC services.                                   | Retrieves user information, SID lookups, and NetBIOS name tables; useful for AD reconnaissance.          |

