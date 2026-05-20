# harness-engineering-skill

A portable, framework-agnostic skill on **harness engineering** — the discipline of building the scaffolding around an LLM that turns it into an agent: the loop, the tools, the context, the plans, the subagents, the budgets, the guardrails, and the recovery paths.

It is deliberately not tied to any one provider's API or framework. It shapes *how* a harness is built, not *which* concrete tool an agent must call. You can use it inside any agent harness that has a mechanism for loading contextual instructions — Claude Code, Cursor, Codex, Cline, Aider, your own custom loop — or just read it.

The skill itself lives in [`SKILL.md`](SKILL.md), with chapters under [`references/`](references/).

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

Source material is listed in [`references/sources.md`](references/sources.md).

## How to use it

The body of `SKILL.md` and every chapter is plain markdown. The only Claude-Code-specific bits are two frontmatter fields (`allowed-tools`, `argument-hint`) which other runtimes simply ignore. So:

### In any harness with a skill / rules / instruction-loading mechanism

If your agent harness can load a markdown file as contextual guidance (Cursor rules, Codex skills, Cline `.clinerules`, Aider conventions, a system-prompt injection in your own loop, an MCP "skill" server, etc.), point it at `SKILL.md`. The chapters under `references/` are designed to be loaded *on demand* by the agent when it's working on that specific layer — if your harness supports progressive disclosure, expose them; if not, `SKILL.md` alone is still useful as a map and stance document.

A minimal integration: feed `SKILL.md` as part of the agent's system prompt, and let it `read` chapters from `references/` when it needs them.

### As a Claude Code skill

Drop the skill into one of the directories Claude Code scans:

**Per-project** (versioned alongside your agent code):

```bash
mkdir -p .claude/skills
git clone https://github.com/hchen13/harness-engineering-skill.git .claude/skills/harness-engineering
```

**Global** (every project on your machine):

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/hchen13/harness-engineering-skill.git ~/.claude/skills/harness-engineering
```

Restart Claude Code (or run `/skills`) and `harness-engineering` will appear in the available-skills list. Claude will load `SKILL.md` (the map) when relevant and pull individual chapters on demand.

### Just read it

It's also a short course on building agent harnesses well. You don't need an agent to get value out of it.

## Where this came from

Distilled from public material by Anthropic and others on building production LLM agents. See [`references/sources.md`](references/sources.md) for the primary sources.

## License

[MIT](LICENSE).
