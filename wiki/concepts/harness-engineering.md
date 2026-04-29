---
type: concept
tags: [ai, autonomous-agents, architecture, pattern, software-engineering, quality]
sources:
  - "https://martinfowler.com/articles/harness-engineering.html"
date-updated: 2026-04-29
---

# Harness Engineering

The practice of designing and iterating on the outer controls that surround a coding agent — everything the *user* builds around the agent to increase the probability of correct output and reduce review toil. Distinct from the agent's built-in harness (system prompt, retrieval, orchestration): harness engineering refers specifically to the user-controllable layer.

## Three-Layer Model

```
[ Model ]
  └─ [ Coding agent's built-in harness ]   ← vendor/tool responsibility
       └─ [ User's outer harness ]          ← harness engineering lives here
```

The user's outer harness wraps around what the coding agent already provides. Teams control this layer via AGENTS.md, skills, MCP servers, hooks, linters, structural tests, and review agents.

## Guides and Sensors

The outer harness has two directions:

**Guides (feedforward)** — shape the agent's context before it acts. Examples: AGENTS.md, how-to skills, reference docs, LSP integration, codemods, bootstrap scripts.

**Sensors (feedback)** — monitor output and trigger self-correction. Examples: linters, type checkers, test suites, structural analysis, code review agents.

### Computational vs. Inferential

Both guides and sensors split into two execution types:

| | Computational | Inferential |
|---|---|---|
| Execution | CPU, deterministic | GPU/NPU, probabilistic |
| Speed | Milliseconds–seconds | Seconds–minutes |
| Cost | Cheap | Expensive |
| Reliability | Consistent | Non-deterministic |
| Examples (guides) | Codemods, bootstrap scripts | AGENTS.md, skills, how-tos |
| Examples (sensors) | Linters, tests, ArchUnit | Review agents, LLM judges |

Computational sensors are cheap enough to run on every commit. Inferential sensors are reserved for post-integration or async drift detection.

## Regulation Dimensions

The harness regulates the codebase across three orthogonal dimensions:

### Maintainability Harness

Internal code quality: duplication, complexity, coverage, style, architectural drift. The most tractable dimension — pre-existing computational tooling covers the structural problems reliably. Inferential sensors partially address semantic problems (redundant tests, over-engineered solutions) but expensively and probabilistically.

What neither catches reliably: misdiagnosis, unnecessary features, misunderstood instructions. Correctness is outside any sensor's remit if the human didn't clearly specify the goal.

### Architecture Fitness Harness

Guides and sensors for architecture characteristics — essentially [[concepts/architectural-fitness-functions]] in agent-aware form. Examples: skills that feed forward performance requirements + performance tests that feed back; logging convention skills + debugging reflection prompts.

### Behaviour Harness (Unsolved)

The open problem. Current practice: functional spec as feedforward + AI-generated test suite as feedback sensor + manual testing. This puts too much faith in AI-generated tests. The [approved fixtures](https://lexler.github.io/augmented-coding-patterns/patterns/approved-fixtures/) pattern shows promise in bounded areas but is not a wholesale answer. Reducing supervision and manual testing for functional behaviour remains unsolved.

## "Keep Quality Left"

Distribute harness controls across the change lifecycle based on cost and speed:

1. **Pre-commit / pre-integration**: fast computational sensors (linters, type checks, fast test suites, basic review agent)
2. **Post-integration pipeline**: repeat fast controls + expensive inferential sensors (architecture review, detailed review, mutation testing)
3. **Continuous / outside change lifecycle**: drift sensors (dead code, dependency scanners, test coverage quality, runtime SLO monitoring)

The earlier an issue is caught, the cheaper it is to fix.

## The Steering Loop

The human's role shifts from reviewing every output to iterating on the harness itself. When the same issue recurs, improve the relevant guides or sensors so it either doesn't happen or gets caught automatically. Agents can help: they can write structural tests, draft rules from observed patterns, scaffold custom linters, or generate how-to guides from codebase archaeology.

## Harnessability

Not all codebases are equally governable. Factors that increase harnessability:
- Strongly typed languages (type checker as free sensor)
- Clearly definable module boundaries (afford constraint rules)
- Frameworks that abstract away details (reduce agent surface area)

Legacy codebases with accumulated technical debt face the hardest problem: the harness is most needed where it is hardest to build. Greenfield teams can bake harnessability in from day one via technology and architecture choices.

## Harness Templates

Emerging pattern: bundle guides and sensors for a common service topology (e.g. CRUD business service, event processor, data dashboard) into a reusable template. Teams pick tech stacks partly based on available harness templates. Faces the same versioning/contribution problems as service templates, amplified by non-deterministic components that are harder to test.

## Related

- [[concepts/agentic-llm-primitives]] — the "autonomous harness" there is the inner ring; harness engineering is the outer ring users build around it
- [[concepts/multi-agent-orchestration]] — the three nested failure loops are an instance of computational feedback sensors in an orchestration-layer harness
- [[papers-articles/harness-engineering-for-coding-agent-users]] — source article
