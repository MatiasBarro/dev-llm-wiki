---
type: concept
tags: [ai, architecture, pattern, autonomous-agents, system-design]
sources:
  - "https://newsletter.systemdesign.one/p/agentic-design-patterns"
date-updated: 2026-05-15
---

# Agentic Patterns

A taxonomy of nine design patterns for LLM-based systems, organized around a single axis: *who controls what happens next — your code or the model?*

## The Escalation Ladder

Before choosing a pattern, establish whether you need one at all.

```
Direct API call
  → Workflow patterns   (steps known in advance; code controls flow)
    → Agent patterns    (steps unknown until runtime; LLM controls flow)
      → Multi-agent     (agents cooperate, control can transfer between them)
```

**Default to the simplest level that handles the failure mode you're solving.** The signal to escalate: you're writing more exception-handling code than actual work, or you keep adding special cases for situations the LLM encounters that you didn't plan for.

A direct API call handles more than most people assume — summarization, classification, extraction, rewriting, translation, code generation with clear specs. Most people skip this level too quickly.

**Don't conflate 70% prototype with architectural failure.** When a system stalls at 70–80%, the real issue is usually prompt quality or missing validation gates, not the pattern.

## The Core Divide: Workflow vs. Agent

| | Workflow patterns | Agent patterns |
|---|---|---|
| Who defines the steps | Your code | The LLM |
| Steps known in advance | Yes | No |
| You define | The execution path | Constraints (tools, budget, stop conditions) |
| Failure modes | Predictable | Emerge at runtime |
| Cost/latency | Predictable | Unpredictable |

## Workflow Patterns

### 1. Prompt Chaining

Break a task into sequential LLM calls. Each call takes the output from the prior step; **validation gates** check the result before passing it forward.

- **Analogy**: CI/CD pipeline — each stage must pass before the next runs
- **Example**: Extract clauses → classify by risk → summarize high-risk items (legal contract review)
- **Tradeoff**: Latency grows linearly with steps; errors carry forward if gates don't catch them

### 2. Routing

A classifier examines input and sends it to the right handler — a different prompt, model, or sub-workflow. Each handler is built for one type of input.

- **Analogy**: Hospital front desk — triage, don't treat
- **Example**: Sierra AI routes across 15+ models; simple queries hit cheap models, complex ones hit expensive ones
- **Tradeoff**: The router is a single point of failure; its accuracy sets the upper bound for the whole system

### 3. Parallelization

Two variants that solve different problems:

- **Sectioning**: split a task into independent parts, run in parallel, merge results. Each worker has a different job.
- **Voting**: run the same task multiple times with different prompts, aggregate for consensus. Every worker has the same job.

**Example**: Sectioning — GitHub Advanced Security runs CodeQL and Snyk on the same PR in parallel. Voting — three prompts analyze the same code for vulnerabilities; flag only if two or more agree.

**Tradeoff**: Cost multiplies with every branch. Partial failure handling (retry one? fail all? proceed without?) must be decided upfront.

### 4. Orchestrator-Workers

A central LLM breaks a task into subtasks, assigns each to worker LLMs, and combines results. The difference from the prior patterns: the subtasks aren't known in advance — the orchestrator figures them out dynamically.

- **Example**: Cursor agent mode — starts workers for tests, docs, and refactoring, adjusting as it finds new problems
- **Not multi-agent**: one central LLM remains in control throughout. In true multi-agent systems, agents can call each other directly without a central coordinator
- **Tradeoff**: Orchestrator can lose track of the original goal, over-decompose simple tasks, or become the bottleneck when workers finish faster than it can combine results

## Agent Patterns

With agent patterns, you stop defining what happens and in what order. Instead of steps, you define constraints: what tools are available, how much the model can spend, when to stop. The model observes, reasons, and chooses what happens next.

### 5. Reflection

The LLM generates output, reviews its own work, and revises. Loops until output is good enough. A two-agent variant uses the same model with two different prompts — one generates, one critiques.

- **Example**: Claude 3.7 Sonnet with structured reflection scores 70.3% on SWE-bench vs. 62.3% single-prompt — same model, eight-point jump from architecture alone
- **Tradeoff**: Quality improvements diminish with each iteration while cost stays constant. The model can only catch errors it already knows how to recognize — works for tasks with measurable criteria (code, math, structured output), stalls when information needed for critique isn't in context

### 6. Tool Use

The LLM selects from a **registry** of available tools (APIs, databases, code execution, search) and calls them as needed. One tool's output often becomes the next tool's input; the model decides the sequence at runtime.

The most widely adopted agent pattern — nearly every production agent calls external tools.

**Permission tiers matter**:
- Read-only tools (queries, file reads, search) — run freely
- Write tools (file edits, API updates) — require confirmation or sandboxing
- High-stakes tools (payments, deletions, external communications) — explicit human approval

**Performance tiers**:
- Direct function calls: ~30ms overhead
- Protocol-based (MCP): 300–500ms per call before the function runs
- LLM-mediated: slowest

