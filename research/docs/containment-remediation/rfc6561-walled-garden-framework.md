# RFC 6561 Walled Garden & Containment Framework

This dossier documents the architectural patterns, remediation lifecycles, and network policy mechanisms defined in IETF RFC 6561 ("Recommendations for the Remediation of Compromised Hosts in Broadband Access Networks").

---

## 1. The Tripartite Remediation Lifecycle

RFC 6561 formalizes a tripartite lifecycle for handling infected broadband subscriber lines:

```
[Phase 1: Detection & Attribution]
High-confidence Bayesian compromise confirmation -> Subscriber mapped to BNG / RADIUS identity
                    |
                    v
[Phase 2: Isolation & Notification (Walled Garden)]
Subscriber quarantined:
  - Malicious traffic blocked at edge
  - Transparent HTTP/DNS redirection to Notification Portal
  - Remediation whitelists enabled
                    |
                    v
[Phase 3: Remediation & Verification]
Subscriber initiates cleanup / updates -> Verification probes confirm host is clean -> Full access restored
```

### Policy Requirements from RFC 6561:
1. **Targeted Containment:** Containment policies must target the specific malicious behavior (e.g., blocking outbound port 23/2323, port 25, or high-rate UDP) while keeping communication channels open for legitimate remediation.
2. **User Education:** The subscriber must be presented with clear, actionable explanations detailing why access is restricted, which device is infected, and what exact steps are required to remediate.
3. **Safe Harbor Access:** The walled garden must never isolate the subscriber completely from the Internet; critical update repositories and antivirus signatures must remain accessible.

---

## 2. Enforcement Mechanisms: DNS RPZ vs. HTTP 302 DNAT vs. BGP Flowspec

Carriers evaluate three primary network redirection and filtering primitives:

| Dimension | DNS Response Policy Zones (RPZ) | HTTP 302 Destination NAT (DNAT) | BGP Flowspec (RFC 5575 / RFC 8955) |
| :--- | :--- | :--- | :--- |
| **Layer of Operation** | Layer 7 (Application - DNS resolver). | Layer 3/4 (Transport - Router / Firewall). | Layer 3/4 (Routing plane across BNG/Edge). |
| **Mechanics** | Recursive DNS server rewrites domain queries (A/AAAA) for quarantined hosts to resolve to the ISP portal IP. | Access router intercepts TCP/80 and TCP/443 packets and DNATs them to the quarantine web server. | Route reflector distributes precise filtering rules (rate-limit, redirect-to-VRF, or drop) to BNG routing engines. |
| **Bypass Vulnerability** | **High.** Easily bypassed if the subscriber configures third-party public resolvers (Google `8.8.8.8`, Cloudflare `1.1.1.1`) or uses DNS-over-HTTPS (DoH / RFC 8484). | **Low.** Operates at the network layer. Subscriber cannot bypass by altering DNS settings. | **Zero.** Enforced directly in hardware ASIC / TCAM forwarding tables of the BNG. |
| **HTTPS Impact** | Causes browser TLS certificate mismatches if the portal IP serves a different certificate than the requested domain. | Triggers severe HSTS / TLS certificate warnings on encrypted sites (unless Captive Portal API RFC 8908 is supported). | Drops or rate-limits specific abusive traffic without intercepting TLS sessions. |
| **Scale & CPU Cost** | Zero router CPU cost; handled by DNS software (BIND9 / Knot Resolver / PowerDNS). | Moderate router CPU / state table consumption on legacy BNGs. | Near zero CPU impact; executes in line-rate hardware forwarding tables. |
