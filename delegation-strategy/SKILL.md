---
name: delegation-strategy
description: Decision guide for choosing between inline work, subagents (Agent tool), parallel agents, and Workflow orchestration. Use when planning a task that could involve searching many files, independent parallel strands, large-scale fan-out, or when tempted to spawn agents — to pick the cheapest level that fits.
license: MIT
---

# Delegation strategy

Pick the **lowest rung** on this ladder that fits the task. Every step up costs latency, tokens, and context-transfer overhead — a subagent starts with zero knowledge of the conversation.

The ladder only goes as high as your harness's tools. Before climbing, check what you actually have — subagent spawning? parallel execution? a workflow/orchestration tool? — and treat your highest supported rung as the ceiling. If a rung is missing, see "Degrading gracefully" below.

## The ladder

1. **Do it yourself** (default) — you know where to look, the work is linear, or it's small.
2. **One subagent** — the search is wide but the answer is narrow, or a specialized agent type fits.
3. **Parallel subagents** — multiple genuinely independent strands of work.
4. **Workflow orchestration** (e.g. Claude Code's `Workflow` tool) — structured fan-out at scale, AND the user explicitly opted in.

## When a subagent earns its cost

The core trade-off is **context economy vs. directness**. An agent burns its own context and returns only its final message.

- **Delegate wide-search / narrow-answer work.** "Which of these 40 files handle auth?" — let an Explore agent read the file dumps and hand back the conclusion. Your context stays clean for the real work.
- **Never delegate single-fact lookups.** If you already know the file, symbol, or value, a direct Grep/Read beats spawning an agent and writing it a prompt.
- **Write the prompt as if to a stranger.** The agent knows nothing you haven't told it: include file paths, constraints already discovered this session, and the shape of answer you want back (a list, a verdict, a path). Most delegation failures are under-specified prompts, not wrong rungs.
- **Fan out independent work.** Launch unrelated investigations as multiple agents in a single message so they run concurrently. Parallelism pays when the strands are slow or numerous, not merely independent — two quick lookups can stay sequential.
- **Isolate parallel writers.** Agents that edit files concurrently need disjoint file sets, or per-agent worktrees if your harness offers them. Logical independence isn't enough to stop them clobbering each other's changes.
- **Match specialized agent types** when your harness defines them (e.g. Claude Code's Explore and Plan agents, docs agents, custom agent definitions). Their tools and prompts are scoped to the job; prefer them over general-purpose.
- **Don't duplicate.** Once you've delegated a search, don't also run it yourself. Wait for the result.
- **Continue, don't respawn.** If your harness can message an existing agent (e.g. Claude Code's SendMessage), follow up with it — it keeps its context — instead of starting a fresh one.

## When workflow orchestration is justified

This rung means a dedicated orchestration tool that runs many agents under deterministic, code-driven control flow (Claude Code's `Workflow` is one; adapt to whatever your harness provides). Two gates, **both** required:

1. **Explicit user opt-in — a hard rule, not a judgment call.** Only run a Workflow if the user asked for multi-agent orchestration in their own words, used an opt-in keyword your harness defines (e.g. Claude Code's "ultracode", per message or enabled for the session), invoked a skill that calls for one, or named a saved workflow. A task that would merely *benefit* from a workflow does not count. Without opt-in: describe what a workflow could do and its rough cost, and let the user choose.
2. **The task's shape needs deterministic orchestration** — control flow that should be code, not model judgment: fan-out over a known work-list (migrations, audits), independent finders + adversarial verification of every finding, loop-until-dry discovery, judge panels over competing designs. Scout inline first to discover the work-list, then orchestrate over it.

If the work is a single investigation or a linear edit, a plain subagent (or rung 1) is correct even when workflows are available.

## Degrading gracefully

When your harness lacks a rung, translate the principle, not the tool:

- **No workflow tool:** emulate rung 4 at rung 3 — decompose into batches of parallel subagents and iterate, with you as the orchestrator. The opt-in gate survives the translation: spawning dozens of agents is a scale decision the user must make explicitly, no matter which tool does the fan-out.
- **No subagents at all:** rung 1 is the whole ladder. The principle becomes context hygiene — read narrowly, summarize findings as you go instead of retaining raw file dumps, and drop intermediate material once distilled. When a task is genuinely a fan-out job your tooling can't express (a 200-file migration, an exhaustive audit), say so and propose splitting it into sessions or steps the user drives, rather than grinding through it badly.

## Failure modes to avoid

- Delegating trivial lookups → pure latency for nothing.
- Doing giant multi-file sweeps inline → context pollution that degrades the rest of the session.
- Escalating to a workflow without opt-in → token surprise; the scale is the user's decision, never inferred.
