DRAFT - NOT LEGALLY REVIEWED
# ARCUS Security Policy & Responsible Vulnerability Disclosure

ARCUS is an infrastructure security and attribution architecture designed for
carrier networks. We treat potential security vulnerabilities in our
specifications, schemas, and code with the utmost urgency.

---

### Scope
This security policy applies to:
* Mathematical logic flaws in the two-tier CGNAT attribution engine that could
  lead to false subscriber identification.
* Cryptographic weaknesses in the BSA 2023 Section 63 Merkle hash chaining or
  timestamp verification routines.
* Ingestion buffer overflows, denial-of-service vulnerabilities, or injection
  vectors in our syslog and IPFIX parsing daemons.
* Authentication bypasses in the administrative and compliance web portal.

### Out of Scope
* Security issues in third-party carrier hardware (e.g., vendor zero-days in
  MikroTik RouterOS or Cisco ASR).
* Volumetric denial-of-service attacks against an operator's physical optical
  distribution network.

---

### How to Report a Vulnerability
If you discover a potential vulnerability, please report it privately:
* **Email:** Send full technical details, proof-of-concept steps, and packet
  traces to: `bsd98574@gmail.com` [OWNER TO CONFIRM].
* **Encryption:** If sensitive, request our PGP public key prior to sending raw
  exploit details.
* **Do NOT open a public GitHub issue** for undisclosed security vulnerabilities.

---

### Our Response Timeline
* **Initial Acknowledgment:** Within forty-eight (48) hours of receipt [OWNER TO CONFIRM].
* **Triage and Assessment:** Within five (5) business days.
* **Remediation Release:** We aim to release a verified security patch within
  fourteen (14) business days of confirmation.

We ask security researchers to practice responsible coordinated disclosure and
allow adequate time to issue fixes before public release.
