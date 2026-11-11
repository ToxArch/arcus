# ARCUS

<p align="center">
  <strong>Autonomous Subscriber Compromise Detection & Regulatory Compliance Architecture for Tier-2/3 Internet Service Providers</strong>
</p>

<p align="center">
  <a href="#overview">Overview</a> •
  <a href="#architectural-pillars">Architectural Pillars</a> •
  <a href="research/">Research Dossier</a> •
  <a href="#repository-layout">Repository Layout</a> •
  <a href="#governance">Governance</a>
</p>

---

## Overview

**ARCUS** is an autonomous cybersecurity detection, Carrier-Grade NAT (CGNAT) attribution, and regulatory compliance architecture engineered specifically for regional Tier-2 and Tier-3 broadband Internet Service Providers (FTTH / Wireless ISP).

Regional ISPs operate under unique operational, economic, and sovereign legal constraints:
1. **The CGNAT Attribution Dilemma:** Thousands of residential subscribers share common public IPv4 address pools under Carrier-Grade NAT (RFC 6888 / RFC 6598). When external abuse notifications arrive citing an incident tuple $\langle IP_{\text{pub}}, Port_{\text{pub}}, t_{\text{incident}} \rangle$, operators cannot determine which physical subscriber loop originated the threat without microsecond-accurate temporal state correlation.
2. **The Base Rate Fallacy:** In residential networks with a low subscriber infection base rate ($P(C) \approx 0.005$), even a 99% accurate detection signature suffers a **False Discovery Rate of 66.8%**. Quarantining customers on un-fused single-rule alerts creates catastrophic false positives and customer support collapse.
3. **Draconian Regulatory Penalties:**
   - **CERT-In Directions (28 April 2022):** Enforces a strict **6-hour reporting SLA** from incident notice, domestic 180-day log retention, and NPLI Stratum-1 clock synchronization.
   - **Jan Vishwas Act, 2023:** Imposes financial penalties up to **₹1 Crore (10,000,000 INR)** for non-reporting, accompanied by executive vicarious liability under Section 85 of the IT Act.
   - **Bharatiya Sakshya Adhiniyam, 2023 (BSA Section 63):** Effective 1 July 2024, supersedes Section 65B of the Indian Evidence Act, enforcing strict cryptographic hash chaining, SHA-256 Merkle trees, and statutory Schedule Part A/B certification for electronic evidence admissibility.

ARCUS solves these structural challenges through deterministic Port Block Allocation (PBA) state tracking, line-rate unsampled flow telemetry, additive Bayesian log-odds multi-signal fusion, automated RFC 6561 / RFC 8908 remediation, and cryptographically defensible compliance packaging.

---

## Architectural Pillars

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

## Pre-Project Research Corpus

The foundation of ARCUS is established through an exhaustive 25-chapter technical research dossier located in the [`research/`](research/) directory. 

### Document Map

| Domain | Focus | Key Dossier Links |
| :--- | :--- | :--- |
| **Research Methodology** | Evidence boundaries & primary source tracking | [Methodology](research/docs/methodology.md) • [Document Map](research/docs/README.md) |
| **Foundations** | Access topologies & telemetry line-rate physics | [Broadband Access](research/docs/foundations/broadband-access-topologies.md) • [Flow Telemetry](research/docs/foundations/telemetry-mechanisms.md) |
| **CGNAT Attribution** | Port Block Allocation & two-tier session recovery | [CGNAT Standards](research/docs/cgnat-attribution/cgnat-architecture-and-standards.md) • [Log Volume Math](research/docs/cgnat-attribution/log-volume-mathematics.md) • [RADIUS Attribution](research/docs/cgnat-attribution/radius-accounting-and-attribution.md) |
| **Threat Intelligence** | Malware disassembly & residential threat vectors | [Mirai Botnet](research/docs/threat-intel/mirai-botnet-family.md) • [Mozi P2P](research/docs/threat-intel/mozi-p2p-botnet.md) • [Residential Proxies](research/docs/threat-intel/residential-proxies-and-abuse.md) • [CPE Vulnerabilities](research/docs/threat-intel/target-cpe-vulnerabilities.md) • [Amplification](research/docs/threat-intel/outbound-abuse-profiles.md) |
| **Detection Analytics** | Line-rate DPI, Bayesian fusion & columnar storage | [Suricata Line-Rate](research/docs/detection-analytics/suricata-line-rate-engine.md) • [Bayesian Fusion](research/docs/detection-analytics/bayesian-log-odds-fusion.md) • [ClickHouse Storage](research/docs/detection-analytics/clickhouse-columnar-storage.md) |
| **Containment & Remediation** | Walled garden, Captive Portal API & probation | [RFC 6561 Framework](research/docs/containment-remediation/rfc6561-walled-garden-framework.md) • [Captive Portal API](research/docs/containment-remediation/captive-portal-api-standards.md) • [Safe Harbors](research/docs/containment-remediation/safe-harbor-remediation-whitelist.md) • [Probation Ladder](research/docs/containment-remediation/verification-and-probation-ladder.md) |
| **Regulatory Compliance** | Sovereign SLA clock, evidence admissibility & law | [CERT-In 6-Hour SLA](research/docs/regulatory-compliance/cert-in-directions-and-sla.md) • [Annexure Schemas](research/docs/regulatory-compliance/annexure-mapping-and-reporting.md) • [Approval Gate](research/docs/regulatory-compliance/human-approval-gate-rationale.md) • [BSA 2023 Sec 63](research/docs/regulatory-compliance/bsa-2023-section-63-evidence.md) |
| **Evidence & Capacity** | Primary standards ledger & commodity server sizing | [Standards Ledger](research/docs/evidence/standards-and-rfc-ledger.md) • [Hardware Sizing](research/docs/evidence/hardware-capacity-specifications.md) |

---

## Repository Layout

```
arcus/
├── README.md                      # Master architecture & project overview
└── research/                      # Complete Pre-Project Research Corpus
    ├── README.md                  # Scope & foundations overview
    ├── index.html                 # Interactive documentation reader
    └── docs/
        ├── README.md              # Document index table
        ├── methodology.md         # Evidence tiers & standards bounds
        ├── foundations/           # BNG, PPPoE/IPoE, GPON, IPFIX
        ├── cgnat-attribution/     # RFC 6888, PBA 99.997% math, RADIUS
        ├── threat-intel/          # Mirai, Mozi, BadBox, CPE CVEs, Abuse
        ├── detection-analytics/   # Suricata, Bayesian fusion, ClickHouse
        ├── containment-remediation/ # Walled Garden, RFC 8908/8910, Safe Harbors
        ├── regulatory-compliance/ # CERT-In SLA, BSA Sec 63, Human Gate
        └── evidence/              # Standards ledger & hardware capacity
```

---

## Governance & Status

* **Status:** Pre-Project Research Complete. System engineering and specification phase active.
* **Organization:** Developed under the [ToxArch](https://github.com/ToxArch) cybersecurity defense organization.
* **License:** Licensing is currently under strategic evaluation. All rights reserved by ToxArch.
