# ARCUS Pre-Project Research Dossier

> **Foundational Engineering Intelligence, Telemetry Mathematics & Statutory Evidence Base for Autonomous Subscriber Compromise Detection & Regulatory Compliance in Regional Tier-2/3 FTTH Networks.**  
> *Developed under the **ToxArch** cybersecurity organization.*

Snapshot Date: **2026-11-11**  
Corpus Status: **Pre-Project Technical Research (Completed)**  
Scope: **Pure Explanatory Architecture, Empirical Data, Malware Disassembly & Statutory Proofs (Zero Untested Code / Zero Post-Project Claims)**

---

## 1. Executive Problem Space

Regional Tier-2 and Tier-3 Internet Service Providers (operating across Indian telecom circles) deliver broadband access to residential and small-business subscribers under unique operational and legal constraints:

1. **Carrier-Grade NAT (CGNAT) Attribution Impasse:** Due to extreme IPv4 exhaustion, hundreds or thousands of subscribers share identical public IPv4 addresses using Carrier-Grade NAT (RFC 6888 / RFC 6598). When sovereign authorities (CERT-In) or international threat feeds report an abuse tuple $\langle IP_{\text{pub}}, Port_{\text{pub}}, t_{\text{incident}} \rangle$, operators cannot determine which physical customer line originated the attack without microsecond-accurate temporal state intersection.
2. **The Dynamic NAT Logging Explosion:** Recording every per-session dynamic NAT translation for 10,000 subscribers generates **2.16 billion records per day (~172.8 GB/day)**, crashing routing engines and overwhelming storage infrastructure.
3. **The Base Rate Fallacy in Subscriber Networks:** With residential infection rates hovering at $\approx 0.5\%$, a high-accuracy detection signature (99% sensitivity, 1% false positive rate) suffers an empirical **False Discovery Rate of 66.8%**. Quarantining customers based on un-fused single-rule alerts creates massive customer churn and operational disruption.
4. **Draconian Statutory Penalties:**
   - **CERT-In Directions (28 April 2022):** Enforces a mandatory **6-hour incident reporting window** from discovery, plus **180-day domestic log retention** and **NPLI Stratum-1 clock synchronization**.
   - **Jan Vishwas Act, 2023:** Escalated penalties under Section 70B(7) of the IT Act up to **₹1 Crore (10,000,000 INR)** per failure to report, backed by executive vicarious liability under Section 85.
   - **Bharatiya Sakshya Adhiniyam, 2023 (BSA Section 63):** Effective 1 July 2024, supersedes Section 65B of the Indian Evidence Act, mandating strict cryptographic chain-of-custody, SHA-256 Merkle trees, and statutory Schedule Part A/B certificates for electronic evidence admissibility.

---

## 2. Pre-Project Architectural Foundations

The diagram below outlines the conceptual relationship between the seven research domains investigated in this corpus:

```
+---------------------------------------------------------------------------------------------------+
|                                  ACCESS LAYER TOPOLOGIES (FND-01)                                 |
|  [Residential FTTH Subscribers] ---> [OLT (GPON/EPON)] ---> [BNG/BRAS (PPPoE / IPoE Dual-Stack)] |
+---------------------------------------------------------------------------------------------------+
                                   |                                  |
               Unsampled IPFIX / NetFlow v9 (FND-02)        RADIUS radacct Telemetry (NAT-03)
                                   v                                  v
+----------------------------------------------------+   +------------------------------------------+
|            LINE-RATE DPI & FLOW SENSING            |   |          CGNAT & AAA ATTRIBUTION         |
|  - Suricata Workers Runmode + Hyperscan (DET-01)   |   |  - RFC 6888 / RFC 6598 Standards (NAT-01)|
|  - Threat Signatures: Mirai, Mozi, Proxies (THR-01)|   |  - Port Block Allocation 99.997% (NAT-02)|
|  - Target CPE Exploit Profiles (THR-04)            |   |  - Two-Tier Temporal Intersection (NAT-03|
+----------------------------------------------------+   +------------------------------------------+
                           \                                          /
                            \                                        /
                             v                                      v
+---------------------------------------------------------------------------------------------------+
|                                 DETECTION & ANALYTICS PIPELINE                                    |
|  - Additive Bayesian Log-Odds Fusion & Base Rate Fallacy Correction (DET-02)                     |
|  - Columnar Analytical Storage & Real-Time Aggregations in ClickHouse (DET-03)                    |
+---------------------------------------------------------------------------------------------------+
                                                  |
                         High Confidence Posterior P(C|E) >= 0.999
                                                  |
                    +-----------------------------+-----------------------------+
                    |                                                           |
                    v                                                           v
+---------------------------------------+   +-------------------------------------------------------+
|    CONTAINMENT & REMEDIATION (REM)    |   |               REGULATORY COMPLIANCE (REG)             |
|  - RFC 6561 Walled Garden (REM-01)    |   |  - CERT-In 6-Hour SLA Clock & Jan Vishwas (REG-01)    |
|  - RFC 8908/8910 Captive API (REM-02) |   |  - Annexure-I/II Incident Mapping Schema (REG-02)     |
|  - Safe Harbor Whitelisting (REM-03)  |   |  - Human-in-the-Loop Approval Gate Rationale (REG-03) |
|  - Verification & Probation (REM-04)  |   |  - BSA 2023 Sec 63 Electronic Evidence Proof (REG-04) |
+---------------------------------------+   +-------------------------------------------------------+
```

