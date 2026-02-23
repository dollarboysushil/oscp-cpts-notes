# Remote Port Forwarding

Remote port forwarding is a technique used to forward traffic from a port on a remote server (the SSH server) to a specified local host and port. This allows services on the attacker's local machine to be accessed from the remote machine or other machines on the remote network.

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

lets say we want to access the webserver running on port 80 of windows (172.16.1.16) machine.\
Currently we (kali) and target(windows) are not in same subnet, so we cannot make connection.

Lets access webserver running on port 80 on windows from kali.

#### Steps to Achieve Remote Port Forwarding

```
ssh -R 8080:172.16.1.16:80 ubuntu@10.10.15.130
```

here, `-R` flag is used to specify remote port forwarding. `8080` is the port on the pivot host `10.10.15.130` that will listen for incoming connections

**`172.16.1.16:80`**: This is the target IP and port where the web server is running (the Windows machine).



After SSH connection is successfully established, we can access the webserver from our local machine by navigating to

```
http://10.10.15.130:8080
```

#### Diagram Representation

To visualize the setup:

* **Kali Machine** (`10.10.15.128`) connects to the **Pivot Host** (`10.10.15.130`).
* The **Pivot Host** listens on port `8080`.
* Traffic sent to `8080` on the **Pivot Host** is forwarded to `172.16.1.16:80`.
