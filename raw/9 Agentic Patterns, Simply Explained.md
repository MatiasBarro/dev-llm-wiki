---
title: "9 Agentic Patterns, Simply Explained"
source: "https://newsletter.systemdesign.one/p/agentic-design-patterns?utm_source=post-email-title&publication_id=1511845&post_id=194925836&utm_campaign=email-post-title&isFreemail=false&r=2l7yfa&triedRedirect=true&utm_medium=email"
author:
  - "[[Neo Kim]]"
published: 2025-11-18
created: 2026-04-23
description: "A hands-on guide to building intelligent systems"
tags:
  - "clippings"
---
## 140: The design decisions behind modern AI systems—how each design pattern works, where it breaks, and when to use it.

- *[Share this post](https://newsletter.systemdesign.one/p/agentic-design-patterns/?action=share) & I'll send you some rewards for the referrals.*

---

When you build software with LLMs [^1], you need to decide how much control to give the model.

*Does your code run every step, or does the model figure out the steps on its own?*

Agentic patterns are design patterns for making that decision. They’re the same architecture choices you already make in any system: *who controls what happens next, what happens on failure, and how data moves between components.*

The difference is that some of those components are now language models.

This newsletter covers nine of those patterns…

On one end, *workflow patterns* where your code controls every step. On the other hand, *agent patterns* where the Large Language Model (**LLM**) decides what to do next.

That boundary is a decision about how much control to hand to the model: at some point, you stop telling the model what to do and start letting it figure that out.

The first question isn’t which pattern to use. It’s whether you need one at all.

Let’s start there...

---

## Find out why enterprise leaders are doubling down on data foundations for AI (Partner)

![](https://substackcdn.com/image/fetch/$s_!s4PY!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fcc0b098a-af83-481b-805b-c1ad861f8c9d_1280x720.gif)

AI is moving fast, but your data foundation isn’t keeping up?

That’s exactly why leaders from JPMorgan, Mercedes-Benz, Siemens, and Roche contributed to this AWS book on agentic analytics.

Here’s what you get:

- Real strategies from enterprise leaders: Learn how top companies are building data foundations for AI systems that actually scale.
- Practical frameworks you can apply: From data strategy to data products, ML, and agentic AI.
- Perspectives from 15+ enterprise leaders: Each chapter brings a unique view from senior leaders across industries.
- Complex topics broken down clearly: Understand what matters without getting lost in theory.

An **[AWS book](https://pages.awscloud.com/awsmp-gim-jqup-adhoc-aim-ent-ai-data-leader-book-1-ent.html?trk=68d21c4e-c3e8-4668-959c-cb996265ef28&sc_channel=el)** designed to help you build data foundations for intelligent agents that scale.

Download it now and learn how to build systems ready for intelligent agents.

(Thanks to [AWS](https://pages.awscloud.com/awsmp-gim-jqup-adhoc-aim-ent-ai-data-leader-book-1-ent.html?trk=68d21c4e-c3e8-4668-959c-cb996265ef28&sc_channel=el) for partnering on this post.)

---

**Inside this newsletter, you’ll get:**

- **The shift from prompts to systems.** Why simple LLM calls break down, and how workflows and agents introduce structured decision-making.
- **A practical escalation ladder.** When to stay simple, when to use workflows, and when agents are actually justified.
- **Workflow patterns.** chaining, routing, parallelization, and orchestrator-workers, with clear tradeoffs in cost, latency, and failure modes.
- **Agent patterns.** Reflection, tool use, ReAct, and planning, focusing on how the model makes decisions and where things go wrong.
- **Evaluator-optimizer loop.** How to add a reliability layer with evaluation, iteration limits, and cost control.
- **Real-world case study.** Combining multiple patterns in an AI code review pipeline, showing how to choose the simplest architecture that works.

---

## Not everything needs an agent

Most tasks don’t need an agent [^2] …

Most tasks don’t even need a workflow…The default should be the simplest setup that works, and you escalate only when that setup breaks.

Start with a **direct API call to an LLM**. You can do more with this than most people think:

- Summarization
- Classification
- Extraction
- Rewriting
- Translation
- Code generation with clear specs

Most people skip this level too quickly!

**Workflow patterns** are the next step up.

Escalate when a task has many steps, and when focused attention at each step improves the output. Between those steps, you define **validation gates**: *checks that verify the output before passing it forward*. The defining feature: *you can write every step down before the system runs.* Your code owns the control flow.

And the LLM handles the details.

**Agent patterns** are for when the number and type of steps are *unknown* until the system runs.

The model needs to look at results, decide what to do next, and choose its own path. For example, say you built a support agent that handles order cancellations. A customer asks to cancel, but the agent doesn’t know upfront whether it needs to check the refund policy, look up the shipping status, or pass the issue to a human.

The steps depend on what it finds…

Here’s how you know it’s time to switch: you write more code handling errors and exceptions than doing the actual work. Or you keep adding special cases for situations the LLM encounters you didn’t plan for.

If you can still write down all the steps before the system runs, stick with a workflow.

![Agentic pattern complexity escalation ladder showing four levels from direct API calls to multi-agent systems with decision criteria at each step-up point](https://substackcdn.com/image/fetch/$s_!SSER!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F6337a830-7cfd-437c-a23d-24bf10f7dafb_2048x1143.jpeg)

Agentic pattern complexity escalation ladder showing four levels from direct API calls to multi-agent systems with decision criteria at each step-up point

Agent patterns sit within multi-agent architectures, i.e., many agents cooperate on the same problem, each with its own role.

One common mistake: you get your system to 70-80% of a prototype and assume the architecture needs upgrading. It usually doesn’t…The real issue is usually prompt quality or missing validation gates. Move to a more complex pattern only after the simpler one has actually failed, not when it feels limiting.

With that baseline set, here are the four workflow patterns:

---

## Workflow patterns: you control the flow

With workflow patterns, your code controls what happens.

You decide the steps, the order, and the checks between them. The LLM handles each step, but your code is in charge…

### 1\. Prompt chaining

Prompt chaining breaks a task into a series of LLM calls.

Each call takes the output from the previous step, and a check runs between them to ensure the result is correct before moving on.

If you’ve ever set up a **CI/CD** (Continuous Integration/Continuous Deployment) pipeline, this is the same idea. A build pipeline compiles, lints, tests, and packages. Each step must pass before the next one runs.

If tests fail, the pipeline stops.

![Prompt chaining workflow pattern showing sequential LLM calls connected by validation gates that stop the pipeline on failure](https://substackcdn.com/image/fetch/$s_!qUdp!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe6f8b6ec-1bc5-4670-9b25-3c2d782f1098_2048x1143.jpeg)

Prompt chaining workflow pattern showing sequential LLM calls connected by validation gates that stop the pipeline on failure

For example, in [legal contract review](https://gradientflow.substack.com/p/ais-biggest-enterprise-test-case), this looks like: extract all clauses from a contract, classify each clause by risk level, then generate a summary of high-risk items for human review. Products like [Thomson Reuters CoCounsel](https://legal.thomsonreuters.com/en/c/cocounsel) and Robin AI run variations of this pipeline.

Each step gets a focused prompt with a narrow task, and a validation gate checks the output before passing it forward.

#### Tradeoffs

Latency grows linearly…

Five steps mean five round-trip. Errors carry forward: if step two misclassifies a clause, steps three through five process bad input. The gates between steps catch these failures early, but they only catch what the conditions you write can catch.

### 2\. Routing

A routing pattern classifies the input and sends it to the right handler: *a different prompt, a different model, or a different sub-workflow.*

Each handler is built for one type of input.

This is like a hospital front desk. A patient walks in, the front desk checks their symptoms, and then sends them to the right doctor. The front desk doesn’t treat anyone. And the doctor doesn’t check in patients.

![Routing pattern architecture showing a classifier directing inputs to cheap, specialized, or expensive model handlers based on input type](https://substackcdn.com/image/fetch/$s_!YT5p!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F36bd3215-0100-4043-9ebd-0589949a309a_2048x1143.jpeg)

Routing pattern architecture showing a classifier directing inputs to cheap, specialized, or expensive model handlers based on input type

[Sierra AI](https://sierra.ai/blog/constellation-of-models) routes across 15+ models this way.

Classification tasks go to one model, tool calling to another, and response generation to a third. Customer support platforms like [Intercom Fin](https://www.intercom.com/help/en/articles/9929230-the-fin-ai-engine) route each customer request based on what the customer wants and how they feel about it, sending requests to either an AI handler or a human agent.

The cost argument is built into the pattern…

Simple queries hit cheap, fast models. While complex ones hit expensive, capable ones. You’re not paying the highest prices for work that a smaller model handles well.

#### Tradeoffs

In 1 word: misclassification.

A complex query routed to the cheaper model produces a poor answer. A simple query routed to the expensive model is just a waste of money.

Either way, the router is a single point of failure, and its accuracy sets the upper limit for the entire system.

### 3\. Parallelization

In software, parallelization means running many tasks simultaneously rather than one after another. With LLMs, it has two variants that solve different problems…

![Parallelization pattern comparing sectioning (split task into independent subtasks and merge) versus voting (run same task multiple times and aggregate for consensus)](https://substackcdn.com/image/fetch/$s_!8m3i!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F244a234b-7085-4ac5-ae83-4a49c35d3810_2048x1143.jpeg)

Parallelization pattern comparing sectioning (split task into independent subtasks and merge) versus voting (run same task multiple times and aggregate for consensus)

**Sectioning** splits a task into independent parts, runs them in parallel, and merges the results. This is like a construction crew. Electricians, plumbers, and carpenters work in the same building at the same time. Their work gets combined at the end.

[GitHub Advanced Security](https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning) works this way: CodeQL and third-party tools like Snyk and Semgrep scan the same pull request in parallel, each producing independent security alerts.

**Voting** runs the same task many times with different prompts and combines the answers. This is like a jury. Each juror looks at the same evidence and gives their own opinion, and the group decides together.

You’d build this for something like a security review: three separate prompts analyze the same code change for vulnerabilities, and you flag an issue only if two or more agree. Running the task many times is more expensive, but it catches fewer false alarms.

In short, *sectioning* gives each worker a different job; *voting* gives every worker the same job.

#### Tradeoffs

Cost multiplies with every parallel branch.

Partial failures are a design decision you need to plan for up front: *if one branch fails, do you retry it, proceed with the others, or fail the whole operation?* There’s no universal answer, and a wrong choice is expensive.

### 4\. Orchestrator-workers

The orchestrator-workers pattern uses one central LLM to break a task into subtasks, assign each subtask to a worker LLM, and combine the results.

The difference from the earlier patterns is that the subtasks aren’t known in advance. Orchestrator figures them out as it goes. This is like a general contractor building a house:

- They hire the right people, but don’t know every subcontractor they’ll need upfront
- Plumber finds a structural problem, so the contractor brings in a structural engineer
- The work plan changes as the project reveals new problems

![Orchestrator-workers pattern showing a central LLM dynamically delegating to worker agents including unknown workers decided at runtime](https://substackcdn.com/image/fetch/$s_!c9_Q!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F042accd5-ff55-4030-acf1-e28ecfcb7f0f_2048x1143.jpeg)

Orchestrator-workers pattern showing a central LLM dynamically delegating to worker agents including unknown workers decided at runtime

[Cursor’s agent mode](https://cursor.com/learn/agents) works this way. It starts with different agents for different parts of the codebase:

- One adding tests
- Another updating documentation
- A third refactoring shared utilities

You’re trusting the LLM to figure out what work needs to be done, not just to do the work you’ve assigned.

This uses several LLMs, but it’s NOT a multi-agent architecture: *a single central LLM remains in control of the entire process.* In multi-agent systems, agents can call each other directly without a central coordinator, and control can transfer between them.

#### Tradeoffs

The orchestrator can lose track of the original goal.

It breaks simple tasks into too many subtasks. And when workers finish faster than it can combine results, the orchestrator itself becomes the bottleneck.

---

These four patterns get gradually more powerful and more expensive…

Chaining and routing are cheap and predictable: you know what will happen before it runs. Parallelization saves time but costs more. Orchestrator-workers handle problems you can’t predict upfront, but it introduces failure modes you can’t predict either. The first three patterns are simple to understand.

Most production systems shouldn’t need to go further than parallelization. When they do, the next four patterns hand control to the model itself...

---

***Reminder: this is a teaser of the subscriber-only newsletter, exclusive to my golden members.***

When you upgrade, you’ll get:

- **High-level architecture of real-world systems.**
- Deep dive into how popular real-world systems work.
- **How real-world systems handle scale, reliability, and performance.**

---

## Agent patterns: LLM controls the flow

With agent patterns, you STOP defining what happens and in what order.

Instead of defining steps, you define constraints: *what tools are available, how much the model can spend, and when to stop*. The model observes, reasons, and chooses what happens next.

![Side-by-side comparison of workflow patterns where you define the fixed execution path versus agent patterns where the LLM controls the flow and you define the boundaries](https://substackcdn.com/image/fetch/$s_!8Y67!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F05738b84-f3c1-43d4-a706-6ed0ae52f5ef_2048x1143.jpeg)

Side-by-side comparison of workflow patterns where you define the fixed execution path versus agent patterns where the LLM controls the flow and you define the boundaries

That trust decision appears differently in each pattern: *how much autonomy, over what actions, with what limits.*

### 5\. Reflection

Reflection is when the LLM generates output, reviews its own work, and then fixes the problems it found.

This loop repeats until the output is good enough. In a two-agent version, you use the same model with different prompts: *one generates, one reviews, and they pass work back and forth.*

This is like submitting a pull request. A reviewer reads your code and leaves comments on what’s wrong. You revise and push again. They review the new version. And this loop continues until the code is good enough to merge.

![Reflection agent pattern showing generate-critique-revise loop with iteration counter and two-agent variant using same model with different prompts](https://substackcdn.com/image/fetch/$s_!ERmq!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9b1b40b7-c53e-4ff4-9f14-36febfed17f7_2048x1143.jpeg)

Reflection agent pattern showing generate-critique-revise loop with iteration counter and two-agent variant using same model with different prompts

A benchmark shows this clearly.

SWE-bench tests whether models can fix actual bugs in open-source projects. [Claude 3.7 Sonnet with a single prompt solves 62.3% of those issues](https://www.anthropic.com/research/swe-bench-sonnet) (submits a fix that passes the project’s test suite). The same model, when used with a more structured setup, solves 70.3%: same model, different architecture, eight-point jump.

The model doesn’t get smarter between iterations; it gets more focused.

Each round of feedback eliminates possibilities, so the model focuses on a smaller set of issues each time.

#### Tradeoffs

The cost increases with each loop.

Each iteration adds time and tokens. Two loops might catch real issues. Five loops might mean the model is repeating the same mistakes instead of fixing them.

You’re trading latency and cost for quality, and the improvements get smaller with each pass.

There’s no separate tool checking the work, no test suite, no external judge.

The model is its own quality check, which is both its strength and its limit. A model can only catch errors it already knows how to recognize. For tasks with measurable criteria (code generation, math, structured output), self-critique usually works.

For tasks that require information not already in context, the loop stops making progress because the model can’t critique what it doesn’t know.

### 6\. Tool use

Most of what an agent needs isn’t in its context window (text the model can see and work with at once).

Tool use is the pattern for getting that data. The model selects from a **registry** of available tools (APIs, databases, code execution, search) and calls them as needed. **Registry** is just a list of tools set up in advance that the model can call.

One tool’s output often becomes the next tool’s input, and the model decides the sequence while the program is running.

![Tool use agent pattern showing LLM selecting from a tool registry of APIs, databases, and code execution, with results feeding back into the next tool selection](https://substackcdn.com/image/fetch/$s_!HGPR!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5f117c2a-b8b0-4435-9f1f-1c64583bf8a0_2048x1143.jpeg)

Tool use agent pattern showing LLM selecting from a tool registry of APIs, databases, and code execution, with results feeding back into the next tool selection

Tool use is the most widely adopted agent pattern.

Nearly every production agent calls external APIs, runs code, or queries databases. Sierra AI runs this across customer service. A customer asks about an order, and the agent authenticates the caller, queries a CRM (Customer Relationship Management) system for order history, checks a knowledge base for the return policy, and issues a refund.

Each of those is a tool call, and the agent decides which to call based on how the conversation unfolds. Coding agents such as Devin and Cursor do the same thing with different tools: file reads, terminal commands, test runners, and git operations.

When you’re building a tool-use agent, real questions are:

- What goes in the registry?
- What permission scope does each tool get?
- Which tools can cause side effects (refunds, database writes) versus which should wait until the model is done gathering information?

That last question matters most…

Limiting what each tool is allowed to do is the difference between a useful agent and a risk:

- **Read-only tools** (database queries, file reads, search) run freely
- **Write tools** (file edits, API updates) require confirmation or sand boxing (running the operation in an isolated environment where it can’t affect production data)
- **High-stakes tools** (payments, deletions, external communications) need explicit human approval before execution

Reads can’t break anything. A bad query wastes time but doesn’t change any data. Writes and payments can’t be undone, so they need a human check.

#### Tradeoffs

Choosing the wrong tool wastes tokens and time…

Tools with side effects create a chain of failures when the model calls them in the wrong order or with bad inputs. And the performance gaps are enormous:

- **Direct function calls** (your code calls a function without LLM reasoning): ~30ms overhead
- **Protocol-based calls** through MCP (Model Context Protocol, a standard for connecting LLMs to external tools, similar to how REST standardizes web APIs): 300-500ms per call before the actual function runs
- **LLM-mediated calls** (the model reasons about which function to call): slower still

The general rule: *use **direct calls** for deterministic operations (operations that produce the same result every time)*. Writing to a database or posting to an API are good examples.

*Use **tool calling** through the model only when the model genuinely needs to decide which tool to use next.*

---

Know someone who wants to learn system design? Consider gifting a subscription:

---

### 7\. ReAct (reason + act)

ReAct stands for **reason, act, observe, repeat**.

The model looks at the current state, takes an action (usually a tool call), observes the result, and reasons again. The cycle repeats until the task is done or the model decides to stop.

![ReAct agent loop architecture showing reason-act-observe cycle with growing context window and while-loop code representation](https://substackcdn.com/image/fetch/$s_!xpZY!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7e223603-ff7b-4d22-a737-85e4475e5576_2048x1143.jpeg)

ReAct agent loop architecture showing reason-act-observe cycle with growing context window and while-loop code representation

For example, open [Claude Code](https://blog.promptlayer.com/claude-code-behind-the-scenes-of-the-master-agent-loop/) and ask it to fix a bug.

It reads your codebase (observe), reasons about what needs to change (think), edits a file (act), runs your tests (observe), and adjusts if something fails. Internally: a `while` loop. As long as the model returns a tool call, execute it, pass the results back, and let the model decide what’s next.

It’s the architecture behind most production agents.

Because the model reasons before acting, you get a **reasoning trace**: a step-by-step record of why the agent did what it did. When something goes wrong, you read the trace rather than guessing.

#### Tradeoffs

Reasoning becomes less focused over long sessions as the model loses track of its original goal.

Action loops occur when the model repeatedly calls the same tool, expecting different results. And the context window fills up as each observation gets added to the conversation. Long-running ReAct agents need context management, or they degrade.

In [one reported case](https://github.com/anthropics/claude-code/issues/34685), Claude Code running on Opus 4.6 with a 1M-token context window started showing circular reasoning (repeating the same arguments in a loop without making progress) at 20% context usage.

By 48%, the model itself recommended starting a fresh session…

### 8\. Planning

The planning pattern breaks a goal into subtasks with dependencies and runs them in order. When something fails, the agent goes back, finds the wrong assumption, and adjusts the remaining steps.

![Planning agent pattern showing goal decomposition into subtask dependency tree with re-planning feedback loop triggered by test failures](https://substackcdn.com/image/fetch/$s_!EznP!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F83dd5846-a6c1-4f4b-981a-6e658532c63f_2048x1143.jpeg)

Planning agent pattern showing goal decomposition into subtask dependency tree with re-planning feedback loop triggered by test failures

For example, give Devin a GitHub issue, and it won’t start editing files right away.

First, it creates a step-by-step plan. Then it waits for your approval. If a test fails halfway through, it replans. In most coding agents, planning and ReAct work together. The planning layer breaks the problem down into steps. Each step runs through a ReAct loop to handle the unpredictable parts of execution.

Planning works best when the problem has clear decomposition points, semi-independent subtasks, and costly mistakes worth preventing with upfront analysis. A code migration across 50 files benefits from planning.

A single function rewrite doesn’t.

#### Tradeoffs

Bad decomposition is worse than no decomposition, because the agent follows a flawed structure early and spends tokens executing a wrong plan.

Plans don’t always adjust when conditions change. Re-planning is expensive in both tokens and latency. And agents struggle with how to break large problems down, getting stuck in analysis loops rather than making progress.

Planning agents are “ [kind of finicky](https://www.youtube.com/watch?v=sal78ACtGTc),“ in Andrew Ng’s words. When they work, they’re impressive. If they don’t, they spend your whole budget planning how to plan.

Each of these four patterns asks you to trust the model with a larger share of the decision-making. That’s the cost of the power they give you.

And none of them have a built-in way to catch when the output isn’t good enough.

That’s what the next part covers...

---

## 9\. Feedback loop: evaluator-optimizer

Reflection works entirely within a single agent.

The *evaluator-optimize* r is a separate component you attach to any pattern in your system. i.e., it’s not a separate pattern…instead, a quality check you add on top of the other patterns.

![Evaluator-optimizer feedback loop showing generator producing output, evaluator checking quality, and three stopping criteria: quality threshold, max iterations, and cost budget](https://substackcdn.com/image/fetch/$s_!_QNO!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa36a7bd8-101c-4a26-99a9-1dcfcf06b9ab_2048x1143.jpeg)

Evaluator-optimizer feedback loop showing generator producing output, evaluator checking quality, and three stopping criteria: quality threshold, max iterations, and cost budget

The generator and evaluator can be:

- Different models
- Different prompts on the same model
- An LLM paired with a deterministic check, like a test suite

The evaluator could be an LLM judge [^3], a test suite, a set of rules, or a human.

Stopping the loop is the harder design problem. A quality threshold alone will loop forever on a hard input because some inputs never get good enough, no matter how many attempts are made. While a fixed iteration count wastes budget on easy problems while giving up too early on hard ones. Most production systems combine both and add a cost budget as a hard ceiling.

You need all three working together…

The pattern attaches to anything you’ve already built.

In a prompt chaining pipeline, you add the evaluator after the final step: *last output gets checked and regenerated until it passes the check*. In a ReAct agent, the evaluator sits outside the loop and judges the final answer rather than individual tool calls.

![](https://substackcdn.com/image/fetch/$s_!xbnc!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0bcfac1c-e378-4710-876f-03418ce17dc1_2048x1143.jpeg)

The clearest case is code generation with test suites.

The generator writes code; the evaluator runs tests; and failures provide feedback for the next pass. Tests give a clear pass/fail result that the generator must satisfy.

Because the evaluation criteria exist outside the model, upgrading the generator improves output without changing anything else in your system.

Beyond code, this works anywhere you can score the output:

- translation scores,
- content safety ratings,
- structured output validation.

The requirement is an evaluation more specific than “looks good.”

#### Tradeoffs

The main failure mode is the generator-evaluator loop, where each improvement gets smaller.

The fifth iteration rarely improves as much as the second, but costs the same. Without a budget limit, a hard input spends tokens indefinitely. A weak evaluator is worse than no evaluator: it makes you think output was checked when it wasn’t, which is harder to recover from than no check at all.

That covers all nine patterns.

Before comparing them, there are systemic problems that show up only at the agent level...

---

Know someone who wants to learn system design? Consider gifting a subscription:

---

## Systemic concerns at the agent level

Workflow patterns fail in predictable places:

- Gate rejects bad output
- Router misclassifies
- Branch times out

Agent patterns fail in ways you won’t expect until production, because the model is making decisions your code used to make.

**State** is the first problem.

An agent’s context window is its short-term memory, and it’s finite. Long-running tasks generate reasoning traces, tool results, and observation piles up faster than the window can hold. If new content arrives and the window is full, the oldest content gets dropped. So earlier decisions get lost.

This is called **agentic amnesia**.

The standard fix is checkpointing: *saving the agent’s state to an external database so it can resume or summarize its history*. But that raises design questions you’ll need to answer: *what to keep, what to discard, and how to summarize state without losing information the agent will need later.*

**Security** is where the failure surface gets serious.

The threat comes from three things: *access to private data*, *exposure to untrusted content* (user input, web pages, third-party API responses), and *a channel that can send data out* (email, API calls, code execution).

Most people don’t think about this until something goes wrong…

The Nibzard handbook calls this the “lethal trifecta.”

Any two of those three are manageable. The missing piece stops the attack. Without private data, there’s nothing to steal. Without untrusted content, there’s no way to inject instructions. Without an exfiltration channel, there’s no way to send data out.

All three together require you to limit what each tool is allowed to do, run tools in isolated environments where they can’t touch real data, and treat every external input as if it might be trying to cause harm.

Then there’s the question of seeing what the agent is doing…

**Observability** works differently from traditional services because agents are non-deterministic (the same input can produce different outputs each time).

The same input produces different tool call sequences, different reasoning paths, and different outputs. You can’t reproduce a failure by replaying inputs. You need the reasoning trace from that specific run. At a minimum, you need structured logging of every tool call, every observation fed back into context, and every decision point.

Without it, you’re diagnosing a black box after something goes wrong.

Error recovery, cost budgeting, and human-in-the-loop checkpoints all matter too, but state, security, and observability are the three that most people underestimate. They’re also part of why some people split work across many agents, each with narrower permissions and fewer ways to fail. That’s a multi-agent architecture.

The fix for all three isn’t a new pattern…

Treat agents like code, not chat interfaces:

- Type your inputs (add type annotations to function parameters)
- Schema your outputs (validate responses against a JSON schema)
- Version your prompts (track changes in version control)
- Test your tool integrations (write integration tests for each tool)

Without this discipline, agents fail in predictable ways that look random but aren’t.

Here’s how to choose between different patterns...

---

## Choosing between patterns

The table below compares all 9 patterns across five dimensions: *cost, latency, blast radius* (how much damage spreads when something fails), *when to use, and when to avoid.*

For the question of whether you need an agent at all, the escalation ladder in “Not everything needs an agent” is the right starting point…

![Nine agentic patterns compared across cost, latency, blast radius, when to use, and when to avoid](https://substackcdn.com/image/fetch/$s_!CVBC!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F38cfa3b9-e640-4482-8d32-3670e5270818_1560x573.jpeg)

The table doesn’t capture interactions between patterns.

Routing and parallelization combine naturally: route by type, then process each type’s subtasks in parallel. This is the most common two-pattern composition in production.

- Tool use and ReAct are often confused: if your tool calls follow a predictable sequence, tool use alone is enough.
- The reason-act-observe loop adds overhead that is only worth it when the sequence isn’t knowable in advance.
- High cost of ReAct and planning doesn’t mean avoid them. It means the value needs to justify the cost. A ReAct agent debugging a production outage is worth the tokens. A ReAct agent answering FAQ questions is not.

How you connect patterns matters as much as which model you use…

[OpenAI’s own SWE-bench research](https://openai.com/index/introducing-swe-bench-verified/) found that changing the architecture built around GPT-4o changed its score from 23% to 33.2%, a ten-point jump, with the same model.

The bottom line: the architecture around the model matters as much as the model itself.

Now it’s time to put this into practice...

---

## Designing an AI code review system

The problem: *an AI code review system for pull requests (**PR**). Small PRs (docs, configs) need quick automated passes. Large PRs with complex logic need deep analysis.*

Both go through the same pipeline.

![AI code review system architecture showing full pipeline from incoming PR through routing, parallel security-style-logic reviews, ReAct investigation, and evaluator-optimizer with failure boundaries at each stage](https://substackcdn.com/image/fetch/$s_!k8PA!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F78184073-45b0-4490-ab5c-016f6851f7b9_2048x1143.jpeg)

AI code review system architecture showing full pipeline from incoming PR through routing, parallel security-style-logic reviews, ReAct investigation, and evaluator-optimizer with failure boundaries at each stage

Start with routing.

A classifier examines each PR: file types, diff size, and affected subsystems. Documentation changes go to a fast model with a style-focused prompt. Config files go to a validation check. Code changes with logic modifications enter the full review pipeline. The pattern is the same as that of any router.

The inputs just happen to be diffs.

![Code review parallelization detail showing three independent reviewers for security, style, and logic running in parallel with merge step and escalation path to ReAct](https://substackcdn.com/image/fetch/$s_!qwAw!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2f4f5fbf-f7bc-44d2-8584-e48359581ac5_2048x1143.jpeg)

Code review parallelization detail showing three independent reviewers for security, style, and logic running in parallel with merge step and escalation path to ReAct

PRs that enter the full pipeline are split into parallel paths.

Three reviewers run simultaneously:

- Security
- Style
- Logic correctness.

Each receives a specialized prompt focused on a single review dimension. They don’t share context, and they don’t wait for each other. Results merge when all three finish.

This is sectioning, not voting: each reviewer looks for different things.

Now let’s say most comments from the parallel reviewers are simple: *“this variable is unused,” “this SQL query isn’t parameterized.”* But some are ambiguous. *“This function might have a race condition, but I’d need to see how it’s called.”* Those get escalated to a ReAct agent.

The ReAct agent investigates like a developer would, using file-read and search tools:

- Reads the flagged code
- Follows imports to understand the call graph (the map of which functions call which other functions)
- Checks test coverage
- Looks at the recent commit history to see if the code was recently changed and why

It might then discover the race condition is real because the function was recently made concurrent. Or it might find that the function is only called from a single-threaded context and clear the flag.

The reasoning trace records every step so that a human reviewer can audit the agent’s logic.

And before the final review goes out, the evaluator-optimizer checks the output.

An evaluation prompt checks whether comments are actionable, reference the right line numbers, and have consistent severity ratings. The evaluator refines *“this might be a problem”* into *“this function allocates on every loop iteration; consider moving the allocation outside the loop.”*

i.e., vague becomes specific.

Yet failure boundaries sit on each stage transition:

- If the router misclassifies a large PR as documentation-only, it gets a shallow review: bad, but recoverable when a human reviewer catches it
- If a parallel reviewer fails, the other two still produce results, and the merge step notes which dimension is missing
- If the ReAct agent loops, a timeout kills the investigation, and the original comment passes through with a “needs human review” flag

Here, orchestrator-workers would add complexity without adding anything.

The subtasks are predictable: you know before the system runs that you need security, style, and logic reviews. When you can list all subtasks in advance, an orchestrator adds unnecessary complexity. This is the simpler setup, and it works.

Planning is the wrong choice for the same reason…

The subtasks are independent, the failure cost per subtask is low (a missed comment, not a broken deployment), and planning overhead exceeds the value. Planning agents also struggle with large codebases, getting stuck in analysis loops, which is the opposite of what a fast review pipeline needs.

![Code review system scale comparison showing simple synchronous flow at 10 PRs per day versus load-balanced parallel architecture at 1000 PRs per day with ReAct as the scaling bottleneck](https://substackcdn.com/image/fetch/$s_!OPFj!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F34a55cbf-704a-4d97-9fda-66a87891be43_2048x1143.jpeg)

Code review system scale comparison showing simple synchronous flow at 10 PRs per day versus load-balanced parallel architecture at 1000 PRs per day with ReAct as the scaling bottleneck

At 10 PRs a day, you run everything synchronously. At 1,000, the routing classifier also works as a load balancer…

It now also distributes work across many reviewer instances to prevent any single one from getting overloaded. The parallel reviewers scale horizontally since each PR’s fan-out is independent.

The ReAct agent is the bottleneck: most expensive, slowest, and the only component that scales sub-linearly (adding more capacity yields less-than-proportional improvement).

Limit concurrent investigations and route only high-severity flags to full ReAct investigation while auto-approving straightforward ones. The evaluator-optimizer stays cheap: one pass per review, low cost per review.

(Four patterns, no orchestrator needed.)

---

## Closing Thoughts

Pick the simplest pattern that handles the failure mode you’re actually solving.

The most successful implementations aren’t complex. They’re built from simple pieces where each pattern does one thing well.

Remember, when a better model comes out, the architecture stays the same. Better models need less instruction to do the same work, so the prompts shrink. And the output gets better. So build for that.

---

Know someone who wants to learn system design? Consider gifting a subscription:

---

![Author Neo Kim; System design case studies](https://substackcdn.com/image/fetch/$s_!bEFk!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8f94ab8c-0d67-4775-992e-05e09ab710db_320x320.png)

👋 Find me on LinkedIn | Twitter Threads Instagram

---

Thank you for supporting this newsletter.

You are now 210,001+ readers strong, very close to 210k. Let’s try to get 211k readers by 25 April. Consider sharing this post with your friends and get rewards.

Y’all are the best.

![system design newsletter](https://substackcdn.com/image/fetch/$s_!6oWl!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2e739087-a910-4643-be36-997b6dd5b4af_800x500.png)

system design newsletter

---

### References

- Anthropic, [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) (2024)
- Anthropic, [SWE-bench with Claude 3.7 Sonnet](https://www.anthropic.com/research/swe-bench-sonnet) (2025)
- OpenAI, [Introducing SWE-bench Verified](https://openai.com/index/introducing-swe-bench-verified/) (2024)
- GitHub Engineering, [Multi-Agent Workflows: How to Engineer Ones That Don’t Fail](https://github.blog/ai-and-ml/generative-ai/multi-agent-workflows-often-fail-heres-how-to-engineer-ones-that-dont/) (2025)
- Nibzard, [Agentic AI Handbook](https://www.nibzard.com/agentic-handbook) (2025)
- Andrew Ng, [What’s next for AI agentic workflows](https://www.youtube.com/watch?v=sal78ACtGTc) (2024)
- Patil et al., [Tool Use Performance Benchmarks](https://arxiv.org/html/2512.08769v1), arXiv (2025)

[^1]: An LLM (large language model) is a type of AI model trained on lots of text to understand and generate human-like language.

[^2]: An agent is an AI system that can take actions, use tools, and make decisions step by step to complete a task, instead of just giving a single answer.

[^3]: An LLM judge is an AI model that evaluates or scores another model's output based on criteria such as quality, correctness, or usefulness.