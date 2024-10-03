# Local Port Forwarding

Local port forwarding is a technique used to forward traffic from a local port on the attacker's machine to a specified remote IP address and port through an intermediary (usually an SSH server). This method allows access to services on the remote server that might not be directly accessible due to firewall rules or network restrictions



## Setup

```
dollarboysushil@kali$ ssh -L 4567:localhost:80 ubuntu@10.10.15.130
ubuntu@10.10.15.130's password:
```

`-L` indicates local port forwarding, and we are forwarding the web app running on port 80  of remote server to port 4567 on our attacker machine.\
Which means, website on attackers\_ip (10.10.15.130:80) can be accessed on our\_ip:4567

Once the SSH session is established, any request sent to `http://localhost:4567` on your attacker machine will be securely forwarded to `http://localhost:80` on the remote server (`10.10.15.130`).



### To forward multiple ports

```
dollarboysushil@kali$ ssh -L 4567:localhost:80 -L 6789:localhost:3306 ubuntu@10.10.15.130
```

