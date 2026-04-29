---
type: paper-article
tags: [ai, multi-agent, autonomous-agents, software-engineering, architecture]
sources:
  - "https://agentfield.ai/blog/beyond-vibe-coding"
date-updated: 2026-04-29
---

# Beyond Vibe Coding: How We Ship Production Code with 200 Autonomous Agents

**Source**: AgentField blog, March 2026
**Bias note**: Written by the team behind AgentField and their open-source SWE-AF framework. The article draws primarily on their own builds as evidence and concludes by linking to their products. Engineering patterns described are real and detailed, but claims should be read with the awareness that the authors are promoting their own tooling.

## Summary

AgentField built a system that orchestrates 200+ Claude Code instances in parallel on a shared codebase. The goal: shift the human role from line-by-line iteration to architectural sign-off on a finished, verified draft PR. The article covers three lessons learned and what they'd do differently.

Real builds cited:
- **Diagrams-as-code CLI in Rust**: 15 issues, 200+ agent invocations, $116
- **Go SDK feature**: 10 issues, 80+ invocations, $19
- **Node.js benchmark**: same architecture scored 95/100 with both cheapest and mid-tier models

## Key Takeaways

### 1. Two LLM primitives, not one

See [[concepts/agentic-llm-primitives]].

Every agent role should map to one of two modes:
- **Constrained call** (`.ai()`): single-shot, structured I/O, no tools, milliseconds, fractions of a cent. Used for routing, classification, guidance generation.
- **Autonomous harness** (`.harness()`): multi-turn, tool-using, goal-driven. Iterates until it delivers a verifiable outcome. Up to 150 tool-use turns; up to $4+ per invocation on complex issues.

A single boolean from a cheap classification call (`needs_deeper_qa`) determines whether an issue runs a lean 2-call path or a thorough 4-call path. The routing is cheap; the work is expensive.

### 2. Three nested failure loops

See [[concepts/multi-agent-orchestration]].

In a 200+ invocation build, failures are the normal path. The system needs three nested control loops:

- **Inner loop**: per-issue retry up to 5×, agent receives QA and code review feedback on each attempt.
- **Middle loop** (issue advisor): activates when the inner loop is exhausted. Five typed recovery actions: `RETRY_MODIFIED`, `RETRY_APPROACH`, `SPLIT`, `ACCEPT_WITH_DEBT`, `ESCALATE_TO_REPLAN`. On its final invocation, the prompt is explicitly biased toward acceptance or escalation to avoid futile retries.
- **Outer loop** (replanner): fires when an entire dependency level has unrecoverable failures. Can restructure the remaining issue graph, skip dependents, reduce scope, or abort.

Key insight: some failures cannot be fixed by retrying (a deadlocked binary, for instance). The right response is to **block**, record a typed debt item with severity, and let the rest of the build continue. The alternative — silently dropping, aborting, or retrying harder — are all worse.

### 3. Checkpoint-based execution

At $116/build and 200+ invocations, restarting from scratch after a crash is not viable. The team checkpoints at every dependency-level boundary, capturing: completed levels, full issue list, replanning count, accumulated debt, git state (branch names, worktree directory mapping, initial commit SHA).

`resume_build()` loads the checkpoint and continues from the exact failure point. Longer workflows accumulate more recovery points, not more fragility.

### 4. Git worktrees for agent isolation + intent-aware merging

Each issue runs in its own git worktree on a dedicated branch. Agents in the same dependency level execute in parallel with no lock contention.

Between levels, a merger agent (not a mechanical `git merge`) uses the architecture spec and conflict annotations from the planning phase to make intent-aware resolution decisions — it understands what each conflicting change was trying to accomplish.

A 6-gate sequence enforces a clean handoff between levels: merge → integration test → debt propagation → split injection → replan → checkpoint.

### 5. Architecture beats model selection

On a Node.js benchmark: same architecture scored 95/100 with Claude Haiku (~$20) and MiniMax M2.5 (~$6). Single-agent approaches with better models scored 59–73.

The verification loops and escalation hierarchy compensate for model capability through iteration: more inner loop cycles, cheaper per cycle, same outcome. Model is a runtime config parameter; each role (coder, reviewer, QA, planner, merger) is assigned independently and can be swapped without changing the architecture.

## What They'd Do Differently

**Cross-agent memory is unsolved.** A shared key-value store propagates conventions (naming patterns, testing idioms, successful import paths) and failure patterns across issues. But the tradeoff between memory breadth (too little → agent 15 repeats agent 1's mistakes) and prompt focus (too much → agent ignores the task) is still being tuned manually.

**Verification loops can mask bad planning.** A final verifier agent checks every acceptance criterion against the codebase and generates targeted fix issues for any gaps. This is how the Go SDK build hit 34/34 criteria — not by getting everything right the first time, but by verifying and closing gaps. The downside: if fix issues are generated on every build, the real problem may be upstream in the planner. They now track fix-issue frequency as a planning quality signal but haven't solved it.

**$116/build is too expensive for rapid iteration.** The architecture is sound at $6 (same score), but model routing is still too coarse. Their answer is risk-proportional allocation: route low-risk issues to cheaper models and cut unnecessary QA passes on straightforward work.
