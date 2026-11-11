# Residential Proxies & Covert Abuse Networks

This dossier analyzes residential proxy networks (e.g., BadBox, 911 S5, IPIDEA, Proxy-in-a-box) that turn legitimate subscriber broadband connections into exit nodes for cybercriminal operations.

---

## 1. Threat Mechanics & Architecture

Residential proxies differ fundamentally from traditional botnets:
- Traditional botnets (Mirai) conduct high-volume, noisy actions (DDoS floods, mass Telnet brute-forcing).
- **Residential proxy agents operate silently:** They serve as relay proxies (SOCKS5 / HTTP CONNECT tunnels).
- Cybercriminals route credential stuffing attacks, financial fraud, CAPTCHA harvesting, and ad fraud through the subscriber's residential IP to bypass anti-fraud filters that trust residential Autonomous System Numbers (ASNs).

```
[Threat Actor / Fraudster]
           |
           v SOCKS5 / TLS Tunnel
[Proxy C2 / Dispatcher Server]
           |
           v Outbound Tunnel Encapsulation
[Compromised Subscriber ONT / TV Box] (Pre-NAT: 100.64.12.34)
           |
           v Clear HTTP/TLS Traffic
[Victim Bank / Web Portal / Target Service]
```

---

## 2. BadBox Firmware Supply-Chain Compromise

The **BadBox** malware operation represents one of the largest supply-chain compromises in residential broadband history:
- **Infection Vector:** Budget Android Open Source Project (AOSP) streaming media players and TV boxes (e.g., AllWinner, Rockchip models sold under brand names like T95, MXQ Pro).
- **Firmware Implant:** Devices arrive from factories with trojanized system binaries pre-flashed in ROM. The primary payload replaces `/system/bin/debuggerd` or hooks `system_server`.
- **Persistence:** Because the malware resides in the read-only `/system` partition, standard Android factory resets **do not remove the infection**.
- **Operation:** Upon boot, the device contacts command-and-control servers, joins a global residential proxy grid, and begins relaying external web requests without the subscriber's knowledge.

---

## 3. Mathematical Detection: Upload/Download Ratio Inversion

Residential broadband traffic is overwhelmingly asymmetric:
- Normal residential subscriber behavior consists of video streaming (Netflix, YouTube), web browsing, and downloading files.
- **Empirical Baseline Residential Ratio:**
  $$\text{Ratio}_{\text{baseline}} = \frac{\text{Bytes}_{\text{down}}}{\text{Bytes}_{\text{up}}} \approx 8:1 \text{ to } 15:1$$
  $$\text{Inverted Ratio} = \frac{\text{Bytes}_{\text{up}}}{\text{Bytes}_{\text{down}}} \approx 0.06 \text{ to } 0.12$$

### Proxy Relay Traffic Inversion Proof:
When a compromised subscriber device acts as a proxy exit node:
1. The proxy client forwards incoming requests from the C2 to the target server.
2. The target server returns response data (e.g., HTML, API responses, images) to the device.
3. The device immediately uploads this exact response back over the tunnel to the threat actor.

For every byte downloaded from the victim server, exactly one byte must be uploaded back to the proxy controller:
$$\text{Bytes}_{\text{outbound}} \ge \text{Bytes}_{\text{inbound}}$$
$$\text{Ratio}_{\text{proxy}} = \frac{\text{Bytes}_{\text{up}}}{\text{Bytes}_{\text{down}}} \ge 1.0 \quad (\text{often } 1.2:1 \text{ to } 2:1 \text{ due to tunnel encapsulation})$$

```
Residential vs. Proxy Flow Profile:
Normal FTTH:  [====================] Down (90%)  [==] Up (10%)
Proxy Node:   [==========] Down (45%)             [===========] Up (55%) -> INVERSION DETECTED
```

### Sustained Connection Cardinality:
In addition to ratio inversion, residential proxies maintain an abnormally high number of concurrent external TCP sessions to diverse financial, e-commerce, and social media domain endpoints:
- A normal residential smart TV connects to 3-5 distinct CDN streaming endpoints.
- A BadBox proxy node opens connections to **500 to 3,000 distinct public destinations per hour**.
