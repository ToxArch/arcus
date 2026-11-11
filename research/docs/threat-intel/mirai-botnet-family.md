# Mirai Botnet Family Disassembly & Network Signatures

This dossier documents the binary architecture, algorithmic scanning mechanisms, and observable telemetry profiles of the Mirai IoT botnet family, derived from binary disassemblies and the canonical source repository leaked in September 2016 by author *Anna-senpai*.

---

## 1. Modular Architecture Overview

Mirai is engineered specifically for cross-compilation across diverse embedded microprocessor architectures:
- MIPS (Big Endian and Little Endian - `mipsel`)
- ARM (ARMv4, ARMv5, ARMv7)
- SuperH (`sh4`)
- PowerPC (`ppc`)
- SPARC
- x86 / x86_64

The binary payload is divided into three core subsystems:
1. **`scanner.c` (Worm Propagation Subsystem):** High-speed asynchronous SYN prober targeting Telnet service ports (TCP/23 and TCP/2323).
2. **`killer.c` (Process Elimination & Host Dominance):** Sweeps `/proc/$PID/exe`, `/proc/$PID/fd`, and `/proc/$PID/maps` to terminate competing botnet binaries (e.g., Qbot/Bashlite) and bound services on ports 22, 23, and 80.
3. **`attack.c` (DDoS Weaponization Engine):** Implements raw socket flooding routines:
   - `attack_udp_generic` / `attack_udp_vse` (Valve Source Engine query amplification).
   - `attack_udp_dns` (DNS request flooding with pseudo-random subdomains).
   - `attack_tcp_syn` / `attack_tcp_ack` (TCP state table exhaustion).
   - `attack_tcp_stomp` (ACK flood with realistic PSH/ACK payload).
   - `attack_gre_ip` / `attack_gre_eth` (Generic Routing Encapsulation flooding).

---

## 2. Scanning Engine (`scanner.c`) Mechanics

### Pseudo-Random IP Address Generation:
Mirai avoids generating uniform 32-bit random integers across the full IPv4 space. Instead, it utilizes a custom Linear Congruential Generator (LCG) and strictly filters out non-target, military, government, and internal subnets.

```
Random IP Candidate (32-bit uint)
          |
          v
[Exclusion Filter Check]
Is candidate within:
  - 127.0.0.0/8    (Loopback)
  - 0.0.0.0/8      (Local identification)
  - 10.0.0.0/8     (RFC 1918 Private)
  - 172.16.0.0/12  (RFC 1918 Private)
  - 192.168.0.0/16 (RFC 1918 Private)
  - 100.64.0.0/10  (RFC 6598 Carrier-Grade NAT)
  - 6.0.0.0/8      (US Army ISC)
  - 7.0.0.0/8      (DoD)
  - 11.0.0.0/8     (DoD Intel)
  - 21.0.0.0/8     (DoD DDN)
  - 22.0.0.0/8     (DoD DISA)
  - 26.0.0.0/8     (DoD DIA)
  - 28.0.0.0/8     (DoD DSI)
  - 29.0.0.0/8     (DoD DSI)
  - 30.0.0.0/8     (DoD DLA)
  - 33.0.0.0/8     (DoD DLA)
  - 55.0.0.0/8     (US Army Information Systems)
  - 56.0.0.0/8     (US Postal Service)
          |
    +-----+-----+
    |           |
 [MATCH]    [NO MATCH]
    |           |
 [DISCARD]   [TARGET DISPATCH]
```

### Critical Detection Implication:
Notice that `100.64.0.0/10` (CGNAT shared space) is explicitly excluded in the original Mirai code! When an infected CPE scans, **all scanner traffic is directed outbound toward public IPv4 ranges** across the ISP transit link. The internal BNG never sees internal-to-internal scanning; it observes massive outbound SYN floods on ports 23 and 2323.

---

## 3. Credential Brute-Forcing Profile

Mirai contains **68 hardcoded credential pairs** obfuscated via a single-byte XOR key (`0xDE` or `0xDEADBEEF` depending on variant):
- Target defaults: `root:xc3511`, `root:vizxv`, `root:admin`, `admin:admin`, `root:888888`, `root:default`, `root:juantech`, `root:123456`, `root:54321`, `support:support`.
- **State Machine:**
  1. Issues TCP SYN to port 23/2323.
  2. Upon TCP 3-way handshake completion, waits for prompt characters (`:`, `>`, `#`, `$`).
  3. Sends username string, reads buffer, sends password string.
  4. Upon successful shell acquisition, transmits verification token `sh` followed by `/bin/busybox MIRAI` or `cat /bin/busybox`.
  5. Dispatches compromised victim IP, port, and credentials to a central **Scan Receiver (Loader)** via raw socket or HTTP POST.

---

## 4. Line-Rate Network Signatures

When an ISP subscriber device is compromised by Mirai:
1. **Flow Asymmetry Inversion:** Extreme ratio of outbound TCP SYN packets to inbound SYN-ACK or RST packets (often >50:1 due to unresponsive target hosts).
2. **Distinctive TCP Window Size:** Scanners frequently use a fixed TCP window size (e.g., `512` or `1024` bytes) with zero TCP options (no Window Scale, no SACK-Permitted).
3. **Payload Signature:** The Telnet authentication sequence and BusyBox verification tokens (`/bin/busybox`) match fixed byte sequences inspectable via Suricata DPI rules.
