# Broadband Access Network Topologies

This dossier examines the access layer infrastructure terminating residential broadband subscribers across regional Tier-2/3 Indian FTTH networks.

## 1. BNG / BRAS Architecture

The Broadband Network Gateway (BNG), historically termed the Broadband Remote Access Server (BRAS), represents the demarcation point between the Layer-2 access network (optical distribution) and the Layer-3 routed IP core.

### Core BNG Functions:
- Subscriber session establishment, authentication, and teardown.
- Policy enforcement, Quality of Service (QoS) bandwidth shaping, and rate limiting.
- IP address assignment (internal private IPv4 and public/private IPv6 delegations).
- Accounting data generation dispatched to central AAA servers via RADIUS.

In Tier-2 and Tier-3 Indian networks, the dominant BNG platform is the **MikroTik Cloud Core Router series (CCR2004, CCR2116, CCR1072)** running RouterOS v7, alongside legacy Cisco ASR 1000/9000 and Huawei NE40E routers deployed by Class-B operators.

---

## 2. PPPoE vs. IPoE Statefulness Comparison

| Dimension | PPPoE (RFC 2516) | IPoE (DHCP-based Access) |
| :--- | :--- | :--- |
| **Encapsulation Overhead** | 8 bytes (PPP 2 bytes + PPPoE 6 bytes); requires MTU clamping to 1492 bytes (`mss-clamping 1452`). | 0 bytes; standard 1500-byte MTU supported natively. |
| **Session Statefulness** | Highly stateful; active PPP keepalive (LCP Echo-Request/Reply). Disconnect is instantaneous upon link-down. | Stateless at Layer 2; relies on DHCP lease time and ARP/NDP polling. Session termination can lag by hours if ungraceful. |
| **Authentication Flow** | Integrated PAP/CHAP during PPP negotiation with BNG acting as RADIUS client. | Out-of-band via DHCP Option 82 injection or web captive portal. |
| **Subscriber Attribution Precision** | **Absolute and Cryptographic:** Every IP packet is bound to a unique 16-bit PPPoE `Session-ID` on the BNG interface. IP spoofing between subscribers is physically impossible at Layer 2. | **Vulnerable without Strict L2 Security:** Requires Dynamic ARP Inspection (DAI), DHCP Snooping, and IP Source Guard (IPSG) on access switches/OLTs; otherwise, rogue CPEs can spoof neighbors' IPs. |
| **Regional ISP Adoption** | **~85% of Tier-2/3 Indian FTTH** networks. | ~15% (primarily high-end metro FTTH and enterprise leased circuits). |

---

## 3. GPON / EPON Optical Access Layer

Regional broadband operators deploy Gigabit Passive Optical Networks (GPON, ITU-T G.984) and Ethernet PON (EPON, IEEE 802.3ah):
- **Optical Distribution Network (ODN):** Optical splitters (typically 1:32 or 1:64) aggregate single-fiber drops from customer premises ONTs into Optical Line Terminals (OLTs) at regional Points of Presence (PoPs).
- **VLAN Provisioning Models:**
  - **1:1 VLAN (VLAN per Subscriber):** Each ONT port is mapped to a dedicated Customer VLAN (C-VLAN) encapsulated inside a Service VLAN (S-VLAN) via QinQ (`802.1ad`). Delivers absolute Layer-2 isolation; the BNG interface maps 1:1 to a physical customer line.
  - **N:1 VLAN (Shared VLAN):** Multiple ONTs share a single broadcast domain. Requires the OLT to implement **PPPoE Intermediate Agent (PPPoE-IA)** or **DHCP Option 82** injection, inserting vendor tags (`Circuit-ID` containing OLT chassis, slot, PON port, and ONT ID) into access frames.
