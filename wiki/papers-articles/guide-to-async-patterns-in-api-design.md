---
type: paper-article
tags: [api, system-design, architecture, pattern, distributed-systems, networking]
sources:
  - "https://blog.bytebytego.com/p/a-guide-to-async-patterns-in-api?utm_source=post-email-title&publication_id=817132&post_id=197567426&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-05-22
---

# A Guide to Async Patterns in API Design

**Source**: ByteByteGo, 2026-05-21

A catalogue of eight async API patterns, organized around when plain request-response breaks down and what to use instead. The article introduces a "four levers" mental model that unifies all the patterns.

## Key Takeaways

**The four levers framework**: every async pattern is a configuration of (1) who initiates the connection, (2) how long the connection stays open, (3) whether a broker sits in the middle, and (4) what delivery guarantees apply. This is more useful than memorizing pattern names in isolation.

**SSE's revival via LLMs**: SSE was widely considered superseded by WebSockets, but it is now the dominant transport for LLM token streaming. One-way server-to-client push is exactly what SSE was built for, and the `EventSource` API with built-in reconnection makes it robust with minimal code.

**The WebSocket trap**: many features built on WebSockets would be better served by SSE. The bidirectional capability is only worth its operational cost when both sides genuinely produce high-frequency events. If only the server ever sends data, SSE is the right tool.

**Webhooks mean you're building a server**: the receiver is a server in everything but name. Production-grade implementations need signature verification, idempotency, retry handling with exponential backoff, and ideally a replay archive. The operational burden falls entirely on the consumer, not the provider.

**GraphQL subscriptions are a contract layer, not a transport**: they sit above WebSockets or SSE the same way a normal GraphQL query sits above HTTP. Comparing subscriptions to WebSockets as alternatives is a category error. They are useful inside a GraphQL ecosystem; adopting GraphQL solely for subscriptions is rarely justified.

**Short polling is still legitimate**: modern HTTP/2 has significantly reduced per-request overhead. For low-frequency events at modest scale, polling is often the simplest correct answer, and switching to a push-based pattern trades away operational simplicity for minimal gain.

## Patterns Covered

Full details in [[concepts/async-api-patterns]].

| Pattern | Core idea |
|---|---|
| Short polling | Client asks on a fixed timer; no special infrastructure |
| Long polling | Server holds the request until an event arrives |
| SSE | Server pushes a stream over one persistent HTTP connection |
| Webhooks | Server calls the client; receiver must act like a server |
| WebSockets | Full-duplex persistent channel; both sides send freely |
| Async APIs | 202 + job ID + poll or callback; for long-running work |
| Message queues | Durable broker decouples producer and consumer in time |
| GraphQL subscriptions | Query-layer contract over WebSockets/SSE |

## Connections

- The async task execution pattern (202 + job ID) overlaps with the Async Task Execution entry in [[papers-articles/guide-to-event-driven-architectural-patterns]]
- Message queues connect directly to [[concepts/event-driven-architecture]] and [[concepts/transactional-outbox]]
