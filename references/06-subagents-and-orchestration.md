# 6. Subagents and Orchestration

Multiple agents are an escalation, not a default. Reach for a second agent
only when one well-equipped agent demonstrably cannot do the job.

## Single agent first

Maximize a single agent with good tools and good context before splitting.
More agents buy a tidy separation of concerns on a diagram, but they cost real
complexity, real tokens, and new failure modes. Most tasks — including most
coding — do not need them.

When a single agent struggles, first ask whether the problem is actually tool
overload (chapter 4) or context volume (chapter 3). Both have cheaper fixes
than a second agent.

## Why multi-agent works — and what that implies

When multi-agent systems help, they help for concrete, mechanical reasons:

- **Token budget.** Several agents spend more total tokens on the problem, in
  parallel, than one agent can spend in a single context window.
- **Parallelism.** Independent directions explored at the same time finish
  sooner.
- **Context isolation.** Each subagent works in its own clean window; the
  noise of its exploration never enters the others'.

The implication is sharp: at an *equal token budget*, a single agent is
roughly competitive with a multi-agent system on reasoning quality.
Multi-agent earns its overhead for **breadth** — parallel, independent
investigation — and for work that genuinely exceeds one context window. It is
not a general intelligence multiplier.

## The real anti-pattern is parallel writes

The danger is not "multiple agents." It is **multiple agents writing in
parallel**. Every action carries implicit decisions; two agents acting at once
make conflicting implicit decisions, and the conflict surfaces as incoherent
combined output that no coordinator can cleanly reconcile.

The safe shape:

> Keep writes single-threaded. Additional agents contribute *intelligence* —
> analysis, review, search, options — not concurrent *actions*.

A reviewer subagent, a research subagent, an agent that proposes options for
one writer to act on — all safe. Two agents editing the same artifact at once
— not.

## Hub-and-spoke orchestration

The default multi-agent structure is hub-and-spoke: one coordinator at the
center, subagents on the spokes, no spoke-to-spoke communication. The
coordinator owns decomposition, context passing, aggregation, recovery, and
final synthesis. Centralizing this buys observability (one place logs every
exchange), consistent error handling, and controlled information flow.

Peer-to-peer handoff — one agent transferring control to another — is a
legitimate alternative where routing itself is the workflow. But for work that
ends in a synthesized result, hub-and-spoke is the safer default, because it
keeps writes coordinated.

## Subagent context is isolated

A subagent starts fresh. It does not inherit the coordinator's history, its
system prompt, or any other subagent's results. Everything it needs must be in
the brief the coordinator hands it. A useful brief states:

- the **objective** — the specific question or outcome;
- the **output format** — what to return, and how;
- **tools and sources** — what to use, scoped to the role;
- **boundaries** — what is in and out of scope;
- the **effort level** — how much to do, scaled to task complexity.

Vague briefs cause the classic multi-agent failures: subagents running
duplicate searches, scope blowing up, effort wildly mismatched to the question.

## Pass structured findings, not prose

When a subagent's output feeds a downstream step, return structured findings,
not a prose blob. Each finding should carry its claim, its source or evidence
reference, a date where relevant, and a confidence or limitation. A synthesis
step can only cite what it was given — content passed without its metadata
cannot be attributed downstream (chapter 3).

## Decomposition is where breadth fails

When a multi-agent result is missing whole *categories* of content —
incomplete in **scope**, not in depth — the cause is almost always the
coordinator's decomposition, not the subagents. A subagent cannot cover a
topic it was never assigned. Adding more subagents does not fix a
decomposition bug; fixing the decomposition does. Diagnose breadth failures at
the hub.

## Context isolation is durable; orchestration cleverness is not

Subagents as **context firewalls** — isolating noisy exploration so it never
pollutes the main agent's window — is a durable pattern. It rests on a fact
about the world: context is finite. That benefit does not expire as models
improve.

Elaborate orchestration logic — rigid role hierarchies, fixed decomposition
templates, forced parallel section-writing — is the opposite. It is
capability-gap scaffolding, and capable models routinely outgrow it. Keep the
isolation; re-test the cleverness.

## Independent review

For a review or critique step, use a *separate* agent instance with no shared
session. An agent reviewing its own work within the same context retains the
reasoning that produced it and is biased toward defending it. A fresh
instance, given only the artifact, evaluates it cold and catches more. This is
a durable pattern — it exploits the value of an independent vantage point, not
a temporary model weakness.
