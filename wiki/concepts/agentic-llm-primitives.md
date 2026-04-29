---
type: concept
tags: [ai, multi-agent, architecture, pattern, autonomous-agents]
sources:
  - "https://agentfield.ai/blog/beyond-vibe-coding"
date-updated: 2026-04-29
---

# Agentic LLM Primitives: Constrained Call vs. Autonomous Harness

Multi-agent systems need two fundamentally different modes of LLM integration. Giving every agent the same kind of access — unconstrained, tool-using, open-ended — makes the system operationally unmanageable: you cannot set SLAs, predict costs, or reason about retry semantics.

## The Two Primitives

### Constrained Call

- Single-shot: one request, one response, no follow-up turns
- Structured input and output (schema-validated)
- No tools, no iteration
- Predictable latency (milliseconds), predictable cost (fractions of a cent)
- Output is a value downstream code can switch on

**Used for**: routing decisions, classification, risk scoring, guidance generation, any place where you need a reliable signal with low variance.

Example output from a planning step:
```
IssueGuidance:
  needs_new_tests: true
  estimated_scope: "medium"
  touches_interfaces: true
  needs_deeper_qa: true
  agent_guidance: "Complex parsing logic with edge cases, run QA in parallel"
```

A single boolean (`needs_deeper_qa`) from a cheap constrained call can determine whether downstream work runs a lean 2-agent path or a thorough 4-agent path.

### Autonomous Harness

- Multi-turn: iterates until it delivers a verifiable outcome
- Full toolset: filesystem access, test execution, git operations
- Goal-driven: receives a goal, produces a result, internal steps are opaque
- Unpredictable duration and cost (up to 150 tool-use turns and $4+ on complex issues)
- You check what it delivered, not how it got there

**Used for**: coding, QA, code review, merging, verification — any task that requires reading and modifying state in a loop.

## Why the Distinction Matters

| Property | Constrained Call | Autonomous Harness |
|---|---|---|
| Latency | Milliseconds | Minutes to hours |
| Cost | Fractions of a cent | Dollars |
| Retry semantics | Trivial re-run | Must checkpoint and resume |
| Output shape | Schema-defined | Goal-verification |
| Parallelism | Trivially parallel | Requires isolation (e.g. git worktrees) |

When you conflate the two, retry logic becomes ambiguous (re-running a 45-minute loop vs. re-running a 200ms classification are completely different operations). Cost prediction becomes impossible. SLA reasoning collapses.

## Origin

This distinction emerged from the SWE-AF multi-agent system described in [[papers-articles/beyond-vibe-coding-200-autonomous-agents]]. The constrained call was recognized on day one; the autonomous harness abstraction took months of failed builds to identify — it wasn't designed in advance, it was extracted from recurring patterns.

## Related

- [[concepts/multi-agent-orchestration]] — how these primitives compose into a full orchestration system with failure loops and checkpointing
