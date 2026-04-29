---
title: "I struggled with system design until I learned these 114 concepts"
source: "https://newsletter.systemdesign.one/p/system-design-core-concepts?utm_source=post-email-title&publication_id=1511845&post_id=187077810&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
author:
  - "[[Neo Kim]]"
published: 2026-02-14
created: 2026-04-22
description: "A deep dive into system design core concepts. Websockets, gateway, distributed cache, and 35 others."
tags:
  - "clippings"
---
### #122: Part 2 - websockets, gateway, distributed cache, and 35 others.

- *[Share this post](https://newsletter.systemdesign.one/p/system-design-core-concepts/?action=share) & I'll send you some rewards for the referrals.*
- *Block diagrams created using [Eraser](https://app.eraser.io/auth/sign-up?ref=neo).*

---

Onwards 'n downwards:

Following is the second of a premium 3-part newsletter series… If you’re just getting started with system design or want a super strong foundation, then this newsletter is for you.

On with part 2 of the newsletter:

\===

Some of these are foundational, and some are quite advanced. ALL of them are super useful to software engineers building distributed systems…

Curious to know how many were new to you:

39. WebSockets
40. API Gateways
41. Distributed Cache
42. Cache Eviction Policies
43. Proxy vs Reverse Proxy
44. HTTP vs HTTPS
45. TCP vs UDP
46. OSI Model
47. TLS/SSL
48. DNS Load Balancing
49. Anycast Routing
50. Object Storage
51. Distributed File Systems
52. Block vs File vs Object Storage
53. Data Compression
54. ACID vs BASE
55. Network Partitions
56. Split-Brain Problem
57. Heartbeats
58. Leader Election
59. Consensus Algorithms
60. Quorum
61. Paxos Algorithm
62. Raft Algorithm
63. Gossip Protocol
64. Clock Synchronization Problem
65. Logical Clocks
66. Lamport Timestamps
67. Vector Clocks
68. Distributed Transactions
69. Two-Phase Commit
70. SAGA Pattern
71. Outbox Pattern
72. Three-Phase Commit
73. Delivery Semantics
74. Change Data Capture
75. Long Polling
76. Server-Sent Events

(…and much more in part 3!)

For each, I’ll share:

- What it is & how it works--in simple words
- A real-world analogy
- Tradeoffs
- Why it matters

Let’s go.

---

### AI code review with your team’s knowledge (Partner)

![](https://substackcdn.com/image/fetch/$s_!ui-Q!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc6649d8e-d6ef-477e-baab-c8cbe718c0f7_2880x2160.png)

**[Unblocked](https://getunblocked.com/code-review/?utm_source=systemdesign&utm_medium=email&utm_campaign=codereview&utm_content=context)** is the only AI code review tool that has a deep understanding of your codebase, docs, and past decisions, giving you thoughtful feedback that feels like it came from your best engineer.

---

## 39\. WebSockets

> WebSockets provide full-duplex, bidirectional communication between client & server over a single, long-lived TCP connection.

Unlike HTTP, where the client always initiates requests, WebSockets allow the server to push data to clients in real-time.

After an initial HTTP handshake, the connection upgrades to the WebSocket protocol. Both the client and the server can then send messages at any time.

![](https://substackcdn.com/image/fetch/$s_!LcJZ!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F400c4e99-1830-4f1d-86b1-3f518e5778ba_790x368.png)

### Analogy

WebSockets is like a phone call where both people can talk and listen simultaneously…

Compare this to HTTP, which is like sending letters back and forth,,, where you wait for a reply before sending the next message.

### Tradeoff

They’re more complex to implement and scale since each connection consumes server resources. Also, load balancing becomes tricky because connections are long-lived and stateful.

Plus, some proxies/firewalls “block” WebSocket upgrades or long-lived connections, so compatibility can vary.

### Why it matters

Use for real-time apps like chat systems, live sports scores, collaborative editing, online gaming, or stock trading platforms. But avoid for simple request-response patterns where HTTP is enough.

## 40\. API Gateways

> An API gateway is a server that acts as a SINGLE entry point for all client requests to your microservices.

It handles request routing, composition, and protocol translation [^1]. Instead of clients calling different microservices directly, they make ‘one call’ to the gateway.

![](https://substackcdn.com/image/fetch/$s_!Zckl!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd3999e1d-12ec-4bff-a0cd-8f7f9e188beb_872x275.png)

### Analogy

An API gateway is like a hotel concierge:

Instead of guests figuring out which department to call, they call the concierge desk. The concierge knows which department to contact and gets back to the guest with answers.

### Tradeoff

They can become a bottleneck or a single point of failure if not deployed redundantly. Besides, they increase latency because of the extra network hop. So the gateway itself needs to scale & be highly available.

### Why it matters

Useful in microservices because it provides clients with a single entry point.

Also, it handles common tasks like authentication, authorization, and rate limiting in one place, and can return different responses for different clients, such as web or mobile apps.

## 41\. Distributed Cache

> Distributed cache spreads cached data across many cache servers instead of a single cache instance.

Each cache node stores a portion of the data, typically determined by consistent hashing [^2]. Popular implementations include Redis Cluster and Memcached.

![](https://substackcdn.com/image/fetch/$s_!qZFX!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5121717e-5e70-44e9-bfec-e5265c39027d_1600x501.png)

### Analogy

Multiple fast-food locations across a city instead of one central kitchen.

Each location stores popular items for quick service. Total capacity increases by opening more locations, and no single location becomes overwhelmed during rush hour.

### Tradeoff

They add operational complexity (partitioning, rebalancing, replication) and can incur overhead during rebalancing/failover. Also, there’s a risk of cache misses when keys get redistributed.

Plus, debugging becomes harder with many nodes.

### Why it matters

Use a distributed cache in high-traffic sites when one cache server can’t handle the traffic, when the data no longer fits in one machine’s memory, or when you need high availability.

Start with a single cache server…Move to a distributed cache setup only when you reach scaling or reliability limits.

## 42\. Cache Eviction Policies

> Cache eviction policies decide which data to remove when the cache is full and new data needs space.

- Least Recently Used (**LRU**) removes the data that has NOT been accessed for the longest time.
- Least Frequently Used (**LFU**) removes the data that is accessed the least often.
- First In, First Out (**FIFO**) removes the oldest data first, based on when it was added.
- Time To Live (**TTL**) automatically removes data after a fixed time period.

![](https://substackcdn.com/image/fetch/$s_!OFUU!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd2c99482-eb4a-4248-a6bc-d113ab177dd5_813x370.png)

### Analogy

Think of your phone storage:

- LRU deletes photos you haven’t opened in a long time.
- LFU deletes photos you rarely look at.
- FIFO deletes the oldest photos first.
- TTL is like a message that automatically disappears after 24 hours.

### Tradeoff

Different policies work well for different access patterns…

- LRU works well when recently accessed data is likely to be used again. Yet it can perform poorly if large amounts of data are accessed only once.
- LFU works well when frequently accessed data stays popular over time, but it reacts slowly if usage patterns change.
- FIFO is simple but does not consider how often or recently data is used.
- TTL ensures data does not stay in the cache forever, but it may remove useful data too early or keep stale data too long.

Each policy has overhead in tracking metadata for eviction decisions.

### Why it matters

Use:

- LRU for general-purpose caching where recent data is likely to be reused.
- LFU when certain data remains popular for long periods.
- TTL when data naturally becomes stale after some time, such as API responses or session data.

Most systems combine TTL with LRU or LFU.

## 43\. Proxy vs Reverse Proxy

> A forward proxy sits between clients and the Internet. It sends requests to external servers on behalf of the client.
> 
> A reverse proxy sits in front of your servers. It receives requests from clients and forwards them to the correct backend server.

With a forward proxy, client is configured to use it. With a reverse proxy, the client usually doesn’t know it exists.

![](https://substackcdn.com/image/fetch/$s_!tDrX!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbf7a4905-c791-4215-a8f3-f22ea30cde09_1536x1024.png)

### Analogy

A forward proxy is like an assistant who makes calls for you, so the person on the other end doesn’t talk with you directly.

A reverse proxy is like a company receptionist. Callers think they are contacting the company directly,,, but the receptionist routes the call internally.

### Tradeoff

Forward proxies can improve privacy, enforce security policies, and filter traffic. Yet they add extra network hops and can increase latency.

Reverse proxies provide load balancing, SSL termination, caching, and protection from direct exposure of backend servers. But they must be deployed redundantly to avoid becoming a single point of failure.

Both require proper configuration to prevent security risks…

### Why it matters

- Use forward proxies in corporate networks for content filtering, monitoring & privacy control.
- Use reverse proxies in production systems for load balancing, SSL termination, traffic routing, and protection against attacks [^3].

Most apps use reverse proxies such as Nginx, HAProxy, or cloud load balancers.

## 44\. HTTP vs HTTPS

> Hypertext Transfer Protocol (HTTP) sends data in ‘plain text’.
> 
> Hypertext Transfer Protocol Secure (HTTPS) is HTTP encrypted using Transport Layer Security (TLS).

HTTPS encrypts communication between the client and server, protecting data from eavesdropping and tampering. The server provides a certificate to prove its identity. Modern browsers mark HTTP sites as “Not Secure.”

![](https://substackcdn.com/image/fetch/$s_!lv4n!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9dd9c4b4-741b-47b0-a218-98bda88e90fd_859x428.png)

### Analogy

HTTP is like sending a postcard. Anyone who intercepts it can read the message.

HTTPS is like sending a sealed, locked box. Even if someone intercepts it, they cannot read or change what’s inside.

### Tradeoff

HTTPS requires managing digital certificates and adds a small performance cost because of the TLS handshake. Yet these costs are minimal compared to the security benefits.

### Why it matters

HTTPS protects against eavesdropping and man-in-the-middle attacks, where attackers intercept or modify traffic.

HTTPS is also a positive ranking factor for search engines and is required for many modern web features, such as HTTP/2, service workers, and secure cookies.

## 45\. TCP vs UDP

> Transmission Control Protocol (TCP) is a connection-oriented protocol that provides reliable, ordered delivery of data.
> 
> User Datagram Protocol (UDP) is connectionless and sends packets without guaranteeing delivery, order, or protection against duplication.

TCP establishes a connection using a handshake, retransmits lost packets, and performs congestion control.

UDP sends packets independently with minimal overhead & no built-in reliability. i.e., UDP is faster but less reliable.

![](https://substackcdn.com/image/fetch/$s_!oc0S!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6888b2b1-b192-445f-ac3a-2d404b932f74_1086x631.png)

### Analogy

TCP is like certified mail with tracking and delivery confirmation.

UDP is like sending postcards. They usually arrive, but they might be lost or arrive out of order…

### Tradeoff

TCP adds latency due to the handshake, acknowledgments, retransmissions, and head-of-line blocking (where a lost packet delays subsequent packets).

UDP doesn’t guarantee delivery or order. If reliability is needed,,, the application code must handle it.

### Why it matters

- Use TCP for web browsing, email, file transfers, database connections, and APIs where accuracy matters more than speed.
- Use UDP for real-time applications such as video calls, live streaming, and online gaming, where low latency is more important than reliability.

NOTE: DNS typically uses UDP for speed, but it can fall back to TCP for large responses or specific operations.

---

***Reminder: this is a teaser of the subscriber-only post, exclusive to my golden members.***

When you upgrade, you’ll get:

- **Full access to system design case studies**
- FREE access to (coming) Design, Build, Scale newsletter series
- **FREE access to (coming) popular interview question breakdowns**

And more!

Get 10x the results you currently get with 1/10th the time, energy & effort.

---

## 46\. OSI Model

> Open Systems Interconnection (OSI) model is a conceptual framework that divides network communication into seven layers:
> 
> Physical, Data Link, Network, Transport, Session, Presentation, and Application.

Each layer performs a specific function and communicates with the layer above and below it.

![](https://substackcdn.com/image/fetch/$s_!2AM7!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7690f4ee-390d-4204-9c8e-33d5731be437_1536x1024.png)

### Analogy

Think of sending a package internationally, where each step handles a different responsibility…

- Physical layer is the physical transport, like a plane or a truck.
- Data Link layer handles local delivery within a network.
- Network layer decides the route.
- Transport layer ensures the package arrives correctly and completely.

### Tradeoff

OSI model is useful for understanding how networks work, but it’s mostly ‘theoretical’.

Real-world networking follows the TCP/IP model, which combines several OSI layers into 4 layers. Modern protocols sometimes blur boundaries between layers, so the OSI model doesn’t match real implementations.

### Why it matters

OSI model is useful as a learning & troubleshooting tool…It helps to isolate problems by thinking in layers.

Although most systems follow the simpler TCP/IP model, the OSI model remains a standard reference for discussing networking concepts.

## 47\. TLS/SSL

> TLS (Transport Layer Security) is a cryptographic protocol that provides secure communication over a network.

It encrypts data in transit, authenticates the server using certificates, and ensures data integrity. TLS handshake establishes encryption keys, the server presents its certificate, and both parties agree on encryption algorithms before data flow.

> Secure Sockets Layer (SSL) is the older protocol that TLS replaced and is now *deprecated*.

![](https://substackcdn.com/image/fetch/$s_!rGh1!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F56e70ca3-916e-42ae-a611-8ca1b47ceed1_1462x710.png)

### Analogy

It’s like meeting someone in person to agree on a secret code before speaking on the phone.

The initial meeting takes time, but afterward, your conversations are encrypted. Even if someone intercepts the call, they cannot understand it.

### Tradeoff

TLS adds a small delay during the handshake and requires extra CPU usage for encryption and decryption. Also, it requires certificate management, including renewal and revocation.

Modern improvements like session resumption and hardware acceleration [^4] significantly reduce performance overhead.

### Why it matters

TLS for any network communication involving sensitive data, which means everything on the internet. Required for HTTPS, recommended for database connections, email, API calls, and internal microservice communication.

The performance overhead is negligible compared to its security benefits.

## 48\. DNS Load Balancing

> DNS load balancing spreads traffic across many servers by returning different IP addresses for the same domain name.

When a client looks up your domain, a DNS server can return different IP addresses using round-robin, weighted distribution, or geographic routing [^5]. The client then connects directly to the IP address it receives.

![](https://substackcdn.com/image/fetch/$s_!7tqN!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4b38e0aa-d10e-418f-ad0c-98a2c09c9157_1600x870.png)

### Analogy

It’s like calling a single hotel reservation number and being routed to a different regional office based on your location…

The phone number stays the same, but you’re connected to different offices behind the scenes.

### Tradeoff

DNS load balancing is NOT very precise because DNS responses are cached by browsers, operating systems, and internet providers. i.e., changes in traffic routing or failover do not happen instantly. Plus, if a server fails, users may still try to connect to it until their cached DNS record expires.

Also, DNS only returns an IP address. It cannot inspect requests, terminate TLS, or perform application-level routing.

### Why it matters

Useful for global applications that need geographic routing so users connect to the nearest data center.

It’s often used as a first layer of routing, combined with traditional load balancers inside each region. Common for CDNs & global SaaS platforms.

## 49\. Anycast Routing

> Anycast routing is a network addressing method in which many servers share the same IP address across different geographic locations.

Network routers automatically direct traffic to the nearest server based on routing protocols and network topology. From the client’s perspective, they connect to a “single” IP address, but the network layer routes them to the closest physical server.

![](https://substackcdn.com/image/fetch/$s_!cSdf!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F74087535-dd6f-407c-8043-8f8f9179325b_1600x873.png)

### Analogy

It’s like calling a national emergency number:

You dial the same number everywhere, but your call is automatically routed to the nearest local center.

### Tradeoff

It requires advanced network configuration and knowledge of Border Gateway Protocol (BGP) [^6]. It works best for stateless or short-lived connections. If routing changes during a long-lived connection, the connection might break.

Plus, Anycast operates at the network level, so it cannot make application-level decisions or perform weighted traffic distribution.

It also requires coordination with internet service providers to advertise routes correctly.

### Why it matters

Anycast is commonly used for DNS infrastructure, content delivery networks (CDNs), and DDoS protection services [^7]. It’s ideal for globally distributed systems handling stateless traffic.

Root DNS servers use anycast.

---

Know someone who wants to learn system design? Consider gifting a subscription:

---

## 50\. Object Storage

> Object storage stores data as discrete objects rather than as files in a hierarchy or blocks on disk.

Each object contains the data, metadata, and a unique identifier. Objects are stored in a flat address space without traditional folder hierarchies. Object storage is accessed via HTTP APIs.

Examples: Amazon S3, Google Cloud Storage & Azure Blob Storage.

![](https://substackcdn.com/image/fetch/$s_!k4tz!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F21ab33ff-6ff6-48b0-a129-ea5840a66fe0_1600x873.png)

### Analogy

Think of a large warehouse where every item has a unique barcode…

Items get stored wherever there is space. You don’t walk through aisles…You scan the barcode, and the system retrieves the item. The warehouse can grow by adding more storage space.

### Tradeoff

It has higher latency than local disk storage.

It’s NOT designed to be mounted as a traditional file system. While updates typically replace the whole object rather than modifying part of it. Providers also charge per storage and API request.

### Why it matters

Object storage is ideal for storing unstructured data such as images, videos, backups, logs, and static assets. It’s commonly used for data lakes, media storage, archival systems, and user-uploaded content.

## 51\. Distributed File Systems

> Distributed file systems spread file storage across many servers while presenting a unified view to clients.

Files get split into chunks and distributed across different servers for parallel access. Chunks then get replicated to improve reliability. Some systems provide standard file system interfaces (like CephFS, GlusterFS). Others, such as HDFS, use their own client APIs rather than acting as a standard-mounted file system.

![](https://substackcdn.com/image/fetch/$s_!FFUF!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0d6bd9d7-7673-4cd3-b9b2-b921ef845cb9_1198x572.png)

### Analogy

Imagine a library where each book gets divided into chapters, stored on different floors:

When you request a book, the system gathers all chapters and presents the complete book. If one floor is damaged, copies of the chapters exist elsewhere.

### Tradeoff

They add significant complexity, introduce latency because of network access, consume network bandwidth, and often perform poorly with many small files.

### Why it matters

Useful for big data analytics processing petabytes of data, video processing and rendering, scientific computing with large datasets, or backup and archival systems.

Plus, they’re essential in Hadoop [^8] ecosystems.

## 52\. Block vs File vs Object Storage

> Block storage divides data into fixed-size blocks with unique addresses.
> 
> File storage organizes data hierarchically in folders & files.
> 
> Object storage stores data as objects with metadata in a flat namespace.

Block storage offers low latency & fine control… File storage provides a familiar structure & easy sharing… Object storage provides massive scalability & rich metadata.

![](https://substackcdn.com/image/fetch/$s_!mmmk!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fdb7aa84a-94b3-4625-9fb2-beb0e9d9f8a6_832x350.png)

### Analogy

Block storage is like LEGO bricks…you control how everything gets built, but you must manage the pieces yourself.

File storage is like a filing cabinet with folders and documents, making it easy to organize and share files.

Object storage is like a warehouse with barcoded items. You don’t care where something is stored; you just retrieve it using its ID.

### Tradeoff

Block storage provides high performance and low latency, but requires managing file systems & lacks metadata features.

File storage is easy to use and share across systems, but has scaling limitations & hierarchical constraints.

Object storage scales well and supports rich metadata, but it has higher latency and isn’t designed to behave like a traditional mounted drive.

### Why it matters

- Use block storage for databases, virtual machine disks, and applications that require low latency & high performance.
- Use file storage for shared network drives, home directories, and applications that expect a traditional file system.
- Use object storage for backups, media files, logs, archives, and cloud-native applications that need massive scalability.

## 53\. Data Compression

> Data compression reduces data size by encoding information more efficiently.

- Lossless compression (such as gzip, Brotli, or LZ4) allows the original data to be restored.
- Lossy compression (such as JPEG for images or MP3 for audio) reduces size by removing less important information, so the original cannot be perfectly reconstructed.

Compression reduces storage and network usage but requires CPU time to compress and decompress data.

![](https://substackcdn.com/image/fetch/$s_!1Fej!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F96d0b3ac-7532-4388-98c7-846effc0d983_749x503.png)

### Analogy

Lossless compression is like using abbreviations in text messages, where the meaning is perfectly recoverable.

Lossy compression is like summarizing a book where you keep the main plot but lose some details. You save space, but can’t recreate the original word-for-word.

### Tradeoff

It increases CPU usage and adds processing delay…

Compressed data usually cannot be modified directly without first decompressing it. Some algorithms are fast but achieve lower compression ratios, while others compress more but use more CPU.

### Why it matters

Use lossless compression for text files, JSON, HTML, CSS, JavaScript, logs, and database backups where exact recovery is required.

Use lossy compression for images, audio, and video where small quality loss is acceptable.

## 54\. ACID vs BASE

> ACID stands for Atomicity, Consistency, Isolation, and Durability.

It defines strict guarantees for transactions. A transaction either completes fully or not at all, follows defined data rules, does not interfere with other transactions, and remains stored even after failures.

> BASE stands for Basically Available, Soft state, Eventually consistent.

It describes a flexible approach used in distributed systems. The system remains available; data may be temporarily inconsistent, but it converges over time.

![](https://substackcdn.com/image/fetch/$s_!zWWL!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5f4c774c-1269-493c-ac2e-62e767353390_1600x873.png)

### Analogy

ACID is like a bank transfer. Money gets either fully moved or not moved at all, and your balance is always correct.

BASE is like counting social media likes. The count may differ slightly across servers for a short time, but eventually all servers show the same number.

### Tradeoff

ACID provides strong guarantees, making application logic simpler since you don’t handle inconsistencies, critical for financial systems and anywhere correctness is non-negotiable. Yet it reduces availability during failures and limits scalability.

BASE provides better availability and scalability and works well in distributed systems, but it complicates application logic because you must handle inconsistencies and conflicts.

### Why it matters

Use ACID when correctness is critical, such as in financial transactions, bookings, and inventory systems. Traditional relational databases support ACID transactions.

Use BASE when high availability and scalability are more important than immediate consistency, such as in social media feeds, analytics, caching layers, or NoSQL databases.

---

Know someone who wants to learn system design? Consider gifting a subscription:

---

## 55\. Network Partitions

> A network partition occurs when a network failure splits a distributed system into isolated groups that cannot communicate with one another.

Nodes within each partition can communicate, but can’t reach nodes in other partitions. When a partition occurs, the system must choose between maintaining consistency or remaining available (CAP theorem).

Network partitions are unavoidable in distributed systems.

![](https://substackcdn.com/image/fetch/$s_!KDpr!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F18f7f261-b008-40a8-b303-eebb0d37162a_1430x756.png)

### Analogy

Imagine a company whose internet connection to another office branch goes down:

Employees in each office can still work together locally, but the two offices cannot communicate. They must decide whether to keep working independently or wait until the connection gets restored.

### Tradeoff

When a partition happens, you must make a tradeoff…

You can either reject requests to preserve consistency or allow operations to continue,,, risking temporary inconsistencies.

Some systems use quorum-based [^9] approaches to balance availability and consistency. There’s NO way to guarantee both full consistency and full availability during a partition.

Detecting partitions quickly reduces impact but adds monitoring & coordination overhead.

### Why it matters

All distributed systems must handle network partitions:

- For systems where correctness is critical, choose a consistency-focused design.
- For systems where uptime matters more, choose an availability-focused design.

Design for partition detection, graceful degradation, and data reconciliation after recovery.

## 56\. Split Brain Problem

> Split brain occurs when network partitions cause different nodes to believe they’re the leader--each making conflicting decisions.

In a system designed for a single leader, a partition might lead each side to believe the other has failed, so both sides elect themselves leader and accept writes.

This leads to data inconsistency, conflicting writes, and potential data corruption when the partition heals.

![](https://substackcdn.com/image/fetch/$s_!_Ejf!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7d02e192-1b36-40c0-b3ab-e19d55c0c9e1_677x536.png)

### Analogy

Imagine two branch managers in a company who lose communication with each other:

Each thinks the other is gone and starts making company wide decisions. When communication is restored, they discover conflicting decisions.

### Tradeoff

To prevent split-brain, systems use coordination mechanisms such as majority voting (quorum), fencing tokens, or external coordination services [^10].

These mechanisms reduce availability during partitions but prevent data corruption. Allowing split-brain and reconciling later is complex and risks permanent conflicts.

### Why it matters

Any system with leader election or shared mutable state must prevent split-brain. It’s critical for databases, distributed locks, consensus systems, or cluster managers.

## 57\. Heartbeats

> Heartbeats are periodic signals sent between nodes in a distributed system to show they are alive.

A node sends a heartbeat message at regular intervals to other nodes or a monitoring system. If heartbeats stop within the expected timeout,,, the system assumes the node has failed and may trigger failover or raise an alert.

![](https://substackcdn.com/image/fetch/$s_!OxLu!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc85d1102-4ee6-481d-8362-3ed544ca244a_815x419.png)

### Analogy

It’s like a security guard checking in with headquarters every 15 minutes…

As long as the check-ins arrive, headquarters knows everything is fine. If they stop, headquarters assumes something is wrong and sends help.

### Tradeoff

They create continuous network traffic.

Also, network delays can cause false failure detection even if a node is healthy. Short intervals detect failures faster but increase false positives and overhead. While long intervals reduce overhead but delay failure detection.

### Why it matters

Heartbeats are essential for leader election, cluster membership tracking, health monitoring & automatic failover in distributed systems.

Choose heartbeat intervals based on how quickly you need to detect failures and how much network overhead you can tolerate.

## 58\. Leader Election

> Leader election is the process by which nodes in a distributed system agree on one node to act as the leader or coordinator.

The leader makes decisions, coordinates work, or acts as the single source of truth. If the leader fails, remaining nodes detect the failure and elect a new leader.

Protocols like Raft and Paxos ensure that only one leader is active at a time, even in the presence of network delays or failures.

![](https://substackcdn.com/image/fetch/$s_!E_5-!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7a82d004-f4fb-4a2a-aecf-363e9d7548e6_1460x709.png)

### Analogy

It’s like a group project where students choose one person to lead:

If the leader becomes unavailable, the group selects a new leader so work can continue. Only “one” leader is chosen to avoid conflicting decisions.

### Tradeoff

It creates a “temporary” single point of failure until failover completes.

The election process adds complexity and can cause instability if leaders frequently fail or recover. Plus, the leader can become a bottleneck if too much responsibility is centralized.

### Why it matters

Leader election is essential for distributed databases with a primary node, distributed locking systems, cluster management, and systems that require a single source of truth. It helps maintain consistency in distributed systems.

## 59\. Consensus Algorithms

> Consensus algorithms enable different nodes in a distributed system to agree on a single value or decision, even if some nodes fail or messages get delayed.

They guarantee:

- Agreement: All nodes commit to the same value.
- Validity: Chosen value was proposed by some node.
- Termination: Nodes eventually reach a decision under expected conditions.

Popular algorithms include Paxos, Raft, and Byzantine Fault Tolerant (BFT) variants.

![](https://substackcdn.com/image/fetch/$s_!yLs7!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8e65db49-23a3-44ad-8f53-e2be5ac6d5c1_1536x1024.png)

### Analogy

Think of a committee making a decision using a voting process:

Even if some members are temporarily disconnected, the group follows rules to ensure only one final decision is accepted & everyone agrees on it.

### Tradeoff

- It’s complex to design & implement correctly.
- It requires multiple message exchanges, which increases latency.
- It requires a quorum (majority) to make progress, so minority partitions cannot continue.
- Standard algorithms such as Paxos and Raft tolerate crash failures but do not handle malicious behavior; Byzantine Fault-Tolerant algorithms are more complex and expensive.

### Why it matters

Consensus is fundamental for distributed databases, leader election, distributed locks, cluster coordination, and maintaining a consistent shared state.

Raft is easier to understand & implement than Paxos.

In practice, it’s better to use mature systems such as etcd, ZooKeeper, or Consul rather than implementing consensus yourself…

## 60\. Quorum

> A quorum is the minimum number of nodes that must agree on an operation for it to be considered successful in a distributed system.

In a 5-node cluster, a common quorum is 3 nodes (a majority). i.e., a write must get acknowledgments from at least 3 nodes before it succeeds.

Quorums are set to a majority (more than half of N) to ensure that two different partitions cannot both make progress at the same time.

In some systems, read quorum (R) and write quorum (W) are configured so that:

```markup
R + W > N
```

This ensures that read & write operations overlap on at least one node, which maintains “strong” consistency.

![](https://substackcdn.com/image/fetch/$s_!bW1H!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9a5849d6-aadb-4b67-90f4-f6e407da812b_1536x1024.png)

### Analogy

It’s like a company board requiring a majority vote:

If there are 9 members and at least 5 votes are required, two separate groups cannot both approve conflicting decisions because neither group would have enough members to reach a majority.

### Tradeoff

- If a majority of nodes are unavailable, the system cannot make progress.
- Waiting for responses from many nodes increases latency.
- Larger quorum sizes improve consistency but reduce availability.

### Why it matters

Quorums are used in distributed databases such as Cassandra and DynamoDB, in leader election, distributed locks, and any system that requires consistent distributed writes.

---

Know someone who wants to learn system design? Consider gifting a subscription:

---

## 61\. Paxos Algorithm

> Paxos is a consensus algorithm that allows distributed nodes to agree on a single value even with failures and network issues.

It involves three main roles:

- Proposers suggest values,
- Acceptors vote on proposals,
- Learners learn the final chosen value.

Paxos guarantees safety, meaning only one value can be chosen.

It guarantees liveness (eventual progress) under certain conditions, such as a stable network and a majority of functioning nodes.

![](https://substackcdn.com/image/fetch/$s_!k5Fh!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ffae1658d-12de-4d64-bb7a-16b17c26bd4d_1536x1024.png)

### Analogy

Think of a formal voting process:

Members propose options & vote in structured rounds. Even if proposals compete or some members are temporarily unavailable, the rules ensure that only one final decision is accepted.

### Tradeoff

- It’s complex to understand & implement correctly.
- It requires many rounds of messaging, which adds latency.
- Progress can slow down if many proposals compete at the same time.

### Why it matters

Paxos is important for understanding distributed consensus theory…

Yet many systems prefer Raft because it offers similar guarantees but is easier to understand and implement.

Systems like ZooKeeper, etcd, and Consul provide consensus as a service, so you don’t need to implement Paxos yourself…

## 62\. Raft Algorithm

> Raft is a consensus algorithm designed to be easier to understand than Paxos while providing similar safety and consistency guarantees.

It works by:

- Leader election: One node becomes the leader.
- Log replication: Leader replicates log entries to follower nodes.
- Commit rules: An entry is committed once a majority of nodes store it.

Raft divides ‘time’ into terms, and in each term, there is at most one leader.

![](https://substackcdn.com/image/fetch/$s_!ix8_!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F41b37cd4-1ac5-4606-bc18-6ded1967644d_872x483.png)

### Analogy

It’s like a classroom electing a class president:

The president proposes decisions and shares them with everyone. If the president leaves, a new election is held. Only one president exists at a time, which avoids confusion.

### Tradeoff

- It requires a majority of nodes to make progress.
- Writes must go through the leader, which can become a bottleneck.
- Replication to a majority adds network latency.

Although simpler than Paxos, it’s still “complex” compared to non-consensus systems.

### Why it matters

Raft is widely used in distributed systems that require consensus, including distributed databases, configuration systems (such as etcd and Consul), cluster managers, and distributed locks.

In practice, you should use a mature Raft-based system rather than implementing Raft yourself.

## 63\. Gossip Protocol

> A gossip protocol is a decentralized communication method in which nodes periodically share information with randomly selected peers.

Each node exchanges state information with selected peers. Those peers then share the information with others. Over time, the information spreads across the system, usually very quickly.

There is no central coordinator…Gossip protocols are fault-tolerant and scale well to large clusters…

![](https://substackcdn.com/image/fetch/$s_!x8kQ!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F461c477f-728b-4be3-9efc-8efce856f573_820x407.png)

### Analogy

It’s like spreading news in a social group:

You tell a few friends; they tell a few others, and after several rounds, almost everyone knows. Some people may hear the same news more than once, but eventually it spreads widely.

### Tradeoff

- Messages might be duplicated.
- Convergence time can vary depending on network conditions.
- It cannot guarantee immediate consistency.

Most gossip systems are probabilistic--they achieve high reliability but NOT absolute guarantees.

### Why it matters

Gossip is used for:

- Cluster membership and failure detection
- State synchronization in distributed databases like Cassandra
- Configuration distribution in large systems
- Service mesh and container orchestration control planes [^11]

It works well in large-scale distributed systems where centralized coordination would not scale…But avoid gossip protocol when strict consistency, low-latency coordination, or guaranteed delivery is required.

## 64\. Clock Synchronization Problem

> Clock synchronization is the problem of keeping time consistent across machines in a distributed system.

Each machine has its own physical clock. These clocks drift over time and do not tick at exactly the same rate. Even when synchronized using Network Time Protocol (**NTP**), clocks can still differ by milliseconds or seconds.

This means you cannot safely use local timestamps to determine the exact order of events across different machines…Two nodes might disagree about which event happened first.

This makes event ordering, conflict resolution, and consistency more complex in distributed systems.

![](https://substackcdn.com/image/fetch/$s_!-A8Q!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd5d0df75-b4ef-4558-a41b-6b79c16b3a76_865x383.png)

### Analogy

Imagine coordinating a meeting where everyone’s watch is slightly off:

You say “ *arrive at 6 PM,”* but some watches are a little fast and others a little slow. When comparing watch times later, you cannot reliably know who actually arrived first.

### Tradeoff

Physical clocks synchronized with NTP are good enough for logs, monitoring, and most business applications.

Yet perfect synchronization is extremely difficult because of:

- Network delays
- Clock drift
- Variable message latency

You can improve precision with GPS or atomic clocks, but they are expensive…

### Why it matters

Understand that physical clocks are unreliable for distributed ordering.

For strict ordering, use logical clocks such as:

- Lamport clocks
- Vector clocks

These tracks “causality” instead of physical time.

## 65\. Logical Clock

> Logical clocks help systems figure out the order of events in distributed systems without using physical clock time.

Instead of relying on wall-clock timestamps, each event gets assigned a number based on causality. i.e., if event A happens before event B, then A gets a smaller number than B.

**Lamport timestamps** use a single counter per node. They can show that one event happened before another, but they cannot determine if two events happened at the same time independently.

**Vector clocks** use an array of counters, one for each node. They can determine whether events are related or happened independently, but they need more storage as the system grows.

![](https://substackcdn.com/image/fetch/$s_!ThRD!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7400a24c-8381-4c47-a07e-c7ab2e957c0f_808x353.png)

### Analogy

Think of numbering messages in a conversation thread…

You don’t care about the exact time on the clock. You only care that message 3 came after message 2. The numbers show the order, even if everyone’s phone clock is slightly different.

### Tradeoff

They do not show real-time.

Lamport clocks cannot detect concurrent events. Vector clocks take up more storage because they store a counter for each node. They also make application logic more complex.

### Why it matters

Distributed databases like DynamoDB or Cassandra for conflict resolution, version control systems for merging changes, distributed debugging to understand event causality, or any system needing to order events without synchronized clocks.

Use Lamport timestamps when you only need simple ordering. Use vector clocks to detect concurrent updates and resolve conflicts.

## 66\. Lamport Timestamp

> Lamport timestamps are a type of logical clock used to order events in distributed systems.

Each node keeps a counter…

- Before an event, the node increases its counter.
- When sending a message, it includes the current counter value.
- When receiving a message, the node sets its counter to the maximum of its current value and the received value, then increments it by 1.

This ensures that if event A happened before event B, then A has a smaller timestamp than B.

![](https://substackcdn.com/image/fetch/$s_!mb_c!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F64f5d763-6d38-4e41-bba0-43aeb3475b0e_707x315.png)

### Analogy

Imagine a shared to-do list where each person keeps their own number counter:

Before adding a new task, you increase your number. If you see someone else has added task 15 and your counter is at 12, move your counter to 16 before adding your next task.

This keeps the ordering consistent across everyone…

### Tradeoff

They cannot detect whether two events happened independently at the same time. Instead, they only guarantee ordering for events that are causally related (partial ordering). Also, they do not represent real-world time.

### Why it matters

Lamport timestamps are useful for tracking event order in distributed logs, debugging, or simple version ordering.

They are enough when you only care about “happened before” relationships.  
But if you need to detect concurrent updates, you should use vector clocks instead...

---

Know someone who wants to learn system design? Consider gifting a subscription:

---

## 67\. Vector Clock

> Vector clocks are logical clocks used to track the order of events (causality) across all nodes in distributed systems.

Each node keeps a list of counters--one counter for every node in the system:

- When a node performs an event, it increases its own counter.
- When it sends a message, it includes the entire list of counters.
- When another node receives the message, it compares the numbers and keeps the highest value at each position.

This helps the system understand if:

- One event happened before another
- Or two events happened independently at the same time (concurrent)

![](https://substackcdn.com/image/fetch/$s_!OUwk!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fccc6a068-68b6-4755-9256-bfa8a0099fec_808x417.png)

### Analogy

Imagine each family member tracking how many photos they have added to a shared album using a list.

Your list might say:

- You added 5 photos
- Your spouse added 3
- Your kid added 2

When you sync, you compare lists.

- If one list has equal or higher numbers everywhere, it’s more up-to-date.
- If some numbers are higher and some lower, both of you made concurrent changes, so you need to merge them.

### Tradeoff

- Every message carries the full list
- List of counters grows with the number of nodes
- This increases storage & network overhead

i.e., vector clocks do NOT scale well for systems with thousands of nodes.

### Why it matters

Useful in distributed databases like Riak or Voldemort for conflict detection and resolution, shopping cart merging in e-commerce, collaborative editing tools, or any eventually consistent system needing to detect concurrent updates.

They help you detect whether changes happened in order or concurrently…But avoid them in very large systems where the growing vector size becomes too expensive.

## 68\. Distributed Transactions

> Distributed transactions occur when a single logical operation spans many databases or services and must succeed or fail as a single unit.

If you update inventory in one service and charge a credit card in another, both actions must succeed together. If one fails,,, both must roll back. This keeps data consistent across systems.

To coordinate this, systems use protocols like Two-Phase Commit (2PC).

![](https://substackcdn.com/image/fetch/$s_!4mjj!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F61bea700-957a-4ce4-a1dd-f57b3a5e6e68_822x350.png)

### Analogy

It’s like booking a flight, hotel, and rental car as one package:

If the hotel booking fails, the flight and car get canceled as well. You don’t end up paying for half the trip.

### Tradeoff

Distributed transactions provide strong consistency & prevent ‘partial’ failures.

But they:

- Require all systems to be available at the same time
- Add extra network round trips for coordination & increase latency
- Create distributed deadlocks
- Limit scalability

Plus, they’re difficult to implement correctly and can affect performance.

### Why it matters

Use distributed transactions when correctness across systems is critical, such as for financial transfers or tightly coupled legacy systems.

But many distributed systems avoid them because they hurt availability and scalability. Instead, they use patterns like sagas or eventual consistency with compensation logic.

## 69\. Two-Phase Commit (2PC)

> Two-Phase Commit is a protocol that ensures a distributed transaction--either commits everywhere or rolls back everywhere.

*Phase 1 - Prepare phase:*

- A coordinator asks all participants if they’re ready to commit.
- Each participant locks its data, prepares the transaction, and votes “yes” or “no”.

*Phase 2 - Commit or Rollback phase:*

- If all participants vote “yes”, the coordinator tells everybody to commit.
- If any participant votes “no”, the coordinator tells everybody to roll back.

This guarantees atomicity across systems…

![](https://substackcdn.com/image/fetch/$s_!XP1y!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F421f0203-a181-4b86-b6f5-8427aa228ca7_835x318.png)

### Analogy

It’s like a group signing a contract:

First, everyone reviews it and says whether they agree. If everyone agrees,,, all sign it. If even one person refuses,,, nobody signs.

### Tradeoff

- Blocks if the coordinator crashes, leaving participants waiting with locked resources
- Requires all participants to be online
- Adds extra network round trips, increasing latency
- Creates a coordinator bottleneck and a potential single point of failure

Put simply, it improves consistency, but hurts availability & scalability.

### Why it matters

Use 2PC when strong ACID guarantees across systems are required, such as in financial systems or tightly coupled distributed databases.

If you use 2PC, the coordinator must be highly available & durable.

## 70\. SAGA Pattern

> SAGA pattern handles distributed transactions by breaking them into a series of small, local transactions.

Each service updates its own database and then sends an event or message to trigger the next step. If one step fails,,, compensating transactions get executed to undo the previous steps.

Unlike two-phase commit, each step commits immediately. There are two common styles:

- Choreography: services react to events from other services without a central controller
- Orchestration: central coordinator tells each service what to do next

![](https://substackcdn.com/image/fetch/$s_!fOCn!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbc0a3610-4328-4e7e-a011-8484b43f8ed6_828x460.png)

### Analogy

Imagine booking parts of a trip one by one:

You book a flight…Then a hotel…Then a rental car.

If the rental car fails, you cancel the hotel and flight manually. Each booking was confirmed immediately. If something fails, you cancel it step by step.

### Tradeoff

SAGAs improve availability because services don’t block each other…They scale well & avoid distributed locks.

But:

- They support only eventual consistency
- You might see “partial” results during execution
- Compensating actions can be complex
- Debugging becomes harder because of the distributed workflow

### Why it matters

- Use SAGAs in microservices architectures where distributed transactions would reduce availability.
- They’re ideal for long-running workflows such as order processing, payments, shipping, or booking systems.
- Use ‘choreography’ for simple event-driven flows.
- Use ‘orchestration’ when workflows are complex and need centralized control.

---

Know someone who wants to learn system design? Consider gifting a subscription:

---

## 71\. Outbox Pattern

> The outbox pattern ensures reliable publishing of events when updating a database.

Instead of:

1. Updating the database
2. Publishing an event

…as two separate steps that could fail independently, you:

- Write the data change
- Write the event to an outbox table

…in the “same” database transaction.

A separate background process then reads the outbox table and publishes the events to a message broker. This approach guarantees the event is NOT lost if the service crashes after committing the database change.

![](https://substackcdn.com/image/fetch/$s_!Oien!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc6ed039b-add4-4275-917a-9ba2cbe61210_819x412.png)

### Analogy

It’s like writing a letter and placing it in an outbox tray:

You write the letter & place it in the tray in one step. A mail carrier then picks up all the letters from the tray and sends them. You never risk updating your records, but you risk forgetting to send the letter.

### Tradeoff

Pros:

- Guarantees at-least-once event delivery
- Keeps database changes and events consistent
- Works within normal ACID database transactions

Cons:

- Adds a background process to publish events & increase complexity
- Introduces a small delay between database write and event publishing
- Might publish duplicate events, so consumers must be idempotent
- Increases database writes because of the outbox table

### Why it matters

Use the outbox pattern in microservices that publish events after database changes.

It’s critical in event-driven systems & SAGA workflows. Plus, it prevents event loss and keeps system state consistent, especially when reliability is more important than immediate delivery.

## 72\. Three-Phase Commit (3PC)

> Three-phase commit extends two-phase commit (2PC). It adds an extra step to reduce blocking if the coordinator fails.

Three phases are:

1. Can-commit: Coordinator asks participants whether they can commit. Each node votes “yes” or “no”.
2. Pre-commit: If all vote “yes”, coordinator tells them to prepare to commit. Then they acknowledge & enter a safe state.
3. Do-commit: Coordinator sends the final commit command, and all nodes commit the transaction.

The ‘pre-commit’ phase allows participants to decide after a timeout whether to proceed if the coordinator crashes, rather than blocking forever…

![](https://substackcdn.com/image/fetch/$s_!FqLX!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F83aab276-8370-4fe0-b1d2-abe3adc66d0d_838x313.png)

### Analogy

It’s like a relay race with three clear steps:

- First, runners get into position.
- Second, they hear “on your marks, get set.”
- Third, they hear “go” and start running.

That extra “get set” step helps. If the starter never says “go,” the runners know something went wrong and can safely reset instead of waiting forever.

### Tradeoff

3PC is non-blocking (unlike 2PC) because it allows participants to recover from coordinator failures using timeouts.

Yet it assumes:

- Bounded network delays--messages arrive within a known time.
- No network partitions.

These assumptions are unrealistic in distributed systems. It also adds another network round-trip, increasing latency compared to 2PC.

### Why it matters

Rarely used in practice because the assumptions don’t hold in real networks with unbounded delays & network partitions.

Most systems use 2PC with timeouts or avoid distributed transactions entirely with sagas or eventual consistency.

## 73\. Delivery Semantics (At-Most-Once, At-Least-Once, Exactly-Once)

> Delivery semantics describe how reliably a distributed system delivers messages.

- **At-most-once** means a message gets delivered at most once. It may be lost, but it will NEVER be duplicated.
- **At-least-once** means a message gets delivered one or more times. It will NOT be lost, but duplicates are possible.
- **Exactly-once** means a message gets delivered only once, with no loss and no duplicates.

![](https://substackcdn.com/image/fetch/$s_!qyUI!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe6065ed0-7bf7-4965-abb2-fc6a5c74c63b_833x398.png)

### Analogy

- At-most-once is like mailing a postcard. It might get lost, but you only send it once.
- At-least-once is like sending the same letter multiple times to make sure it arrives. The receiver may get duplicates.
- Exactly-once is like registered mail with tracking and strict checks, so the letter gets delivered only once.

### Tradeoff

- At-most-once is simple and fast, but messages can be lost.
- At-least-once guarantees delivery, but the receiver must handle duplicates. This requires idempotency.
- Exactly-once sounds ideal, but it is very difficult in distributed systems. It requires coordination, deduplication, and state tracking.

### Why it matters

- Use at-most-once for metrics or logs where occasional loss is acceptable.
- Use at-least-once for most reliable systems, especially when you can make operations idempotent.
- Use exactly-once semantics only when duplicates cause serious problems, such as financial operations.

## 74\. Change Data Capture (CDC)

> Change Data Capture is a method for tracking changes in a database and sending those changes to other systems.

CDC captures inserts, updates, and deletes from database transaction logs and publishes them as events. This enables real-time data synchronization without polling the database repeatedly or adding triggers that slow down writes.

![](https://substackcdn.com/image/fetch/$s_!zgsN!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fcf342a58-e719-479d-b4f8-7cbecb639430_806x335.png)

### Analogy

It’s like a reporter sitting inside a courthouse and reporting new filings immediately:

Instead of checking the courthouse website every hour, the reporter watches events unfold and shares updates in real time.

### Tradeoff

- It adds infrastructure complexity.
- Consumers depend on the database schema, so schema changes must be handled carefully.
- It increases coupling between systems.
- Transaction log formats differ across databases, making implementations database-specific.

### Why it matters

CDC is useful for:

- Synchronizing microservices without tight coupling
- Building materialized views or search indexes from database changes
- Feeding data warehouses for analytics
- Implementing event-driven architectures

Popular tools include Debezium and Maxwell.

## 75\. Long Polling

> Long polling is a technique in which the client makes a request to the server and keeps the connection open until new data is available or a timeout occurs.

Once data arrives, or the timeout expires, the server responds, and the client immediately makes another request. This simulates real-time updates over HTTP without WebSockets.

![](https://substackcdn.com/image/fetch/$s_!Huqp!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F747de49a-a6bd-438a-a720-42baad0d7037_809x337.png)

### Analogy

It’s like calling a restaurant and asking, *“Is my order ready?*”

Instead of saying *“No, call back later,”* they keep you on the line until the food is ready. Once they answer, you hang up and immediately call again to wait for the next update.

### Tradeoff

Long polling:

- Is easier to implement than WebSockets
- Works through firewalls and proxies that may block WebSockets
- Uses standard HTTP

But:

- It’s less efficient because each response requires a new request
- It adds overhead from repeated connections
- It still uses server resources while connections stay open
- It can add small delays between updates

### Why it matters

Use long polling when:

- WebSockets aren’t supported
- You need near real-time updates
- You’re working in restricted corporate networks

For high-scale real-time systems, WebSockets or other streaming methods are usually efficient.

## 76\. Server-Sent Events (SSE)

> Server-Sent Events enable servers to push updates to clients over a single HTTP connection.

The client opens a single connection, and the server pushes data whenever new information becomes available. SSE is one-way communication. Data flows only from server to client. The client cannot send messages back over the same connection.

Plus, SSE automatically reconnects if the connection drops.

![](https://substackcdn.com/image/fetch/$s_!3Sri!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F39d76699-f615-41c2-a29f-2d27ac7243ff_722x280.png)

### Analogy

It’s like subscribing to a live news ticker:

You open the page once, and headlines keep appearing automatically. You don’t send anything back. You just receive updates as they happen.

### Tradeoff

- It only supports one-way communication
- Browsers limit the number of open connections per domain
- It’s not supported in very old browsers

### Why it matters

Use SSE for:

- Live dashboards
- Notifications and alerts
- Stock prices or sports scores
- Any real-time updates that only need server-to-client communication

Use WebSockets when you need two-way communication between client and server.

## Final words

Again, none of those concepts exists in isolation--they’re building blocks that work together. You should understand the tradeoffs of each concept to make good architectural decisions.

Remember this:

*“Start simple. Add complexity only when the problem demands it. And when you do, you’ll know exactly which of these concepts to apply.”*

---

Know someone who wants to learn system design? Consider gifting a subscription:

---

![Author Neo Kim; System design case studies](https://substackcdn.com/image/fetch/$s_!bEFk!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8f94ab8c-0d67-4775-992e-05e09ab710db_320x320.png)

👋 Find me on LinkedIn | Twitter Threads Instagram

---

Thank you for supporting this newsletter.

You are now 200,001+ readers strong, very close to 201k. Let’s try to get 201k readers by 20 February. Consider sharing this post with your friends and get rewards.

Y’all are the best.

![system design newsletter](https://substackcdn.com/image/fetch/$s_!6oWl!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2e739087-a910-4643-be36-997b6dd5b4af_800x500.png)

system design newsletter

[^1]: - Request routing: Sending a client’s request to the correct backend service.
- Request composition: Combining responses from multiple services into a single response for the client.
- Protocol translation: Converting one communication format or protocol (like HTTP) into another (like gRPC) so different systems can talk to each other.

[^2]: Consistent hashing is a method of distributing data across multiple servers so that when a server is added or removed, only a small portion of the data needs to be moved.

[^3]: - SSL termination: The reverse proxy handles encryption and decryption of HTTPS traffic, so backend servers don’t need to manage certificates or encryption themselves.
- Traffic routing: The reverse proxy determines which backend server handles each incoming request.
- Protection against attacks: The reverse proxy can filter malicious traffic, limit request rates, and hide internal servers from direct exposure to the internet.

[^4]: Session resumption allows a client to reconnect without repeating the full handshake, and hardware acceleration leverages specialized CPU features to encrypt data more quickly, making TLS faster and more efficient.

[^5]: - DNS server: A server that translates a domain name (like example.com) into an IP address.
- Round-robin: DNS rotates through a list of IP addresses, giving a different one to each request in turn.
- Weighted distribution: Some IP addresses are returned more often than others, based on assigned weights (for example, more powerful servers get more traffic).
- Geographic routing: DNS returns the IP address of the server closest to the user’s location.

[^6]: - Border Gateway Protocol (BGP): The routing protocol that internet routers use to decide how traffic moves between networks.
- Internet service provider (ISP): A company that provides internet connectivity and routes traffic between networks.
- Advertise routes: Announcing to other networks that a server or IP address is reachable through a specific path.

[^7]: - Content Delivery Network (CDN): A network of servers around the world that stores and delivers website content from locations close to users.
- DDoS protection service: A system that protects websites from being overwhelmed by large amounts of malicious traffic.
- Root DNS servers: Top-level DNS servers that help direct domain lookups to the correct part of the Internet’s naming system.

[^8]: Hadoop ecosystem comprises open-source tools built around Hadoop that work together to store, process, and analyze very large datasets across many distributed servers.

[^9]: Quorum: A method where only a majority of nodes must agree before an operation succeeds.

[^10]: - Fencing tokens: Unique numbers given to a leader so that older leaders cannot continue making changes after losing authority.
- External coordination services: Separate systems (like ZooKeeper or etcd) that help nodes agree on leadership and shared state.

[^11]: - Service mesh control plane is the part of a service mesh that manages communication rules, security policies, and traffic routing between services.
- Container orchestration control plane is the central system that manages containers by scheduling them on machines, monitoring their health, and handling scaling and networking.