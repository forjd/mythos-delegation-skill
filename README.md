# delegation-strategy

[![Agent Skills spec](https://img.shields.io/badge/Agent%20Skills-compliant-blue)](https://agentskills.io/specification)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Install with skills.sh](https://img.shields.io/badge/skills.sh-install-black)](https://skills.sh)

An [Agent Skill](https://agentskills.io) that teaches coding agents when to delegate — and, just as often, when not to.

Agents with subagent tools tend to fail in one of two directions: they spawn agents for lookups they could answer with a single grep, or they grind through forty-file sweeps inline and pollute their own context. This skill gives the agent a four-rung ladder and tells it to pick the lowest rung that fits.

## The ladder

| Rung | Use when |
| --- | --- |
| 1. Do it yourself | You know where to look, the work is linear, or it's small. **The default.** |
| 2. One subagent | The search is wide but the answer is narrow, or a specialized agent type fits. |
| 3. Parallel subagents | Multiple genuinely independent strands of work. |
| 4. Workflow orchestration | Structured fan-out at scale, **and** the user explicitly opted in. |

Every step up costs latency, tokens, and context-transfer overhead — a subagent starts with zero knowledge of the conversation. The skill spells out the heuristics for each rung: which searches earn a subagent, why single-fact lookups never do, how to write the hand-off prompt, when parallel fan-out pays (and how to keep parallel writers from clobbering each other), and the two hard gates a Workflow must pass (explicit user opt-in plus a task shape that needs deterministic orchestration).

The full text is in [`delegation-strategy/SKILL.md`](delegation-strategy/SKILL.md) — about 50 lines, loaded only when the agent is planning a task that could involve delegation. Codex and ChatGPT-compatible skill metadata lives in [`delegation-strategy/agents/openai.yaml`](delegation-strategy/agents/openai.yaml).

## Install

With the [skills.sh](https://skills.sh) CLI (the skill installs under its spec name, `delegation-strategy`):

```sh
npx skills add forjd/mythos-delegation-skill
```

or with Bun:

```sh
bunx skills add forjd/mythos-delegation-skill
```

Add `-g` to install user-level (active in every project) rather than project-level.

### Codex / ChatGPT-compatible install

For Codex, copy or symlink the skill directory into your Codex skills folder:

```sh
git clone https://github.com/forjd/mythos-delegation-skill.git
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
ln -s "$(pwd)/mythos-delegation-skill/delegation-strategy" \
  "${CODEX_HOME:-$HOME/.codex}/skills/delegation-strategy"
```

Then ask Codex to use `$delegation-strategy`, or let it trigger implicitly when the task involves subagents, parallel review, large searches, or orchestration decisions.

### Claude Code install

Clone this repo, then symlink the skill directory into Claude Code's skills directory. For user-level install:

```sh
git clone https://github.com/forjd/mythos-delegation-skill.git
ln -s "$(pwd)/mythos-delegation-skill/delegation-strategy" ~/.claude/skills/delegation-strategy
```

For a single project, copy or symlink `delegation-strategy/` into that repo's `.claude/skills/` instead.

## How it works

Agent Skills load progressively. At startup the agent sees only the skill's name and description (~100 tokens); the description is written to trigger when the agent is planning work that could involve searching many files, parallel strands, or large fan-out. Only then does the agent load the full instructions.

The skill is distilled from delegation guidance in Anthropic's Claude Fable 5 harness prompt, then generalized for Agent Skills-compatible runtimes including Claude Code and Codex. It's most useful for agents that *don't* carry this guidance natively: subagents of subagents, SDK-built agents, and other Agent Skills-compatible tools.

The ladder degrades gracefully. Agents without a workflow tool emulate rung 4 with batches of parallel subagents (the user-opt-in rule still applies); agents with no subagent tools at all get the rung-1 principles — context hygiene, and telling the user when a task is really a fan-out job their tooling can't express.

## Repository layout

```
mythos-delegation-skill/
├── delegation-strategy/
│   ├── agents/
│   │   └── openai.yaml   # Codex / ChatGPT-compatible UI metadata
│   └── SKILL.md          # the skill: frontmatter + instructions
├── LICENSE
└── README.md
```

## Development

The skill follows the [Agent Skills specification](https://agentskills.io/specification). To validate after editing:

```sh
npx skills-ref validate ./delegation-strategy
```

For Codex compatibility, also run the Codex skill validator if available:

```sh
python3 /path/to/skill-creator/scripts/quick_validate.py ./delegation-strategy
```

Codex uses only the `name` and `description` frontmatter fields to decide when to load a skill, so keep trigger wording in `description` and UI-facing metadata in `delegation-strategy/agents/openai.yaml`.

Issues and PRs welcome — especially counterexamples where the ladder gives the wrong answer.

## License

[MIT](LICENSE)
