# Telemetry Mechanisms: NetFlow v9 & IPFIX

This dossier analyzes flow telemetry export standards, Information Elements, and sampling economics across regional ISP core transit links.

## 1. Protocol Evolution: NetFlow v5 vs. v9 vs. IPFIX (RFC 7011)

- **NetFlow v5:** Legacy fixed 7-field format. Lacks IPv6, MPLS, variable-length fields, and vendor enterprise IDs. Obsolete in modern carrier networks.
- **NetFlow v9 (RFC 3954):** Dynamic template-based format developed by Cisco. Decouples field definitions from record structures.
- **IPFIX (RFC 7011):** The official IETF successor to NetFlow v9. Standardizes template encoding, variable-length Information Elements (IE), enterprise-specific information elements (PEN), and native Carrier-Grade NAT event reporting (RFC 8158).

---

## 2. Critical Information Elements for Compromise Detection

| IPFIX Element ID | Canonical Field Name | Telemetry Purpose in Compromise Detection |
| :--- | :--- | :--- |
| **8** | `sourceIPv4Address` | Originating subscriber private IPv4 address (pre-NAT). |
| **12** | `destinationIPv4Address` | Target victim IP or external botnet C2 destination. |
| **7** | `sourceTransportPort` | Ephemeral client source port. |
| **11** | `destinationTransportPort` | Target service port (Telnet 23, CWMP 7547, DNS 53, etc.). |
| **4** | `protocolIdentifier` | Layer 4 protocol (TCP=6, UDP=17, GRE=47). |
| **6** | `tcpControlBits` | Cumulative TCP flags observed across session lifetime (SYN, ACK, RST, FIN). |
| **1** | `octetDeltaCount` | Total octets transferred across the unidirectional flow. |
| **2** | `packetDeltaCount` | Total packets transferred across the unidirectional flow. |
| **152** | `flowStartMilliseconds` | Microsecond-accurate session initiation timestamp. |
| **153** | `flowEndMilliseconds` | Microsecond-accurate session termination timestamp. |

---

## 3. Sampling Rates and Detection Degradation Mathematics

Carriers frequently apply packet sampling (e.g., 1:1000) on core transit interfaces to prevent router CPU exhaustion. In subscriber compromise detection, this introduces catastrophic blind spots:

### Mathematical Impact on Low-Volume C2 Beaconing:
Let an infected IoT device emit a stealthy Command and Control (C2) keepalive beacon consisting of $n=1$ packet every 60 seconds.
Under a sampling rate of $R = 1/1000$:
The probability of detecting the beacon in a given 60-second window is:
$$P(	ext{Detect}) = rac{1}{1000} = 0.001 \quad (0.1\%)$$
The probability that the C2 beacon goes completely undetected over a 24-hour observation window ($N = 1,440 	ext{ intervals}$) is:
$$P(	ext{Undetected in 24h}) = (1 - 0.001)^{1440} pprox 0.2367 \quad (23.7\%)$$
Nearly a quarter of active botnet beacons are completely invisible under 1:1000 sampling.

**Conclusion:** Reliable compromise detection mandates **1:1 unsampled flow capture on aggregation links** using high-throughput collectors like **GoFlow2**, or direct optical SPAN/TAP line-rate inspection.
