---
type: concept
tags: [reliability, system-design, distributed-systems, operations]
sources:
  - "https://blog.bytebytego.com/p/ep212-data-warehouse-vs-data-lake?utm_source=post-email-title&publication_id=817132&post_id=195380781&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
date-updated: 2026-06-10
---

# SLA, SLO, SLI

Three related terms for defining, measuring, and promising service reliability. They nest inside each other: SLI feeds SLO, SLO feeds SLA.

## The three terms

**SLI (Service Level Indicator)**
The raw metric being measured. It should be expressed as a ratio or rate — not an absolute number.

> Example: ratio of successful login requests to total valid login requests over the last 5 minutes.

An SLI tells you where the service stands *right now*.

**SLO (Service Level Objective)**
An internal target defined on top of an SLI, usually over a rolling time window.

> Example: login availability SLI should stay above 99.9% over a rolling 28-day window.

An SLO tells you where the service *should* be. Breaching an SLO is a signal to investigate before customers notice — it's an internal alarm, not a contractual obligation.

**SLA (Service Level Agreement)**
The external contract with customers. Set *lower* than the SLO to provide a buffer.

> Example: contractual availability promise to customers is 99.5% per calendar month. If breached, service credits are owed.

An SLA tells customers *what they can expect* and what compensation they receive if that expectation is violated.

## Why SLO > SLA matters

If SLO and SLA are set to the same value (e.g., both 99.9%), then any availability drop immediately breaches the customer contract. The SLO buffer exists so engineering can react to degradation before it becomes a legal/commercial issue.

```
SLI: actual measurement (e.g., 99.94% availability today)
SLO: internal target (e.g., 99.9%) — breach triggers internal alert
SLA: customer promise (e.g., 99.5%) — breach triggers credits/penalties
```

## Practical notes

- SLIs should measure what customers actually experience, not internal proxies. "Requests served in <200ms" is better than "CPU < 80%."
- SLOs require a time window. Availability over 1 hour and over 28 days are very different numbers.
- Error budgets (the complement of SLO) are a useful concept: if your SLO is 99.9%, your error budget is 0.1% — how much unreliability you can "spend" before triggering remediation.
- Setting the right SLO for a new service is hard. Starting with SLOs informed by past incident data and customer feedback is better than guessing.
