---
type: concept
tags: [api, system-design, architecture, pattern, distributed-systems, networking]
sources:
  - "https://blog.bytebytego.com/p/a-guide-to-async-patterns-in-api?utm_source=post-email-title&publication_id=817132&post_id=197567426&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-05-22
---

# Async API Patterns

Plain request-response breaks in four situations: the work takes too long to fit inside a single HTTP timeout, the server has events the client can't predict, both sides need to exchange messages at high frequency continuously, and the producer and consumer aren't online at the same time. Every async API pattern exists to address one of these four failure modes.

## The Four Levers

Every pattern is a specific configuration of four mechanical choices:

1. **Who initiates** — does the client pull, or does the server push?
2. **Connection duration** — short-lived per request, or long-lived persistent?
3. **Broker in the middle** — direct connection, or durable intermediary?
4. **Delivery guarantees** — at-most-once, at-least-once, exactly-once?

## Patterns

### Short Polling
Client sends regular GET requests on a fixed timer. Server responds with whatever it has and closes the connection. No special infrastructure required. Most requests return empty when nothing has changed.

- **Best for**: low-frequency events where operational simplicity outweighs latency
- **Cost**: wasted requests; largely mitigated by HTTP/2's lower per-request overhead

### Long Polling
Client sends a request; server holds it open (tens of seconds) until an event arrives or a timeout fires, then responds. Client immediately re-issues the request.

- **Best for**: environments where WebSockets are unreliable (e.g. behind corporate firewalls)
- **Cost**: server must manage many open connections; connection wasted on every timeout

### Server-Sent Events (SSE)
Client opens a single long-lived HTTP connection. Server pushes a stream of text-formatted events as they occur. One direction only: server → client. The browser's `EventSource` API handles reconnection automatically.

- **Best for**: one-way server-to-client streams — notably LLM token streaming, where SSE is now the dominant transport
- **Cost**: unidirectional; client cannot send messages over the same connection
- **Note**: many features built on WebSockets would be served better by SSE

### Webhooks
Inverts HTTP direction: when an event occurs, the provider sends an HTTP POST to a URL registered by the consumer. The webhook receiver is a server in everything but name.

Production requirements:
- **Signature verification** — confirms events came from the provider, not an attacker
- **Retry handling with exponential backoff** — the receiver will go down
- **Idempotency** — the same event will be delivered more than once under normal operation
- **Replay archive** — lets consumers recover events missed during outages

- **Best for**: server-to-server integration where the receiver isn't a browser and doesn't want a persistent connection
- **Cost**: all operational responsibility falls on the receiver

### WebSockets
HTTP connection upgraded to a persistent, full-duplex channel. Both sides can send messages at any time without request-response framing.

- **Best for**: collaborative editors, multiplayer games, live trading interfaces — any case where **both** sides genuinely produce high-frequency events
- **Trap**: using WebSockets when only the server sends events; SSE handles that case more cleanly at lower operational cost

### Async APIs (Job Pattern)
Client submits a request (POST). Server immediately returns `202 Accepted` with a job ID and status URL. Computation runs separately. Client either polls the status URL or registers a webhook for the callback.

- **Best for**: long-running compute — video transcoding, ML inference, bulk exports, report generation
- **Cost**: client must track a state machine across multiple interactions; server needs status endpoints, job storage, and result delivery

### Message Queues
A broker sits between producer and consumer, holding messages durably. Producer writes to the broker; consumer reads on its own schedule. The two never connect directly.

Benefits:
- Messages survive consumer downtime
- Fan-out: the same message reaches many consumers
- Crash recovery: broker retains the message if the consumer crashes mid-processing
- Natural backpressure during demand spikes

- **Best for**: time-shifted communication — producer and consumer on different clocks, fan-out to many consumers
- **Cost**: stateful infrastructure to operate; ordering and delivery guarantees vary between brokers

See also: [[event-driven-architecture]], [[transactional-outbox]]

### GraphQL Subscriptions
A contract layer, not a transport. A subscription is a GraphQL query that requests an ongoing stream of updates when specific data changes. The underlying transport is almost always WebSockets or SSE.

- **Key distinction**: subscriptions operate at the application/query layer; WebSockets operate at the transport layer. Comparing them as alternatives is a category error.
- **Best for**: teams already invested in GraphQL who want consistent client-side semantics across queries, mutations, and live data
- **Rarely worth**: adopting GraphQL solely to get subscriptions

## Pattern Selection Summary

| Situation | Pattern |
|---|---|
| Low-frequency events, simplicity first | Short polling |
| Near-real-time, firewalls block WebSockets | Long polling |
| Server streams to client (incl. LLM output) | SSE |
| Server-to-server event delivery, no persistent connection | Webhooks |
| Both sides send high-frequency events | WebSockets |
| Work takes longer than HTTP timeout allows | Async API (job pattern) |
| Producer and consumer on different clocks | Message queues |
| GraphQL ecosystem, live queries | GraphQL subscriptions |
