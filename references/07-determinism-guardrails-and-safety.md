# 7. Determinism, Guardrails, and Safety

A language model is probabilistic. Most of the time that is the point — it is
why the agent can handle situations you did not script. But some things must
hold *every* time, and "most of the time" is not a property you can prompt
your way to.

## The decision rule

> Prompts and instructions are probabilistic — they work most of the time.
> Code, hooks, gates, and schemas are deterministic — they work every time.
> The choice between them is not about whether the prompt is good enough. It
> is about whether the consequence of a *single* failure justifies a
> deterministic guarantee.

Use deterministic enforcement when one failure is unacceptable:

- financial side effects, payments, transfers;
- irreversible or destructive operations;
- security and access-control boundaries;
- credential or secret exposure;
- compliance and regulatory requirements;
- a user-visible commitment the system must honor.

Use prompt-level guidance for everything low-stakes — formatting, style,
ordering preferences, tone. There a rare deviation costs little, and the
flexibility is worth more than the guarantee.

This is the same minimalism discipline as the rest of the course, applied to
control: do not wrap the whole agent in deterministic rails, and do not leave
a high-stakes boundary to a prompt.

## Mechanisms

- **Hooks / interception** — code that runs at a fixed point in the loop and
  can block, modify, or redirect an action. The mechanism for injecting
  deterministic behavior into an otherwise probabilistic system.
- **Prerequisite gates** — a tool stays unavailable until its preconditions
  are met (a destructive action gated on a confirmation; a write gated on a
  prior validation). The gate returns an error that forces correct ordering.
- **Schemas** — constrain the *shape* of an output or tool call.
- **Permissions / scoped access** — the agent simply cannot reach what it is
  not allowed to reach.

Timing is part of correctness: a gate must fire **before** the action. A check
that runs *after* an irreversible effect has already happened has verified
nothing it can act on.

## Guardrails are a separate, layered defense

Keep guardrails *outside* the agent's own instructions. A rule that lives in
the system prompt is something the agent can be argued, confused, or injected
out of. A guardrail the agent cannot see and cannot route around is a real
boundary.

Design properties that hold across runtimes:

- **Layered.** No single guardrail is sufficient; combine several — input
  validation, an output check, a pre-action gate.
- **Cheap to run.** A guardrail can be a fast small-model classifier or plain
  code; it should not cost as much as the action it protects.
- **Fail closed at high stakes.** When a guardrail trips, the protected action
  does not happen.
- **Necessary but not sufficient.** Guardrails sit *on top of* real
  engineering security — authentication, authorization, sandboxing. They do
  not replace it.

## Rate tools by risk

Make the risk explicit. Rate each tool — low / medium / high — by
reversibility, read versus write access, the permissions it requires, and its
financial or external impact. Then let the rating drive harness behavior:
low-risk tools run freely; high-risk tools trigger a gate, a confirmation, or
human escalation. A risk-rated toolset turns "be careful" into something the
harness enforces rather than something the model is asked to remember.

## A schema guarantees shape, not correctness

Structured output through a schema eliminates *malformed structure* — missing
fields, bad types, invalid syntax. It does nothing about *semantic* errors: a
value in the wrong field, a total that does not add up, a confidently
fabricated number. The shape is right and the content is wrong.

So schema validation is necessary but never the whole story. Semantic
correctness needs actual checks — verification code, cross-field consistency
rules, a review pass (chapters 2 and 10). Schema design can help: make a field
nullable so the model can honestly decline instead of inventing a value; add
an explicit "uncertain" value; capture a stated total *and* a computed total
so a mismatch is detectable.

## Human escalation

Some situations should leave the agent. Escalate on:

- an **explicit human request** — honor it immediately, without first
  "trying anyway";
- a **policy gap** — the rules are silent on this situation (distinct from a
  policy *violation*, where the rules give a clear answer the agent can
  apply);
- a **genuine inability to make progress** — after a real attempt, with
  evidence of what failed.

Do not escalate on signals that do not correlate with needing a human. User
sentiment ("the user seems annoyed") is not task complexity, and a model's
self-reported confidence is poorly calibrated — it is often confident and
wrong. Neither is a sound trigger.

When escalating, hand off a **self-contained** summary. The human picking it
up cannot see the agent's transcript. The handoff must carry the situation,
the relevant facts and figures, what the agent found, and a recommended
action — everything needed to act without reconstructing the run.

## Least privilege

Prefer the narrowest tool that does the job. A scoped `load_document` that
validates its input beats a generic `fetch_url`; a read-only role gets no
write tool. Narrow tools reduce the blast radius of a wrong call and make the
agent's intent legible. Most safety is bought here, quietly, before any
guardrail fires.
