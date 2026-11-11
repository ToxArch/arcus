# CERT-In Directions, Statutory Mandates & The 6-Hour SLA Clock

This dossier analyzes the statutory requirements, regulatory timelines, and legal penalties governing cybersecurity incident reporting for Indian Internet Service Providers (ISPs).

---

## 1. Statutory Foundations: Section 70B & Directions 2022

Under the **Information Technology Act, 2000 (Section 70B)**, the Indian Computer Emergency Response Team (CERT-In) is designated as the national nodal agency for incident response and cybersecurity directives.

On **28 April 2022**, CERT-In issued binding Directions under Section 70B(6) (effective 28 June 2022 / 25 September 2022 for MSMEs):
- **Mandatory 6-Hour Reporting Window:** Any service provider, intermediary, data center, body corporate, or ISP must report cybersecurity incidents to CERT-In **within 6 hours of noticing or being brought to notice of such incidents**.
- **Log Retention Mandate:** Intermediaries and service providers must mandate the maintenance of system and ICT logs for a period of **180 days (6 months)** within Indian jurisdiction.
- **Clock Synchronization Mandate:** All ICT systems must synchronize their system clocks to the Network Time Protocol (NTP) servers of the **National Physical Laboratory of India (NPLI)** or the National Informatics Centre (NIC).

---

## 2. Jan Vishwas Act, 2023 & Penalty Architecture

Historically, non-compliance with CERT-In directions under Section 70B(7) carried criminal imprisonment of up to 1 year and a fine up to ₹1,00,000.

The **Jan Vishwas (Amendment of Provisions) Act, 2023** (Act No. 18 of 2023) fundamentally overhauled IT Act penalties, replacing minor criminal prosecutions with severe financial penalties adjudicated by designated Adjudicating Officers:

| Statutory Provision | Historical Penalty (Pre-2023) | Amended Penalty (Jan Vishwas Act, 2023) | Enforcement Entity |
| :--- | :--- | :--- | :--- |
| **Section 70B(7)** (Failure to report or furnish info to CERT-In) | Imprisonment up to 1 year OR fine up to ₹1 Lakh, or both. | **Penalty up to ₹1,00,00,000 (1 Crore INR)** per failure to report or comply. | CERT-In / MeitY Adjudicating Officer. |
| **Section 43A / DPDP Act 2023** (Failure to protect personal data) | Compensation to affected victim. | **Up to ₹250,00,00,000 (250 Crore INR)** under Digital Personal Data Protection Act 2023 for significant data breaches. | Data Protection Board of India (DPBI). |
| **Section 85 (IT Act)** (Offences by Companies) | Directors/executives liable unless lack of knowledge proved. | **Executive Vicarious Liability:** Managing directors, CTOs, and designated CISO personally liable for corporate non-compliance. | Courts of Law / Adjudicating Authority. |

---

## 3. The 4-Phase 6-Hour SLA State Machine

Because the 6-hour clock begins the moment an incident is "noticed or brought to notice", an automated security architecture must structure incident handling into four rigorous time-gated phases:

```
t=0h                                 t=2h               t=4.5h           t=5h              t=6h
 |------------------------------------|-------------------|----------------|-----------------|
 [Phase 1: Ingestion & Verification]  [Phase 2: Review]   [Phase 3: Auth]  [Phase 4: Transmit]
 - Bayesian Posterior >= 0.999        - Human CISO Review - Cryptographic  - Dispatch to CERT-In
 - CGNAT + RADIUS Attribution Bound   - Evidence audit    - BSA Cert Sign  - Webhook / Encrypted
 - Incident Package Generated         - "Approve/Reject"  - Merkle Chain   - Acknowledgment
```

| Phase | Time Window | System Actions | Human Actions |
| :--- | :--- | :--- | :--- |
| **Phase 1: Verification & Assembly** | $T \in [0, 2\text{ hours}]$ | Detection engine verifies Bayesian score $\ge 0.999$; correlates CGNAT PBA + RADIUS session; compiles Annexure-II JSON dossier with raw PCAP and IPFIX records. | None (Autonomous processing). |
| **Phase 2: Security Analyst Review** | $T \in [2, 4.5\text{ hours}]$ | Incident locked in review queue; automated countdown timer alerts security desk; auto-escalates via SMS/PagerDuty if unassigned. | Duty security engineer reviews subscriber identity, attack vector, and evidence artifacts. |
| **Phase 3: Executive Authorization** | $T \in [4.5, 5\text{ hours}]$ | System generates Section 63 BSA electronic evidence certificate; computes Merkle root; pre-populates official CERT-In dispatch form. | CISO / Designated Incident Officer reviews and applies digital signature (`sign_token`). |
| **Phase 4: Sovereign Dispatch** | $T \in [5, 6\text{ hours}]$ | Secure API dispatch or encrypted PGP email transmission to `incident@cert-in.org.in`; captures HTTP/SMTP receipt token; closes SLA clock. | Monitors transmission confirmation. |
