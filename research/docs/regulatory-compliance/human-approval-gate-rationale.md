# Human-in-the-Loop Approval Gate: Legal & Operational Rationale

This dossier details why fully autonomous, un-gated regulatory incident dispatch to sovereign authorities violates statutory due process and creates severe operational liabilities for Internet Service Providers.

---

## 1. Statutory Due Process & Legal Representation

Under Indian jurisprudence and corporate law:
- **Formal Legal Submissions:** An incident report dispatched to CERT-In or the Department of Telecommunications (DoT) is not an informal log message; it is a **formal legal submission made by the corporate entity under statutory penalty**.
- **Section 85 of the IT Act, 2000:** Imposes personal criminal and civil vicarious liability upon the directors and designated officers of the company for false, negligent, or misleading statements made to sovereign authorities.
- An automated software daemon or machine learning engine cannot hold legal power of attorney, cannot be cross-examined in a court of law, and cannot be held legally liable under the Bharatiya Nyaya Sanhita, 2023.

---

## 2. Operational Risks of Unsupervised Autonomous Dispatch

### Risk 1: The False Regulatory Cascade
If an autonomous engine dispatches incident reports without human verification:
- A sudden burst of benign traffic misclassified by a flawed signature rule could flood CERT-In with hundreds of false incident reports within minutes.
- This triggers official inquiries, on-site audits, statutory show-cause notices under Section 70B(7), and severe reputational damage with licensing authorities (DoT/TRAI).

### Risk 2: Premature Tipping Off / Contempt Risks
Certain incidents may involve lawful interception orders or confidential investigations directed by intelligence agencies under Section 69 of the IT Act. Autonomous dispatch without compliance officer clearance risks disclosing active investigations to unauthorized parties.

---

## 3. The 3-Tier Human Gate Architecture

To reconcile the mandatory 6-hour SLA clock with statutory human authorization, the system enforces a **3-Tier Gate**:

```
[Autonomous Detection & Assembly Engine]
                 |
                 v
   [Is Confidence >= 0.999 AND Threat Critical?]
          |                      |
       (YES)                    (NO)
          |                      |
          v                      v
[Tier 1: Analyst Triage]   [Logged in Advisory Queue]
   (Time: <= 2 Hours)      (No Regulatory Dispatch)
          |
   [Approved by Analyst]
          |
          v
[Tier 2: CISO / Officer Authorization Gate]
   (Time: <= 4.5 Hours)
   - Review Evidence Dossier
   - Verify BSA Sec 63 Hash
   - Apply Corporate Electronic Signature
          |
   +------+------+
   |             |
[APPROVED]   [REJECTED / FALSE POSITIVE]
   |             |
   v             v
[Tier 3:     [Suppressed; Feedback Fed to
 Sovereign    Bayesian Prior Calibration]
 Dispatch]
```

### Fail-Safe SLA Escalation:
If the designated CISO or security engineer does not act on a critical incident within **4 hours (2 hours before statutory breach)**, the system automatically triggers multi-channel escalation:
1. PagerDuty high-priority emergency call.
2. SMS alert with one-time verification token.
3. Automated fallback to secondary designated officer.
