---
type: concept
tags: [databases, storage, reliability, foundational, system-design]
sources:
  - "[[concepts/b-trees]]"
  - "[[concepts/lsm-trees]]"
date-updated: 2026-06-10
---

# Write-Ahead Log (WAL)

A durability mechanism used by virtually all database storage engines. The core rule: **before any write is considered complete, it must first be recorded sequentially to an append-only log on disk.** If the system crashes before in-memory changes are flushed to their final location, the WAL can be replayed on restart to recover what would otherwise be lost.

## Why it works

Writing to the WAL is *sequential* — fast, because the disk appends to a single location. Writing to the actual data structure ([[b-trees|B-Tree]] pages or [[lsm-trees|LSM]] memtable→SSTable) may involve *random* I/O and multiple steps — slower, and interruptible mid-way.

The WAL acts as the source of truth for "did this write happen?" regardless of whether the main structure has caught up.

## In B-Trees

Each insert or update is logged to the WAL before the page is modified. If a page split is interrupted mid-crash, the WAL replays the intent and finishes it cleanly on recovery. See [[b-trees]].

## In LSM Trees

The memtable lives entirely in memory, so it would vanish on crash. The WAL is what makes memtable writes durable — every write goes to the WAL first, then the memtable. On recovery, the WAL is replayed to rebuild the in-memory state before the system comes back online. See [[lsm-trees]].

## Cost

The WAL adds one extra sequential write per logical write, contributing to write amplification in both storage engines. See [[comparisons/b-trees-vs-lsm-trees]] for how WAL fits into the broader read/write/space amplification tradeoff.

## Key property: sequential writes only

The WAL is append-only by design. Sequential disk writes are significantly faster than random writes, so the durability guarantee comes at relatively low latency cost — especially compared to the random I/O that modifying a B-Tree page would require.
