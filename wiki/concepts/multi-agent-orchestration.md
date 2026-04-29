---
type: concept
tags: [ai, multi-agent, architecture, distributed-systems, autonomous-agents, pattern]
sources:
  - "https://agentfield.ai/blog/beyond-vibe-coding"
date-updated: 2026-04-29
---

# Multi-Agent Orchestration

Patterns for coordinating multiple autonomous LLM agents working in parallel on a shared artifact (e.g. a codebase). The core challenge is the **convergence problem**: getting N autonomous processes to produce one coherent result requires primitives for isolation, failure recovery, and state reconciliation.

## Issue Dependency Graph

Work is decomposed into issues organized in dependency levels. Issues within the same level are independent and execute in parallel; issues in the next level depend on the outputs of the current level.

Autonomous planners tend to decompose work into finer-grained, more parallelizable graphs than humans would — often 50–100 issues with dependency structures a human planner wouldn't attempt because the coordination cost would be prohibitive. That coordination cost is what the orchestration system absorbs.

## Agent Isolation: Git Worktrees

Each issue runs in its own git worktree on a dedicated branch (`issue/01-project-scaffold`, `issue/02-types-module`, etc.). This provides:
- No lock contention between agents working in the same dependency level
- Clean rollback if an issue fails unrecoverably
- A clear integration boundary: branches merge into an integration branch between levels

A **merger agent** (not a mechanical `git merge`) performs intent-aware conflict resolution using the architecture spec and file conflict annotations from the planning phase. When two issues modify the same file, the merger understands what each change was trying to accomplish.

## Three Nested Failure Loops

In a large multi-agent build, failures are the normal path. A single retry loop is insufficient; the system needs three nested control loops.

### Inner Loop (per-issue retry)

- Runs up to 5 iterations per issue
- The agent retries itself with feedback from QA and code review agents
- Handles failures that the same agent can fix given better information (e.g. a missing module export caught by the reviewer)
- Does **not** handle failures the agent structurally cannot fix (e.g. a binary with an infinite loop that times out on every invocation)

### Middle Loop (issue advisor)

Activates when the inner loop is exhausted. Has five typed recovery actions:

| Action | Effect |
|---|---|
| `RETRY_MODIFIED` | Relax acceptance criteria; record the gap as typed debt |
| `RETRY_APPROACH` | Same criteria, different strategy |
| `SPLIT` | Break into smaller sub-issues; inject into remaining levels |
| `ACCEPT_WITH_DEBT` | Close enough; record each gap as typed, severity-rated debt |
| `ESCALATE_TO_REPLAN` | Cannot be fixed locally; restructure remaining work |

On its final invocation, the advisor's prompt is explicitly biased toward `ACCEPT_WITH_DEBT` or `ESCALATE_TO_REPLAN` to avoid burning budget on futile retries.

### Outer Loop (replanner)

Fires when an entire dependency level has unrecoverable failures. Sees the full execution state and can:
- Skip downstream issues that depended on the failed work
- Restructure the remaining issue graph
- Reduce scope
- Abort

Previous replan decisions are fed back on subsequent invocations to prevent repeating failed strategies. If the replanner itself crashes (LLM timeout, malformed output), the system defaults to **continue** rather than abort — for expensive workflows, graceful degradation beats fail-fast.

## Typed Debt

When an issue is blocked or accepted with gaps, the failure is recorded as a structured debt item — not a log message:

```json
{
  "type": "dropped_acceptance_criterion",
  "criterion": "integration tests pass end-to-end",
  "issue_name": "integration-tests",
  "severity": "high",
  "justification": "CLI binary deadlocks on invocation; test suite correct but binary needs runtime debugging beyond automated repair"
}
```

Downstream issues receive `debt_notes` so they can work around known gaps rather than building on assumptions that no longer hold. The final PR includes a full debt section visible to the human reviewer.

## Level Gate Sequence

Between every dependency level, six sequential gates enforce a clean handoff:

1. **Merge gate**: integrate completed issue branches into the integration branch
2. **Integration test gate**: validate merged code still works together
3. **Debt gate**: process completed-with-debt results; propagate `debt_notes` downstream
4. **Split gate**: inject any sub-issues from SPLIT recoveries into remaining levels
5. **Replan gate**: invoke the replanner if failures escalated
6. **Checkpoint**: serialize full execution state to disk

Any gate can pause for human approval in regulated environments; the gate sequence itself is constant.

## Checkpointing

At significant per-build cost (e.g. $116 for 200+ invocations), restarting from scratch after a crash is not viable. The checkpoint captures:
- Completed dependency levels
- Full issue list and status
- Replan count and history
- Accumulated debt items
- Git state: integration branch, original branch, initial commit SHA, worktree directory mapping

`resume_build()` loads the checkpoint and continues from the exact failure point. Longer workflows accumulate more recovery points, not more fragility — this inverts the usual relationship between workflow length and fragility.

## Architecture vs. Model Selection

The same orchestration architecture scored 95/100 on a benchmark with the cheapest available model. Single-agent approaches using more capable models scored 59–73 on the same task. The verification loops and escalation hierarchy compensate for model capability through iteration. The model is a runtime config parameter:

```json
{
  "models": {
    "default": "sonnet",
    "coder": "haiku",
    "qa": "haiku",
    "architect": "sonnet"
  }
}
```

Each role is assigned independently. This turns model selection into an empirical cost/quality tradeoff question rather than a bet on a single model being smart enough.

## Cross-Agent Memory (Unsolved)

A shared key-value store propagates conventions and failure patterns across issues within a build: naming patterns, project structure idioms, testing conventions discovered by early agents, failure traps to avoid, concrete import paths from completed issues.

The unsolved tradeoff: too little context and later agents repeat earlier mistakes; too much context and the prompt is so long the agent ignores the actual task. Keeping the store simple (structured schemas, synchronous updates at lifecycle points) makes it reliable but requires manually defining what gets stored and when.

## Related

- [[concepts/agentic-llm-primitives]] — the two fundamental LLM integration modes that compose into this system
- [[papers-articles/beyond-vibe-coding-200-autonomous-agents]] — source article with real build data and failure examples
