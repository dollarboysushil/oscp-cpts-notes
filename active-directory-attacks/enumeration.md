# Enumeration

## External Reconnaissance

External reconnaissance involves gathering information from public sources to identify the target’s network footprint and domain details.

**Tools for External Recon**

1. [**BGP Toolkit (https://bgp.he.net/)**](https://bgp.he.net/)
   * **Purpose:** Finds IP address ranges, ASNs, and BGP routing information.
   * **Use Case:** Search for the organization's ASN or name to discover registered IP ranges and subnets.
2. [**Whois Lookup (https://whois.domaintools.com/)**](https://whois.domaintools.com/)
   * **Purpose:** Retrieves domain registration details (owner, registrar, contact info).
   * **Use Case:** Provides domain ownership history, useful for identifying related domains and administrative contacts.
3. [**ViewDNS.info (https://viewdns.info/)**](https://viewdns.info/)
   * **Purpose:** Offers multiple tools like DNS records, reverse DNS, and reverse IP lookups.
   * **Use Case:** Finds hosted domains on an IP, DNS configurations, and verifies global DNS propagation.
4. [**Shodan (https://www.shodan.io/)**](https://www.shodan.io/)
   * **Purpose:** Search engine for internet-connected devices and services.
   * **Use Case:** Finds open ports, services, and vulnerabilities associated with the target’s IP addresses.
5. [**Censys (https://censys.io/)**](https://censys.io/)
   * **Purpose:** Internet search engine for enumerating devices and services.
   * **Use Case:** Gathers information on SSL certificates, web servers, and open ports.
6. [**DNSDumpster (https://dnsdumpster.com/)**](https://dnsdumpster.com/)
   * **Purpose:** DNS recon tool for mapping domain infrastructure.
   * **Use Case:** Identifies subdomains, MX records, and hosts for domain mapping.
7. [**Spyse (https://spyse.com/)**](https://spyse.com/)
   * **Purpose:** Provides information on domains, IP addresses, SSL certificates, and technologies used.
   * **Use Case:** Useful for comprehensive data collection on an organization’s internet footprint.
8. [**Netcraft (https://www.netcraft.com/)**](https://www.netcraft.com/)
   * **Purpose:** Website reconnaissance tool for discovering hosting details and technologies.
   * **Use Case:** Checks site history, IP addresses, and hosting providers.

These tools assist in passive information gathering, which helps in understanding the target’s external infrastructure without alerting them to your activities.

## Internal Reconnaissance

Internal reconnaissance involves gathering information about the target's internal network once access is gained. The goal is to map the network, identify high-value targets, and discover potential vulnerabilities.

When performing internal reconnaissance, the initial goal is to identify active hosts, map the network, and gather basic information about the environment.

**1. Ping Sweep using fping**

* **Purpose:** Identifies active hosts on a network by sending ICMP echo requests.
*   **Command Example:**

    ```bash
    fping -a -g 192.168.1.0/24 2>/dev/null
    ```
* **Explanation:**
  * `-a` shows only live hosts.
  * `-g` generates a list of IPs for the specified subnet.
  * This command scans the entire subnet to find active devices.

**2. DNS Resolution using nslookup**

* **Purpose:** Queries DNS servers for domain name information, translating hostnames to IP addresses.
*   **Command Example:**

    ```bash
    nslookup <hostname>
    ```
* **Use Cases:**
  * Translate a hostname to its IP address.
  * Find the DNS server information for the domain.
* **Tip:** Can also perform reverse lookups with an IP address to find associated hostnames.

**3. Nmap Scan on All Hosts**

* **Purpose:** Conducts network scanning to identify open ports, services, and operating systems.
*   **Basic Command Example:**

    ```bash
    nmap -sP 192.168.1.0/24
    ```
* **Explanation:**
  * `-sP` performs a ping scan to list active hosts.
*   **Full Port Scan Example:**

    ```bash
    nmap -p- 192.168.1.1
    ```
* **Explanation:**
  * `-p-` scans all 65,535 ports on a target host to discover open services.

**Practical Workflow**

1. **Start with a Ping Sweep:** Use `fping` to identify active hosts on the network.
2. **Perform DNS Lookups:** Use `nslookup` to resolve hostnames and gather DNS information.
3. **Use Nmap for Port Scanning:** Run Nmap scans on identified hosts to map open ports and services.
