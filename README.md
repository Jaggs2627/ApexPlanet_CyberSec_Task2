### Step 1: Reconnaissance 
### Part A: Passive Reconnaissance
Define and explain how a security analyst uses the following open-source intelligence (OSINT) tools without interacting with the target network:

WHOIS Lookup: Explain that this queries public registries to find domain ownership, registration dates, name servers, and administrative contact emails.

NSLookup: Explain that this queries DNS servers to resolve domain names into IP addresses and uncover specific records (like MX records for mail servers).

Google Dorking: Provide examples of advanced search operators used to find leaked configurations or sensitive files (e.g., site:target.com filetype:sql).

Shodan: Explain that this is a search engine for internet-connected devices that gathers banner data from public-facing ports, mapping out vulnerable infrastructure without hitting them directly.

### Part B: Active Reconnaissance
Explain the conceptual difference when you transition from passive gathering to active targeting:

Ping Sweep: Document how an analyst sends ICMP Echo Request packets across an entire subnet range (e.g., 10.0.2.1 to 10.0.2.254) to see which host machines are online and responding.

Banner Grabbing: Document how an analyst establishes a basic connection to an open port (like Port 21 or 80) to read the text string sent back by the service, exposing the exact software name and version number.

---

## Step 2: Port & Service Scanning Report

### Target IP: 10.0.2.3
### Detected OS: Linux 2.6.X (Ubuntu)

| Port | Protocol | State | Service | Software Version |
| :--- | :--- | :--- | :--- | :--- |
| 21 | TCP | Open | ftp | vsftpd 2.3.4 |
| 22 | TCP | Open | ssh | OpenSSH 4.7p1 Debian 8ubuntu1 |
| 23 | TCP | Open | telnet | Linux telnetd |
| 25 | TCP | Open | smtp | Postfix smtpd |
| 80 | TCP | Open | http | Apache httpd 2.2.8 ((Ubuntu) DAV/2) |
| 139 | TCP | Open | netbios-ssn | Samba smbd 3.X - 4.X |
| 445 | TCP | Open | netbios-ssn | Samba smbd 3.X - 4.X |
| 3306 | TCP | Open | mysql | MySQL 5.0.51a-3ubuntu5 |
| 5432 | TCP | Open | postgresql | PostgreSQL DB 8.3.0 - 8.3.7 |
| 8180 | TCP | Open | http | Apache Tomcat/Coyote JSP engine 1.1 |

---

## Step 3: Vulnerability Scanning & Risk Analysis

Using Nikto, an automated web application vulnerability scanner, a comprehensive security audit was executed against the target web server on `http://10.0.2.3`. Below is the risk classification based on the enterprise severity matrix:

### CRITICAL VULNERABILITIES
* **Exposed Administrative Interfaces (/phpMyAdmin/)**
  * **Finding:** The database management interface `phpMyAdmin` was discovered completely exposed without network restrictions.
  * **Risk:** Attackers can attempt brute-force credential attacks directly against the root database administrator portal to compromise backend storage entirely.

### HIGH VULNERABILITIES
* **Directory Indexing / Browsable Folders (`/doc/`, `/test/`)**
  * **Finding:** Directory indexing is globally enabled (e.g., `CVE-1999-0678`).
  * **Risk:** Anyone can browse the raw folder architecture via their browser, exposing source code files, backups, or internal test configurations.

### MEDIUM VULNERABILITIES
* **Information Disclosure via Scripting (`/phpinfo.php`)**
  * **Finding:** The active server configuration function `phpinfo()` is publicly accessible (`CWE-552`).
  * **Risk:** This leaks crucial target intelligence including environmental variables, compilation paths, module versions, and underlying kernel architecture details.
* **HTTP TRACE Method Enabled**
  * **Finding:** The server replies to HTTP TRACE requests, indicating vulnerability to Cross-Site Tracing (XST).
  * **Risk:** Can allow attackers to intercept and steal session tokens or sensitive cookies moving through client browsers.

### LOW / INFORMATIONAL VULNERABILITIES
* **Missing Security Headers (`X-Frame-Options`, `X-Content-Type-Options`)**
  * **Finding:** Missing modern defensive browser security configurations.
  * **Risk:** Leaves web clients exposed to Clickjacking attacks or MIME-sniffing exploits.

---

## Step 4: Packet Analysis with Wireshark

A live packet capture was conducted on interface `eth0` to monitor unencrypted credentials and analyze network-layer denial-of-service anomalies.

### 1. Cleartext Credential Extraction (FTP Sniffing)
* **Methodology:** An interactive session was established with the target system via standard FTP (`port 21`). 
* **Analysis:** Because FTP is an unencrypted legacy protocol, all transmission occurs in cleartext. By filtering the packet stream for the `ftp` protocol, the authentication handshake was intercepted in plain text, exposing explicit user credentials:
  * **Captured Username:** `msfadmin`
  * **Captured Password:** `msfadmin`

### 2. DDoS Simulation Analysis (SYN Flood Attack)
* **Methodology:** A high-velocity synchronization flood was simulated against the target web server using `hping3` with the following flags: `sudo hping3 -S -p 80 --flood --rand-source`.
* **Analysis:** The network interface captured a massive influx of traffic (approx. 297,000+ packets). 
* **Wireshark Filter Applied:** `tcp.flags.syn == 1 and tcp.flags.ack == 0`
* **Findings:** The filter isolated thousands of inbound connection requests where the SYN flag is raised but no corresponding Acknowledgment (ACK) packet exists. Because the source IPs were dynamically randomized/spoofed by our tool, the target server was forced to keep thousands of half-open TCP connections in its memory allocation tables, exhausting its system resources and preventing legitimate users from accessing the service.

---

## Step 5: Firewall Basics with iptables

To defend the target infrastructure against the active reconnaissance mapping and denial-of-service threats analyzed in previous steps, network-layer packet filtering was configured using the native Linux kernel utility `iptables`.

### 1. Defensive Policy Implementation
The following host-based packet filtering rules were appended to the `INPUT` chain:
* **Rule 1 (HTTP Mitigation):** `sudo iptables -A INPUT -p tcp --dport 80 -j DROP`
  * **Objective:** Drops all inbound TCP handshakes targeting the web server (Port 80/www). This effectively makes the web server invisible to automated version scans and drops malicious flood streams before processing.
* **Rule 2 (Reconnaissance Mitigation):** `sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP`
  * **Objective:** Drops inbound ICMP Echo Requests. This prevents attackers from discovering whether the machine is active during an automated subnet network sweep.

### 2. Active Policy Verification
The rules table was verified using `sudo iptables -L -v`, confirming both defensive chains are actively monitoring the interfaces with empty hit counters, ready to discard unauthorized ingress packets.
