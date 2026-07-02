# mythos-delegation-skill

A Claude Code agent skill: a decision guide for when to work inline, spawn subagents, run agents in parallel, or escalate to Workflow orchestration. Distilled from the Claude Fable 5 harness guidance.

The skill itself is [SKILL.md](SKILL.md).

## Install

**User-level** (available in every project) — symlink the repo into your skills directory:

```sh
ln -s "$(pwd)" ~/.claude/skills/delegation-strategy
```

**Project-level** — copy or symlink it into a repo's `.claude/skills/delegation-strategy/`.
