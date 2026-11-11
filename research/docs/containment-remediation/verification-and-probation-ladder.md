# Verification Probing & Probation Ladder Architecture

This dossier documents the automated state machine, verification tests, and tiered probation mechanisms required to safely transition a subscriber from quarantine back into full production routing.

---

## 1. Automated Verification Probing

Once a subscriber clicks "Remediation Complete" on the notification portal (or when the automated quarantine timer expires), the system initiates active and passive verification:

```
[Subscriber Clicks 'Remediate']
             |
             v
[Active Synthetic Probe]
Scan subscriber pre-NAT IP (100.64.x.y) for open vulnerable ports:
  - TCP/23 (Telnet)
  - TCP/2323 (Mirai Telnet alt)
  - TCP/7547 (CWMP SOAP)
  - TCP/5555 (Open ADB)
  - UDP/53 (Open DNS Resolver)
             |
      +------+------+
      |             |
 [Port Open]   [Port Closed / Filtered]
      |             |
 [FAIL]             v
 [Keep in      [Passive Telemetry Window (15 Min)]
  Quarantine]  Monitor NetFlow/IPFIX for:
                 - No SYN flood scans
                 - No UDP reflection bursts
                 - Normal Up/Down ratio
                    |
             +------+------+
             |             |
        [Anomalies]   [Clean Stream]
             |             |
        [FAIL]             v
        [Extend]      [Release to Probationary Status]
```

---

## 2. The 30-Minute Probation Ladder

Releasing an infected host immediately to unrestricted access creates severe relapse risks if the malware is dormant or waiting for a C2 beacon cycle. The system utilizes a **Tiered Probation Ladder**:

| Probation Stage | Duration | Network Policy Enforced | Progression Criteria | Relapse Penalty |
| :--- | :--- | :--- | :--- | :--- |
| **Stage 1: Quarantine** | Dynamic | Walled garden; HTTP/DNS redirection; outbound abuse blocked. | Passing synthetic probe and passive 15-min quiet window. | N/A |
| **Stage 2: Strict Probation** | 30 Minutes | Full Internet access restored, **except** outbound Telnet (TCP 23/2323), SMTP (TCP 25), and CWMP (7547) remain strictly blocked at BNG. | Zero high-rate alerts or Bayesian confidence spikes during window. | Instant regression to Stage 1 with 2-hour quarantine floor. |
| **Stage 3: Full Unrestricted** | Permanent | Standard subscriber policy; 1:1 flow telemetry monitoring continues. | Continuous clean operation. | Standard Bayesian detection ladder. |

### Operational Advantages:
- Eliminates customer frustration by restoring web browsing, video streaming, and essential communications within minutes of cleanup.
- Maintains a safety net protecting external networks from re-infection during the initial high-risk observation window.
