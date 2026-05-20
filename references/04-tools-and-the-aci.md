# 4. Tools and the Agent-Computer Interface

A tool is a contract between a deterministic system and a non-deterministic
agent. The set of tools — the agent-computer interface, or ACI — deserves the
same design investment a team puts into a human-facing UI. On hard agentic
tasks, tuning the tools often moves the result more than tuning the prompt.

## Build high-leverage tools, not endpoint wrappers

The most common tool-design mistake is exposing one tool per underlying API
endpoint or function. That pushes orchestration the agent should not be doing
into the agent, and burns context on intermediate results.

Instead, design tools around *what the agent is trying to accomplish*, and let
each tool consolidate the steps under the hood. Rather than `list_users`,
`list_events`, and `create_event`, expose a `schedule_event` tool that finds a
free slot and books it. One call, one result, less room for error.

Match tools to the agent's affordances. The agent has limited context, not
limitless memory — so a tool that returns "all records, paginate yourself" is
a poor fit. A tool that *searches* and returns the few relevant records fits
how the agent actually works.

## The tool description is routing logic

The description is the primary mechanism the model uses to choose a tool. It
is not metadata — it is the routing logic. Write it as if onboarding a
teammate. A production-grade description states five things:

1. **Purpose** — what the tool does, unambiguously.
2. **Inputs** — each parameter's type, format, constraints, and whether it is
   required. Use unambiguous parameter names.
3. **Examples** — concrete cases where this tool is the right call.
4. **Limits and edge cases** — what the tool does *not* do.
5. **Boundaries** — when to use this tool versus a neighboring one.

Small description refinements produce large routing improvements. When the
agent picks the wrong tool, fix the description before reaching for a
classifier or more tools — it is the lowest-effort, highest-leverage lever.

## Scope the toolset

Too many tools degrade selection. The metric is not the raw count but
**overlap**: a dozen well-differentiated tools can work fine; a handful with
fuzzy, overlapping purposes will not. The litmus test:

> If a human cannot say which tool should be used in a given situation, the
> agent cannot either.

Give each agent or subagent only the tools its role needs. Namespacing
(consistent prefixes) helps the model — and you — keep boundaries clear. When
selection misfires, the fix order is: sharpen descriptions → rename for
disambiguation → consolidate or split tools → only then split the agent.

## Structured errors

A tool error is an input to the agent's next decision, so it must carry
decision-relevant structure. "Operation failed" tells the agent nothing. Every
error should classify itself:

| Category | Meaning | Retry? |
|---|---|---|
| Transient | timeout, rate limit, service unavailable | yes, after a delay |
| Validation | malformed or out-of-range input | yes, after fixing the input |
| Business / policy | a rule forbids the operation | no — escalate or take another path |
| Permission | access denied | no — escalate or re-authorize |

Each error payload should state what was attempted, whether a retry is useful,
any partial results obtained, and a suggested alternative when the tool can
name one.

One distinction is worth enforcing on its own:

> An **access failure** (the tool could not reach the data) is not the same as
> a **valid empty result** (the tool ran and found nothing).

Conflating them causes two opposite bugs: retrying forever against a query
that correctly returns nothing, or treating a real outage as "no data." An
empty result is a successful answer — mark it as not-an-error, and do not
retry it.

## Token-efficient results

A tool result spends context on every later turn (chapter 3). Design results
to be economical:

- Default to a concise representation; offer a detailed mode when the agent
  asks for it.
- Support pagination, filtering, and field selection.
- Return semantically meaningful values, not opaque identifiers, where the
  agent will reason over them.
- When truncating, say so, and steer the agent toward a narrower request.

## A named, well-specified surface beats freeform output

When the agent must produce something structured — a diff, a record, a
command — a dedicated tool with a precise schema outperforms asking for the
structure in prose. The schema constrains the shape, and a model is more
reliable producing into a known surface than free-forming it. (The schema
guarantees *shape*, not *correctness* — see chapter 7.)

## Tools-as-code, for scale

When an agent has access to many tools, two costs grow: every tool's schema
sits in context whether or not it is used, and every intermediate result
passes back through the model. At a large enough scale this dominates the
token budget.

The pattern that addresses it: expose tools as a **code API** the agent
*writes code against*, instead of as direct calls. The agent imports only the
tool definitions a task needs (progressive disclosure), and loops, filtering,
and intermediate data stay in the execution environment — only the distilled
result returns to the model. The token reduction can be an order of magnitude
or more.

This is a capability-and-scale tradeoff, not a free win: it needs a secure
sandbox with resource limits and monitoring. Adopt it when tool-definition
overhead or result volume is actually hurting you — and treat it as a pattern
that tracks model and platform capability, so revisit it as both improve.

## Improve tools from transcripts

Tool design is iterative and eval-driven. Prototype, run the agent on
realistic tasks, read the *transcripts*, and refine the tools the agent
actually fumbled. A capable model can do much of this analysis itself — given
its own failed transcripts, it can often suggest the description or schema
fix. (Chapter 10.)
