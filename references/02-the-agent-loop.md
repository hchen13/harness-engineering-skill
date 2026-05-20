# 2. The Agent Loop

The loop is the irreducible core of an agent. Long-horizon success depends
less on one well-written prompt and more on the loop the model operates
inside.

## The shape of the loop

A capable agent runs one feedback cycle:

> **gather context → take action → verify work → repeat**

- **Gather context** — pull in what the next decision needs: read files,
  search, inspect the environment.
- **Take action** — call tools, run code, produce output.
- **Verify work** — check the action against ground truth.
- **Repeat** — feed the result back, then continue or stop.

Every iteration must gain *ground truth from the environment* — a real tool
result, a test outcome, a file that now exists. An agent that plans several
steps ahead without observing the environment between them compounds error.
Keep the cycle tight.

## Completion is a typed signal, not text

The hardest loop bug is deciding when the agent is done. The rule:

> Loop control is driven by structured runtime signals, never by parsing the
> model's prose.

Runtimes expose a structured completion signal — a stop-reason field, a
finish or turn event, a final-output tool call, a turn with no tool calls.
Whatever your runtime exposes, *that* is the completion signal. Three
anti-patterns, all unreliable:

- **Parsing natural language** ("I'm done", "let me know if…"). Prose is
  ambiguous; a model says "done" mid-task and keeps working after "finished."
- **Content-type checking** — inspecting whether the last block is text. A
  text block is not a completion signal.
- **An iteration cap as the primary mechanism.** A turn cap is a safety net
  (chapter 8), not the intended way a task ends.

A loop has other legitimate exit conditions: a designated final-output tool
fired, an unrecoverable error, a budget cap reached. Each is a typed condition
the harness checks — not a string it greps.

## Feed every result back

Any tool result that affects reasoning must be available to the next model
turn. An agent acting on a result it can no longer see is reasoning from
memory of the environment, not the environment.

If a result is too large to keep inline, persist it and return a stable
preview plus a reference the agent can re-open. Do not silently drop it.
(Chapter 3 covers managing result volume.)

## Verification is the highest-leverage step

Of the four beats, **verify** is the one most often missing and the one with
the largest payoff. An agent that can check and improve its own output catches
mistakes before they compound, self-corrects when it drifts, and improves as
it iterates. Giving the agent a way to verify its work is the single
highest-leverage thing a harness can provide.

The failure mode it prevents has a name — the **trust-then-verify gap**: the
model produces a plausible-looking result that does not actually hold up, and
nothing in the loop catches it.

Verification mechanisms, in descending order of robustness:

1. **Rules-based / deterministic feedback** — type-checkers, linters, tests,
   schema validators, compilers. The strongest form: the result either passes
   a defined rule or it does not, and the failure says exactly what broke.
   Prefer this whenever the domain admits it.
2. **Environmental / observational feedback** — run the thing and observe:
   execute the script, render the page, diff the output against an expected
   state.
3. **Model-as-judge** — a separate model call scores the output against an
   explicit rubric. Use it where rules cannot reach (open-ended quality), and
   know it is the least robust form and adds latency. (Chapter 10.)

A verification step is durable. It addresses a fact about the world — work can
be wrong — not a fact about the model. A stronger model uses good
verification *better*, not less.

## Retrying with the error in hand

When verification fails, a retry is useful only if the model is told what
broke. A useful retry includes three things:

- the original input,
- the failed output,
- the precise verification error.

Without the specific error the model has no signal and tends to reproduce the
same mistake.

Know the boundary: retries fix outputs that were *misplaced, malformed, or
miscomputed* — the information was there and was mishandled. Retries do not
fix *missing* information. If a required input genuinely does not exist,
retrying loops forever; the loop should instead surface the gap (chapter 8).

## What the loop should not be

- It should not decide completion by reading prose.
- It should not plan many steps deep without observing the environment.
- It should not depend on the turn cap to end a normal task.
- It should not discard a tool result that later reasoning depends on.

A loop that holds these four lines is correct for most agents. Everything else
in this course is about what flows *through* it.
