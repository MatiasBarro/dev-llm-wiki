---
title: "A Guide to Async Patterns in API Design"
source: "https://blog.bytebytego.com/p/a-guide-to-async-patterns-in-api?utm_source=post-email-title&publication_id=817132&post_id=197567426&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
author:
  - "[[ByteByteGo]]"
published: 2026-05-21
created: 2026-05-22
description: "In this article, we will look at each of these patterns in detail, along with their advantages."
tags:
  - "raw"
---
The default model for client-server communication is request-response. The client sends a request, the server returns a response, and the connection closes. This handles the majority of what most software needs to do, and it covers an enormous range of practical scenarios in modern web applications.

It doesn’t handle everything. Some work takes too long to complete inside a single request. Some events happen on the server’s schedule, not the client’s. Some interactions are continuous rather than one-shot. And some messages need to outlive the moment they were sent.

Async API patterns are the techniques engineers use for these cases, and the list has grown over the years. It now includes short polling, long polling, server-sent events, WebSockets, webhooks, async APIs with status polling, message queues, and GraphQL subscriptions. Each has its own design and preferred use case. What they share is one purpose, which is to extend what’s possible beyond a single HTTP request and response.

In this article, we will look at each of these patterns in detail, along with their advantages. We’ll start by looking at where request-response stops fitting and then walk through each pattern in turn.

