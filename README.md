# mythos-delegation-skill

A Claude Code agent skill: a decision guide for when to work inline, spawn subagents, run agents in parallel, or escalate to Workflow orchestration. Distilled from the Claude Fable 5 harness guidance.

The skill itself is [SKILL.md](SKILL.md).

## Install

Via the [skills](https://skills.sh) CLI (installs as `delegation-strategy`):

```sh
npx skills add forjd/mythos-delegation-skill
```

```sh
bunx skills add forjd/mythos-delegation-skill
```

Add `-g` to install user-level (available in every project) instead of project-level.

### Manual install

**User-level** — clone this repo and symlink it into your skills directory:

```sh
ln -s "$(pwd)" ~/.claude/skills/delegation-strategy
```

**Project-level** — copy or symlink it into a repo's `.claude/skills/delegation-strategy/`.
