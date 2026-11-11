# ARCUS Pre-Project Research Document Map

Master index of technical research dossiers supporting autonomous subscriber compromise detection, Carrier-Grade NAT (CGNAT) attribution, and regulatory compliance for regional Tier-2/3 Internet Service Providers, developed under the **ToxArch** organization.

---

## Complete Document Index

| Chapter ID | Technical Domain | Document Title | Primary Focus | Canonical File Link |
| :--- | :--- | :--- | :--- | :--- |
| **META-01** | **Methodology** | Research Methodology & Evidence Boundary | Evidence boundaries, verification tiers, primary source tracking | [methodology.md](methodology.md) |
| **FND-01** | **Access Topologies** | Broadband Access Topologies | BNG/BRAS architectures, PPPoE vs. IPoE statefulness, GPON optical splitters | [broadband-access-topologies.md](foundations/broadband-access-topologies.md) |
| **FND-02** | **Telemetry Standards** | Telemetry Mechanisms: NetFlow v9 & IPFIX | NetFlow v9 vs. IPFIX (RFC 7011), sampling rate degradation math, flow templates | [telemetry-mechanisms.md](foundations/telemetry-mechanisms.md) |
| **NAT-01** | **CGNAT Standards** | CGNAT Architecture & Standards | RFC 6888, RFC 6598, Dynamic vs. PBA vs. Deterministic NAT | [cgnat-architecture-and-standards.md](cgnat-attribution/cgnat-architecture-and-standards.md) |
| **NAT-02** | **Logging Economics** | CGNAT Log Volume Mathematics | Mathematical derivation of 2.16B rows/day vs. PBA 60k rows/day (99.997% reduction) | [log-volume-mathematics.md](cgnat-attribution/log-volume-mathematics.md) |
| **NAT-03** | **AAA & Attribution** | RADIUS Accounting & Attribution | FreeRADIUS session accounting, critical attributes, two-tier temporal intersection, orphaned sessions | [radius-accounting-and-attribution.md](cgnat-attribution/radius-accounting-and-attribution.md) |
| **THR-01** | **Mirai Disassembly** | Mirai Botnet Family Disassembly | Modular architecture, scanner.c LCG logic, 68 credentials, exclusion subnets | [mirai-botnet-family.md](threat-intel/mirai-botnet-family.md) |
| **THR-02** | **Mozi P2P Botnet** | Mozi P2P Botnet Architecture | Kademlia DHT overlay, ECDSA384 command verification, Realtek/Huawei exploits | [mozi-p2p-botnet.md](threat-intel/mozi-p2p-botnet.md) |
| **THR-03** | **Residential Proxies** | Residential Proxies & Covert Abuse Networks | BadBox firmware rootkits, 911 S5 trojans, upload/download ratio inversion | [residential-proxies-and-abuse.md](threat-intel/residential-proxies-and-abuse.md) |
| **THR-04** | **Target Hardware** | Target CPE Hardware Vulnerabilities | Syrotech/Netlink ONTs, TP-Link/D-Link routers, CP PLUS cameras, default credentials | [target-cpe-vulnerabilities.md](threat-intel/target-cpe-vulnerabilities.md) |
| **THR-05** | **Abuse Profiles** | Outbound Abuse Profiles & Amplification | Amplification factor mathematics, DNS/NTP/SSDP/Memcached matrices, BCP 38 | [outbound-abuse-profiles.md](threat-intel/outbound-abuse-profiles.md) |
| **DET-01** | **Suricata Line-Rate** | Suricata Line-Rate Packet Inspection | Workers runmode, AF_PACKET MMAP vs. DPDK, Hyperscan MPM, asymmetric stream | [suricata-line-rate-engine.md](detection-analytics/suricata-line-rate-engine.md) |
| **DET-02** | **Bayesian Scoring** | Bayesian Multi-Signal Fusion | Base Rate Fallacy proof, additive log-odds fusion, likelihood ratios, correlation damping | [bayesian-log-odds-fusion.md](detection-analytics/bayesian-log-odds-fusion.md) |
| **DET-03** | **Analytical Storage** | ClickHouse Columnar Storage | ClickHouse columnar engine, SIMD vectorization, compression codecs, ReplacingMergeTree, TTLs | [clickhouse-columnar-storage.md](detection-analytics/clickhouse-columnar-storage.md) |
| **REM-01** | **RFC 6561 Quarantine** | RFC 6561 Walled Garden Framework | Tripartite lifecycle, DNS RPZ vs. HTTP 302 DNAT vs. BGP Flowspec | [rfc6561-walled-garden-framework.md](containment-remediation/rfc6561-walled-garden-framework.md) |
| **REM-02** | **Captive Portal API** | Captive Portal API Standards | IETF RFC 8908 & RFC 8910, DHCP Option 114, IPv6 RA Option 37, OS CNA handling | [captive-portal-api-standards.md](containment-remediation/captive-portal-api-standards.md) |
| **REM-03** | **Safe Harbors** | Safe Harbor Remediation Whitelists | Essential OS/AV update domains, Cyber Swachhta Kendra, split-horizon DNS routing | [safe-harbor-remediation-whitelist.md](containment-remediation/safe-harbor-remediation-whitelist.md) |
| **REM-04** | **Probation Ladder** | Verification Probing & Probation Ladder | Active synthetic scanning, 30-minute tiered probation, automated quarantine clearance | [verification-and-probation-ladder.md](containment-remediation/verification-and-probation-ladder.md) |
| **REG-01** | **CERT-In Directions** | CERT-In Directions & The 6-Hour SLA | Section 70B IT Act, 6-hour SLA clock trigger standards, 4-phase countdown state, Jan Vishwas fines | [cert-in-directions-and-sla.md](regulatory-compliance/cert-in-directions-and-sla.md) |
| **REG-02** | **Annexure Mapping** | Annexure-I Categories & Reporting Schemas | 20 mandatory incident categories mapped to flow/DPI triggers, Annexure-II schema | [annexure-mapping-and-reporting.md](regulatory-compliance/annexure-mapping-and-reporting.md) |
| **REG-03** | **Human Approval** | Human-in-the-Loop Approval Gate Rationale | Legal due process, Section 85 vicarious corporate liability, 3-tier gate architecture | [human-approval-gate-rationale.md](regulatory-compliance/human-approval-gate-rationale.md) |
| **REG-04** | **BSA 2023 Section 63** | BSA Section 63 Digital Evidence Admissibility | Electronic evidence admissibility replacing IEA 65B, Merkle chains, WORM storage, NPLI NTP | [bsa-2023-section-63-evidence.md](regulatory-compliance/bsa-2023-section-63-evidence.md) |
| **EVD-01** | **Standards Ledger** | Primary Standards, Directives & Research Ledger | Canonical IETF RFCs, Gazette orders, statutory provisions, and technical citations | [standards-and-rfc-ledger.md](evidence/standards-and-rfc-ledger.md) |
| **EVD-02** | **Hardware Sizing** | Hardware Sizing & Capacity Specifications | Commodity 1U/2U server specifications across 5k, 15k, and 30k subscriber tiers | [hardware-capacity-specifications.md](evidence/hardware-capacity-specifications.md) |
