---
type: concept
tags: [distributed-systems, databases, architecture, pattern, event-driven, audit]
sources:
  - "https://blog.bytebytego.com/p/a-guide-to-event-driven-architectural?utm_source=post-email-title&publication_id=817132&post_id=197472195&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-05-15
---

# Event Sourcing

Rather than storing the current state of an entity, store every change as an immutable event in an append-only log. Current state is computed by replaying those events.

Conventional approach: an `orders` table with a `status` column that gets overwritten. History is lost unless audit logging is explicitly built on top.

Event sourcing approach: the order *is* the sequence `OrderCreated → OrderShipped → OrderCancelled`. The table is a derived view.

## Two Anchoring Components

- **Event store** — durable append-only log of every event; functions as both database and message broker; the single source of truth
- **Read models** — projections built by replaying events into a queryable form; typically eventually consistent with the event store

## Benefits

- **Complete audit trail** built in — no need to add separate audit logging
- **Temporal queries** — reconstruct state at any point in the past by replaying events up to that moment
- **Concurrency** — events are stored sequentially, making conflict reasoning easier
- Natural fit with [[concepts/event-driven-architecture]] — the event store doubles as the broker for downstream consumers

## Costs

- **Schema evolution is hard** — when an event's shape changes, every old event in the store must still replay correctly, potentially years later; this is the biggest practical pain point
- **Querying current state** requires replay or carefully maintained projections; no simple `SELECT`
- **Onboarding** — mental model differs from standard CRUD; new engineers take longer to get productive

## When to Use

Strong fit for: finance, compliance, healthcare — any domain where the audit trail is genuinely valuable and required.

Oversold for: most CRUD applications where history is incidental rather than core. The costs are real and ongoing; don't adopt it for the wrong reasons.

## Relationship to Other Patterns

Event sourcing pairs naturally with [[concepts/event-driven-architecture#Consume-and-Project]] — read models are projections built from the event store. The [[concepts/transactional-outbox]] problem doesn't apply here because the event store *is* the database; writes and events are the same operation.
