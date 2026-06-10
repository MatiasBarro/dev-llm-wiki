---
type: concept
tags: [databases, storage, algorithm, foundational, system-design]
sources:
  - "https://blog.bytebytego.com/p/b-trees-vs-lsm-trees-comparison-and?utm_source=post-email-title&publication_id=817132&post_id=195215951&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-06-10
---

# B-Trees (B+ Trees)

A B-Tree is a self-balancing tree that organizes data on disk in sorted, fixed-size blocks called **pages** (typically 4 KB each). Every key is maintained in sorted order across the tree, so any lookup requires traversing from root to leaf — typically only 3–4 levels deep even for millions of records.

Real databases almost universally use the **B+ Tree** variant, not plain B-Trees:
- **Internal nodes** store only keys (no values) — used purely for routing.
- **All values live at the leaf level** — one predictable place to find data.
- **Leaves are linked together** in a chain, enabling fast range scans: find the start key, then follow leaf pointers sequentially without backtracking up the tree.

## Write path

1. Find the correct page for the key.
2. Insert in sorted position.
3. If the page is full, split it into two pages and update the parent node.
4. Record the operation in a write-ahead log (WAL) for crash safety.

Each write involves **random I/O** — the target page could be anywhere on disk. Under heavy write loads this becomes a bottleneck.

## Amplification profile

| Dimension | B+ Tree |
|---|---|
| Read amplification | Low — 3–4 page reads to reach any key; upper levels usually cached |
| Write amplification | Moderate — page splits touch multiple pages + WAL; predictable |
| Space amplification | Low — partially empty pages after splits, but no duplicate versions |

## When B-Trees win

- **Read-heavy and mixed workloads** — most web applications, transactional systems (OLTP).
- **Range queries** — the linked-leaf structure makes scans efficient.
- **Predictable latency** — no background compaction competing with queries.

Most relational databases (PostgreSQL, MySQL InnoDB) use B+ Tree storage engines.

## See also

- [[lsm-trees]] — the alternative approach that defers sort cost to background compaction
- [[comparisons/b-trees-vs-lsm-trees]] — full trade-off analysis with the three amplifications framework
