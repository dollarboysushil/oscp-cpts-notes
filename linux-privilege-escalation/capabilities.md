# Capabilities

Capabilities split the traditionally all-or-nothing root privileges into distinct units that can be independently enabled or disabled for processes.



**Common Capabilities**

Here are some common capabilities available in Linux:

| Capability             | Description                                                                    |
| ---------------------- | ------------------------------------------------------------------------------ |
| `CAP_CHOWN`            | Change file ownership.                                                         |
| `CAP_DAC_OVERRIDE`     | Bypass file read, write, and execute permission checks.                        |
| `CAP_DAC_READ_SEARCH`  | Bypass file read permission checks.                                            |
| `CAP_FOWNER`           | Bypass permission checks on operations that normally require the file's owner. |
| `CAP_NET_ADMIN`        | Perform various network-related operations, such as configuring interfaces.    |
| `CAP_NET_BIND_SERVICE` | Bind to network ports below 1024.                                              |
| `CAP_SYS_ADMIN`        | Perform a wide range of administrative tasks.                                  |
| `CAP_SYS_MODULE`       | Load and unload kernel modules.                                                |
| `CAP_SYS_RAWIO`        | Perform raw I/O operations.                                                    |
| `CAP_SYS_TIME`         | Modify the system clock.                                                       |



#### Linux Capabilities: Viewing and Checking

**Viewing Capabilities**

You can view the capabilities of a binary using the `getcap` command. For example:

```bash
getcap /path/to/binary
```

**Setting Capabilities**

To set capabilities on a binary, use the `setcap` command. For example, to give a binary the capability to bind to low-numbered ports:

```bash
sudo setcap 'cap_net_bind_service=+ep' /usr/bin/somebinary
```

**Removing Capabilities**

To remove capabilities, use the `setcap` command with a minus sign:

```bash
sudo setcap 'cap_net_bind_service=-ep' /usr/bin/somebinary
```

**Checking Effective Capabilities**

You can check the effective capabilities of a running process using the `capsh` command:

```bash
capsh --print
```



#### Linux Capabilities for Privilege Escalation

Here’s a list of Linux capabilities that can be leveraged for privilege escalation (priv esc) if not used correctly. Misconfigurations or overly permissive settings can lead to security vulnerabilities:

1. **CAP\_CHOWN**
   * Allows changing file ownership. If a binary with this capability is compromised, an attacker can take ownership of sensitive files.
2. **CAP\_DAC\_OVERRIDE**
   * Bypasses file read, write, and execute permission checks. This capability allows access to files that are normally restricted, potentially exposing sensitive data.
3. **CAP\_DAC\_READ\_SEARCH**
   * Bypasses file read permission checks. This can be exploited to read files that should otherwise be inaccessible.
4. **CAP\_FOWNER**
   * Bypasses permission checks on operations that normally require the file's owner. An attacker can manipulate files without being the owner, leading to unauthorized access.
5. **CAP\_NET\_ADMIN**
   * Grants the ability to perform network-related operations, such as modifying network interfaces and routing tables. Misuse can lead to network manipulation and privilege escalation.
6. **CAP\_NET\_BIND\_SERVICE**
   * Allows binding to network ports below 1024. If a service running with this capability is vulnerable, it can allow an attacker to hijack the service.
7. **CAP\_SYS\_ADMIN**
   * This broad capability allows performing a variety of administrative tasks. Improper use can lead to severe privilege escalation, as it provides control over many system functions.
8. **CAP\_SYS\_MODULE**
   * Permits loading and unloading kernel modules. Exploiting this capability can allow attackers to load malicious modules into the kernel.
9. **CAP\_SYS\_RAWIO**
   * Grants access to raw I/O operations, which can be abused to perform arbitrary read/write operations on devices.
10. **CAP\_SYS\_TIME**
    * Allows modification of the system clock. Changing the system time can be used to manipulate logs and other time-sensitive data, hiding malicious activities.





If there is harmful capabilities set on a binary, we can use this capability to escalate privilege\
For example, if `CAP_SETUID` capability is set then this can be used as a backdoor to maintain privileged accss by manipulating its own process UID

```
cp $(which python) .
sudo setcap cap_setuid+ep python

./python -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

&#x20;Refer the og site [https://gtfobins.github.io/#+capabilities](https://gtfobins.github.io/#+capabilities) for more.

