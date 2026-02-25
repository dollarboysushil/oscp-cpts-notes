# Dynamic Port Forwarding

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



Dynamic port forwarding is a technique used in SSH that allows us to create a SOCKS proxy server. This enables us to route traffic through the SSH connection dynamically to any port on the remote server or through any other hosts accessible from that remote server.

* **Setup**: When we establish a dynamic port forwarding session, an SSH client listens on a specified local port and forwards traffic to the remote server, allowing connections to any host and port through the SSH tunnel.
* **Traffic Flow**:
  * Any application that supports SOCKS proxy (like web browsers, curl, etc.) can connect to the local port.
  * The SSH server will route this traffic through the established SSH connection to the desired destination.

## Example

```bash
ssh -D [local_port] [user]@[remote_server]
```

`-D` flag indicates we want to create a SOCKS proxy

```bash
ssh -D 1080 user@10.10.15.130
```

In this example:

* The SSH client will listen on `localhost:1080`.
* Any traffic directed to this port will be forwarded through the SSH tunnel to the remote server and then on to the final destination.

We must edit  `/etc/proxychains.conf` file to inform proxychains that we must use port 1080.\
add this into conf file

`socks4 127.0.0.1 1080`



## Using Proxychains to Access the Web Server

Once Proxychains is configured, you can use it to route your requests through the SOCKS proxy. Given the diagram with the setup:

* **Attacker (Kali)**: `10.10.15.128`
* **Pivot Host (Ubuntu)**: `10.10.15.130`
* **Target (Windows)**: `172.16.1.16` (running a web server on port `80`)

To access the web server on the Windows target through Proxychains, use the following command:

```bash
proxychains curl http://172.16.1.16
```

or we can use proxychains with metasplit

```bash
proxychains msfconsole
```

