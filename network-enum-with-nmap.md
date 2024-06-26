# Network Enum with Nmap

the TCP-SYN scan (`-sS`) The TCP-SYN scan sends one packet with the SYN flag and, therefore, never completes the three-way handshake, which results in not establishing a full TCP connection to the scanned port.

| Nmap Option        | Description                                                            |
| ------------------ | ---------------------------------------------------------------------- |
| 10.10.10.0/24      | Target network range.                                                  |
| -sn                | Disables port scanning.                                                |
| -Pn                | Disables ICMP Echo Requests                                            |
| -n                 | Disables DNS Resolution.                                               |
| -PE                | Performs the ping scan by using ICMP Echo Requests against the target. |
| --packet-trace     | Shows all packets sent and received.                                   |
| --reason           | Displays the reason for a specific result.                             |
| --disable-arp-ping | Disables ARP Ping Requests.                                            |
| --top-ports=\<num> | Scans the specified top ports that have been defined as most frequent. |
| -p-                | Scan all ports.                                                        |
| -p22-110           | Scan all ports between 22 and 110.                                     |
| -p22,25            | Scans only the specified ports 22 and 25.                              |
| -F                 | Scans top 100 ports.                                                   |
| -sS                | Performs an TCP SYN-Scan.                                              |
| -sA                | Performs an TCP ACK-Scan.                                              |
| -sU                | Performs an UDP Scan.                                                  |
| -sV                | Scans the discovered services for their versions.                      |
| -sC                | Perform a Script Scan with scripts that are categorized as "default".  |
| --script \<script> | Performs a Script Scan by using the specified scripts.                 |
| -O                 | Performs an OS Detection Scan to determine the OS of the target.       |
| -A                 | Performs OS Detection, Service Detection, and traceroute scans.        |
| -D RND:5           | Sets the number of random Decoys that will be used to scan the target. |
| -e                 | Specifies the network interface that is used for the scan.             |
| -S 10.10.10.200    | Specifies the source IP address for the scan.                          |
| -g                 | Specifies the source port for the scan.                                |
| --dns-server \<ns> | DNS resolution is performed by using a specified name server           |



| State      | Description                                                                                                                                                                                           |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| open       | This indicates that the connection to the scanned port has been established. These connections can be TCP connections, UDP datagrams as well as SCTP associations.                                    |
| closed     | When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an RST flag. This scanning method can also be used to determine if our target is alive or not. |
| filtered   | Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we get an error code from the target.                |
| unfiltered | This state of a port only occurs during the TCP-ACK scan and means that the port is accessible, but it cannot be determined whether it is open or closed.                                             |
| open       | filtered                                                                                                                                                                                              |
| closed     | filtered                                                                                                                                                                                              |

