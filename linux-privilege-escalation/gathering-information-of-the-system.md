# Gathering Information of the System

To escalate privileges on a Linux system, it’s crucial to gather as much information about the environment as possible. This helps identify potential weaknesses or misconfigurations that can be exploited. Below are some key commands that can be used for enumeration, along with a few additional ones to broaden the assessment.

#### Linux Privilege Escalation: Environment Enumeration

1.  **Check OS Information:**

    ```bash
    cat /etc/os-release
    ```

    Contains details about the operating system version, distribution, and other useful information.
2.  **Inspect the PATH Variable:**

    ```bash
    echo $PATH
    ```

    Reveals directories in the system's PATH, which could expose writable or insecure paths.
3.  **List Environment Variables:**

    ```bash
    env
    ```

    Lists all environment variables. Sensitive information like credentials might be exposed.
4.  **Check Kernel Version:**

    ```bash
    uname -a
    ```

    Displays kernel version, system architecture, and other details. Certain versions may have known vulnerabilities.
5.  **List Available Shells:**

    ```bash
    cat /etc/shells
    ```

    Shows available login shells. Vulnerable or misconfigured shells can provide opportunities for escalation.
6.  **View Routing Table:**

    ```bash
    route
    # or
    netstat -rn
    ```

    Displays the routing table, helping identify available network interfaces.
7.  **Check ARP Table:**

    ```bash
    arp -a
    ```

    Shows the ARP table, revealing other hosts the target machine communicates with.
8.  **List SUID and SGID Files:**

    ```bash
    find / -perm /4000 2>/dev/null
    ```

    Finds SUID binaries, which run with the file owner's privileges (often root).
9.  **Check for Running Processes:**

    ```bash
    ps aux
    ```

    Lists all running processes. Look for processes running as root or with elevated privileges.
10. **Check for Installed Packages:**

    ```bash
    dpkg -l   # For Debian-based systems
    rpm -qa   # For Red Hat-based systems
    ```

    Lists installed packages. Some might have known vulnerabilities or misconfigurations.
11. **Check Crontab Entries:**

    ```bash
    crontab -l
    cat /etc/crontab
    ```

    Lists scheduled cron jobs. Misconfigured cron jobs running as root can be exploited.
12. **Check Active Network Connections:**

    ```bash
    netstat -tuln
    ```

    Displays active network connections and listening ports, which can help identify services running as root.
13. **View Mounted File Systems:**

    ```bash
    mount
    ```

    Lists mounted file systems. Uncommon or insecure mounts can offer opportunities for privilege escalation.
14. **Check Writable Directories for Other Users:**

    ```bash
    find / -writable -type d 2>/dev/null
    ```

    Identifies world-writable directories that could be leveraged to inject malicious files.
15. **Inspect sudo Privileges:**

    ```bash
    sudo -l
    ```

    Lists what commands the current user is allowed to run with sudo, revealing potential escalation paths.
