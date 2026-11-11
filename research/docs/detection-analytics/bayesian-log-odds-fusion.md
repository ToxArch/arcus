# Bayesian Multi-Signal Fusion & Base Rate Fallacy Proof

This dossier establishes the mathematical proof of the Base Rate Fallacy in ISP compromise detection and details the additive Bayesian Log-Odds fusion model required for robust, false-positive-resistant subscriber classification.

---

## 1. The Base Rate Fallacy in ISP Networks

In carrier cybersecurity, operators often assume that an intrusion detection rule with "99% accuracy" ($1\% \text{ False Positive Rate}$) is sufficient to trigger automated subscriber quarantine. **In high-volume ISP networks, this assumption is mathematically catastrophic.**

### Mathematical Proof:
Let $C$ represent the event that a subscriber host is truly compromised:
$$C \in \{0, 1\}$$

Let $P(C)$ represent the **prior probability (base rate)** of subscriber infection across a regional residential broadband network. Empirical industry measurements indicate:
$$P(C) = 0.005 \quad (0.5\% \text{ of subscribers infected at any time})$$
$$P(\neg C) = 1 - P(C) = 0.995 \quad (99.5\% \text{ clean})$$

Let $T^+$ represent a positive detection alert from a signature or heuristic detection rule.
Assume an exceptional, high-precision detection signature with:
- **True Positive Rate (Sensitivity):** $P(T^+ \mid C) = 0.99 \quad (99\%)$
- **False Positive Rate (Fallout):** $P(T^+ \mid \neg C) = 0.01 \quad (1\%)$

Using Bayes' Theorem, the **Posterior Probability** that a subscriber is truly compromised given an alert $T^+$ is:

$$P(C \mid T^+) = \frac{P(T^+ \mid C) \cdot P(C)}{P(T^+ \mid C) \cdot P(C) + P(T^+ \mid \neg C) \cdot P(\neg C)}$$

Substituting the empirical values:

$$P(C \mid T^+) = \frac{0.99 \times 0.005}{(0.99 \times 0.005) + (0.01 \times 0.995)}$$

$$P(C \mid T^+) = \frac{0.00495}{0.00495 + 0.00995} = \frac{0.00495}{0.01490} \approx 0.3322 \quad (33.22\%)$$

### Operational Consequence:
Despite a 99% accurate detection rule, **66.78% of all generated alerts are FALSE POSITIVES**:
$$\text{False Discovery Rate (FDR)} = 1 - P(C \mid T^+) = 66.78\%$$

If an ISP configured automated walled-garden quarantine based on single-rule alerts, **two out of every three quarantined customers would be completely innocent**, triggering catastrophic customer support churn, regulatory complaints, and brand damage.

---

## 2. Additive Bayesian Log-Odds Formulation

To overcome the Base Rate Fallacy, multiple independent evidence signals (e.g., DPI signature match, NetFlow scan burst, DNS sinkhole hit, abuse threat feed) must be fused mathematically.

### Log-Odds Formulation:
The posterior odds are related to prior odds via the Bayes Factor (Likelihood Ratio $LR$):
$$\text{Odds}(C \mid E) = \text{Odds}(C) \times \prod_{i=1}^{k} LR_i$$

Taking the natural logarithm yields a strictly additive computational model:
$$\ln \text{Odds}(C \mid E_1, \dots, E_k) = \ln \text{Odds}(C) + \sum_{i=1}^{k} \ln LR_i$$

Where:
- $\text{Odds}(C) = \frac{P(C)}{1 - P(C)} = \frac{0.005}{0.995} \approx 0.005025$
- $\ln \text{Odds}_{\text{prior}} = \ln(0.005025) \approx -5.293$
- $LR_i = \frac{P(E_i \mid C)}{P(E_i \mid \neg C)}$ (Likelihood Ratio of evidence $E_i$)

### Likelihood Ratio Table for Detection Signals:

| Evidence Signal ($E_i$) | Detection Mechanism | $P(E_i \mid C)$ | $P(E_i \mid \neg C)$ | Likelihood Ratio ($LR_i$) | Weight ($\ln LR_i$) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **DPI Mirai Telnet Payload** | Suricata rule match (`sh /bin/busybox`) | 0.85 | 0.0001 | **8,500** | **+9.047** |
| **Spamhaus DROP IP Hit** | Flow collector destination match | 0.90 | 0.001 | **900** | **+6.802** |
| **High-Rate Port Scan** | >100 failed SYNs/sec on port 23/2323 | 0.70 | 0.005 | **140** | **+4.941** |
| **DNS Sinkhole Query** | Querying known C2 DGA domain | 0.80 | 0.002 | **400** | **+5.991** |
| **Upload Ratio Inversion** | Up/Down byte ratio > 1.2 sustained 1 hr | 0.65 | 0.020 | **32.5** | **+3.481** |

---

## 3. Correlation Damping & Confidence Thresholds

In production networks, security signals are rarely 100% statistically independent (e.g., a Telnet brute-force scanner triggers both a flow rate alert and a DPI rule alert). Unchecked additive fusion would cause **confidence over-inflation**.

### Damping Formulation:
To prevent over-inflation from correlated telemetry sources, a damping coefficient $\alpha_i \in (0, 1]$ is applied:
$$\ln \text{Odds}_{\text{posterior}} = \ln \text{Odds}_{\text{prior}} + \sum_{i=1}^{k} \alpha_i \ln LR_i$$

Where $\alpha_i = 1.0$ for orthogonal telemetry (e.g., DNS sinkhole vs. NetFlow volume) and $\alpha_i = 0.5$ for co-dependent signals (e.g., multiple alerts from the same Suricata flow).

### Posterior Probability Conversion:
$$\text{Posterior Probability } P(C \mid E) = \frac{1}{1 + e^{-\ln \text{Odds}_{\text{posterior}}}}$$

```
Confidence Action Matrix:
P(C|E) >= 0.999 (Posterior Odds >= +6.9): Mandatory Tier-1 Containment (Walled Garden)
P(C|E) >= 0.900 (Posterior Odds >= +2.2): Tier-2 Advisory (SMS/Email warning, rate-limit)
P(C|E) <  0.900 (Posterior Odds <  +2.2): Telemetry observation only (No customer action)
```

### Temporal Evidence Decay:
Evidence does not remain valid indefinitely. A half-life decay function updates accumulated log-odds over time:
$$\ln \text{Odds}(t) = \ln \text{Odds}(t_0) \cdot e^{-\lambda (t - t_0)}$$
Where $\lambda = \frac{\ln 2}{T_{half}}$ with half-life $T_{half} = 6 \text{ hours}$.
