# ClickHouse Columnar Storage & Telemetry Query Optimization

This dossier analyzes the columnar database architecture, storage engine choices, and compression mechanics required to ingest, store, and query billions of ISP telemetry and attribution records.

---

## 1. Columnar vs. Row-Oriented Storage Economics

In network traffic analysis, analytical queries typically aggregate over a small subset of columns (e.g., `SUM(bytes)`, `COUNT(DISTINCT dst_ip)`) across billions of rows.

```
Row-Oriented Engine (PostgreSQL / MySQL):
Row 0: [Timestamp, SrcIP, DstIP, SrcPort, DstPort, Bytes, Packets, Flags]
Row 1: [Timestamp, SrcIP, DstIP, SrcPort, DstPort, Bytes, Packets, Flags]
-> Querying 'Bytes' requires scanning ALL columns of every row from disk!

Column-Oriented Engine (ClickHouse):
Timestamp: [T0, T1, T2, T3, T4, ...]
SrcIP:     [S0, S1, S2, S3, S4, ...]
Bytes:     [B0, B1, B2, B3, B4, ...]
-> Querying 'Bytes' reads ONLY the contiguous 'Bytes' block from disk!
```

### Advantages for ISP Telemetry:
1. **SIMD Vectorized Execution:** ClickHouse processes data in vectorized blocks (typically 65,536 values per vector), executing CPU vector instructions (AVX-512) directly over columnar arrays.
2. **Extreme Compression Codecs:** Because similar data types are stored contiguously:
   - IP addresses compressed via **DoubleDelta** or **Gorilla** codecs achieve **85-92% size reduction**.
   - Timestamps compressed via **DoubleDelta + ZSTD(1)** achieve **95% size reduction**.
   - Raw disk footprint is **6x to 10x smaller** than PostgreSQL or Elasticsearch.

---

## 2. Table Engine Architecture: ReplacingMergeTree & AggregatingMergeTree

### `ReplacingMergeTree` for CGNAT PBA State:
CGNAT port block allocation records have a lifecycle (allocation followed by release). Using `ReplacingMergeTree(version)` allows background merging to automatically deduplicate and update records with the latest state without locking tables.

### `AggregatingMergeTree` for 15-Minute Flow Rollups:
Raw 1:1 NetFlow / IPFIX streams produce hundreds of millions of rows per day. Long-term trend analysis and compliance audits require rolling these into time-bucketed summaries:
- ClickHouse **Materialized Views** aggregate flows into 15-minute windows on-the-fly:
  - `AggregateFunction(sum, UInt64)` for total bytes and packets.
  - `AggregateFunction(uniqExact, IPv4)` for distinct destination cardinality (detecting scanning worms).
- Ingestion query performance: Aggregated queries execute across millions of buckets in **sub-50 milliseconds**.

---

## 3. Two-Year Retention Lifecycle via TTL (Time-to-Live)

Under Indian cybersecurity regulations (CERT-In Directions, April 2022), ISPs must retain subscriber access and transaction records for **not less than 180 days (6 months)** and certain security incident logs for up to **5 years**.

ClickHouse native table TTLs automate tiered storage without external cron scripts:
- **Hot Tier (NVMe SSD):** Raw 1:1 flow telemetry retained for 30 days for immediate incident triage.
- **Warm Tier (SATA HDD / ZFS Array):** Moved automatically via `TTL event_date + INTERVAL 30 DAY TO VOLUME 'slow_disks'`.
- **Cold Rollup Tier:** Raw records deleted after 180 days (`TTL event_date + INTERVAL 180 DAY DELETE`), while `AggregatingMergeTree` summary tables are retained for 2 years (730 days).
