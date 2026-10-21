# Network Traffic Sniffing, Packet Analysis & HTTP Protocol Inspection

## 📌 Project Overview
This project demonstrates end-to-end network traffic sniffing, packet analysis, and credential extraction using **tcpdump** and **Wireshark** in a Linux environment. By capturing network streams between a client browser and a containerized vulnerable application (**DVWA** hosted on **Docker**), this project showcases how unencrypted HTTP protocols expose authentication credentials and session management cookies over network segments.

---

## 🛠️ Tools & Technologies
* **Operating System:** Kali Linux
* **Packet Capture Tools:** `tcpdump`, Wireshark
* **Target Application:** DVWA (Damn Vulnerable Web Application)
* **Containerization:** Docker
* **Network Inspection Utilities:** `ifconfig`, `ip route`, `pwd`

---

## 🚀 Execution Workflow & Methodology

### 1. Host Environment & Interface Verification
* Verified system environment and directory paths for packet capture storage (`pwd`).
* Identified physical and virtual network interfaces using `ifconfig` and routing tables via `ip route` (`eth0`, `lo`, `docker0` / bridge interfaces).

### 2. Application Deployment
Deployed the DVWA container environment locally mapped on HTTP port `80` using Docker:
```bash
sudo systemctl start docker
sudo docker run --rm -it -p 80:80 vulnerables/web-dvwa
3. CLI Packet Capture with tcpdump

Initiated a raw network packet capture on the active interface using tcpdump and saved the output stream directly into a .pcap file for offline analysis:
sudo tcpdump -i eth0 -s 0 -w packetdump.pcap
4. Deep Packet Inspection with Wireshark

Loaded packetdump.pcap into Wireshark and performed granular protocol analysis:

    DNS Inspection: Analyzed domain name resolution requests and responses.

    HTTP POST Credential Capture: Isolated POST request methods (http.request.method == "POST") and expanded the HTML Form URL Encoded payload tree to inspect plain-text authentication tokens (username and password).

    Session Cookie Tracking: Located HTTP responses containing 302 Found status codes to extract Set-Cookie headers and analyze session hijacking vectors.

🔍 Key Findings

    Plaintext Credential Exposure: Unencrypted HTTP traffic allows passive eavesdroppers to inspect authentication payloads directly within the packet detail view.

    Session Hijacking Vectors: Inspected cleartext session cookies transferred during browser-server interactions, emphasizing the risk of session interception.

🛡️ Remediation Strategies

    Enforce HTTPS (TLS/SSL encryption) across all web interfaces to prevent plain-text credential interception.

    Implement secure cookie attributes (Secure and HttpOnly flags) to protect session tokens against network sniffing and script-based exposure.

    Deploy HSTS (HTTP Strict Transport Security) to prohibit protocol downgrade attacks.