![](https://substackcdn.com/image/fetch/$s_!LYCw!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F76f47049-9f51-40f2-a453-794a63b7d322_2250x2624.png)

## Request-Response and Its Limits

Request-response works well because the exchange is short, stateless, and self-contained. There are four common situations where this stops being enough, and every async pattern in common use exists to address one of them.

- When the work itself takes a long time, like a request that transcodes a video or runs a machine learning inference job, the work won’t fit inside the seconds-long window that most HTTP infrastructure tolerates before timing out.
- When the server has events the client can’t predict, like a new chat message arriving or an order status changing, the event happens on the server’s clock, and the client has no way to know when to ask for it.
- When both sides need to keep producing events at high frequency, like two people editing the same document or two players in the same match, the result is a continuous stream of small messages flowing in both directions.
- When the producer of an event and its consumer aren’t running on the same clock, the producer might have news right now while the consumer is offline, slow, or one of many systems that need to receive it.

Underneath all the patterns we’ll cover are four mechanical levers. They determine who initiates the connection, how long the connection stays open, whether a third party sits in the middle, and what guarantees the design provides on delivery.

Every pattern in this article is a specific configuration of these four levers. Let’s look at them one by one.

## Short Polling

Short polling is the simplest answer to the question. The client asks the server, on a fixed schedule, whether anything new has happened. Most of those requests come back empty, and the ones that find new data return it before the cycle continues.

The mechanics are straightforward and require no special infrastructure. A timer on the client side fires every few seconds, the client sends a regular GET request, the server responds with whatever it has, and the connection closes. Each request is ordinary HTTP, and the server has no idea that any of them belong to a polling pattern.

![](https://substackcdn.com/image/fetch/$s_!Bwc6!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F295a681b-cd13-4a77-930a-100118289820_1734x1496.png)

The strength of short polling is that nothing about it is special. There are no held connections, no protocol upgrades, and no infrastructure beyond what every web service already runs. The cost is the wasted requests when nothing has changed.

Modern HTTP, especially HTTP/2, has narrowed that cost considerably. The per-request overhead is much lower. For low-frequency events at a modest scale, polling is often the simplest correct answer, and switching to a push-based pattern would lose operational simplicity for no major gain.

## Long Polling

Long polling is an upgraded version of short polling. The client sends a request, but the server doesn’t respond immediately. Instead, the server holds the request open, sometimes for tens of seconds, until either an event happens or a timeout fires. Then the response comes back, and the client immediately issues another request.

![](https://substackcdn.com/image/fetch/$s_!rgra!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fbcc9e269-68dd-4d34-b230-8a6de40b4d5f_1788x1410.png)

From the client’s perspective, this looks like a push, since new data arrives the moment it exists with whatever latency the network adds. Underneath, it’s still a pull facilitated by an ordinary HTTP request that the server happens to take a long time to answer.

The tradeoff has shifted away from long polling over time. Servers have to manage many open connections, which complicates scaling, and the connection is wasted on every timeout. Long polling still appears in environments where WebSockets are unreliable, like behind corporate firewalls, but for most use cases, SSE or WebSockets do the same job more cleanly.

## Server-Sent Events

Server-Sent Events take the held-connection idea and formalize it.

The client opens a single long-lived HTTP connection to the server, and the server uses it to push a stream of text-formatted events whenever it has something to send. There are no client-issued follow-up requests, and the result is one open connection carrying many messages in a single direction.

![](https://substackcdn.com/image/fetch/$s_!yx-A!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F13f6bce1-cdc7-48a6-8041-86f8ded14048_1814x1372.png)

SSE is cleaner than long polling in three concrete ways:

- The browser’s EventSource API handles reconnection automatically when the connection drops.
- The text format is human-readable and easy to debug.
- The server doesn’t have to fake the held connection with timeouts, since the protocol expects a long-lived stream from the start.

For a few years, SSE looked like a pattern that had been quietly superseded by WebSockets. However, the current generation of large language model interfaces relies heavily on SSE, because token streaming from server to client is exactly the one-way push pattern SSE was built for. SSE has become one of the most heavily used async transports in modern software.

When the server has a stream of updates, and the client doesn’t need to talk back during that stream, SSE is usually the right answer.

## Webhooks

Webhooks invert the usual direction of HTTP. Instead of the client calling the server when it wants something, the server calls the client when something happens. More precisely, when an event occurs at a provider, the provider sends an HTTP POST to a URL that the consumer registered earlier.

A webhook receiver is a server in everything but name. It exposes a publicly reachable endpoint and treats incoming events the way any HTTP service treats incoming requests. The consumer takes on all the work a server does and inherits all the responsibilities a server has. This is the most important thing to keep in mind before we build anything with webhooks.

![](https://substackcdn.com/image/fetch/$s_!Teeo!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6a95ba6f-0ae2-49bd-a0ef-77a2b00c135e_1734x1410.png)

Why does this design exist when SSE and WebSockets can also push?

This is because webhooks fit server-to-server integration, where the receiver isn’t a browser, isn’t necessarily online when the event happens, and doesn’t want to maintain a persistent connection to the provider.

The cost of webhooks is paid almost entirely on the receiver’s side. For example:

- A production webhook handler needs signature verification so the endpoint can confirm events came from the provider rather than an attacker.
- It needs retry handling with exponential backoff because the receiver will go down at some point.
- It needs idempotency because the same event will be delivered more than once under normal operation.
- Production designs typically also include a replay archive that lets consumers recover events they missed during outages.

## WebSockets

WebSockets answer a different question from the notification patterns.

While the notification patterns deal with the server informing the client of events, WebSockets handle both sides, keeping a conversation going at high frequency, in both directions, for as long as the application needs.

A WebSocket connection starts as an ordinary HTTP request with an upgrade header. If the server agrees to the upgrade, the connection switches protocols, and from that point on, both client and server can send messages to each other at any time without the request-response framing. The connection stays open until one side closes it or the network gives up.

![](https://substackcdn.com/image/fetch/$s_!J8mu!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F008d397e-335c-4a87-8779-1995440529c6_1788x1410.png)

WebSockets clearly win for a small set of applications:

- Collaborative document editors are a classic example, where every keystroke from one user becomes an event for every other user.
- Multiplayer games depend on the same kind of interaction, with every action by one player needing to reach every other player within milliseconds.
- Live trading interfaces work similarly, with user actions and market updates flowing constantly in both directions.

The common thread across all of these is that both sides genuinely produce events at high frequency throughout the session.

The trap with WebSockets is using them when only one side actually has anything to say. Many features built on WebSockets would be served better by SSE, since the client never actually pushes anything during normal operation. The bidirectional capability is worth its operational cost only when both directions are actually carrying meaningful traffic.

## Async APIs

Async APIs cover a different situation from any pattern we’ve covered so far. The work itself takes longer than a single request can wait for, so the question shifts from when the server learned about an event to when the slow computation will be ready.

The pattern is consistent across the systems that use it. The client submits a request, often as a POST. The server immediately returns a 202 Accepted response with a job identifier and a URL where the result will eventually live. The actual computation happens separately on the infrastructure that the server uses for slow work. The client either polls the status URL until the result is ready or registers a webhook that the server calls back when finished.

![](https://substackcdn.com/image/fetch/$s_!kMYe!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fef52affc-ee4d-4555-b2b8-cbb407874df1_1734x1496.png)

This pattern fits any compute operation that takes longer than the seconds-scale window of normal HTTP. Common examples include video transcoding, machine learning inference jobs that involve large models, bulk data exports, and report generation.

The cost is the state machine the client now has to track across multiple distinct interactions, plus the additional surface area on the server side for status endpoints, job storage, and result delivery.

## Message Queues

Message queues address the situation where the producer of an event and its consumer aren’t operating on the same clock. The producer has news to share, but the consumer might be offline, slow, or one of many. There might not be any moment when both are reachable together. None of the patterns we’ve covered so far handles this well, because all of them assume both parties are active at the same time.

![](https://substackcdn.com/image/fetch/$s_!ADYN!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fccd3bb43-ea58-467d-b869-9b4e81ef0483_2256x1276.png)

The answer is to put a third party in the middle. The producer writes its event to a broker, which is a piece of infrastructure designed to hold messages durably. Consumers read from the broker on their own schedules, in whatever order works for them. The producer and the consumer never connect to each other directly.

Several benefits follow from the broker’s role:

- Messages remain available even when consumers are temporarily down.
- The same message can be read by many consumers, which makes fan-out trivial.
- A consumer that crashes mid-processing can pick up where it left off because the broker still has the message.
- Consumers process at their own pace, which gives the system natural backpressure when demand spikes.

Queues blur the line between API pattern and infrastructure. Many systems that look webhook-driven from the outside use queues internally for delivery.

The cost is greater, since we’re now operating a stateful piece of infrastructure, and ordering and delivery guarantees vary enough between brokers.

## GraphQL Subscriptions

GraphQL subscriptions are different in kind from the patterns we’ve covered so far. Every other pattern in this article is a transport for moving messages between two parties. Subscriptions sit on top of a transport and shape what flows over it.

A subscription is a GraphQL query that requests an ongoing stream of updates whenever specific data changes. Unlike a normal query that returns once and ends, a subscription keeps producing results over time. The client expresses what it wants to be notified about using the same syntax it uses for normal queries. The server pushes events that match the subscription whenever they occur. Underneath, the actual transport is almost always WebSockets, sometimes SSE.

![](https://substackcdn.com/image/fetch/$s_!Eute!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fffdfc5f8-1fad-4672-ab57-022d27661962_1924x1410.png)

This is why putting subscriptions in a feature comparison alongside WebSockets creates confusion. Subscriptions and WebSockets operate at different levels in the stack. A subscription is a contract layer that uses WebSockets the same way a normal GraphQL query uses HTTP.

Subscriptions are a good choice when we’re already invested in GraphQL and want consistent client-side semantics across queries, mutations, and live data. Adopting GraphQL solely for subscriptions is rarely worthwhile.

## Summary

In this article, we have looked at async patterns for APIs in detail. Here are the key learning points in brief:

- Async API patterns exist because plain request-response can’t handle four situations: slow work, server-driven events the client can’t predict, continuous bidirectional conversation, and time-shifted communication between parties that aren’t online together.
- Every pattern is a configuration of four underlying levers: who initiates the connection, how long it stays open, whether a broker sits in the middle, and what delivery guarantees apply.
- Short polling is a respectable default for low-frequency events. Modern HTTP makes the per-request cost lower than older guidance assumes, and the simplicity often wins.
- Long polling holds the request open until something happens, trading server-side connection management for better latency than fixed-interval polling.
- Server-Sent Events handle one-way server-to-client streams over a single persistent HTTP connection. They are the dominant transport for LLM token streaming, and many features built with WebSockets would be served better by SSE.
- Webhooks reverse the usual client-server roles. The receiver acts as a server, and production-grade implementations need signature verification, retries with backoff, idempotency, and ideally a replay archive.
- WebSockets fit when both sides genuinely need to keep talking. The bidirectional capability is only worth the cost when both directions actually carry meaningful traffic.
- Async APIs separate the request from the result for long-running work. The client receives a job ID immediately and either polls a status endpoint or waits for a callback.
- Message queues introduce a broker that decouples producer and consumer in time and identity, enabling durability and fan-out at the cost of operating stateful infrastructure.
- GraphQL subscriptions are a contract layer over a transport like WebSockets or SSE, not a transport in their own right. They are useful inside GraphQL ecosystems, but rarely a reason to adopt GraphQL on their own.