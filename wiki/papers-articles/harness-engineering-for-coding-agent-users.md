---
type: paper-article
tags: [ai, autonomous-agents, software-engineering, quality, architecture, pattern]
sources:
  - "https://martinfowler.com/articles/harness-engineering.html"
date-updated: 2026-04-29
---

# Harness Engineering for Coding Agent Users

**Author**: Birgitta Böckeler | **Published**: April 2026 | **Via**: martinfowler.com

A framework for the outer controls that users build around coding agents — narrowing the broad "Agent = Model + Harness" definition to the specific layer users control.

## Core Argument

A well-built outer harness does two things: increases the probability the agent gets it right the first time, and provides a self-correcting feedback loop before issues reach human review. The goal is lower review toil, higher quality, and fewer wasted tokens.

## Key Framework

**Guides (feedforward)** → agent → **Sensors (feedback)** → self-correction loop → human steering

Both guides and sensors split into:
- **Computational**: deterministic, fast, cheap (tests, linters, LSPs, codemods)
- **Inferential**: probabilistic, slow, expensive (review agents, LLM judges, semantic analysis)

Full treatment: [[concepts/harness-engineering]]

## Three Regulation Dimensions

| Dimension | State | Hardest problem |
|---|---|---|
| Maintainability | Well-tooled | Semantic issues (misdiagnosis, overengineering) — no reliable sensor |
| Architecture fitness | Tractable | Keeping guides and fitness functions in sync |
| Behaviour | Unsolved | AI-generated tests aren't trustworthy enough to reduce manual supervision |

## Industry Examples Cited

- **OpenAI**: layered architecture enforced by custom linters + structural tests + recurring "garbage collection" agent scans for drift. Conclusion: "Our most difficult challenges now center on designing environments, feedback loops, and control systems."
- **Stripe (Minions)**: pre-push hooks running relevant linters via heuristic; "blueprints" integrating sensors into agent workflows; emphasis on shifting feedback left.
- **Thoughtworks**: increasing API quality with agents + custom linters; "janitor army" for code quality; tackling architecture drift with mixed computational/inferential sensors.

## Open Questions

- How do you keep a harness coherent as it grows — guides and sensors not contradicting each other?
- How far can agents be trusted to make sensible trade-offs when instructions and feedback signals conflict?
- If sensors never fire, is that high quality or inadequate detection?
- We need harness coverage metrics analogous to code coverage and mutation testing.
- Tooling that treats feedforward and feedback controls as a system (not scattered across delivery steps) doesn't exist yet.

## Related

- [[concepts/harness-engineering]] — full concept breakdown with regulation dimensions, the steering loop, and harnessability
- [[concepts/agentic-llm-primitives]] — inner-harness primitives; this article is the outer layer
- [[concepts/multi-agent-orchestration]] — orchestration-layer harness as a concrete instance of the framework
