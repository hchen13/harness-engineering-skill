# 5. Planning and Task Decomposition

A plan is a tool for tracking and steering work. It is useful exactly as long
as it stays subordinate to the goal — and harmful the moment it becomes a
script the agent follows off a cliff.

## A plan is mutable state, not a script

Treat a plan as visible, mutable execution state:

- it is **created** from an initial understanding of the task;
- items move through execution **one at a time**;
- it is **reshaped** when understanding changes — reality outranks the plan;
- meaningful changes carry a short **reason**, so the trail is auditable.

A plan helps the model hold a long task together and lets a user see progress.
It must not prevent a useful result when the world turns out messier than the
first guess. If the plan and the territory disagree, the plan is what is wrong.

## Two decomposition shapes

- **Fixed pipeline** — a predetermined sequence of steps, each feeding the
  next. Reliable and predictable; cannot adapt to a surprising finding. Right
  for well-understood, repeatable tasks.
- **Adaptive decomposition** — the agent generates the next subtask from what
  it just learned; the plan evolves as it goes. Flexible; less predictable.
  Right for open-ended work whose shape is unknown up front.

The choice is driven by **ambiguity, not difficulty**. A hard but
well-specified task — a precise bug with a clear reproduction — wants direct
execution. An easy-sounding task with three plausible interpretations wants an
exploration-and-plan phase first. Difficulty is about effort; ambiguity is
about whether you yet know what to do.

## Explore, then plan, then execute

For a task that is ambiguous or spans many files, separate the phases:

1. **Explore** — read, search, understand. Change nothing.
2. **Plan** — commit to an approach while the picture is fresh.
3. **Execute** — apply the plan.

This is not plan *or* execute; for cross-cutting work it is plan *then*
execute. For a small, unambiguous change, skip straight to execution — a
planning phase that adds no information is just overhead.

## Recitation

On a long task, the original goal scrolls out of the model's recent attention.
Periodically re-stating the plan — keeping a short, current to-do list the
agent rewrites as it works — pulls the goal back into the high-attention
region of the context. This exploits a durable fact about attention
(chapter 3).

One caution, as an illustration of this whole course's thesis: making the
agent narrate its plan in prose has, for some coding agents on recent models,
flipped from best practice to actively harmful — verbose plan narration can
make the model stop early. *Recitation as compact state* is durable;
*narration as a prompted ritual* is model-generation-sensitive. Re-test it.

## Attention dilution

Asking an agent to process many items in a single pass — review thirty files,
analyze twenty findings — gives the early items full attention and the later
ones a shallow pass. The tell-tale signs: thorough analysis up front and
superficial analysis later, the same issue flagged in one item and missed in
another.

This is an architectural problem, not a model-quality problem, and a better
prompt does not fix it. The fix is a **multi-pass** structure:

- a **local pass** per item, each with the full attention budget;
- a separate **integration pass** for relationships and cross-cutting
  concerns.

A larger context window does not solve this — the issue is attention
distribution, not space.

## Plan guards prevent abandonment, not adaptation

If the harness enforces anything about plans, enforce that work is not
*silently dropped* — not that the plan is followed literally.

A good plan guard asks: *is every open item either completed, or explicitly
resolved with a reason and its impact recorded?* A bad guard asks only whether
every item is literally marked done — which either traps the agent when an
item is genuinely impossible, or teaches it to mark things done that are not.

Let an item close because it was completed, or because the agent made a
justified, auditable decision that it cannot or should not be pursued now. The
final output should then reflect any unresolved items and how they affect the
result.

## Keep the plan surface small

One simple "update the plan" affordance — create, advance an item, reshape,
with a reason — covers almost all of this. Reach for specialized plan tooling,
lifecycle taxonomies, and elaborate resolution states only when a *measured*
need shows the simple surface failing. A heavy plan-state machine is itself a
capability-gap bet; do not pay for it before the evidence does.
