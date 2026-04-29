---
title: "B-Trees vs LSM Trees: Comparison and Trade-Offs"
source: "https://blog.bytebytego.com/p/b-trees-vs-lsm-trees-comparison-and?utm_source=post-email-title&publication_id=817132&post_id=195215951&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
author:
  - "[[ByteByteGo]]"
published: 2026-04-23
created: 2026-04-23
description: "In this article, we will look at B-Trees and LSM trees in detail, along with the trade-offs associated with each of them."
tags:
  - "clippings"
---
Every database has to solve the same basic problem.

Data lives on disk, and accessing disk is slow. Every read and every write eventually has to reach the disk, and how a database organizes data on that disk determines everything about its performance.

Over decades of research, two dominant approaches have emerged.

- B-Trees keep data sorted on disk so reads are fast, but pay for it on every write.
- LSM Trees buffer writes in memory and flush them to disk in bulk, making writes cheap but reads more expensive.

Neither approach is better. They represent two different approaches, and understanding the tradeoff between them is one of the most useful mental models in system design.

In this article, we will look at B-Trees and LSM trees in detail, along with the trade-offs associated with each of them.

![](https://substackcdn.com/image/fetch/$s_!zFEV!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F745700fd-435c-44be-b049-ae5bc1392636_2250x2624.png)

## The Problem with Disk Access

To understand why these two approaches exist, we need to start with the problem they’re both trying to solve.

Applications work with data in memory, but memory is volatile, meaning it loses everything when power is cut. Therefore, at some point, every piece of data that matters has to be persisted to disk. And disk access, whether on a spinning hard drive or an SSD, is orders of magnitude slower than memory access. Where a memory read takes nanoseconds, a disk read can take microseconds at best, milliseconds at worst. The gap is enormous.

Not all disk access is equally expensive, though.

Sequential access, where data is read or written in a continuous stream, is significantly faster than random access, where the system jumps to different locations on disk. This difference matters because the way a database organizes its data determines whether reads and writes are sequential or random.

The entire game of storage engine design comes down to minimizing how often the disk is touched, and when it must be touched, making the access pattern as efficient as possible. Every time a query runs or a row is inserted, this problem is what the database is trying to handle. B-Trees were the first dominant answer to this problem.

## B-Trees

A B-Tree organizes data on disk into fixed-size blocks called pages, typically 4KB each. These pages are arranged in a tree structure where every key is kept in sorted order.

When a new key is inserted, the tree finds the right page and places the key in its correct sorted position. When a page gets too full, it splits into two pages, and the parent node is updated to reflect the new structure.

Consider a simple example. Imagine a database storing user accounts indexed by user ID. If we insert user IDs 10, 25, 37, 42, and 55, the B-Tree places them across its pages in sorted order. Now, when a query asks for user 37, the tree starts at the root, sees that 37 falls between two boundary keys, follows the right pointer, and lands directly on the page containing 37. Only 3-4 pages are read from disk, even if the table holds millions of users.

![](https://substackcdn.com/image/fetch/$s_!6CZz!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc4003041-b793-4501-95c7-6db8ee534d4c_2362x1902.png)

This upfront investment in sorting pays off immediately at read time. To find any single key, the walk down the tree involves comparing the search key at each level to decide which branch to follow. Even with millions of records, a B-Tree is typically only 3-4 levels deep.

One important detail to understand here is that when databases say “B-Tree,” they almost always mean B+ Tree, a variant that makes a significant structural improvement.

In a B+ Tree, the internal nodes of the tree store only keys, not values. All the actual data lives at the leaf level. The leaves are then linked together in a chain, so once the starting point is found, scanning through a range of keys is just a matter of following leaf pointers sequentially. Going back to the user accounts example, a query like “find all users with IDs between 25 and 50” would locate 25 in the leaf nodes and then simply follow the chain through 37 and 42 without ever jumping back up the tree. This is why range queries are fast in relational databases. The data is already sorted, and the leaves are already linked.

![](https://substackcdn.com/image/fetch/$s_!5xG4!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F81339da8-3bf6-4e87-8be0-da6dd3334cba_3328x2022.png)

However, all of this organization costs something.

Every write has to find the correct page, potentially split pages, and update parent nodes. These operations involve random I/O, which is exactly the expensive kind of disk access we discussed earlier. This is because the page that needs updating could be anywhere on disk. Under heavy write loads, this can become a bottleneck. The database is spending time maintaining perfect organization that might not be needed yet.

This is the B-Tree’s fundamental deal in which the cost of organization is paid upfront on every write, so that every read is cheap. If a workload reads more than it writes, this is a great deal. If the workload is write-heavy, every insert pays this tax to maintain a structure that favors reads.

But what if that cost were deferred instead?

## LSM Trees

An LSM (Log-Structured Merge) Tree takes the opposite approach.

Instead of sorting data into the right place on every write, it buffers writes in memory in a structure called a memtable. This is just a sorted in-memory data structure like a balanced binary tree or a skip list (a layered linked list designed for fast lookups). Since memory access is fast, writes land almost instantly.

When the memtable fills up (typically a few megabytes), the entire data is flushed to disk as a single sorted file called an SSTable, short for Sorted String Table. Two properties of SSTables matter.

- First, they are sorted by key, so we can efficiently jump to any key within them without scanning the whole file.
- Second, they are immutable, meaning once written, an SSTable is never modified. New writes always go to the memtable, never to an existing SSTable.

See the diagram below:

![](https://substackcdn.com/image/fetch/$s_!49Ji!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F3ee2b9fd-8ef6-418f-9c44-46ef6fe7bbc0_1890x1888.png)

A concrete example can help make things clear. Imagine an application logging click events with timestamps as keys. Events arrive rapidly, and we have timestamp 1001, then 1002, then 1003, and so on. Each one is written to the memtable in memory. After a few thousand events, the memtable flushes to disk as SSTable-1, containing events 1001 through 5000, sorted. More events arrive and fill another memtable, which is flushed as SSTable-2 with events 5001 through 9000. Now there are two sorted files on disk. If a query now asks for event 3500, the system has to figure out which SSTable contains it.

This is where the costs start showing up. Over time, SSTables accumulate on disk. There might be dozens or hundreds of them, each representing a different batch of writes from a different point in time. The same key can appear in multiple SSTables if it was updated several times. This is the mess that LSM Trees trade for fast writes.

To manage this mess, LSM Trees run a background process called compaction.

Compaction reads multiple SSTables, merges them (keeping only the most recent version of each key), and writes out a new, larger SSTable. This is essential for keeping the system functional, but it is not free. Compaction consumes CPU and disk I/O. During compaction, the system is doing significant work that competes with actual reads and writes. If writes come in faster than compaction can process, the number of SSTables grows unchecked, and performance degrades. This is one of the biggest operational challenges with LSM-based systems.

Even within compaction, the read-vs-write tension reappears.

Different compaction strategies exist, and they make different tradeoffs. Some favor write throughput by doing less frequent, larger merges. Others favor read performance by keeping SSTables more aggressively organized.

See the diagram below:

![](https://substackcdn.com/image/fetch/$s_!Bjn7!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9e8f3dab-27d8-4216-a1ce-5a8ad8068ce1_2908x2092.png)

The other major cost is applicable to the read path. To find a key, the system first checks the memtable. If the key isn’t there, it has to check SSTables on disk. But which SSTable to check? The key could be in any of them. Without help, every SSTable would have to be searched, which is painfully slow.

This is where bloom filters come in.

A bloom filter is a small, memory-resident data structure attached to each SSTable. When queried with a key, it gives one of two answers. Either “this key is definitely not in this SSTable” or “this key might be in this SSTable.” It can produce false positives but never false negatives. This means the system can skip SSTables that definitely don’t contain the key, reducing the number of disk reads. Without bloom filters, LSM Tree reads would be impractical. But bloom filters themselves use memory, and when there are thousands of SSTables, that memory cost adds up.

See the diagram below:

![](https://substackcdn.com/image/fetch/$s_!-ITt!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ffe72f508-0711-4b06-976f-e99d79b8e49a_2910x2022.png)

One thing both structures share is that they both use write-ahead logs for crash safety. Before any write is considered complete, it’s recorded in a sequential log on disk. If the system crashes, this log can be replayed to recover data that was in memory but hadn’t been flushed yet.

## Comparing the Main Costs

There’s a useful framework for reasoning about storage engine tradeoffs called the three amplifications. These are read amplification, write amplification, and space amplification.

![](https://substackcdn.com/image/fetch/$s_!lk2T!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb2a0c05a-50ff-49f5-a2bd-0365551cb8f3_2362x1502.png)

Write amplification measures how many physical writes to disk result from a single logical write. In other words, when one key-value pair is inserted, how many times does data actually get written to disk?

- In LSM Trees, the answer can be surprisingly high. A piece of data is first written to the write-ahead log, then to the memtable flush (SSTable), and then potentially rewritten multiple times as compaction merges it through levels. A single logical write might result in 10-30x physical writes over the lifetime of the data.
- B-Trees have write amplification too (page splits cause writes to multiple pages, and the write-ahead log adds another write), but it tends to be lower and more predictable.

Read amplification measures how many physical disk reads are needed to serve a single logical read.

- In a B-Tree, this is straightforward. Walking down the tree takes 3-4 page reads, and often fewer because the upper levels tend to be cached in memory.
- In an LSM Tree, the answer depends on how many SSTables exist and how effective the bloom filters are. In the worst case, several SSTables might need to be checked before finding the key. For keys that don’t exist at all, the system has to verify absence across multiple files, which is even more expensive. Bloom filters help here, but false positives still cause unnecessary reads.

Space amplification measures how much extra disk space the storage engine uses beyond the raw size of the data.

- LSM Trees can temporarily store multiple versions of the same key across different SSTables until compaction cleans them up. This means actual disk usage might be significantly higher than the logical data size.
- B-Trees have less severe space amplification, though they can leave partially empty pages after splits.

The main takeaway is that all three cannot be minimized simultaneously.

Reducing write amplification tends to increase read amplification, and vice versa. Reducing both tends to increase space amplification. B-Trees accept higher write amplification in exchange for lower read amplification. LSM Trees accept higher read and space amplification in exchange for lower initial write latency, though their total write amplification through compaction can actually be higher.

When evaluating a database, the main question developers should ask is which amplification is acceptable to pay for.

## Choosing the Right Approach

Understanding the three costs framework is important, but what does it mean for the decisions we’ll actually face?

Read-heavy and mixed workloads, which describe most web applications and transactional systems, tend to favor B-Trees. Data is read far more often than it’s written, so paying a higher write cost to keep reads fast is a sensible tradeoff. This is why most relational databases use B-Tree storage engines. On the other hand, write-heavy workloads like logging, event ingestion, metrics collection, and time-series data tend to favor LSM Trees. The volume of incoming writes is so high that paying for in-place updates on every one of them becomes a bottleneck.

But the crossover point is blurry, and the practical answers can vary a lot. It depends on key size, value size, access patterns, hardware, and how well compaction is tuned. There is no clean formula.

The hardware landscape has also shifted the balance. LSM Trees originally gained a major advantage from sequential writes on spinning hard drives, where sequential I/O was dramatically faster than random I/O. SSDs have narrowed this gap considerably. Random writes on an SSD are much cheaper than on an HDD, which means B-Trees’ random write pattern is less punishing than it used to be.

In practice, the choice is rarely about picking a data structure directly.

The decision is about picking a database, and it has already made this choice. Most relational databases use B+ Trees. In contrast, most write-optimized distributed databases use LSM Trees. Some databases even offer both modes, allowing the choice to be made based on workload. Knowing what’s underneath helps in understanding why a database behaves the way it does and whether its default bet matches the actual access patterns.

Both approaches are also converging over time.

B-Tree systems have adopted ideas from the LSM world, and LSM systems have developed techniques like bloom filters and tiered caching that narrow the read performance gap.

## Summary

In this article, we’ve looked at B-Trees and LSM Trees in detail. Here are the key learning points in brief:

- B-Trees and LSM Trees are two different approaches to organizing data on disk, and each one represents a deliberate tradeoff between read performance and write performance.
- B-Trees keep data sorted in place across fixed-size pages, making reads fast and predictable (typically 3-4 disk reads to find any key) at the cost of slower, more expensive writes.
- Most databases actually use B+ Trees, a variant where all values live at the leaf level, and leaves are linked together, which is what makes range queries efficient.
- LSM Trees buffer writes in memory (the memtable) and flush them to disk as sorted, immutable files (SSTables), making writes extremely fast but shifting the cost to reads and background maintenance.
- Compaction, the process of merging SSTables, is not a free background task. It consumes CPU and disk I/O, and if it falls behind, the system degrades. This is the biggest operational challenge with LSM-based systems.
- Bloom filters make LSM Tree reads practical by quickly eliminating SSTables that don’t contain a given key, but they come with their own memory cost.
- The three amplification framework (read, write, and space amplification) is the clearest way to evaluate any storage engine. All three cannot be minimized at once, and every database picks which cost it’s willing to pay.
- The read-vs-write tradeoff isn’t unique to storage engines. It’s a pattern that shows up across all of system design, and learning to spot it is more valuable than memorizing any single data structure.