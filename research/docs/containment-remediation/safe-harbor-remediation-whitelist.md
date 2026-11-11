# Safe Harbor Remediation Whitelists

This dossier specifies the essential domains, IP prefixes, and update repositories that must remain fully accessible to quarantined subscribers to enable autonomous system remediation.

---

## 1. Principles of Safe Harbor Whitelisting

When a residential subscriber is placed into RFC 6561 quarantine:
- Complete isolation makes self-remediation impossible, forcing technicians to visit the premises or requiring the subscriber to use alternate mobile data connections.
- A **Safe Harbor Whitelist** provides unrestricted outbound access to verified security update infrastructure while continuing to block arbitrary Internet browsing and abusive traffic.

---

## 2. Essential Whitelist Catalog

| Vendor / Platform | Service Category | Mandatory Domains / IP Endpoints | Port / Protocol |
| :--- | :--- | :--- | :--- |
| **National CERT / Government** | National Botnet Cleaning | `csk.gov.in`, `cyberswachhtakendra.gov.in`, `cert-in.org.in` | TCP 80, 443 |
| **Microsoft Windows** | OS Security & Windows Defender | `windowsupdate.microsoft.com`, `update.microsoft.com`, `download.windowsupdate.com`, `wdcp.microsoft.com` | TCP 80, 443 |
| **Apple iOS / macOS** | OS Updates & Gatekeeper | `swscan.apple.com`, `swcdn.apple.com`, `updates.cdn-apple.com`, `ocsp.apple.com` | TCP 80, 443 |
| **Google Android** | Google Play Protect & AOSP | `play.google.com`, `android.clients.google.com`, `update.googleapis.com` | TCP 443 |
| **Antivirus Signatures** | Quick Heal / Seqrite | `download.quickheal.com`, `update.quickheal.com` | TCP 80, 443 |
| **Antivirus Signatures** | Kaspersky | `downloads.kaspersky-labs.com`, `dnl-*.kaspersky-labs.com` | TCP 80, 443 |
| **Antivirus Signatures** | Malwarebytes | `data-cdn.mbamupdates.com`, `telemetry.malwarebytes.com` | TCP 443 |
| **Linux / IoT Repositories** | Ubuntu / Debian Security | `security.ubuntu.com`, `archive.ubuntu.com`, `security.debian.org` | TCP 80, 443 |

---

## 3. Enforcement Architecture: DNS Split-Horizon & IP Passthrough

Safe harbor access is enforced via a two-layer mechanism:
1. **DNS Split-Horizon:** The walled-garden DNS server resolves all arbitrary domain names to the local remediation portal IP, **except** domains matching the Safe Harbor Whitelist, which are resolved via standard recursive resolution.
2. **Access Control Lists (ACLs):** Access routers and BNGs maintain an IP prefix whitelist allowing outbound TCP/80 and TCP/443 traffic toward major CDN prefixes (Akamai, Cloudflare, Fastly, Microsoft Azure) serving these update services.
