# harness-engineering-skill

A portable markdown reference on **harness engineering** — the discipline of building the scaffolding around an LLM that turns it into an agent: the loop, the tools, the context, the plans, the subagents, the budgets, the guardrails, and the recovery paths.

It is framework- and provider-agnostic. It shapes *how* a harness is built, not *which* concrete tool any agent must call. The whole repo is plain markdown — usable by any agent harness with a mechanism for loading contextual instructions, and readable on its own as a short course.

The entry point is [`SKILL.md`](SKILL.md). Chapters live under [`references/`](references/).

## What's inside

| Chapter | Topic |
|---|---|
| [`01-the-harness-and-the-model.md`](references/01-the-harness-and-the-model.md) | Overall stance — what to add, what to remove, layer boundaries, where minimalism is wrong |
| [`02-the-agent-loop.md`](references/02-the-agent-loop.md) | The loop itself — iteration, completion signals, verification |
| [`03-context-engineering.md`](references/03-context-engineering.md) | What the model sees — context curation, compaction, external state, provenance |
| [`04-tools-and-the-aci.md`](references/04-tools-and-the-aci.md) | Tool surfaces — schemas, descriptions, errors, scoping, tools-as-code |
| [`05-planning-and-decomposition.md`](references/05-planning-and-decomposition.md) | Plans and task breakdown — plan state, decomposition shape, attention |
| [`06-subagents-and-orchestration.md`](references/06-subagents-and-orchestration.md) | Multiple agents — when to split, orchestration, context isolation |
| [`07-determinism-guardrails-and-safety.md`](references/07-determinism-guardrails-and-safety.md) | Deterministic control — gates, guardrails, irreversible actions, escalation |
| [`08-budgets-and-recovery.md`](references/08-budgets-and-recovery.md) | Bounding work — budgets, caps, error recovery, resumption |
| [`09-skills-and-instructions.md`](references/09-skills-and-instructions.md) | Instructions and skills — portable methodology, progressive disclosure |
| [`10-evaluation-and-iteration.md`](references/10-evaluation-and-iteration.md) | Knowing the harness is good — evals, observability, the add/measure/remove loop, review checklist |

Source material is in [`references/sources.md`](references/sources.md).

## Use it

### Read it

It's a short course on harness engineering. The text stands on its own — no agent required.

### Feed it to any agent harness

The minimal integration, which works regardless of what harness you're using: include `SKILL.md` in the agent's system prompt or initial context, and give the agent a way to read individual chapters from `references/` on demand. That's the whole protocol — markdown in, agent decisions out.

If your harness has a more specific mechanism for contextual instructions (rules files, skills directories, conventions docs, MCP skill servers, etc.), drop the repo there. The format was designed so that an agent reading `SKILL.md` first sees the map (the chapter table), then loads only the chapters relevant to the task — progressive disclosure works the same way in any harness that lets the agent read files.

Clone the repo:

```bash
git clone https://github.com/hchen13/harness-engineering-skill.git
```

Then point your harness at it. A few common targets:

- **Claude Code** — `.claude/skills/harness-engineering/` (per-project) or `~/.claude/skills/harness-engineering/` (global)
- **Any harness with a rules/conventions/skills directory** (Cursor, Codex, Cline, Aider, Continue, custom loops, …) — drop the repo into that directory, or point your harness's config at `SKILL.md`
- **Your own agent loop** — load `SKILL.md` into the system prompt; expose a `read_file` tool over `references/`

### A note on the frontmatter

`SKILL.md` carries a YAML frontmatter block. The fields `name`, `description`, and the body content are portable. The fields `allowed-tools` and `argument-hint` are Claude Code conventions and are simply ignored by other harnesses — no need to strip them.

## Where this came from

Distilled from public material by Anthropic and others on building production LLM agents. See [`references/sources.md`](references/sources.md).

## License

[MIT](LICENSE).
