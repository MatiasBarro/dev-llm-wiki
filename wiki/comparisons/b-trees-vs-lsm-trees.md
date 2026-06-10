---
type: comparison
tags: [databases, storage, system-design, foundational, algorithm]
sources:
  - "https://blog.bytebytego.com/p/b-trees-vs-lsm-trees-comparison-and?utm_source=post-email-title&publication_id=817132&post_id=195215951&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-06-10
---

# B-Trees vs LSM Trees

Both structures solve the same problem: data lives on disk, and disk is slow. Sequential access is significantly faster than random access, so every storage engine is really an exercise in controlling access patterns. B-Trees and LSM Trees represent two opposing answers.

| | [[concepts/b-trees\|B+ Tree]] | [[concepts/lsm-trees\|LSM Tree]] |
|---|---|---|
| **Write path** | Find correct page → insert in-place → split if full | Write to memtable (memory) → flush to SSTable (disk, sequential) |
| **Read path** | Walk tree root→leaf, 3–4 page reads | Check memtable → check SSTables (with bloom filters) |
| **On-disk format** | Mutable fixed-size pages, always sorted | Immutable SSTables, sorted within each file |
| **Background work** | None | Compaction (merge SSTables, evict stale versions) |
| **Crash safety** | Write-ahead log (WAL) | Write-ahead log (WAL) |

## The Three Amplifications Framework

The clearest way to evaluate any storage engine. All three cannot be minimized simultaneously — every design picks which cost to pay.

**Write amplification** — physical writes per logical write:
- B-Trees: moderate, predictable (WAL + page splits)
- LSM Trees: low initial latency, but data is rewritten through compaction levels (10–30× total)

**Read amplification** — disk reads per logical read:
- B-Trees: low (3–4 page reads; upper levels usually cached)
- LSM Trees: higher; depends on SSTable count and bloom filter effectiveness; worst case for non-existent keys

**Space amplification** — extra disk usage beyond raw data size:
- B-Trees: low (partially empty pages)
- LSM Trees: high temporarily (multiple key versions coexist between compaction runs)

> Reducing write amplification tends to increase read amplification and vice versa. Reducing both tends to increase space amplification.

## When to choose each

**B-Trees (most relational databases — PostgreSQL, MySQL InnoDB)**
- Read-heavy or mixed workloads (most web applications, OLTP)
- Range queries — linked leaves make scans efficient
- Predictable, consistent latency required

**LSM Trees (Cassandra, RocksDB, LevelDB, ScyllaDB, HBase)**
- Write-heavy workloads: logging, event ingestion, metrics, time-series
- Append-mostly access patterns
- Can tolerate higher read latency and compaction I/O spikes

## The SSD caveat

LSM Trees' original write advantage came from sequential flushes on spinning HDDs — sequential I/O was dramatically faster than random I/O. SSDs have narrowed this gap considerably. Random writes on SSD are cheap enough that B-Trees' penalty is much smaller than it used to be. The crossover point is workload- and hardware-dependent; there is no clean formula.

## Convergence

The two approaches are borrowing from each other over time. B-Tree systems have adopted write-buffering ideas from LSM. LSM systems use bloom filters and tiered caching to narrow the read performance gap. The distinction between the two is becoming less sharp at the implementation level, even if the core tradeoff remains.

## Key mental model

The read-vs-write tradeoff isn't unique to storage engines — it's a pattern that appears across all of system design. B-Trees pay the organization cost upfront on every write so reads are cheap. LSM Trees defer that cost to background compaction so writes are cheap. Recognizing this pattern is more durable than memorizing either structure.
