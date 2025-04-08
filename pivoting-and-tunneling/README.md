# Pivoting & Tunneling

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Pivoting

Pivoting refers to the method of using one compromised machine to access and attack other machines on the same network that are not directly accessible from the attacker's machine. It enables attackers to expand their control beyond the initial foothold.

### Tunneling

Tunneling is the process of encapsulating one network protocol within another. This is often used to bypass firewalls and other network restrictions, enabling communication between the attacker and the target machine.

Types are:&#x20;

**SSH Tunneling**: A secure method that allows you to forward ports over an encrypted SSH connection.\
**VPN Tunneling**: Establishing a secure connection to a remote network.\
**HTTP Tunneling**: Encapsulating non-HTTP traffic in HTTP requests to evade network filters.

### Port Forwarding

Port forwarding is the technique of forwarding network ports from one network node to another, enabling external users to connect to services hosted on a private network. This is commonly used in both networking and penetration testing.

Types:\
**Local Port Forwarding**: Redirecting traffic from a local port to a specified remote server and port.\
**Remote Port Forwarding**: Redirecting traffic from a remote port to a specified local server and port.\
**Dynamic Port Forwarding**: Creating a SOCKS proxy to dynamically forward connections as needed.

