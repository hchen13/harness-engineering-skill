# 1. The Harness and the Model

This chapter is the stance the rest of the course rests on: what a harness is,
why every part of it is provisional, and how to decide what to add and what to
remove.

## What a harness is

A harness is the scaffolding that turns a language model into an agent.
Concretely it includes:

- the **loop** that calls the model, executes tool calls, and decides when to
  stop;
- the **tools** and the agent-computer interface;
- **context management** — what the model sees each turn, and what is
  persisted outside the conversation;
- **planning** and task decomposition;
- **subagents** and orchestration;
- **budgets, guardrails, and permissions** that bound the agent;
- **recovery** — what happens when a step fails.

The harness is not the model. It is also not any particular API schema or
agent framework. A provider's request format and a framework's class hierarchy
are implementation surfaces; the harness concerns in this course apply
identically across all of them.

## Workflows and agents

Two things are often both called "agents":

- A **workflow** orchestrates model calls and tools through predefined code
  paths. The control flow is yours.
- An **agent** directs its own process — it decides which tools to call, and
  in what order. The control flow is the model's.

Most harness design is choosing, per concern, how much control flow to hand to
the model. The trend is toward handing over more, but the choice is always
local: a high-stakes step can stay a fixed code path inside an otherwise
model-driven agent.

## Every module encodes a capability assumption

The central principle of this course:

> Every component in a harness encodes an assumption about what the model
> cannot do on its own.

A retry-with-reformatting wrapper assumes the model mishandles a malformed
response. A step-by-step plan template assumes the model will not decompose
the task itself. A long list of "always do X" instructions assumes the model
will not do X by default. A self-critique pass assumes the model ships
first-draft mistakes.

Each assumption is a bet against the model. Model capability improves, so the
bets keep expiring. The job of harness engineering is to keep the harness
matched to the *current* model — which means removing modules about as often
as adding them.

## The scaffold lifecycle

The healthy lifecycle of a capability-gap module:

1. A failure is observed. You build scaffolding to compensate for the gap.
2. A model upgrade closes the gap.
3. You remove the scaffolding — and the agent can now use the new capability
   directly, which frees you to take on harder problems.

Step 3 is the one teams skip, and skipping it has a specific cost beyond
clutter: stale scaffolding **masks the upgrade**. A harness that constrains the
model to its old behavior — narrow tools, rigid plans, over-specified
prompts — prevents the new capability from ever expressing itself. You paid
for a better model and cannot see it. This is *capability overhang*: the gap
between what the model can do and what your harness lets it do.

So removing stale scaffold is not housekeeping. It is how you collect the
value of a model upgrade, and it is competitive.

## The ceiling moves; it does not shrink

It is tempting to conclude "better models → smaller harness → eventually no
harness." That is wrong.

When a model gets stronger, two things happen at once: old capability-gap
modules become redundant, *and* the agent can now attempt larger tasks that
were previously out of reach — tasks with their own unfamiliar failure modes.
The capability ceiling does not shrink; it migrates. Context-anxiety patches
disappear and multi-day memory policies appear in their place.

Practical consequence: the harness surface area stays roughly constant across
model generations. What changes is its *composition*. Harness engineering is
therefore continuous — not a one-time build, and not a march toward zero.

## Durable vs. temporary: the litmus test

The single most useful distinction in this course:

> A module is **durable** if it addresses a fact about the *world*. It is
> **temporary** if it addresses a fact about the *current model*.

Facts about the world do not change when the model improves:

| Durable concern | Why it never expires |
|---|---|
| Irreversibility / side-effect control | A correct answer cannot un-send an email or un-drop a table. |
| Security and scope enforcement | Sandboxing and scoped permissions bound an adversary, not a confused model. |
| Finite context | Context windows are bounded and attention degrades; isolation economics are physics, not a model flaw. |
| Crash and interruption recovery | Processes die and long tasks get paused regardless of how smart the model is. |
| Observability and tracing | A non-deterministic system you cannot inspect is one you cannot debug — at any capability level. |
| Evaluation | You cannot manage what you do not measure; true at every capability level. |
| Verification / back-pressure | Tests and type-checks let the agent self-correct; a stronger model uses them *better*, not less. |

Facts about the current model *do* change:

- The model calls tools unreliably → so you avoid tool calls or hand-parse
  function calls.
- The model over- or under-gathers context → so you prompt it to be
  "thorough" or to "stop early."
- The model decomposes tasks poorly → so you impose a rigid decomposition.
- The model needs heavy instruction → so you write an instruction-dense
  system prompt.
- The model ships first-draft errors on simple work → so you bolt on a
  self-critique pass.

Every item in the second list is a candidate for deletion on the next model
upgrade. Every item in the first list stays.

## The ratchet: add reactively, remove reactively

Do not preemptively scaffold, and do not preemptively minimize. Both are
guessing.

- **Add** a module when you *observe* a failure the model cannot handle on its
  own — backed by an eval or a real transcript, not a hypothetical.
- **Remove** a module when you *observe* that a more capable model no longer
  needs it — by re-running the eval with the module disabled.

When you add a temporary module, record two things with it: the **capability
assumption** it encodes ("this model mishandles malformed tool output") and
the **re-test trigger** ("re-run eval Y on the next model upgrade"). Untagged
scaffolding is how a harness silently rots into a cage. Chapter 10 makes this
loop concrete.

## Where minimalism is wrong

"Prefer the simplest harness" is the default, but it has real limits. Push
back on minimalism in these cases:

- **The harness is the benchmarked artifact.** A score attached to a model
  name is really a model-plus-harness score, and the harness often dominates
  the variance — a weaker model on a better harness routinely beats a stronger
  model on a worse one. "Minimal" must mean *free of capability-gap cruft*,
  never *absent*. You still invest heavily in the scaffold that converts model
  capability into reliable task completion.
- **Constrained compute or latency.** When you cannot afford the tokens or the
  wall-clock of an open-ended agentic loop, a purpose-built deterministic path
  can outperform a general one. Scaling is not always available to you.
- **Poor data or no clean reward signal.** In messy domains where "correct" is
  hard to define, you cannot just wait for a smarter model — there is no clean
  target for it to hit. Structure does real work there.
- **Structure that exploits invariants.** Some added structure tracks facts
  about the world, not the model: a stable prompt prefix for cache reuse,
  keeping failed attempts visible so the model can learn from them, re-stating
  the goal to fight attention decay. These survive model upgrades — do not
  delete them in the name of minimalism.
- **Minimalism presupposes investment elsewhere.** Replacing a large bespoke
  toolset with "the model writes code against a filesystem" only works if the
  data substrate is clean and well-organized. Simplicity in one layer is
  bought with discipline in another.

## Layer boundaries

Keep these layers separate. Mixing them is the most common structural mistake.

- **Skill** — methodology: how to think about a task, what evidence matters,
  what a good result looks like. Portable; names no concrete tool, API field,
  or runtime component. (Chapter 9.)
- **Harness** — runtime control: loop, budgets, plans, subagents, context,
  permissions, recovery. Runtime-specific; may name concrete tools.
- **Tool** — a single capability with a clear schema, crisp description,
  explicit boundaries, and structured results. No hidden policy. (Chapter 4.)
- **Policy / guardrail** — deterministic gates at high-stakes boundaries,
  separate from the agent's instructions so they cannot be reasoned around.
  (Chapter 7.)
- **UI** — renders state and evidence for the user. Never the *only* place a
  workflow invariant lives.

When changing a harness, name the layer first. A fix in the wrong layer —
methodology hardcoded as a tool sequence, a safety invariant living only in a
prompt — is a structural bug even when it appears to work.
