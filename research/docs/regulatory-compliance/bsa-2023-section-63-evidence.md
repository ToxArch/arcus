# Bharatiya Sakshya Adhiniyam, 2023 (BSA Section 63) Digital Evidence Admissibility

This dossier details the statutory requirements for electronic evidence admissibility under Section 63 of the Bharatiya Sakshya Adhiniyam, 2023 (replacing Section 65B of the Indian Evidence Act, 1872), establishing the cryptographic integrity standards required for ISP forensic telemetry.

---

## 1. The Legislative Transition: IEA 65B to BSA Section 63

On **1 July 2024**, the Parliament of India repealed the Indian Evidence Act, 1872 (IEA) and enacted the **Bharatiya Sakshya Adhiniyam, 2023 (Act No. 47 of 2023)**:
- Section 65B of the Indian Evidence Act is replaced by **Section 63 of the BSA, 2023 ("Admissibility of electronic records")**.
- Section 63 formalizes that any information contained in an electronic record produced by a computer, whether on paper, optical, or magnetic media, shall be deemed to be a document and admissible in evidence without further proof, **provided specific conditions are satisfied**.

---

## 2. Mandatory Statutory Conditions under BSA Section 63(2)

For network logs, flow records, and CGNAT attribution tables to be admissible as evidence in Indian courts or regulatory proceedings, four conditions must be met:

1. **Lawful Control & Production:** The computer system generating the record must have been regularly used to store or process information for the purposes of the lawful activities carried on by the ISP during the period.
2. **Regular Ingestion Stream:** Information of the kind contained in the record must have been regularly fed into the computer in the ordinary course of business.
3. **Operational Integrity:** Throughout the material period, the computer system was operating properly, or if inoperative/out of order for any part of that period, such disruption did not affect the accuracy of the electronic record.
4. **Authenticity of Content:** The information contained in the record reproduces or is derived from such information fed into the computer in the ordinary course of business.

---

## 3. Statutory Schedule Certificate (Part A and Part B)

Unlike the historical unstandardized Section 65B certificate, the BSA 2023 establishes a statutory template in the **Schedule to the Act**, divided into two mandatory components:

### Part A (Certificate by the Person in Charge):
Signed by the CISO or systems administrator specifying:
- Description of the computer device (Make, Model, Serial No., IP Address).
- Operating system version, database software version, and cryptographic hash algorithm utilized.
- Affirmation that the system operated normally without unauthorized access or data corruption.

### Part B (Certificate by the Technical Expert):
Signed by the forensic expert or technical authority confirming:
- Cryptographic hash verification of the stored raw telemetry files.
- Absence of manual tampering or deletion across the audit trail.

---

## 4. Cryptographic Proof Architecture: Merkle Trees & WORM Storage

To satisfy the stringent burden of proof under BSA Section 63 without human doubt:

```
[Raw IPFIX Flow Batch]   [CGNAT PBA Event Batch]   [RADIUS radacct Batch]
         |                          |                         |
     SHA-256                    SHA-256                   SHA-256
      (H_0)                      (H_1)                     (H_2)
         \                        /                         /
          \                      /                         /
           v                    v                         /
          [Internal Node H_01]                           /
                   \                                    /
                    \                                  /
                     v                                v
                   [Merkle Root Hash H_Root (SHA-256)]
                                  |
                                  v
             [Immutable WORM Storage / Append-Only Ledger]
             [Timestamped via NPLI Stratum-1 Time Reference]
```

### Cryptographic Standards:
1. **Per-Block Hashing:** Every 15-minute batch of flow and NAT records is sealed with a **SHA-256** checksum upon creation.
2. **Merkle Hash Chain:** Sequential batch hashes are linked into a Merkle DAG (Directed Acyclic Graph). Modifying a single historical row alters the Merkle root, immediately exposing any tampering.
3. **Write-Once-Read-Many (WORM) Retention:** Logs are committed to WORM storage (e.g., ZFS dataset with read-only immutable flags `chattr +i` or S3 Object Lock in Compliance mode) for the statutory 180-day retention duration.
4. **Time Certification:** All timestamps are certified against **NPLI Stratum-1 NTP**, ensuring that clock drift cannot be raised as a defense to challenge evidence authenticity.
