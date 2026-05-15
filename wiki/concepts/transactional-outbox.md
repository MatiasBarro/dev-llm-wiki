---
type: concept
tags: [distributed-systems, databases, pattern, event-driven, reliability]
sources:
  - "https://blog.bytebytego.com/p/a-guide-to-event-driven-architectural?utm_source=post-email-title&publication_id=817132&post_id=197472195&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-05-15
---

# Transactional Outbox Pattern

Guarantees that a database write and an event publication either both happen or neither does, without requiring a distributed transaction.

## The Dual-Write Problem

A service often needs to do two things atomically: update its own database and publish an event announcing the change. These are two separate systems. If the database write succeeds but the broker publish fails, the event is silently lost. Downstream services never learn about the change. The system is inconsistent with no easy way to detect it.

This cannot be fixed by retrying the publish — the service may have already crashed. It cannot be fixed by publishing first — then the database write might fail and the event would describe something that didn't happen.

## The Solution

Write the event into the **same database** as the business data, in **the same transaction**. A separate relay process then forwards events from the outbox to the broker.

```
1. Begin transaction
2. Write business data to Order Table
3. Write event row to Outbox Table  ← same transaction
4. Commit

5. Relay process polls Outbox Table
6. Publishes event to broker
7. Marks outbox row as sent
```

Either both the business data and the outbox row commit, or neither does. The relay can be retried as many times as needed without affecting business data.

## Trade-offs

- **At-least-once delivery** — the relay may publish successfully, crash before marking the row as sent, and publish again on the next poll. Consumers must be idempotent.
- **Small latency** — milliseconds between the database commit and the event reaching the broker (the relay polling interval).
- **Operational overhead** — the outbox table must be maintained, polled, and cleaned up. The relay process must be monitored.

## Why It Matters

Most production event-driven systems rely on some version of this pattern whenever a service publishes events while writing to its own database — even when the pattern is not explicitly named in the architecture. It is foundational, not optional.

## Relationship to Other Patterns

Used by each participant service in a [[concepts/saga-pattern]] to ensure reliable event publication. Sits within the broader [[concepts/event-driven-architecture]] as a reliability primitive. Not needed with [[concepts/event-sourcing]] because the event store and the database are the same system.
