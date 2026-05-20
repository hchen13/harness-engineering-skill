# 10. Evaluation and Iteration

A harness is not finished when it runs. It is finished when you can *measure*
that it works, and when you have a loop for keeping it matched to the model as
the model changes. This chapter is how the rest of the course is operated.

## The harness is what you are actually measuring

A benchmark score attached to a model name is really a *model-plus-harness*
score, and the harness frequently dominates the variance — the same model on a
better harness solves dramatically more, and a weaker model on a better
harness routinely beats a stronger model on a worse one.

Two consequences:

- Harness work is high-leverage. Improving the scaffold is often cheaper and
  more reliable than waiting for a better model.
- You cannot tell whether a harness change helped without measuring it. "It
  seems better" is not a signal. An eval is.

## Build evals, even rough ones

There is no universal, agreed-upon framework for evaluating agent quality —
the measurement problem currently runs ahead of the tooling. That is not a
reason to skip evaluation; it is a reason to build your own, however modest.

- **Evaluate the end state, not the process.** For an agent that changes the
  world, judge whether the correct final state was reached — not whether the
  agent followed a particular sequence of steps. There are many valid paths;
  pinning the eval to one of them measures conformity, not success.
- **Use model-as-judge where rules cannot reach.** For open-ended quality, a
  separate model call scoring the output against an *explicit written rubric*
  is workable. Keep the rubric concrete (named criteria, not "is it good"),
  and remember it is the least robust check you have (chapter 2).
- **Keep humans in the loop.** People testing an agent find edge cases evals
  miss. Human review is not a temporary crutch — it is a permanent part of the
  measurement story.
- **A small eval set beats none.** Even a handful of representative tasks, run
  on every harness change, catches regressions that ad-hoc testing will not.

## Observability: you cannot debug what you cannot see

Agents are non-deterministic — the same input produces different runs — so a
harness you cannot inspect is one you cannot debug. Trace every run: the tool
calls, the results, the plan changes, the budget consumption, the subagent
exchanges.

Evaluate the **trajectory**, not only the final response. A run can end with a
clean-looking answer while something went wrong in the middle — a wrong turn
it recovered from, an out-of-scope action, a side effect that should not have
happened. Judging only the final message would score that run a success.
Observability is durable infrastructure: it addresses a fact about the world
(non-determinism), not a fact about the model.

## Iterate from transcripts

Improve the harness from what the agent *actually did*, not from what you
imagine it does. Read the transcripts of failed runs. Find the specific point
where the run went wrong — the misleading tool description, the missing
context, the prompt line being misread — and fix that. A capable model can do
much of this diagnosis itself: given its own failed trace, it can often
identify the harness flaw and propose the fix.

Patch surgically. A failed run usually has one or two root causes, not ten;
resist rewriting the whole prompt or restructuring the whole loop in response
to a single failure.

## The add / measure / remove loop

This is how the course's central thesis is operated in practice — the routine
that keeps a harness matched to the current model.

**Adding a module**

1. Observe a real failure — in an eval or a transcript, not a hypothetical.
2. Add the smallest module that fixes it.
3. Confirm with the eval that it actually fixed it.
4. Tag the module with the **capability assumption** it encodes ("this model
   mishandles X") and a **re-test trigger** ("re-run eval Y on model
   upgrade").

**Removing a module**

1. On a model upgrade — or periodically — take a module tagged as a
   capability-gap patch.
2. Disable it and re-run its eval.
3. If the eval still passes, the model has outgrown the module. Delete it.
4. If it still fails, the module is still earning its place. Keep it, and note
   the date.

Without this loop a harness only accumulates. Capability-gap modules pile up,
each quietly constraining the agent to an older model's behavior, until the
harness is a cage. The loop is what makes minimalism real: not a one-time
cleanup, but a standing process.

## Review checklist

Before approving a harness design or change, ask:

1. **Layer.** Is the change in the right layer — skill, harness, tool, policy,
   UI — and not, say, methodology hardcoded as a tool sequence?
2. **Assumption.** What does this module assume the model cannot do? Is that
   still true of the current model?
3. **Durable or temporary.** Does it address a fact about the world (keep) or
   a fact about the current model (tag and schedule a re-test)?
4. **Agency.** Does it give the model room with better state and affordances,
   or does it just block behavior?
5. **Completion.** Is loop completion driven by a structured signal, not by
   parsing prose?
6. **Verification.** Can the agent verify its work? Is the strongest available
   form of feedback being used?
7. **Caps.** Are budgets and caps recovery boundaries that produce useful
   partial output — not silent failures?
8. **Errors.** Are tool errors typed for recovery? Is an access failure
   distinguished from a valid empty result?
9. **Adaptation.** Can the agent change the plan when reality changes, with an
   auditable reason?
10. **Determinism.** Are deterministic gates reserved for high-stakes,
    irreversible actions — and present at *all* of them?
11. **Context.** Is the context the smallest high-signal set? Is durable state
    externalized? Is provenance preserved through compaction and synthesis?
12. **Measurability.** Can you tell whether this change helped? Is there an
    eval or a trace that would show it?

A change that cannot answer 1–3 has not been thought through. A change that
fails 5, 7, 8, or 10 is a correctness bug, not a matter of preference.