---

## 3. Modular Research Directory Structure

This repository is organized into focused, modular technical dossiers following rigorous evidence and citation standards:

```
research/
|-- README.md                                   # Master research overview & executive summary
`-- docs/
    |-- README.md                               # Complete document map & cross-reference table
    |-- methodology.md                          # Evidence boundaries, verification tiers & scope
    |-- foundations/                            # Network architecture & telemetry primitives
    |   |-- broadband-access-topologies.md      # BNG/BRAS, PPPoE vs IPoE, GPON optical splitters
    |   `-- telemetry-mechanisms.md             # NetFlow v9 vs IPFIX, sampling rate degradation math
    |-- cgnat-attribution/                      # Large-scale NAT & subscriber identity resolution
    |   |-- cgnat-architecture-and-standards.md # RFC 6888, RFC 6598, Dynamic vs PBA vs Deterministic
    |   |-- log-volume-mathematics.md           # 2.16B vs 60k rows/day derivation (99.997% reduction)
    |   `-- radius-accounting-and-attribution.md# RADIUS attributes, 2-tier inversion, orphaned sessions
    |-- threat-intel/                           # Binary disassembly & residential threat intelligence
    |   |-- mirai-botnet-family.md              # scanner.c LCG, 68 credentials, exclusion subnets
    |   |-- mozi-p2p-botnet.md                  # Kademlia DHT, ECDSA384 verification, CPE exploits
    |   |-- residential-proxies-and-abuse.md    # BadBox AOSP rootkits, 911 S5, Up/Down ratio inversion
    |   |-- target-cpe-vulnerabilities.md       # Syrotech/Netlink ONTs, TP-Link/D-Link, CP PLUS cameras
    |   `-- outbound-abuse-profiles.md          # Amplification factors (DNS, NTP, SSDP, Memcached), BCP 38
    |-- detection-analytics/                    # High-throughput analytics & mathematical fusion
    |   |-- suricata-line-rate-engine.md        # Workers runmode, AF_PACKET MMAP vs DPDK, Hyperscan MPM
    |   |-- bayesian-log-odds-fusion.md         # Base Rate Fallacy proof, additive log-odds, damping
    |   `-- clickhouse-columnar-storage.md      # SIMD vectorization, compression codecs, TTL policies
    |-- containment-remediation/                # Subscriber isolation, notification & safe harbor
    |   |-- rfc6561-walled-garden-framework.md  # Tripartite lifecycle, DNS RPZ vs DNAT vs BGP Flowspec
    |   |-- captive-portal-api-standards.md     # RFC 8908 & RFC 8910, DHCP Option 114, OS CNA handling
    |   |-- safe-harbor-remediation-whitelist.md# Critical OS/AV update domains, CSK, split-horizon DNS
    |   `-- verification-and-probation-ladder.md# Active synthetic scans, 30-min probation, auto-release
    |-- regulatory-compliance/                  # Sovereign compliance & electronic evidence law
    |   |-- cert-in-directions-and-sla.md       # Section 70B, 6-hr SLA countdown, Jan Vishwas penalties
    |   |-- annexure-mapping-and-reporting.md   # 20 Annexure-I categories mapped to telemetry, Annexure-II
    |   |-- human-approval-gate-rationale.md    # Legal due process, Section 85 liability, 3-tier gate
    |   `-- bsa-2023-section-63-evidence.md     # BSA Sec 63 Schedule Part A/B, Merkle chains, NPLI NTP
    `-- evidence/                               # Primary ledgers & hardware capacity baselines
        |-- standards-and-rfc-ledger.md         # Primary source ledger (RFCs, Gazette orders, CVEs)
        `-- hardware-capacity-specifications.md # Sizing matrices for 5k, 15k, and 30k subscriber tiers
