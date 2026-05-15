# Wiki Index

## Concepts
| Page | Summary | Tags |
|---|---|---|
| [[concepts/agentic-llm-primitives]] | Two fundamental LLM integration modes: constrained single-shot call vs. autonomous multi-turn harness | ai, multi-agent, architecture, pattern |
| [[concepts/multi-agent-orchestration]] | Patterns for coordinating parallel autonomous agents: three nested failure loops, git worktree isolation, checkpointing, typed debt | ai, multi-agent, architecture, distributed-systems |
| [[concepts/harness-engineering]] | Framework for the outer controls users build around coding agents: guides (feedforward), sensors (feedback), three regulation dimensions (maintainability, architecture fitness, behaviour) | ai, autonomous-agents, architecture, pattern, quality |
| [[concepts/event-driven-architecture]] | Communication style where services publish facts rather than calling each other; four components, six patterns, when EDA is the wrong choice | distributed-systems, system-design, architecture, pattern, event-driven |
| [[concepts/event-sourcing]] | Store every change as an immutable event; current state derived by replay; built-in audit trail, hard schema evolution, oversold outside audit-heavy domains | distributed-systems, databases, architecture, pattern, event-driven |
| [[concepts/saga-pattern]] | Coordinates multi-service workflows via local transactions + compensating actions; choreography vs orchestration; compensating actions are business decisions | distributed-systems, system-design, architecture, pattern, event-driven |
| [[concepts/transactional-outbox]] | Solves the dual-write problem by writing events to the same DB transaction as business data; relay process forwards to broker; at-least-once delivery | distributed-systems, databases, pattern, event-driven, reliability |

## Technologies
| Page | Summary | Tags |
|---|---|---|

## Papers & Articles
| Page | Summary | Tags |
|---|---|---|
| [[papers-articles/beyond-vibe-coding-200-autonomous-agents]] | Lessons from orchestrating 200+ Claude Code instances on a shared codebase: two LLM primitives, three failure loops, checkpoint execution | ai, multi-agent, autonomous-agents, software-engineering |
| [[papers-articles/harness-engineering-for-coding-agent-users]] | Framework for outer harness controls (guides + sensors, computational vs inferential, three regulation dimensions); industry examples from OpenAI, Stripe, Thoughtworks | ai, autonomous-agents, software-engineering, quality |
| [[papers-articles/guide-to-event-driven-architectural-patterns]] | Six EDA patterns (Competing Consumer, Consume and Project, Event Sourcing, Async Task Execution, Transactional Outbox, Saga) with composability guidance | distributed-systems, system-design, architecture, pattern, event-driven |

## Comparisons
| Page | Summary | Tags |
|---|---|---|
