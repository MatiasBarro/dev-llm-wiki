---
title: "A Guide To Event-Driven Architectural Patterns"
source: "https://blog.bytebytego.com/p/a-guide-to-event-driven-architectural?utm_source=post-email-title&publication_id=817132&post_id=197472195&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
author:
  - "[[ByteByteGo]]"
published: 2025-12-15
created: 2026-05-15
description: "Distributed systems are built out of services that need to communicate, and the simplest way to do that is for one service to call another directly and wait for a response."
tags:
  - "raw"
---
Distributed systems are built out of services that need to communicate, and the simplest way to do that is for one service to call another directly and wait for a response. This pattern works well for small systems and predictable workloads.

However, as systems grow, it tends to produce tight coupling between services, fragile failure behavior, and bottlenecks at the slowest component in any chain of calls.

Event-driven architecture is an alternative communication model where services publish events when something meaningful happens, and other services react to those events on their own time. The patterns related to this architecture are the established techniques for handling the new problems that the model introduces.

In this article, we will start with the basics of how event-driven systems are structured, look at why synchronous communication starts to break down at scale, and then walk through six patterns that solve specific problems EDA introduces.

![](https://substackcdn.com/image/fetch/$s_!-BJY!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F57ee764c-c720-46a6-9c66-dbb53a5e646e_2250x2624.png)

## The Foundations of Event-Driven Architecture

Event-driven architecture is a communication style where services don’t call each other directly. Instead, when something meaningful happens in one service, that service publishes a record of the fact, and any other service that cares about it reacts on its own schedule. The crucial word is fact.

An event describes something that has happened, like OrderPlaced or PaymentCompleted, rather than a command instructing another service to do something. This shift from telling other services what to do to broadcasting what has happened is the entire mental model.

The consequence of that shift is asynchronous flow.

The service that publishes an event doesn’t wait for anyone to process it and continues with its own work. A consumer might process the event in five milliseconds or five minutes, and from the publisher’s perspective, that timing doesn’t affect its own progress.

Four components make this work:

- Producers are the services that emit events. They publish to a broker without knowing or caring who consumes what they send.
- Consumers subscribe to specific event types and react when those events appear. Each consumer is responsible for its own logic and its own state.
- Brokers sit between producers and consumers, holding events durably and routing them to the right places. The broker is doing real work here, handling ordering, persistence, delivery guarantees, and partitioning.
- Event streams are the continuous sequences of events flowing through the system. They are how services stay in sync without ever talking directly.

![](https://substackcdn.com/image/fetch/$s_!3g6p!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa7d443b9-e026-434b-9934-6d39dbb71d41_2136x1104.png)

The benefits associated with EDA, like decoupling, scalability, and fault tolerance, all flow from these four components working together.

Producers don’t depend on consumers being available. Consumers can scale independently based on their own load. If one consumer fails, the others keep working. The broker absorbs traffic spikes by buffering events until consumers catch up. None of this is magic. It is what happens when services no longer need to wait on each other to make progress.

## Why Not Just Call Another Service Directly?

The simplest way for two services to communicate is for one to call the other directly. Service A makes an HTTP request to Service B, waits for the response, and continues. This is direct, easy to debug, and works fine when the system is small.

It also has three specific failure modes that get worse as the system grows.

- The first is cascading failure. When Service B is slow or down, Service A is slow or down too, because A is blocked waiting for B. If Service C calls A, then C is also stuck. A single failing service can take down a chain of dependents.
- The second is tight coupling. When A calls B’s API directly, any change to B’s interface forces a change in A. Over time, this turns into a coordination problem where deploying one service requires checking the others.
- The third is bottlenecks. In a chain of synchronous calls, the system is only as fast as the slowest link, and adding more services to the chain compounds the latency.

EDA addresses all three. The producer doesn’t wait, so a slow consumer doesn’t slow down the producer. Services don’t share APIs, only event schemas, so changes are easier to roll out independently. And because work is buffered in the broker, traffic spikes get absorbed instead of cascading.

This doesn’t mean synchronous calls are wrong. They are often the right answer when the caller genuinely needs an immediate response, like a user clicking a button and expecting to see the result right away. EDA is a different tool, not a better one. It earns its place by solving problems that synchronous communication can’t.

![](https://substackcdn.com/image/fetch/$s_!LwFE!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fdd63c1e7-55d2-4fad-be47-a5a5c390f40a_3244x2064.png)

Despite the advantages, shifting to events introduces new challenges of its own.

Each of the patterns covered next exists because of a specific way an event-driven system solves a certain challenge or overcomes a problem.

## The Competing Consumer Pattern

Consider a single consumer pulling messages from a queue and processing them one at a time. As event volume grows, the consumer falls behind, the queue keeps lengthening, and latency increases. Adding more events to the queue doesn’t help because the consumer itself is the bottleneck. The system needs parallel processing.

The Competing Consumer pattern solves this by running multiple instances of the consumer, all pulling from the same shared queue. The broker, often something like RabbitMQ, hands each message to exactly one of them. As the load increases, more consumer instances are added. As the load decreases, instances are removed. The system scales horizontally, and the broker handles the distribution.

See the diagram below:

![](https://substackcdn.com/image/fetch/$s_!3-1R!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8692ce9a-1c1d-403d-8228-64d66ac80069_2482x1258.png)

The mechanics are straightforward. Each message in the queue goes to one consumer. If a consumer crashes mid-processing, the broker redelivers the message to another instance. This redelivery is what makes the pattern fault-tolerant, though it has consequences.

Consumers must be idempotent, meaning that processing the same message twice produces the same result as processing it once. Without idempotency, redelivery causes duplicate work and an inconsistent state.

The other tradeoff is ordering. With a single consumer, messages are processed in the order they arrive. With multiple competing consumers, that guarantee is gone. When order matters, the queue can be partitioned so related messages always go to the same consumer, or sequence numbers can be embedded in the messages themselves.

This pattern shows up wherever there is high-volume background work that needs to be processed reliably and at scale.

## The Consume and Project Pattern

Some systems get overwhelmed by read traffic.

Consider an Order Service that holds order data, and several downstream services that need to query orders in different ways. Reports compete with transactional queries, dashboards lag, and certain use cases need order data joined with customer data, which means querying both the Order Service and the Customer Service repeatedly.

The Consume and Project pattern solves this by building separate read-optimized views. A dedicated service consumes events from the primary system and writes its own database, shaped specifically for the queries that need to be supported. The primary system stops being the central read store, and each consumer maintains its own projection.

See the diagram below:

![](https://substackcdn.com/image/fetch/$s_!6g6M!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F46e04b83-df82-4a6c-b3f1-3296ef5bc2ba_2102x1308.png)

The flow above follows a clear sequence.

The Order Service publishes an OrderCreated event whenever a new order is placed. An Order Projection Service consumes these events, enriches them with customer data fetched from the Customer Service, and writes the combined result into its own Customer-Order materialized view. Client services that need to read this combined data query the projection directly instead of going to the original services.

The tradeoff is eventual consistency. The projection lags behind the source, sometimes by milliseconds, sometimes longer if the consumer falls behind. For dashboards and reports, this is acceptable. For workflows that need to see their action reflected immediately, the lag is worth thinking about before adopting this pattern. The pattern also introduces multiple databases that need to be operated, monitored, and backed up, which adds operational overhead.

## Event Sourcing

The conventional approach to data is to store the current state of an entity. In a typical order management system, an order has a status field that holds whatever the latest value is, and any prior values are overwritten as the order progresses. The history of how the order got there is gone unless audit logging has been explicitly built on top.

Event Sourcing inverts this approach.

Rather than storing the current state, the system stores every change as an event in an append-only log, and the current state is computed by replaying those events. For example, in an order management system, the order itself is no longer a row that gets updated. It is the result of applying a sequence of events such as OrderCreated, OrderShipped, and OrderCancelled in the order they occurred.

![](https://substackcdn.com/image/fetch/$s_!-BqP!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F345dca0f-cdbe-449f-871e-411c9cf8520b_2218x1402.png)

Two components anchor this pattern.

- The event store is a durable append-only log of every event that has ever happened in the system, functioning as both a database and a message broker.
- Read models are projections built by replaying events into a queryable form, and they are typically eventually consistent with the event store.

When the current state of an order is needed, the system either replays the relevant events from the store or queries a pre-built read model that has been kept up to date as new events arrive.

The benefits are real. A complete audit trail comes built in because the events are the source of truth, which is valuable in domains where compliance or detailed history matters. Temporal queries become possible, allowing the state of an order at any point in the past to be reconstructed by replaying events up to that moment. Concurrency conflicts are also easier to reason about because events are stored sequentially.

The costs are also real. Schema evolution is hard. When the shape of an event changes, every old event in the store still needs to replay correctly, sometimes years later. Querying the current state requires either replay or carefully maintained projections. New engineers take longer to onboard because the mental model differs from the standard CRUD approach most are used to.

Event Sourcing is a strong fit for finance, compliance, and any domain where the audit trail is genuinely valuable.

## The Async Task Execution Pattern

Some work is slow, unreliable, or both. For example, sending emails involves a third-party service that might rate-limit the caller, generating a report can take thirty seconds, and calling an external API sometimes times out. Performing such work inline, on the user’s request thread, is a poor design choice. The user waits too long, and any failure becomes visible as their failure.

The Async Task Execution pattern moves this kind of work out of the request path.

A job scheduler or producer puts a task on a queue managed by an event broker. Worker instances pick up tasks and execute them. If a task fails, it is retried according to a defined retry policy. If it keeps failing, it is moved to a dead letter queue for manual investigation.

![](https://substackcdn.com/image/fetch/$s_!9X36!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8153383b-5fd5-40f3-b700-4342516cd14e_1978x1104.png)

The mechanics build on familiar pieces. Tasks become messages, workers become consumers, and retries are part of the consumer logic.

What makes this pattern distinct is the retry policy itself. Most implementations use exponential backoff, where the delay between retries grows with each attempt, so a flaky downstream service has time to recover instead of being hammered by a tight retry loop. Many setups also use multiple queues with different priorities, so urgent work gets processed before background work.

Idempotency matters here for the same reason it mattered with Competing Consumers. A task might be retried multiple times, and each retry must produce the same result as a single successful run. When a task sends an email, the worker must ensure retries don’t send the same email three times. This usually involves tracking what has already been completed and checking before acting again.

The main cost is observability. Work happens out of a straight flow, which makes debugging harder. When a user reports that their report never arrived, tracing the cause involves stepping through queues, workers, retry attempts, and dead letters. Distributed tracing helps, but it adds its own complexity to the system.

## The Transactional Outbox Pattern

Event-driven systems run into a structural problem early on. A service often needs to do two things atomically, such as updating its own database and publishing an event saying that the change occurred.

When the database write succeeds but the event publish fails, the event is lost. Downstream services never find out about the change. The system is now inconsistent, and there is no easy way to detect it.

This is called the dual-write problem, and it has no clean solution as long as the database and the broker are treated as separate systems. They cannot be placed inside a single transaction because they don’t share one.

The Transactional Outbox pattern solves this by writing the event to the same database as the business data, in the same transaction.

Consider an Order Service processing a new order. The service updates the Order Table with the new order details and inserts a corresponding event into an Outbox Table, both within a single database transaction. A separate process scans the Outbox Table for new events, publishes them to the message broker, and marks them as sent. If the publish step fails for any reason, the event stays in the outbox until the next attempt succeeds.

![](https://substackcdn.com/image/fetch/$s_!b1z8!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2ff3c76c-52c2-44ad-b369-4b9cdd0b8ee1_2508x1540.png)

The mechanics work because the database transaction is the atomic unit. Either both the business data and the outbox row are committed, or neither is. The publish step happens after the transaction commits, and it can be retried as many times as needed without affecting the business data.

The cost is at-least-once delivery. The publisher might successfully publish a message, fail to mark it as sent due to a crash, and then publish it again on the next poll. Consumers must handle this through idempotency. There is also a small delay between the database commit and the event reaching the broker, usually in the millisecond range, which is worth keeping in mind.

Despite its straightforward implementation, this pattern is foundational. Most production event-driven systems rely on some version of it whenever a service publishes events while writing to its own database, even when the pattern is not explicitly named in the architecture.

## The Saga Pattern

Some business processes touch multiple services. For example, placing an order involves an Order Service, a Payment Service, and a Shipping Service. Each has its own database, and each owns part of the overall transaction. There is no shared database transaction that can span all three because the whole point of the architecture is that they are independent.

The Saga pattern handles this by breaking the process into a sequence of local transactions, each running inside one service. After each step, the service publishes an event that triggers the next step. If any step fails, the saga runs compensating actions to undo the previous steps and bring the system back to a consistent state.

![](https://substackcdn.com/image/fetch/$s_!qD0U!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa83dc728-f0bc-4a9d-8e5d-70a62c3dd4d2_2218x1688.png)

In the order example, the Order Service creates the order and triggers the saga. An Order Saga Orchestrator coordinates the flow by sending a Process Payment command to the Payment Service, which charges the card and publishes a PaymentCompleted event.

The orchestrator then sends an Initiate Shipping command to the Shipping Service, which arranges shipment and publishes an OrderShipped event. The orchestrator updates the order status accordingly. When shipping fails, the saga issues a refund through the Payment Service and marks the order as cancelled. Each step is a normal database transaction inside its own service, and the coordination happens through events flowing through the broker.

There are two ways to coordinate a saga.

- In choreography, each service knows what to do based on the events it sees, and there is no central coordinator.
- In orchestration, a dedicated service tells each participant what to do and tracks the progress of the workflow.

Choreography keeps logic distributed, but can become hard to reason about as workflows grow. Orchestration centralizes the logic at the cost of introducing a coordinator service. Both are valid, and the right choice depends on the complexity of the workflow.

The hardest part of sagas is compensating actions. The naive description treats them as automatic rollbacks, but they are rarely that clean. A confirmation email cannot be unsent. Reversing a payment might mean a refund, a credit, or a manual intervention, depending on policy. Real compensating actions are business decisions disguised as technical recovery, and they need careful design.

Sagas also accept eventual consistency as part of the deal. There is a window where some steps have been completed, and others have not, and during that window, the system is in a partial state. For long-running workflows that span minutes or hours, this is unavoidable. The pattern manages the partial state rather than eliminating it.

## When EDA Is the Wrong Choice

Each of these patterns earns its place by solving a specific problem, but EDA itself is a deliberate trade, and there are systems where the trade is not worth making.

For example, a small monolith with one team, one database, and modest traffic gets very little from EDA beyond operational overhead. A broker is a new component to monitor, scale, and recover. Consumers add complexity. Eventual consistency creates UX issues that did not exist before. When the simple synchronous design works, it should be kept.

Workflows that genuinely require strong consistency belong in a single transaction inside a single database. Financial transfers, medical records, and inventory reservations often fall into this category. Splitting them across services purely to feel modern produces worse systems, not better ones. A transaction is the right tool when atomicity across operations is non-negotiable.

Products where users expect immediate, accurate feedback after an action will struggle with eventual consistency. When a user clicks a button, and the next screen needs to show the result accurately, the lag in a projection or the in-flight state of a saga will surface as bugs from the user’s perspective. The architecture has to fit the product’s UX promises.

One thing worth highlighting is that mature event-driven systems rarely rely on just one of these patterns.

For example, a typical e-commerce flow might use a Saga to coordinate an order placement across the Order, Payment, and Shipping services, with each of those services using the Transactional Outbox pattern to publish events reliably to the broker. Downstream services that send notifications or build dashboards might consume those events using Competing Consumers to scale, and a separate read service might apply Consume and Project to maintain a customer-facing order summary view.

See the diagram below:

![](https://substackcdn.com/image/fetch/$s_!w2VS!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff2c5cd29-a6ba-4d69-885c-d1e635cfb7d5_2548x2144.png)

In other words, patterns compose, and the right question is not which pattern to choose for the entire system, but it is which pattern fits which part of the system. Many beginners think they are picking one. In practice, several are stacked together to handle different concerns at different layers.

EDA is a design decision. The simplicity of synchronous calls and the certainty of transactions are exchanged for decoupling, scalability, and resilience. That trade is worth it for many systems and wrong for many others, and knowing which applies is the actual skill.

## Summary

In this article, we have looked at event-driven architecture in detail. Here are the key learning points:

- Event-driven architecture is a communication style where services publish events about facts rather than calling each other directly. This shift from commands to facts is the foundational mental model.
- The four core components of an EDA system are producers, consumers, brokers, and event streams. The broker is doing real work, not just forwarding messages.
- EDA exists because synchronous request-response creates coupling, cascading failures, and bottlenecks as systems grow. It addresses these specific problems rather than being a generic upgrade.
- Each EDA pattern solves a specific problem. Competing Consumer scales out processing, Consume and Project offloads read traffic, Event Sourcing captures complete history, Async Task Execution handles slow or unreliable work, Transactional Outbox guarantees reliable event publication, and Saga coordinates workflows across services.
- Idempotency and eventual consistency are foundational requirements, not optional details. Any production EDA system has to handle them deliberately.
- Event Sourcing is powerful but oversold. It is the right choice for audit-heavy domains and often unnecessary elsewhere.
- Mature systems combine several patterns at once rather than picking just one. A real e-commerce flow might use Saga, Transactional Outbox, Competing Consumer, and Consume and Project together at different layers of the system.
- EDA is a deliberate trade between simplicity and resilience. It is the right choice for many systems and overkill for others, and knowing the difference is the actual skill.