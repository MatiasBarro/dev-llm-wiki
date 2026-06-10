---
type: concept
tags: [databases, storage, algorithm, foundational, system-design]
sources:
  - "https://blog.bytebytego.com/p/b-trees-vs-lsm-trees-comparison-and?utm_source=post-email-title&publication_id=817132&post_id=195215951&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-06-10
---

# LSM Trees (Log-Structured Merge Trees)

An LSM Tree defers the cost of organizing data. Instead of sorting on every write, it buffers writes in memory and periodically flushes them to disk in bulk — making writes cheap at the expense of read complexity and background maintenance.

## Write path

1. Every write lands in an in-memory sorted structure called the **memtable** (usually a balanced BST or skip list). Memory writes are fast.
2. When the memtable fills (typically a few MB), it is flushed to disk as a **SSTable** (Sorted String Table) — a sorted, **immutable** file. Flushing is sequential I/O, which is efficient.
3. A **write-ahead log (WAL)** records each write before the memtable for crash recovery.

## The accumulation problem

SSTables pile up over time. The same key can appear in multiple SSTables (if updated repeatedly). This creates two problems:

- **Read cost**: to find a key, the system must check the memtable, then potentially many SSTables on disk.
- **Space cost**: multiple versions of the same key coexist until cleaned up.

## Compaction

A background process that reads multiple SSTables, merges them (keeping only the most recent version of each key), and writes a new, larger SSTable. This is essential but not free — it competes with live reads and writes for CPU and disk I/O.

If writes outpace compaction, SSTables accumulate unchecked and performance degrades. **Compaction management is the primary operational challenge of LSM-based systems.**

Different compaction strategies trade off write throughput against read performance (e.g., size-tiered vs. leveled compaction).

## Bloom filters

Each SSTable carries a **bloom filter** — a compact, memory-resident probabilistic structure that answers: "is this key in this SSTable?"

- **Never false negatives**: if the filter says "no," the SSTable is skipped entirely.
- **Occasional false positives**: the filter might say "maybe" when the key isn't there, causing an unnecessary disk read.

Without bloom filters, LSM reads would require scanning every SSTable. With them, most SSTables are eliminated cheaply. The tradeoff: bloom filters consume memory, and at scale (thousands of SSTables) that adds up.

## Amplification profile

| Dimension | LSM Tree |
|---|---|
| Read amplification | High without bloom filters; reduced but not eliminated with them |
| Write amplification | Low initial latency, but data is rewritten through compaction levels — total can reach 10–30× |
| Space amplification | High temporarily — multiple versions of keys exist between compaction runs |

## Hardware note

LSM Trees' original write advantage came from sequential flushes on spinning HDDs, where sequential I/O vastly outperformed random I/O. SSDs have narrowed this gap, making B-Trees' random writes less expensive and blurring the traditional rule of thumb.

## When LSM Trees win

- **Write-heavy workloads**: logging, event ingestion, metrics, time-series data.
- **Append-only access patterns**: where the same key is rarely re-read after writing.

Used by: Cassandra, RocksDB, LevelDB, ScyllaDB, HBase.

## See also

- [[b-trees]] — the alternative that pays sort cost upfront on every write
- [[comparisons/b-trees-vs-lsm-trees]] — full trade-off analysis with the three amplifications framework
