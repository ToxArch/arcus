# Captive Portal API Standards: RFC 8908 & RFC 8910

This dossier details the modern IETF standards developed to resolve the severe user experience failures, TLS certificate warnings, and HSTS lockouts caused by legacy captive portal interception.

---

## 1. The HTTPS Interception Breakdown

In legacy broadband networks, captive portals intercept outbound HTTP/HTTPS traffic using Destination NAT (DNAT) or transparent HTTP proxies. In the modern encrypted web, this produces catastrophic failures:

```
[Client Browser] ---(GET https://google.com)---> [Legacy BNG Interceptor]
                                                              |
                                           (Rewrites to Portal 10.0.0.1)
                                                              |
                                                              v
[Client Receives Certificate for portal.isp.com instead of google.com]
                                                              |
                                                              v
           [BROWSER BLOCKS WITH SEVERE SECURITY ERROR]
           "Your connection is not private (NET::ERR_CERT_COMMON_NAME_INVALID)"
```

- **HSTS (RFC 6797) Enforcement:** Modern browsers strictly prohibit users from clicking past certificate errors on domains employing HTTP Strict Transport Security (HSTS).
- **Result:** The subscriber sees only broken websites, cannot reach the ISP notification portal, assumes the network is completely dead, and floods customer support centers.

---

## 2. RFC 8910: Captive-Portal DHCP & IPv6 RA Options

IETF RFC 8910 solves discovery without packet tampering by enabling access routers to explicitly inform the client operating system of the captive portal endpoint:

- **DHCPv4 Option 114:** Specifies a URI pointing to the captive portal API:
  - Option Code: `114` (Captive-Portal).
  - Value: `https://portal.isp.net/api/captive-portal`.
- **IPv6 Router Advertisement (RA) Option 37:** Carries the identical URI in standard IPv6 neighbor discovery advertisements.
- **Operating System Reaction:**
  - Upon receiving Option 114, modern OS clients (Windows 10/11, macOS, iOS, Android, Linux NetworkManager) recognize that captive network conditions exist.
  - The OS pauses standard application networking and directly contacts the specified API endpoint without waiting for browser user interaction.

---

## 3. RFC 8908: Captive Portal API Specification

RFC 8908 defines the RESTful JSON schema exchanged between client operating systems and the portal:

```json
{
  "captive": true,
  "user-portal-url": "https://portal.isp.net/remediation?session=92b7c41a",
  "venue-info-url": "https://isp.net/support",
  "seconds-remaining": 3600,
  "can-extend-session": false
}
```

### OS Captive Network Assistant (CNA) Handling:
1. When `captive: true` is returned, the client operating system automatically spawns a dedicated system browser window (the **Captive Network Assistant** / WebSheet).
2. The CNA loads `user-portal-url` natively.
3. The subscriber is immediately shown the security advisory:
   - Specific infected device MAC address and hostname.
   - Nature of compromise (e.g., Mirai Telnet scanner, BadBox proxy).
   - Step-by-step remediation guide and links to authorized removal utilities.
4. Once the verification probe passes, the API returns `captive: false`, and the OS automatically closes the sheet and restores full Internet routing.
