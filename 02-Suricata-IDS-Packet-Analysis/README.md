# Project 02: Network Intrusion Detection System (NIDS) Implementation & Packet Analysis

## Executive Summary
This project demonstrates the deployment, configuration, and validation of **Suricata Intrusion Detection System (IDS)** alongside **Wireshark** for live network monitoring and deep packet inspection (DPI). Operating within a virtualized enterprise subnet (`192.168.3.0/24`), custom signature rules were engineered to detect reconnaissance sweeps, aggressive TCP port scans, and web-tier application attack vectors in real time.

---

## Network Architecture & Lab Topology

| Component | Role | OS / Platform | IP Address |
| :--- | :--- | :--- | :--- |
| **Defensive Host / Sensor** | NIDS & Packet Analyzer | Ubuntu Desktop 64-bit | `192.168.3.129` |
| **Adversary Host** | Traffic Generator / Attacker | Kali Linux | `192.168.3.132` |
| **Network Hypervisor** | Virtual Switch / NAT Gateway | VMware Workstation Pro | `192.168.3.0/24` |

---

## Phase 1: Custom IDS Signature Engineering

Three tailored detection signatures were authored in `/etc/suricata/rules/local.rules` to capture reconnaissance, active port scans, and Layer 7 directory traversal attacks:

```text
# 1. ICMP Ping Sweep / Echo Probe Detection
alert icmp any any -> $HOME_NET any (msg:"SCAN ICMP Ping Sweep / Probe Detected"; itype:8; sid:1000001; rev:1;)

# 2. Nmap TCP SYN Port Scan Attempt
alert tcp any any -> $HOME_NET any (msg:"SCAN Nmap TCP SYN Port Scan Attempt"; flags:S; threshold:type threshold, track by_src, count 5, seconds 2; sid:1000002; rev:1;)

# 3. HTTP Suspicious Directory Traversal Probe
alert tcp any any -> $HOME_NET 80 (msg:"ATTACK Possible Directory Traversal /etc/passwd Probe"; content:"/etc/passwd"; nocase; sid:1000003; rev:1;)
Phase 2: Threat Simulation & Adversary Probing
From the adversary station (192.168.3.132), synthetic network probes were launched sequentially against the monitored host:

ICMP Discovery: Standard echo requests targeting the host gateway.

Stealth Port Scan: nmap -sS -p 20-30 transmitting rapid TCP SYN packets across targeted port ranges.

Application Exploit Payload: An HTTP GET parameter injection attempting an /etc/passwd local file inclusion (LFI) traversal.

Phase 3: Real-Time Detection & Alert Generation
Suricata monitored active network interface ens33 in live passive capture mode. As packets traversed the interface, signatures evaluated the headers and application payloads, writing alert records directly to /var/log/suricata/fast.log:

Incident Log Breakdown:
SID 1000001: Flagged Layer 3 ICMP echo requests originating from 192.168.3.132.

SID 1000002: Rate-limiting threshold detected rapid SYN packet bursts within a 2-second sampling window.

SID 1000003: Deep packet inspection parsed incoming HTTP streams on port 80 and matched the payload substring /etc/passwd.

Phase 4: Deep Packet Inspection & Dissection (Wireshark)
To validate the alert correlation, Wireshark was deployed to dissect the Layer 7 HTTP transaction:

Frame Analysis: Packet capture on ens33 verified source 192.168.3.132 connecting to destination 192.168.3.129:80.

Hex/ASCII Dissection: Packet data at offset 0x0040 verified the exact URI structure: GET /index.html?file=/etc/passwd HTTP/1.1.

Security Takeaways & Hardening Insights
Signature-Based Defense: Suricata's multi-threaded rule engine offers real-time visibility into active network scanning and perimeter probing.

Threshold Tuning: Configuring threshold:type threshold suppresses alert fatigue while maintaining high-fidelity detection against port-scanning utilities.

Defense-in-Depth: Combining perimeter IDS alerts with raw packet capture (PCAP) allows security analysts to perform root-cause verification during incident response triage.


4. Scroll down to **Commit changes**, enter `Add Project 02 documentation`, and click **Commit changes**.

---

**Step 3: Update the Main Repository README**

1. Go back to your main repo landing page (`Cybersecurity-Portfolio`).
2. Click the pencil icon on `README.md` to edit.
3. Add **Project 02** to your project directory table so visitors can navigate right into it:

```markdown
## Projects Portfolio

| Project | Description | Tech Stack |
| :--- | :--- | :--- |
| [01-OPNsense-Enterprise-Segmentation](./01-OPNsense-Enterprise-Segmentation) | Enterprise firewall architecture with segmented DMZ, Management, and Internal LAN subnets. | OPNsense, VMware, Networking |
| [02-Suricata-IDS-Packet-Analysis](./02-Suricata-IDS-Packet-Analysis) | Network Intrusion Detection
