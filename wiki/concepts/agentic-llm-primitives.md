---
type: concept
tags: [ai, multi-agent, architecture, pattern, autonomous-agents]
sources:
  - "https://agentfield.ai/blog/beyond-vibe-coding"
  - "https://newsletter.systemdesign.one/p/agentic-design-patterns"
date-updated: 2026-05-15
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

## Mapping to the 9-Pattern Taxonomy

The two primitives map directly onto the workflow/agent divide described in [[concepts/agentic-patterns]]:

| Primitive | Corresponds to | Who controls flow |
|---|---|---|
| Constrained Call | Direct API call or workflow patterns (chaining, routing, parallelization) | Your code |
| Autonomous Harness | Agent patterns (reflection, tool use, ReAct, planning) | The LLM |

The orchestrator-workers pattern sits at the boundary — a central LLM makes routing decisions dynamically, but your code still invokes it and checks results. It behaves like an autonomous harness at the orchestration level while each worker call can be either primitive.

The practical implication: the choice of primitive should precede the choice of pattern. If a task requires schema-validated output with predictable cost (routing decision, risk scoring, guidance generation), reach for the constrained call regardless of how many downstream agents will act on it. Only escalate to the autonomous harness when the task requires reading and modifying state in a loop with an open-ended goal.

## Related

- [[concepts/agentic-patterns]] — the full 9-pattern taxonomy and escalation ladder that maps onto these two primitives
- [[concepts/multi-agent-orchestration]] — how these primitives compose into a full orchestration system with failure loops and checkpointing
