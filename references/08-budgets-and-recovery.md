# 8. Budgets and Recovery

An agent loop can run too long, spend too much, or wander. Budgets bound it,
and recovery decides what happens when a step — or the whole run — goes wrong.
Both are harness state, and both must fail *gracefully*, never silently.

## Budgets are harness state

A budget is a limit the harness tracks and acts on — tokens, tool calls,
wall-clock time, money, iterations. Use at least two thresholds, because they
trigger different behavior:

- **Soft budget** — the agent should stop *broadening*. Stop opening new
  threads of investigation, consolidate what it has, and decide: finish now,
  ask the user, or continue along one narrow line.
- **Hard budget** — the agent should stop *new substantive work* and move into
  a finalization turn.

A turn cap (chapter 2) is the hardest budget of all — the loop's safety net.
None of these is the *intended* way a task ends; they are how a task ends
badly less badly.

## Caps trigger recovery, not silent failure

The cardinal rule:

> A budget cap must never produce an empty result. Reaching a cap moves the
> agent into a checkpoint or finalization turn — it does not abort the run.

When a cap is hit, the agent should produce a user-visible state:

- what it was trying to do;
- what it found — the partial result, stated plainly;
- what is still missing;
- how the missing piece affects confidence in the result;
- the best answer or safest action available *given* what is known;
- what a continuation would specifically try next.

A run that hits a budget and returns nothing has wasted the whole budget. A
run that hits a budget and returns honest partial work with a clear next step
has not.

## Stopping criteria belong in the agent's view too

Budgets enforced only by the harness are blunt. The agent should also know,
in-context, when *it* should stop — and capable models stop well when told
how. Give it explicit early-stop criteria rather than only a hard wall: stop
when the next action is clear, stop when independent findings converge, take
one more focused pass if signals conflict and then proceed.

This is a recent inversion worth internalizing. Older guidance pushed agents
to gather exhaustively. Capable models now *over*-gather by default — they
call one more tool when they already have the answer. The useful instruction
today is more often "stop" than "be thorough." Re-test which one your model
needs.

## Error compounding is the central operational risk

Agents are long chains of dependent steps, so a small early error does not
stay small — it sends every later step down a wrong trajectory. And restarts
are expensive: re-running a long agent from the top discards everything it
correctly did.

So design for **resumption from where the agent was**, not restart from zero:

- have the agent periodically write a **state manifest** — current phase, what
  has been explored, key findings, next step, open questions;
- on resume, the agent re-reads that manifest and the durable state
  (chapter 3) instead of redoing the work;
- on resume, run a quick baseline check before continuing — if the world is
  now in a broken state, detect it before building on it;
- keep a recovery point (version-control history is a natural one) so a bad
  change can be reverted to a known-good state.

## Partial failure should not kill the run

When one step or one subagent fails, the run should not collapse. The failing
component reports what happened in structured form (chapter 4): the failure
category, what was attempted, any partial results, a suggested alternative.
The coordinator (chapter 6) then assesses the damage and continues — with the
remaining results, or a targeted retry of just the failed part.

Two failure modes to design against explicitly:

- **Silent suppression** — a failed step that returns an empty success. The
  output looks complete and the gap is invisible. This is the most dangerous
  outcome, worse than a loud failure.
- **Total termination** — one failed step discarding the good work of every
  other step.

The correct middle path is **annotated partial completion**: the result is
delivered, with the gaps and their cause stated openly ("this section is
incomplete because the source was unreachable"). A visible gap can be acted
on; a hidden one cannot.

## What is durable here

All of it. Finite budgets, error compounding, crashes and interruptions,
expensive restarts — these are facts about running real software, not facts
about the model. Recovery and graceful degradation do not become unnecessary
as models improve. A more capable model attempts longer and more valuable
runs, which makes losing one *more* costly. This chapter only gets more
important over time.
