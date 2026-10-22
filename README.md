# Network Traffic Sniffing, Packet Analysis & HTTP Protocol Inspection

📌 **Project Overview**

This project demonstrates end-to-end network traffic sniffing, packet analysis, credential extraction, and session hijacking exploitation using `tcpdump` and Wireshark in a Linux environment. By capturing network streams between a client browser and a containerized vulnerable application (DVWA hosted on Docker), this project showcases how unencrypted HTTP protocols expose authentication credentials and session management cookies over network segments, enabling passive eavesdropping and session takeover attacks.

---

🛠️ **Tools & Technologies**

* **Operating System:** Kali Linux
* **Packet Capture & Analysis Tools:** `tcpdump`, Wireshark, `curl`
* **Target Application:** DVWA (Damn Vulnerable Web Application)
* **Containerization:** Docker
* **Browser & Web Tools:** Firefox Developer Tools (Storage/Cookies Manager)
* **Network Inspection Utilities:** `ifconfig`, `ip route`, `pwd`

---

🚀 **Execution Workflow & Methodology**

### 1. Host Environment & Interface Verification
* Verified system environment and directory paths for packet capture storage (`pwd`).
* Identified physical and virtual network interfaces using `ifconfig` and routing tables via `ip route` (`eth0`, `lo`, `docker0` / bridge interfaces).

### 2. Application Deployment
* Deployed the DVWA container environment locally mapped on HTTP port 80 using Docker:
  ```bash
  sudo systemctl start docker
  sudo docker run --rm -it -p 80:80 vulnerables/web-dvwa
  ```
* Configured DVWA security settings:
  * **Security Level:** `Low`
  * **PHPIDS:** `Disabled`

### 3. CLI Packet Capture with tcpdump
* Initiated raw network packet captures on the active interface using `tcpdump` and saved output streams directly into `.pcap` files for offline analysis:
  ```bash
  sudo tcpdump -i eth0 -s 0 -w packetdump.pcap
  ```

### 4. Deep Packet Inspection with Wireshark
* Loaded `.pcap` captures into Wireshark and performed granular protocol analysis:
  * **DNS Inspection:** Analyzed domain name resolution requests and responses.
  * **HTTP POST Credential Capture:** Isolated POST request methods (`http.request.method == "POST"`) and expanded the HTML Form URL Encoded payload tree to inspect plain-text authentication tokens (`username` and `password`).
  * **Session Cookie Tracking:** Located HTTP response frames containing `302 Found` status codes (Packet 6) to extract cleartext `Set-Cookie` headers:
    ```http
    Set-Cookie: PHPSESSID=8p11r43coac3lcsb6hgpr10uo3; path=/
    ```

### 5. Session Hijacking Exploitation & Proof of Concept (PoC)
* **Manual Cookie Injection (Browser):**
  * Opened an unauthenticated Private Browsing window navigating to `http://127.0.0.1/login.php`.
  * Accessed Developer Tools (**Storage** -> **Cookies**) and injected the stolen `PHPSESSID` value (`8p11r43coac3lcsb6hgpr10uo3`).
  * Navigated to `http://127.0.0.1/index.php` and refreshed the session.
  * Successfully bypassed login controls and gained direct administrative access to the DVWA dashboard without providing credentials.
* **CLI Session Replay (curl):**
  * Replayed the authenticated state via command-line using only the stolen session token:
    ```bash
    curl -i http://127.0.0.1/index.php -H "Cookie: PHPSESSID=8p11r43coac3lcsb6hgpr10uo3"
    ```
  * Verified that the server returned an `HTTP 200 OK` response containing the authenticated DVWA dashboard HTML content (`Welcome to Damn Vulnerable Web Application!`).

---

🔍 **Key Findings**

* **Plaintext Credential Exposure:** Unencrypted HTTP traffic allows passive eavesdroppers to inspect authentication payloads directly within the packet detail view.
* **Active Session Hijacking:** Cleartext session management cookies (`PHPSESSID`) transmitted over HTTP can be stolen and replayed by an attacker to completely takeover active user sessions without needing valid credentials.

---

🛡️ **Remediation Strategies**

* **Enforce HTTPS (TLS/SSL Encryption):** Encrypt all web traffic across all interfaces to prevent plain-text credential and cookie interception in transit.
* **Implement Secure Cookie Attributes:**
  * `Secure`: Restricts cookie transmission exclusively to encrypted HTTPS connections.
  * `HttpOnly`: Prevents client-side scripts (e.g., XSS) from accessing session tokens.
  * `SameSite=Strict` / `SameSite=Lax`: Mitigates Cross-Site Request Forgery (CSRF) vectors.
* **Session Renewal & Invalidation:** Regenerate session identifiers immediately upon privilege elevation or successful login, and implement strict server-side timeout/invalidation rules upon logout.
* **Deploy HSTS (HTTP Strict Transport Security):** Prohibit protocol downgrade attacks across network streams.


