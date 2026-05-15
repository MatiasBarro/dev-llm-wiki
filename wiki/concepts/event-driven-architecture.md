---
type: concept
tags: [distributed-systems, system-design, architecture, pattern, event-driven]
sources:
  - "https://blog.bytebytego.com/p/a-guide-to-event-driven-architectural?utm_source=post-email-title&publication_id=817132&post_id=197472195&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-05-15
---

# Event-Driven Architecture

A communication style where services publish events (facts about things that happened) rather than calling each other directly. The foundational mental shift: `OrderPlaced` (fact) instead of `shipmentService.ship(order)` (command).

## Four Core Components

- **Producers** — emit events; don't know or care who consumes them
- **Consumers** — subscribe to event types and react on their own schedule
- **Broker** — durable intermediary that handles ordering, persistence, delivery guarantees, and partitioning (e.g. Kafka, RabbitMQ)
- **Event streams** — continuous sequences of events flowing through the system

The broker is doing real work, not just forwarding messages.

## Why EDA Exists

Synchronous request-response fails at scale in three specific ways:

1. **Cascading failure** — slow or down Service B blocks Service A, which blocks Service C
2. **Tight coupling** — shared APIs mean changing B forces changes in A; coordinated deploys
3. **Bottlenecks** — chain latency compounds; system speed = slowest link

EDA addresses all three. Producers don't wait. Services share event schemas, not APIs. The broker buffers traffic spikes.

EDA is not universally better — synchronous calls are correct when the caller genuinely needs an immediate response.

## The Six Patterns

### Competing Consumer
Multiple consumer instances pull from the same queue; the broker delivers each message to exactly one. Scales horizontally. Requires **idempotency** (broker redelivers on crash) and sacrifices strict ordering (mitigated by partitioning or sequence numbers).

### Consume and Project
A dedicated service consumes events from a primary system and writes its own read-optimized database (a materialized view). Offloads read traffic from the source. Trade-off: **eventual consistency** — the projection lags behind the source.

### Event Sourcing
Store every change as an event in an append-only log; current state is computed by replaying events. Built-in audit trail, temporal queries, easier concurrency reasoning. Cost: schema evolution is hard (old events must replay correctly years later), onboarding is slower. See [[concepts/event-sourcing]].

### Async Task Execution
Slow or unreliable work (emails, reports, external API calls) is moved off the request thread onto a queue. Workers pick up tasks with retry policies (typically exponential backoff) and dead-letter queues for persistent failures. Requires idempotency. Cost: observability — debugging requires tracing through queues, retry attempts, dead letters.

### Transactional Outbox
Solves the dual-write problem: the event is written to the same database as the business data in the same transaction, then a relay process forwards it to the broker. At-least-once delivery; consumers must be idempotent. See [[concepts/transactional-outbox]].

### Saga
Coordinates multi-service workflows via a sequence of local transactions and compensating actions on failure. Two coordination styles: choreography (each service reacts to events, no central coordinator) vs. orchestration (dedicated coordinator service). See [[concepts/saga-pattern]].

## Patterns Compose

Production systems stack several patterns simultaneously. A typical e-commerce flow:
- **Saga** coordinates order placement across Order, Payment, Shipping
- **Transactional Outbox** ensures each service publishes events reliably
- **Competing Consumer** scales notification and dashboard consumers
- **Consume and Project** maintains a customer-facing order summary view

The question is not which pattern to use for the whole system — it's which pattern fits which part.

## Cross-Cutting Requirements

**Idempotency** and **eventual consistency** are foundational, not optional. At-least-once delivery is the standard guarantee; every consumer must handle duplicate messages safely.

## When EDA Is the Wrong Choice

- Small monolith with one team and modest traffic — operational overhead outweighs benefit
- Workflows requiring strong consistency (financial transfers, inventory reservations) — use a single transaction in a single database
- Products where users expect immediate accurate feedback — eventual consistency surfaces as visible bugs
