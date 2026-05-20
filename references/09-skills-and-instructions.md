# 9. Skills and Instructions

Instructions are how a harness carries knowledge to the agent: how to do a
task, what conventions to follow, what a good result looks like. The design
question is *what* knowledge, loaded *when*.

## Two kinds of instruction

- **Always-on instructions** — loaded into every run: broad conventions,
  durable standards, the things true regardless of the task.
- **On-demand skills** — loaded only when a task needs them: a specific
  methodology, a specialized workflow, domain procedure.

The split matters because context is finite (chapter 3). Putting
task-specific procedure into always-on context spends attention budget on
every unrelated run. Putting universal standards into a skill means they are
absent when they are needed. Match each piece of knowledge to how often it is
actually relevant.

## Always-on context must stay lean

The failure mode of an always-on instruction file is bloat. Past a certain
size it stops being read carefully — an over-stuffed instruction set causes
the model to *ignore* instructions, including the important ones. More
instruction is not more compliance.

The pruning test for every line:

> "Would removing this line cause the agent to make a mistake?" If not, cut it.

If a piece of guidance is relevant only to one kind of task, it does not
belong in always-on context. It belongs in a skill, or in a rule scoped to
load only when that kind of work is happening.

## Progressive disclosure

A skill should reveal itself in layers, so its cost is paid only when it is
used:

1. **Name and one-line description** — always visible, cheap. Enough for the
   agent to know the skill exists and when it applies.
2. **The skill body** — loaded when the agent judges the skill relevant. The
   core methodology.
3. **Bundled references** — deeper material (chapters, examples, scripts)
   loaded only when a specific part is needed.

This keeps the always-visible footprint tiny while supporting arbitrarily deep
material underneath. This very skill is built that way: a short `SKILL.md` and
ten chapters loaded on demand.

## A skill is methodology, not a tool script

The most important rule for a portable skill:

> A skill says how to *think* about a task. It does not say which exact tool
> to call.

A good skill describes: what evidence matters, how to tell fact from inference
from speculation, how to reason when evidence is missing, how to express
uncertainty, how to handle opposing views, what a useful deliverable contains.

A skill should *not* contain: "call this exact tool," "use this JSON shape,"
"query this endpoint," "follow this fixed tool sequence." Those bind the
methodology to one runtime. The day the runtime changes, the skill is wrong —
and a skill that hardcodes a tool sequence also suppresses the model's own
judgment about how to satisfy an evidence need.

If a skill must connect to concrete tools, that mapping belongs in a harness
adapter or in the tool descriptions (chapter 4), not in the skill body. The
skill stays portable; the harness binds it to the runtime.

"Like an onboarding guide for a new hire" is the right mental model: you teach
a new colleague how the work is done and how to think about it, and trust them
to operate the specific tools.

## Skills can carry deterministic code

Methodology is not the only thing a skill can bundle. A skill can also ship
small scripts for the deterministic sub-steps of its workflow — parsing,
sorting, a fixed transformation. Letting the agent run a bundled script for
those steps is more reliable than having it do the work token by token, and it
keeps that work out of the context window. Use the model for judgment; use
code for the mechanical parts.

## Why this chapter is durable

Progressive disclosure, the on-demand/always-on split, methodology-not-scripts
— these rest on facts about the world: context is finite, and a methodology
outlives any particular toolset. They do not expire as models improve. What a
more capable model changes is the *altitude* of the instructions — a stronger
model needs less hand-holding and a more methodology-shaped, less
procedure-shaped skill (chapter 3). The structure stays; the verbosity drops.
