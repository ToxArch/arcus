# Suricata Line-Rate Packet Inspection Architecture

This dossier documents the architectural principles, packet acquisition engines, and stream reassembly configurations required for deterministic line-rate Deep Packet Inspection (DPI) in multi-gigabit carrier aggregation environments.

---

## 1. Threading Architecture: Runmodes Comparison

Suricata supports three primary threading execution modes:

```
Workers Runmode Architecture:
[NIC RX Queue 0] ---> [Thread 0: Decode -> Stream -> Detect -> Output] (Pinned to Core 0)
[NIC RX Queue 1] ---> [Thread 1: Decode -> Stream -> Detect -> Output] (Pinned to Core 1)
[NIC RX Queue 2] ---> [Thread 2: Decode -> Stream -> Detect -> Output] (Pinned to Core 2)
[NIC RX Queue 3] ---> [Thread 3: Decode -> Stream -> Detect -> Output] (Pinned to Core 3)
```

| Dimension | `workers` Runmode | `autofp` (Automatic Flow Pinning) | `single` Runmode |
| :--- | :--- | :--- | :--- |
| **Pipeline Model** | Unified pipeline. Each thread executes packet capture, decoding, stream tracking, signature detection, and alert logging autonomously. | Decoupled pipeline. Dedicated capture threads receive packets, hash flow tuples, and dispatch packets to worker threads via mutex queues. | Single-threaded. All execution occurs on one CPU core. |
| **Context Switching Overhead** | **Near Zero.** Zero inter-thread locks or mutex contention. Packets remain in the L1/L2 cache of the assigned CPU core throughout their lifecycle. | **High.** Severe inter-thread lock contention and cache invalidation as packets are passed between capture threads and worker queues. | Zero (restricted to single core capacity). |
| **Throughput Scaling** | Linear scaling ($O(N)$ with number of physical CPU cores). The industry standard for carrier deployments. | Sub-linear scaling; bottlenecks at 5-8 Gbps due to memory queue locking. | Bound to ~1 Gbps maximum. |
| **Hardware Requirement** | Requires a multi-queue NIC supporting **Hardware RSS (Receive Side Scaling)** to hash flows across RX queues. | Operates on single-queue legacy NICs. | Development/testing only. |

---

## 2. Kernel Bypass & Packet Acquisition: AF_PACKET vs. DPDK

Capturing 10 Gbps to 40 Gbps of transit traffic without packet drops requires bypassing standard Linux kernel socket overhead:

| Acquisition Mechanism | Ring Buffer Mechanism | CPU Utilization | Zero-Copy Capability | Driver Constraints |
| :--- | :--- | :--- | :--- | :--- |
| **Standard `libpcap` / `AF_INET`** | Kernel socket buffer (`sk_buff`) with memory copies. | 100% CPU on interrupt storms; drops packets > 1 Gbps. | No. Double copy (NIC -> Kernel -> User space). | Generic Linux network drivers. |
| **`AF_PACKET` (v3) + `PACKET_MMAP`** | Circular ring buffer shared between Linux kernel and user space (`mmap`). | Moderate (uses hardware FANOUT flow hashing). Handles 5-15 Gbps on modern Xeon/EPYC. | **Yes (`PACKET_MMAP`).** Eliminates kernel-to-user copy. | Supported natively by all standard Linux network drivers. |
| **DPDK (Data Plane Development Kit)** | Complete kernel bypass. Dedicated user-space poll-mode drivers (PMD) polling NIC PCIe registers directly. | 100% continuous polling on assigned cores (dedicated spinning threads). Sustains 40-100 Gbps line-rate. | **Absolute Zero-Copy.** User-space memory pools (`mbuf`) allocated from hugepages. | Requires DPDK-compliant hardware (Intel 82599, X520, X710, E810; Mellanox ConnectX-4/5/6). |

---

## 3. Hyperscan Multi-Pattern Matcher (MPM) Acceleration

Signature evaluation across tens of thousands of rules (Emerging Threats Pro / custom ISP rules) is computationally prohibitive using standard string searching algorithms (Aho-Corasick or Boyer-Moore):
- **Intel Hyperscan Integration:** Suricata utilizes Hyperscan, a high-performance regular expression matching library developed by Intel:
  - Uses Non-deterministic Finite Automata (NFA) and Deterministic Finite Automata (DFA) graph representations.
  - Leverages **Intel AVX2, AVX-512, and ARM Neon SIMD vector instructions** to evaluate hundreds of regex patterns simultaneously in a single CPU cycle.
  - **Empirical Throughput:** Provides a **3.8x to 5.2x speedup** in signature matching latency compared to legacy Aho-Corasick algorithms.

---

## 4. Asymmetric Midstream Flow Handling

In ISP transit aggregation topologies, routing asymmetry is common: outbound packets travel across Transit Provider A, while return inbound packets return via Transit Provider B.
- If Suricata monitors only one transit interface, it observes only unidirectional half-flows.
- **Midstream Session Pickup:** Suricata's TCP stream engine must be configured with `midstream: true` and `async-oneside: true`.
- This ensures the engine does not drop un-handshaked TCP packets, enabling signature matching on malicious payload bytes even when initial TCP SYN/SYN-ACK handshakes occurred over an unmonitored transit link.
