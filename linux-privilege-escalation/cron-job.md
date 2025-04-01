# Cron Job

**Cron jobs** are scheduled tasks in Unix-like operating systems that automatically execute commands or scripts at specified intervals or times. Managed by the cron daemon, these tasks can be set to run daily, weekly, monthly, or at specific times, allowing for automation of repetitive tasks such as backups, updates, and system maintenance.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can verify whether a cron job is active by using pspy, a command-line utility that allows us to observe running processes without requiring root access. This tool enables us to monitor commands executed by other users, including cron jobs. It operates by scanning the procfs filesystem. To use pspy, we can execute the following command

```bash
dollarboysushil@kali $./pspy64 
```

pspy github page; [https://github.com/DominicBreuker/pspy](https://github.com/DominicBreuker/pspy)



## Identifying Cron Jobs

*   To view user-specific cron jobs, use the command:

    ```bash
    crontab -l
    ```
* To view system-wide cron jobs, check files in `/etc/crontab`, `/etc/cron.d/`, and `/var/spool/cron/crontabs/`.



## Exploiting Cron Jobs:

* **Modifying Scripts**: If you find a world-writable script executed by a cron job, you can modify it to execute arbitrary commands.
* **Creating a Malicious Script**: If the cron job runs a specific script, you could create a script with the same name and place it in a directory that is executed before the legitimate script.
* **Race Conditions**: Exploit race conditions by quickly replacing a script while the cron job is running.
* **Exploiting Environment Variables**:Cron jobs may rely on environment variables that are not properly set or sanitized. An attacker can manipulate these variables in the job's environment to alter the behavior of the command being executed.
* **Path Manipulation**:If a cron job uses commands that are not specified with full paths, an attacker can exploit the `PATH` environment variable. By placing malicious executables in a directory that appears earlier in the `PATH`, the attacker can cause the cron job to execute the malicious version instead of the intended command.
* **Symlink Attacks**:If a cron job writes output to a file, an attacker can create a symlink from that file to a sensitive file or a script that they control. When the cron job runs, it may overwrite the symlink target, leading to privilege escalation or data loss.
* **Utilizing Default Shell Behavior**:Certain shell features can be exploited if cron jobs are running with a shell that has specific behaviors (like `bash`). For example, an attacker might use command substitution or other constructs in a script to execute arbitrary commands.
* **Adding to the Crontab**:If a user with sufficient privileges has a cron job configured to allow others to add entries (e.g., using `crontab -e`), an attacker can insert their own commands to execute arbitrary code at scheduled intervals.
* **Exploiting Missing User Permissions**:If a cron job runs as a user with elevated privileges and doesn’t restrict which scripts or binaries it executes, an attacker can create or modify files that the cron job interacts with, leading to potential privilege escalation.
* **Manipulating Command Output**:If a cron job’s output is logged to a file that is writable by all users, an attacker can manipulate this file or replace it with a malicious one to control the output of the job and potentially execute arbitrary code.
* **Job Timing and Timing Attacks**:Understanding the timing of cron jobs can allow an attacker to launch a timing attack. By knowing when a cron job runs, an attacker can exploit any race conditions or manipulate scripts just before execution.
* **Local File Inclusion (LFI)**:If a cron job includes or requires files without properly validating the input, an attacker could exploit this to include malicious files that execute when the job runs.
