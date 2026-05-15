---
type: paper-article
tags: [distributed-systems, system-design, architecture, pattern, event-driven]
sources:
  - "https://blog.bytebytego.com/p/a-guide-to-event-driven-architectural?utm_source=post-email-title&publication_id=817132&post_id=197472195&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-05-15
---

# A Guide To Event-Driven Architectural Patterns

ByteByteGo, December 2025.

Survey of event-driven architecture (EDA) fundamentals and six production patterns that solve the problems EDA introduces.

## Core Argument

Synchronous request-response communication creates three failure modes as systems grow: cascading failures (slow service blocks its callers), tight coupling (shared APIs force coordinated deploys), and bottlenecks (chain latency compounds). EDA addresses all three by having services publish facts rather than calling each other directly.

The trade is real: simplicity and transactional certainty are exchanged for decoupling, scalability, and resilience. Not always worth it.

## The Six Patterns

| Pattern | Problem Solved |
|---|---|
| Competing Consumer | Single consumer can't keep up with volume |
| Consume and Project | Read traffic overwhelms the source service |
| Event Sourcing | Current-state storage loses history |
| Async Task Execution | Slow/unreliable work blocks the request path |
| Transactional Outbox | Database write + event publish can't be made atomic |
| Saga | Multi-service workflows have no shared transaction |

See dedicated pages: [[concepts/event-driven-architecture]], [[concepts/event-sourcing]], [[concepts/saga-pattern]], [[concepts/transactional-outbox]].

## Key Takeaways

- **Events are facts, not commands.** `OrderPlaced` not `ship(order)`. This shift is the entire mental model.
- **Idempotency is non-optional.** At-least-once delivery means every consumer must handle duplicate messages safely.
- **Patterns compose.** A real e-commerce flow stacks Saga + Transactional Outbox + Competing Consumer + Consume and Project simultaneously at different layers. Beginners think they pick one; production systems use several.
- **Event Sourcing is oversold.** Strong fit for audit-heavy domains (finance, compliance). Schema evolution — old events must replay correctly years later — makes it expensive elsewhere.
- **When EDA is wrong:** small monoliths, workflows requiring strong consistency, products where users expect immediate accurate feedback after an action.
