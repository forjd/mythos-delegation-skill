---
name: delegation-strategy
description: Decision guide for choosing between inline work, subagents (Agent tool), parallel agents, and Workflow orchestration. Use when planning a task that could involve searching many files, independent parallel strands, large-scale fan-out, or when tempted to spawn agents — to pick the cheapest level that fits.
---

# Delegation strategy

Pick the **lowest rung** on this ladder that fits the task. Every step up costs latency, tokens, and context-transfer overhead — a subagent starts with zero knowledge of the conversation.

## The ladder

1. **Do it yourself** (default) — you know where to look, the work is linear, or it's small.
2. **One subagent** — the search is wide but the answer is narrow, or a specialized agent type fits.
3. **Parallel subagents** — multiple genuinely independent strands of work.
4. **Workflow** — structured fan-out at scale, AND the user explicitly opted in.

## When a subagent earns its cost

The core trade-off is **context economy vs. directness**. An agent burns its own context and returns only its final message.

- **Delegate wide-search / narrow-answer work.** "Which of these 40 files handle auth?" — let an Explore agent read the file dumps and hand back the conclusion. Your context stays clean for the real work.
- **Never delegate single-fact lookups.** If you already know the file, symbol, or value, a direct Grep/Read beats spawning an agent and writing it a prompt.
- **Fan out independent work.** Launch unrelated investigations as multiple agents in a single message so they run concurrently.
- **Match specialized agent types** (Explore, Plan, docs agents, custom `.claude/agents/*`). Their tools and prompts are scoped to the job; prefer them over general-purpose.
- **Don't duplicate.** Once you've delegated a search, don't also run it yourself. Wait for the result.
- **Continue, don't respawn.** Use SendMessage to follow up with an existing agent (it keeps its context) instead of starting a fresh one.

## When a Workflow is justified

Two gates, **both** required:

1. **Explicit user opt-in — a hard rule, not a judgment call.** Only run a Workflow if the user asked for multi-agent orchestration in their own words, used the "ultracode" keyword / has it on for the session, invoked a skill that calls for one, or named a saved workflow. A task that would merely *benefit* from a workflow does not count. Without opt-in: describe what a workflow could do and its rough cost, and let the user choose.
2. **The task's shape needs deterministic orchestration** — control flow that should be code, not model judgment: fan-out over a known work-list (migrations, audits), independent finders + adversarial verification of every finding, loop-until-dry discovery, judge panels over competing designs. Scout inline first to discover the work-list, then orchestrate over it.

If the work is a single investigation or a linear edit, a plain subagent (or rung 1) is correct even when workflows are available.

## Failure modes to avoid

- Delegating trivial lookups → pure latency for nothing.
- Doing giant multi-file sweeps inline → context pollution that degrades the rest of the session.
- Escalating to a workflow without opt-in → token surprise; the scale is the user's decision, never inferred.