Use direct calls for deterministic operations. Use tool calling through the model only when the model genuinely needs to decide which tool to use next.

**Tradeoff**: Wrong tool wastes tokens; tools with side effects create failure chains when called in the wrong order.

### 7. ReAct (Reason + Act)

**Reason, act, observe, repeat.** The model reasons about the current state, takes an action (usually a tool call), observes the result, and reasons again. This is a `while` loop: as long as the model returns a tool call, execute it, pass results back, let the model decide what's next.

- **Example**: Claude Code — reads codebase, reasons about what to change, edits a file, runs tests, adjusts if something fails
- **Foundation**: the architecture behind most production agents
- **Reasoning trace**: because the model reasons before acting, you get a step-by-step record of why it did what it did — essential for debugging

**Tradeoff**: Reasoning degrades over long sessions; action loops occur (model calls same tool repeatedly); context window fills as observations accumulate. In one reported case, Claude Code on Opus 4.6 showed circular reasoning at 20% context usage and recommended a fresh session at 48%.

### 8. Planning

Decompose a goal into subtasks with dependencies, execute in order. When something fails, find the wrong assumption and replan.

- **Example**: Devin creates a step-by-step plan before editing any files, waits for approval, replans on test failure
- **Pairs with ReAct**: planning layer breaks the problem into steps; each step runs through a ReAct loop for unpredictable execution details
- **When it helps**: clear decomposition points, semi-independent subtasks, costly mistakes worth preventing with upfront analysis (code migration across 50 files, yes; single function rewrite, no)

**Tradeoff**: Bad decomposition is worse than no decomposition — the agent follows a flawed structure and burns tokens executing the wrong plan. Re-planning is expensive. Planning agents are "kind of finicky" (Andrew Ng) — when they work they're impressive; when they don't, they spend your whole budget planning how to plan.

## Evaluator-Optimizer (Add-On Layer)

The evaluator-optimizer is **not a standalone pattern** — it's a quality layer you attach to any other pattern. A generator produces output; a separate evaluator checks it; failures feed back as input for the next pass.

The evaluator can be: an LLM judge, a test suite, a set of rules, or a human.

**The hard design problem is stopping criteria.** Three stopping conditions, and you need all three working together:
1. Quality threshold — output is good enough
2. Max iterations — prevents infinite loops on hard inputs
3. Cost budget ceiling — hard cap regardless of quality

A quality threshold alone loops forever on inputs that never improve. A fixed iteration count wastes budget on easy problems. A weak evaluator is worse than no evaluator: it creates false confidence.

**Attaches to anything**: in a chaining pipeline, add it after the final step; in a ReAct agent, add it outside the loop to judge the final answer rather than individual tool calls.

## Systemic Concerns at the Agent Level

Workflow patterns fail in predictable places (gate rejects bad output, router misclassifies, branch times out). Agent patterns fail in ways you won't expect until production.

**Agentic amnesia**: an agent's context window is finite. Tool results and reasoning traces accumulate faster than the window can hold; oldest content gets dropped. Fix: checkpoint state to an external store so the agent can resume or summarize history.

**Security — the lethal trifecta**: three things must coexist for a serious breach — private data, untrusted content (user input, web pages, third-party API responses), and an exfiltration channel (email, API calls, code execution). Any two are manageable; remove one leg and the attack fails. All three together require: per-tool permission scoping, sandboxed execution environments, and treating every external input as potentially adversarial.

**Observability**: agents are non-deterministic — the same input can produce different tool call sequences, different reasoning paths, and different outputs. You can't reproduce a failure by replaying inputs. You need structured logging of every tool call, every observation fed back into context, and every decision point. At minimum: reasoning traces from specific runs.

**Treat agents like code, not chat interfaces**:
- Type your inputs (function parameter annotations)
- Schema your outputs (validate against JSON schema)
- Version your prompts (track in version control)
- Test your tool integrations (write integration tests per tool)

## Pattern Composition

Patterns combine. The most common two-pattern composition in production: **routing + parallelization** — route by type, then process each type's subtasks in parallel.

Tool use and ReAct are often confused: if your tool calls follow a predictable sequence, tool use alone is enough. The reason-act-observe loop adds overhead only worth paying when the sequence isn't knowable in advance.

The architecture around the model matters as much as the model itself. OpenAI's SWE-bench research found that changing the architecture built around GPT-4o moved its score from 23% to 33.2% — same model, different patterns.

## Related

- [[concepts/agentic-llm-primitives]] — the workflow/agent divide maps directly onto constrained call vs. autonomous harness
- [[concepts/multi-agent-orchestration]] — how agents compose into systems where control can transfer between agents
- [[concepts/harness-engineering]] — outer controls (guides, sensors, observability dimensions) that wrap agent systems
- [[papers-articles/9-agentic-patterns-simply-explained]] — source article with case study and full comparison table
