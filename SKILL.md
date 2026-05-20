---
name: harness-engineering
description: >
  Apply current harness-engineering best practices when designing, reviewing,
  or implementing the scaffolding around an LLM agent — agent loop, tools,
  context, planning, subagents, guardrails, budgets, recovery, skills, and
  evaluation. Framework- and API-agnostic. Load before changing an agent
  runtime or proposing harness architecture.
# allowed-tools / argument-hint are read by Claude Code; other runtimes ignore them.
allowed-tools: Read, Grep, Glob
argument-hint: "[harness design, review, or code area]"
---

# Harness Engineering

A harness is everything around the model that turns a language model into an
agent: the loop that runs it, the tools it can call, the context it sees, the
plans it keeps, the subagents it spawns, the budgets and guardrails that bound
it, and the recovery paths when something fails.

The harness is not the model, and it is not any particular API or framework.
The same harness concerns apply whether the model is reached through one
provider's request schema or another's, and whether the loop is hand-written
or run by an agent framework. Advice tied to one API's fields does not
survive a provider change; advice about loop shape and state management does.

This skill is a short course on building that harness well. It is
deliberately portable — it shapes *how* a harness is built, not *which*
concrete tool an agent must call. Read it before designing an agent runtime,
reviewing a harness change, or deciding whether to add or remove a piece of
agent scaffolding.

## The core idea

Every harness module encodes an assumption that the model cannot do something
on its own. A retry wrapper assumes the model mishandles a transient failure.
A rigid plan template assumes the model will not decompose well. A verbose
instruction block assumes the model needs to be told. Each assumption is a bet
against the model — and model capability keeps improving, so the bets keep
expiring.

When a model upgrade closes a capability gap, the module built to patch that
gap is no longer neutral. It is dead weight, and worse, it *masks* the
upgrade: scaffolding that constrains the model to its old behavior prevents
the new capability from ever showing. Removing stale scaffold is therefore not
housekeeping — it is how you collect the value of a better model.

But this does not mean "delete the harness." The capability ceiling *moves*;
it does not shrink. A stronger model attempts harder tasks, and harder tasks
have their own failure modes. The harness surface area stays roughly constant;
its *composition* churns. The discipline is continuous garbage collection, not
demolition.

The test for what to keep:

- **Durable** modules address facts about the *world* — irreversibility,
  finite context, processes that crash, adversarial input, the need to
  measure. They never expire. Keep them regardless of model capability.
- **Temporary** modules address facts about the *current model* — a specific
  capability gap. They expire. Tag each one with the assumption it encodes and
  a trigger to re-test it; delete it when a model upgrade makes it redundant.

Minimalism here is a maintenance discipline, not a starting architecture. You
do not preemptively strip structure, and you do not preemptively scaffold. You
add a module when an observed failure demands it, and you remove it when an
upgrade makes it redundant.

## Default stance

- Give the model room to solve the task. Make state, evidence, tool effects,
  budgets, and recovery paths *explicit* rather than plugging every hole with
  another rule.
- Start with the simplest loop that could work on a capable model. Add
  structure only in response to a *measured* failure.
- Reserve deterministic enforcement for places where a single probabilistic
  failure is unacceptable. Everywhere else, prefer better affordances.
- Keep the layers separate: skill (methodology), harness (runtime control),
  tool (capability), policy (deterministic gates), UI (rendering).
- Treat every module as provisional. Know which assumption it rests on.

## How to use this course

`SKILL.md` is the map. The chapters live in `references/`; load a chapter when
you are working on that part of the harness. Each is self-contained.

| Chapter | Load when you are working on… |
|---|---|
| [`01-the-harness-and-the-model.md`](references/01-the-harness-and-the-model.md) | the overall stance — what to add, what to remove, layer boundaries, where minimalism is wrong |
| [`02-the-agent-loop.md`](references/02-the-agent-loop.md) | the loop itself — iteration, completion signals, verification |
| [`03-context-engineering.md`](references/03-context-engineering.md) | what the model sees — context curation, compaction, external state, provenance |
| [`04-tools-and-the-aci.md`](references/04-tools-and-the-aci.md) | tool surfaces — schemas, descriptions, errors, scoping, tools-as-code |
| [`05-planning-and-decomposition.md`](references/05-planning-and-decomposition.md) | plans and task breakdown — plan state, decomposition shape, attention |
| [`06-subagents-and-orchestration.md`](references/06-subagents-and-orchestration.md) | multiple agents — when to split, orchestration, context isolation |
| [`07-determinism-guardrails-and-safety.md`](references/07-determinism-guardrails-and-safety.md) | deterministic control — gates, guardrails, irreversible actions, escalation |
| [`08-budgets-and-recovery.md`](references/08-budgets-and-recovery.md) | bounding work — budgets, caps, error recovery, resumption |
| [`09-skills-and-instructions.md`](references/09-skills-and-instructions.md) | instructions and skills — portable methodology, progressive disclosure |
| [`10-evaluation-and-iteration.md`](references/10-evaluation-and-iteration.md) | knowing the harness is good — evals, observability, the add/measure/remove loop, review checklist |

[`references/sources.md`](references/sources.md) lists the primary material
this course is built from.

When reviewing or changing a harness, start with chapter 1 (the stance) and
chapter 10 (the review checklist), then load the chapters specific to the
layer you are touching.