```

---

## 4. Master Document Index & Navigation

| ID | Title | Summary of Key Research Findings | File Link |
| :--- | :--- | :--- | :--- |
| **META-01** | [Research Methodology](docs/methodology.md) | Evidence tiers (Tier 1 Sovereign Law down to Tier 4 Threat Intel), verification boundaries | [methodology.md](docs/methodology.md) |
| **FND-01** | [Broadband Access Topologies](docs/foundations/broadband-access-topologies.md) | MikroTik/Cisco BNGs, PPPoE cryptographic session binding vs. IPoE DAI/IPSG requirements | [broadband-access-topologies.md](docs/foundations/broadband-access-topologies.md) |
| **FND-02** | [Telemetry Mechanisms](docs/foundations/telemetry-mechanisms.md) | NetFlow v9 vs IPFIX (RFC 7011); proof that 1:1000 packet sampling causes a 23.7% botnet C2 miss rate | [telemetry-mechanisms.md](docs/foundations/telemetry-mechanisms.md) |
| **NAT-01** | [CGNAT Standards](docs/cgnat-attribution/cgnat-architecture-and-standards.md) | RFC 6888 requirements (hairpinning, paired pooling), RFC 6598 `100.64.0.0/10` routing constraints | [cgnat-architecture-and-standards.md](docs/cgnat-attribution/cgnat-architecture-and-standards.md) |
| **NAT-02** | [Log Volume Mathematics](docs/cgnat-attribution/log-volume-mathematics.md) | Rigorous derivation of 2.16B rows/day ($172.8\text{ GB}$) vs PBA 60k rows/day ($4.8\text{ MB}$) | [log-volume-mathematics.md](docs/cgnat-attribution/log-volume-mathematics.md) |
| **NAT-03** | [AAA & Attribution](docs/cgnat-attribution/radius-accounting-and-attribution.md) | FreeRADIUS schema, two-tier temporal intersection algorithm, orphaned session recovery formula | [radius-accounting-and-attribution.md](docs/cgnat-attribution/radius-accounting-and-attribution.md) |
| **THR-01** | [Mirai Botnet Disassembly](docs/threat-intel/mirai-botnet-family.md) | Source audit of `scanner.c`, 68 hardcoded XOR credentials, US DoD/USPS IP exclusion filter | [mirai-botnet-family.md](docs/threat-intel/mirai-botnet-family.md) |
| **THR-02** | [Mozi P2P Architecture](docs/threat-intel/mozi-p2p-botnet.md) | BitTorrent Kademlia DHT overlay, ECDSA384 cryptographic command authentication, CPE exploits | [mozi-p2p-botnet.md](docs/threat-intel/mozi-p2p-botnet.md) |
| **THR-03** | [Residential Proxies](docs/threat-intel/residential-proxies-and-abuse.md) | BadBox AOSP firmware rootkits, 911 S5 monetization, mathematical proof of Up/Down ratio inversion | [residential-proxies-and-abuse.md](docs/threat-intel/residential-proxies-and-abuse.md) |
| **THR-04** | [Target CPE Hardware](docs/threat-intel/target-cpe-vulnerabilities.md) | Syrotech/Netlink ONTs, CP PLUS cameras, default credentials, TR-069 / CWMP SOAP exploits | [target-cpe-vulnerabilities.md](docs/threat-intel/target-cpe-vulnerabilities.md) |
| **THR-05** | [Outbound Abuse Profiles](docs/threat-intel/outbound-abuse-profiles.md) | Amplification factor table (DNS 54x, NTP 1000x, Memcached 51000x), BCP 38 ingress filtering | [outbound-abuse-profiles.md](docs/threat-intel/outbound-abuse-profiles.md) |
| **DET-01** | [Suricata Line-Rate](docs/detection-analytics/suricata-line-rate-engine.md) | Workers runmode zero-lock scaling, AF_PACKET MMAP ring buffer vs DPDK, Hyperscan AVX-512 | [suricata-line-rate-engine.md](docs/detection-analytics/suricata-line-rate-engine.md) |
| **DET-02** | [Bayesian Scoring](docs/detection-analytics/bayesian-log-odds-fusion.md) | Mathematical proof of Base Rate Fallacy, additive log-odds fusion, correlation damping $\alpha$ | [bayesian-log-odds-fusion.md](docs/detection-analytics/bayesian-log-odds-fusion.md) |
| **DET-03** | [ClickHouse Storage](docs/detection-analytics/clickhouse-columnar-storage.md) | Column-oriented storage, SIMD vectorization, ZSTD/DoubleDelta compression, TTL lifecycle | [clickhouse-columnar-storage.md](docs/detection-analytics/clickhouse-columnar-storage.md) |
| **REM-01** | [RFC 6561 Quarantine](docs/containment-remediation/rfc6561-walled-garden-framework.md) | Tripartite lifecycle, DNS RPZ spoofing vs HTTP 302 DNAT vs BGP Flowspec rate-limiting | [rfc6561-walled-garden-framework.md](docs/containment-remediation/rfc6561-walled-garden-framework.md) |
| **REM-02** | [Captive Portal API](docs/containment-remediation/captive-portal-api-standards.md) | RFC 8908 & RFC 8910, DHCP Option 114, resolving HSTS/TLS certificate warning lockouts | [captive-portal-api-standards.md](docs/containment-remediation/captive-portal-api-standards.md) |
| **REM-03** | [Safe Harbor Whitelists](docs/containment-remediation/safe-harbor-remediation-whitelist.md) | Essential OS/AV update domains, Cyber Swachhta Kendra, split-horizon DNS routing | [safe-harbor-remediation-whitelist.md](docs/containment-remediation/safe-harbor-remediation-whitelist.md) |
| **REM-04** | [Probation Ladder](docs/containment-remediation/verification-and-probation-ladder.md) | Active synthetic probing, 30-minute tiered probation window, automated quarantine clearance | [verification-and-probation-ladder.md](docs/containment-remediation/verification-and-probation-ladder.md) |
| **REG-01** | [CERT-In Directions & SLA](docs/regulatory-compliance/cert-in-directions-and-sla.md) | Section 70B IT Act, 6-hour SLA clock trigger standards, Jan Vishwas Act 2023 ₹1 Crore penalties | [cert-in-directions-and-sla.md](docs/regulatory-compliance/cert-in-directions-and-sla.md) |
| **REG-02** | [Annexure Mapping](docs/regulatory-compliance/annexure-mapping-and-reporting.md) | Complete mapping of 20 Annexure-I categories to network telemetry, Annexure-II JSON schema | [annexure-mapping-and-reporting.md](docs/regulatory-compliance/annexure-mapping-and-reporting.md) |
| **REG-03** | [Human Approval Gate](docs/regulatory-compliance/human-approval-gate-rationale.md) | Why autonomous dispatch violates law; Section 85 corporate vicarious liability; 3-tier gate | [human-approval-gate-rationale.md](docs/regulatory-compliance/human-approval-gate-rationale.md) |
| **REG-04** | [BSA 2023 Section 63](docs/regulatory-compliance/bsa-2023-section-63-evidence.md) | Section 63 BSA electronic evidence admissibility replacing IEA 65B, Merkle trees, NPLI NTP | [bsa-2023-section-63-evidence.md](docs/regulatory-compliance/bsa-2023-section-63-evidence.md) |
| **EVD-01** | [Standards & RFC Ledger](docs/evidence/standards-and-rfc-ledger.md) | Canonical index of all RFCs, statutory acts, Gazette orders, academic papers, and CVEs | [standards-and-rfc-ledger.md](docs/evidence/standards-and-rfc-ledger.md) |
| **EVD-02** | [Hardware Sizing](docs/evidence/hardware-capacity-specifications.md) | Commodity 1U/2U server sizing across 5,000, 15,000, and 30,000 subscriber networks | [hardware-capacity-specifications.md](docs/evidence/hardware-capacity-specifications.md) |

---

## 5. Scope Boundary & Research Principles

In strict adherence to academic and engineering rigor:
- **No Untested Implementation Claims:** This research corpus documents verified network protocols, disassemblies of real-world malware, published empirical datasets, mathematical proofs, and sovereign legal enactments. It explicitly avoids making speculative claims regarding the performance or completion of unverified proprietary code.
- **Strictly Local & Defensible:** All documents reside locally in markdown format, enabling direct offline auditability, versioning, and integration into subsequent engineering stages.
- **Zero Remote Push:** Maintained strictly within local storage per compliance and data governance protocols.
