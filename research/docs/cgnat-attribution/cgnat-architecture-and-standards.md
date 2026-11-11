# Carrier-Grade NAT (CGNAT) Architecture & Standards

This dossier details the standards, address space allocations, and operational architectures of Large-Scale Carrier-Grade Network Address Translation (CGNAT/LSN) in ISP access networks.

---

## 1. Regulatory & Standards Foundations

### RFC 6598 (IANA-Reserved IPv4 Prefix for Shared Address Space)
- **Prefix Allocated:** `100.64.0.0/10` (Total: 4,194,304 addresses, spanning `100.64.0.0` to `100.127.255.255`).
- **Standard Purpose:** Dual-stack transition and residential subscriber aggregation without colliding with private internal RFC 1918 networks (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) or publicly routable unicast space.
- **Routing Rules:** Strictly non-routable on the global public Internet. ISP edge routers must drop inbound and outbound advertisements of `100.64.0.0/10` across external BGP peerings (RFC 6598 Section 3).

### RFC 6888 (Common Requirements for Carrier-Grade NATs)
Carrier-Grade NAT differs fundamentally from Customer Premises Equipment (CPE) NAT:
- **Hairpinning Requirement (REQ-1):** The CGNAT must support hairpinning, allowing two subscribers behind the same CGNAT public IP pool to communicate directly via their translated public endpoints.
- **Address Pooling (REQ-2):** Must support "Paired" address pooling (all concurrent sessions from a specific internal IP map to the same external public IP) to prevent breaking multi-connection protocols like SIP, FTP, and banking portals.
- **Port Preservation & Parity (REQ-3):** Port parity (preserving even/odd port alignment for RTP/RTCP) must be maintained where possible.
- **Session Lifetimes (REQ-4):**
  - Established TCP sessions: Minimum idle timeout of **2 hours** (120 minutes).
  - Transitory TCP sessions (SYN sent, FIN wait): 4 minutes.
  - UDP sessions: Minimum idle timeout of **5 minutes** (RFC 4787 recommendation: 2 minutes minimum, 5 minutes default).
  - ICMP query sessions: Minimum idle timeout of **60 seconds**.

---

## 2. Translation Allocation Models: Dynamic vs. PBA vs. Deterministic

The choice of port allocation model fundamentally dictates the logging volume, attribution feasibility, and subscriber capacity.

| Dimension | Dynamic NAT44 (Per-Session) | Port Block Allocation (PBA / MAP-E style) | Deterministic NAT (RFC 7422) |
| :--- | :--- | :--- | :--- |
| **Port Mapping Granularity** | 1:1 per transport layer flow. Each outbound TCP/UDP flow is dynamically assigned an ephemeral port from the pool. | Chunk-based. A contiguous block of ports (e.g., 256, 512, or 1024 ports) is pre-allocated to a subscriber upon first connection. | Fixed mathematical formula. Subscriber private IP maps statically to a predefined public IP and port range. |
| **Logging Frequency** | **Every connection establishment and termination.** Produces 2,000 to 5,000 log records per day per subscriber. | **Only block allocation and release events.** Typically 2 to 6 log records per subscriber per day. | **Zero per-session logs.** Translation mapping is computed algorithmically via static formula: $Port_{start} = Offset + (IP_{private} \pmod N) 	imes BlockSize$. |
| **Storage & I/O Overhead** | **Extreme (Terrabyte-scale).** Unusable for Tier-2/3 ISPs without massive distributed database clusters. | **Minimal (Megabyte-scale).** Over 99.9% reduction in I/O and storage requirements while retaining absolute attribution. | **Zero log overhead.** However, mathematically rigid and inflexible. |
| **Port Exhaustion Resilience** | High statistical multiplexing efficiency. Idle ports return to the global pool immediately. | Good efficiency. Inactive subscribers consume a block; can implement secondary dynamic blocks on demand. | **Very Poor.** Heavy users cannot exceed their fixed quota even if 90% of pool ports are idle. Breaks modern web apps. |
| **Attribution Precision** | Exact down to the millisecond, provided high-speed logging does not drop packets during peak bursts. | **Exact.** Intersecting public IP + port with block allocation time span identifies the exact private subscriber IP. | Exact via static reverse calculation. Lacks dynamic session history. |

---

## 3. CGNAT Topology & Placement in Regional Networks

In regional Tier-2/3 FTTH networks, CGNAT is deployed in one of two configurations:

```
Centralized CGNAT Model:
[Subscribers / OLTs] ---> [BNG (MikroTik/Cisco)] ---(100.64.0.0/10)---> [Core CGNAT Appliance] ---(Public IPs)---> [Upstream Transit]
                                                                        ^
                                                                        | High-Volume Logging
                                                                   [Attribution Engine]

Distributed BNG-Colocated Model:
[Subscribers / OLTs] ---> [BNG with Integrated CGNAT] ---(Public IPs)---> [Upstream Transit]
                                 ^
                                 | NetFlow IPFIX / Syslog PBA
                           [Attribution Engine]
```

### Centralized Core CGNAT:
- Deployed between the aggregation BNGs and the upstream transit border routers.
- High port multiplexing efficiency across multiple regional PoPs.
- Concentrates high-bandwidth logging onto a dedicated appliance (e.g., A10 Networks Thunder, F5 BIG-IP, or carrier-grade Linux DPDK appliances).

### Distributed BNG-Colocated CGNAT:
- Implemented directly on the access BNG (e.g., MikroTik RouterOS `action=src-nat` with port-block jump rules, or Cisco ASR Service Modules).
- Reduces core transit encapsulation overhead.
- Requires robust, low-latency log export from edge routers to prevent CPU degradation on the routing plane.
