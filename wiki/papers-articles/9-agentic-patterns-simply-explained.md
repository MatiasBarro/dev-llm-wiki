---
type: paper-article
tags: [ai, architecture, pattern, autonomous-agents, system-design]
sources:
  - "https://newsletter.systemdesign.one/p/agentic-design-patterns"
date-updated: 2026-05-15
---

# 9 Agentic Patterns, Simply Explained

**Author**: Neo Kim (System Design Newsletter)
**Published**: 2025-11-18

A practical guide to the nine design patterns for LLM-based systems, organized around a single axis: *who controls the flow — your code or the model?*

## Key Takeaways

**The escalation ladder is the starting point.** Before choosing a pattern, ask whether you need one at all. Default to a direct API call; escalate to workflow patterns when a task has many steps and focused attention at each step improves output; escalate to agent patterns only when the number and type of steps are unknown until runtime. The signal to escalate: you're writing more exception-handling code than actual work.

**Workflow vs. agent is a clean divide.** In workflow patterns, your code defines every step and the LLM handles the details. In agent patterns, you define constraints (tools available, cost budget, stop conditions) and the LLM decides the steps. See [[concepts/agentic-patterns]] for the full taxonomy.

**Orchestrator-workers is not a multi-agent architecture.** A single central LLM remains in control throughout. In multi-agent systems, agents can call each other directly without a central coordinator, and control can transfer between them — that's the distinction.

**The evaluator-optimizer is an add-on, not a standalone pattern.** It attaches to any other pattern as an external quality layer. The hard design problem is the stopping criteria: a quality threshold alone loops forever on hard inputs; a fixed iteration count wastes budget on easy ones. Production systems need all three — quality threshold + max iterations + cost budget ceiling.

**Three systemic concerns dominate at the agent level**: agentic amnesia (context window fills up, fix with checkpointing), security "lethal trifecta" (private data + untrusted content + exfiltration channel; any two are manageable, all three require sandboxing), and observability (non-deterministic agents require structured reasoning traces — you can't reproduce failures by replaying inputs). See [[concepts/harness-engineering]] for the broader framework.

**Architecture matters as much as the model.** OpenAI's SWE-bench research showed changing the architecture around GPT-4o moved its score from 23% to 33.2% — same model, different patterns. Claude 3.7 Sonnet with a structured reflection setup scores 70.3% vs. 62.3% with a single prompt.

## Pattern Summary

| # | Pattern | Type | Cost | Who decides next step |
|---|---|---|---|---|
| 1 | Prompt Chaining | Workflow | Low | Code (sequential) |
| 2 | Routing | Workflow | Low | Code (classifier) |
| 3 | Parallelization | Workflow | Medium | Code (parallel) |
| 4 | Orchestrator-Workers | Workflow | High | Central LLM + code |
| 5 | Reflection | Agent | Medium | LLM (self-critique) |
| 6 | Tool Use | Agent | Medium | LLM (tool selection) |
| 7 | ReAct | Agent | High | LLM (reason-act-observe loop) |
| 8 | Planning | Agent | Very High | LLM (goal decomposition) |
| +  | Evaluator-Optimizer | Add-on | Varies | External quality layer |

## Case Study: AI Code Review Pipeline

The article closes with a concrete composition: **Routing → Parallelization (sectioning) → ReAct → Evaluator-Optimizer**.

- Router classifies PRs by type (docs, config, logic) and routes to appropriate handlers
- Parallel reviewers (security, style, logic) run simultaneously via sectioning
- Ambiguous findings escalate to a ReAct agent that reads the call graph and commit history
- Evaluator-optimizer tightens vague comments into actionable, line-specific feedback

Orchestrator-workers and planning were explicitly rejected: subtasks are predictable in advance, so a central LLM coordinator adds complexity with no benefit.

## Related

- [[concepts/agentic-patterns]] — full taxonomy of all 9 patterns with escalation ladder
- [[concepts/agentic-llm-primitives]] — how the workflow/agent divide maps to constrained call vs. autonomous harness
- [[concepts/multi-agent-orchestration]] — how agents compose into multi-agent systems
- [[concepts/harness-engineering]] — outer controls (guides, sensors, observability) around agent systems
