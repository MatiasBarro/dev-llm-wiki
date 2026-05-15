---
type: concept
tags: [distributed-systems, system-design, architecture, pattern, event-driven, transactions]
sources:
  - "https://blog.bytebytego.com/p/a-guide-to-event-driven-architectural?utm_source=post-email-title&publication_id=817132&post_id=197472195&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-05-15
---

# Saga Pattern

Coordinates multi-service business workflows when no shared database transaction can span all participants.

The problem: placing an order touches Order Service, Payment Service, and Shipping Service — each with its own database. There is no distributed transaction that can span all three without re-introducing the tight coupling EDA exists to avoid.

The solution: break the process into a sequence of local transactions, one per service. Each step publishes an event that triggers the next. If any step fails, **compensating actions** undo the completed steps to restore consistency.

## Two Coordination Styles

### Choreography
Each service reacts to events it sees and publishes the next event. No central coordinator. Logic is distributed across participants.

- Pros: simple, no single point of failure
- Cons: workflow logic is scattered; hard to reason about as complexity grows; no single place to observe overall progress

### Orchestration
A dedicated Saga Orchestrator service directs each participant (sends commands, listens for results) and tracks workflow state.

- Pros: logic is centralized and observable; easier to debug and modify
- Cons: introduces a coordinator service that must be maintained and scaled

Both are valid. Choreography works well for simple, stable workflows. Orchestration earns its place when workflows have many steps, conditional branches, or complex failure handling.

## Example: Order Placement

1. Order Service creates order → triggers saga
2. Orchestrator → `ProcessPayment` command → Payment Service charges card → `PaymentCompleted`
3. Orchestrator → `InitiateShipping` command → Shipping Service arranges shipment → `OrderShipped`
4. If shipping fails: Orchestrator → refund via Payment Service → order marked cancelled

## Compensating Actions

The hardest part. The naive description treats them as automatic rollbacks, but they rarely are:

- A confirmation email cannot be unsent
- Reversing a payment might mean a refund, a credit, or manual intervention — depending on business policy
- Inventory reservation release may need to account for concurrent reservations

Real compensating actions are **business decisions disguised as technical recovery**. They require careful design and explicit product decisions, not just technical implementation.

## Eventual Consistency

Sagas accept partial state as unavoidable. During a multi-step workflow, some steps have completed and others haven't. For long-running workflows spanning minutes or hours, this window can be significant. The pattern manages the partial state rather than eliminating it.

## Relationship to Other Patterns

Each service in a saga typically uses the [[concepts/transactional-outbox]] to publish its events reliably. The saga itself is a pattern within [[concepts/event-driven-architecture]].
