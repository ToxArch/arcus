# Hardware Sizing & Capacity Specifications

This dossier provides empirical hardware capacity sizing matrices, compute allocations, and storage calculations for deploying the detection and attribution architecture across three standard regional ISP tiers: Tier-A (5,000 subscribers), Tier-B (15,000 subscribers), and Tier-C (30,000 subscribers).

---

## 1. Traffic & Ingestion Workload Profiles

| Dimension | Tier-A (5,000 Subs) | Tier-B (15,000 Subs) | Tier-C (30,000 Subs) |
| :--- | :--- | :--- | :--- |
| **Aggregated Peak Transit Throughput** | 2.5 Gbps to 5.0 Gbps | 8.0 Gbps to 15.0 Gbps | 18.0 Gbps to 30.0 Gbps |
| **Unsampled Flow Export Rate (IPFIX)** | 6,250 flows/sec | 18,750 flows/sec | 37,500 flows/sec |
| **Raw DPI Mirror Bandwidth (10% SPAN)** | 500 Mbps (symmetric) | 1.5 Gbps | 3.0 Gbps |
| **CGNAT PBA Event Rate (Average)** | 0.35 events/sec | 1.05 events/sec | 2.10 events/sec |
| **RADIUS Accounting Packet Rate** | ~18 packets/sec | ~55 packets/sec | ~110 packets/sec |

---

## 2. Server Hardware Sizing Matrix (Single 1U/2U Appliance)

| Hardware Component | Tier-A Specification | Tier-B Specification | Tier-C Specification |
| :--- | :--- | :--- | :--- |
| **Form Factor** | 1U Rackmount Server | 1U/2U Rackmount Server | 2U Rackmount Server |
| **Processor (CPU)** | 1x AMD EPYC 4344P (8 cores / 16 threads, 3.8 GHz) or Intel Xeon E-2388G | 1x AMD EPYC 8324P (32 cores / 64 threads) or Intel Xeon Silver 4410Y | 2x AMD EPYC 9124 (2x 16 cores / 64 threads) or 1x EPYC 9354 (32c/64t) |
| **System Memory (RAM)** | 32 GB DDR5 ECC (4800 MT/s) | 64 GB DDR5 ECC (4800 MT/s) | 128 GB DDR5 ECC (4800 MT/s, 8-channel) |
| **Network Interface Cards (NIC)** | 1x Dual-Port 10G SFP+ (Intel X520-DA2 or X710-DA2) | 1x Quad-Port 10G/25G SFP28 (Intel E810-XXVDA4 or Mellanox ConnectX-5) | 1x Dual-Port 40G/100G QSFP28 (Mellanox ConnectX-5 / ConnectX-6 Dx) |
| **Hot Storage (NVMe SSD)** | 2x 960 GB NVMe PCIe 4.0 in RAID 1 (Read-Intensive, 1 DWPD) | 2x 1.92 TB NVMe PCIe 4.0 in RAID 1 (Mixed-Use, 3 DWPD) | 4x 1.92 TB NVMe PCIe 4.0 in RAID 10 (Enterprise, 3 DWPD) |
| **Cold Storage (Archive HDD)** | 2x 4 TB Enterprise SATA HDD in RAID 1 (ZFS mirror) | 4x 8 TB Enterprise SATA HDD in RAID 10 (ZFS mirror) | 4x 16 TB Enterprise SATA HDD in RAID 10 (ZFS pool) |
| **Power Supply** | Redundant 550W 80 PLUS Platinum | Redundant 750W 80 PLUS Platinum | Redundant 1100W 80 PLUS Titanium |

---

## 3. Subsystem Core & Resource Allocations

To guarantee deterministic zero-packet-drop performance, CPU cores are strictly partitioned using Linux CPU isolation (`isolcpus`, `taskset`):

```
Tier-B 32-Core CPU Allocation Map:
[Core 0-1]   : Linux OS, System Daemons, NPLI NTP Daemon, SSH Management
[Core 2-5]   : ClickHouse Analytical Storage Engine (SIMD Vector Execution)
[Core 6-7]   : GoFlow2 Unsampled IPFIX Collector + Kafka / Redis Stream Ingestion
[Core 8-9]   : RADIUS Accounting Collector + Two-Tier Inversion Engine
[Core 10-11] : Bayesian Log-Odds Fusion Pipeline + SLA State Machine
[Core 12-27] : Suricata DPI Workers Runmode (16 pinned worker threads, AF_PACKET MMAP)
[Core 28-31] : Reserve Headroom / Background Compression Merge Tasks
```

### Storage Endurance & IOPS Requirements:
- **ClickHouse Ingestion IOPS:** Under PBA and sampled flow rollups, continuous write IOPS remains below **800 IOPS**, well within the 50,000+ random write IOPS delivered by enterprise NVMe SSDs.
- **Drive Write Endurance:** At 15,000 subscribers, total daily write volume across all engines is approximately **18 GB/day**. A standard 1.92 TB NVMe drive rated at 1 DWPD (1.92 TB/day) operates at less than **1% of its daily write endurance limit**, ensuring a multi-year hardware lifespan.
