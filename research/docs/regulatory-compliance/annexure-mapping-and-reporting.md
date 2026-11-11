# CERT-In Annexure-I Categories & Reporting Schemas

This dossier maps the 20 mandatory cybersecurity incident categories defined in Annexure-I of the CERT-In Directions 2022 to network telemetry indicators, and specifies the formal Annexure-II reporting structure.

---

## 1. Annexure-I Category Mapping Matrix

Under the 28 April 2022 Directions, ISPs must report incidents falling under the following 20 categories within 6 hours. Below is the direct mapping to network detection primitives:

| Category ID | Annexure-I Mandatory Incident Type | Network Flow / DPI Telemetry Indicator |
| :--- | :--- | :--- |
| **CAT-01** | Targeted scanning/probing of critical networks and systems | Mass outbound TCP SYN scans (>100 dst/sec) to external subnets; Mirai `scanner.c` profile. |
| **CAT-02** | Compromise of critical systems/information | Unauthorized root shell access on BNG/router; administrative credential brute-forcing. |
| **CAT-03** | Unauthorized access of IT systems/data | NetFlow flow records indicating access to restricted management subnets from subscriber IP. |
| **CAT-04** | Defacement of website or intrusion into a website | Web application firewall (WAF) HTTP alerts; unauthorized HTTP PUT/POST bursts. |
| **CAT-05** | Malicious code attacks such as spreading of virus/worm/trojan | High-confidence Suricata signature matches (Mirai, Mozi, Gafgyt payload transfers). |
| **CAT-06** | Attack on servers such as Database, Mail and DNS | Outbound DNS query floods, SMTP spam runs (TCP/25 floods), SQL injection patterns. |
| **CAT-07** | Identity Theft, spoofing and phishing attacks | Outbound traffic matching phishing domain lists; residential proxy relaying spoofed headers. |
| **CAT-08** | Denial of Service (DoS) and Distributed Denial of Service (DDoS) | High-volume outbound UDP floods (DNS/NTP/SSDP reflection), SYN floods > 50k pps. |
| **CAT-09** | Attacks on Critical Information Infrastructure (CII) | Outbound connection attempts originating from subscriber IP toward NCIIPC-designated IP ranges. |
| **CAT-10** | Attacks on Fake Mobile Apps | Malicious app C2 beaconing signatures; reverse-engineered app protocol headers. |
| **CAT-11** | Unauthorized access to social media accounts | High-rate residential proxy connection bursts toward login endpoints of major platforms. |
| **CAT-12** | Security incidents affecting Cloud computing systems | Unauthorized API credential harvesting; outbound connections to cloud metadata endpoints (`169.254.169.254`). |
| **CAT-13** | Incidents affecting Big Data, Blockchain, Virtual Assets | Monero (XMR) Stratum mining protocol headers (`mining.subscribe`, `mining.authorize`). |
| **CAT-14** | Incidents affecting Digital Payment Systems | Man-in-the-middle / DNS spoofing attempts targeting banking and UPI payment gateways. |
| **CAT-15** | Incidents affecting Satellite base stations | Unauthorized telecommand protocols or illegal satellite gateway downlinks. |
| **CAT-16** | Attacks on Internet of Things (IoT) devices | Mirai/Mozi exploit propagation against ONTs, cameras, and residential smart routers. |
| **CAT-17** | Attacks on Smart Grid / SCADA | Industrial protocol traffic (Modbus TCP/502, DNP3 TCP/20000, IEC 60870-5-104) on residential lines. |
| **CAT-18** | Attacks on industrial control systems (ICS) | Outbound exploitation attempts targeting programmable logic controllers (PLCs). |
| **CAT-19** | Ransomware attacks | Mass external traffic to known Ransomware payment domains and Tor exit gateways. |
| **CAT-20** | Data Breach / Data Leak | Multi-gigabyte uncharacteristic outbound bulk data exfiltration over encrypted tunnels. |

---

## 2. Annexure-II Incident Reporting Data Schema

CERT-In mandates specific factual disclosures in the official Incident Reporting Form (Annexure-II). A standards-compliant system automatically structures the incident dossier according to this schema:

```
CERT-In Annexure-II Mandatory Structure:
1. Contact Information:
   - Name of Organization / ISP
   - Designated Incident Officer / CISO Name
   - Contact Phone & Registered 24x7 Email Address

2. Incident Characterization:
   - Annexure-I Category ID (CAT-01 to CAT-20)
   - Date and Exact Time of Incident Discovery (NPLI-synchronized UTC+05:30)
   - Duration and Ongoing Status

3. Technical Telemetry Details:
   - Affected Subscriber Identifier (KYC/Account ID)
   - Pre-NAT Private IPv4 / IPv6 Address
   - Carrier-Grade NAT Translated Public IPv4 + Port Range
   - Terminating BNG Identifier & Physical NAS-Port
   - Target Victim Destination IP / Port
   - Protocol & Attack Volume (Packets/Sec, Megabits/Sec)

4. Forensic Artifacts:
   - Cryptographic SHA-256 Hash of PCAP / Flow Ledger
   - Merkle Tree Root Hash
   - Statutory Section 63 BSA Evidence Certificate Reference
```
