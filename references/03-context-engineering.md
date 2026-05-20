# 3. Context Engineering

Prompt engineering — finding the right words for one prompt — is now a subset
of a larger discipline:

> **Context engineering** is curating and maintaining the optimal set of
> tokens the model sees during inference — the system prompt, tools, message
> history, tool results, retrieved data, external state — across the whole
> run, not one turn.

The question is no longer "what is the best phrasing" but "what configuration
of context most reliably produces the behavior I want."

## Context is a finite, decaying resource

A bigger context window does not make context free. Model attention degrades
as a context fills — recall of any single fact drops as the surrounding token
count rises. Treat the context window as an **attention budget**: every token
added depletes it. The goal of context engineering:

> Find the smallest possible set of high-signal tokens that make the desired
> outcome likely.

This reframes most context work as *subtraction*. More context is not safer;
it dilutes attention and buries the facts that matter.

## System prompts: the right altitude

A system prompt fails in two opposite directions:

- **Too specific** — brittle, hardcoded if-this-then-that logic. High
  maintenance, and it breaks on the first case it did not anticipate.
- **Too vague** — high-level guidance that assumes shared context the model
  does not have. It gives no concrete signal.

Aim for the altitude between them: specific enough to guide behavior, general
enough to let the model apply judgment. Organize the prompt into clear
sections (role, instructions, tool guidance, output format). Start minimal on
a capable model and add only in response to an observed failure — an
instruction-dense prompt is itself a capability-gap patch, and an over-stuffed
prompt makes the model *ignore* instructions rather than follow more of them.

## Just-in-time over pre-loading

Do not pre-load everything the agent might need. Prefer to hold lightweight
references — file paths, queries, links, IDs — and let the agent load the
underlying data at runtime, when a step actually needs it.

This is why open-ended agentic exploration has largely displaced loading a
fixed pre-built index for many tasks: an agent that explores adapts to what it
finds, and the metadata it meets on the way (folder names, timestamps, naming
conventions) is itself signal. A hybrid is fine — pre-load a small, stable
core for speed; explore for the long tail.

The shift here is concrete and recent: "retrieve everything up front into the
prompt" was correct when models explored poorly. Capable models explore well.
Re-test that assumption.

## Long-horizon work: externalize state

A long task does not fit in one context window, and a bigger window does not
fix it — context degrades before it fills. The durable solution is to move
state *out* of the conversation onto a substrate the agent can re-read: the
file system.

Treat a task spanning many context windows like a project staffed in shifts:
each shift starts with no memory of the last, so the state must live somewhere
both can read. Useful externalized state:

- a **progress log** — what is done, what is next;
- a **structured task list** the agent updates as it works;
- **scratchpad / research notes** the agent writes findings into and re-reads;
- version-control history as an audit trail and a recovery point.

The agent should re-read the durable note rather than trust its memory of tool
output from many turns ago. A persisted file is also more inspectable and
recoverable than context that has scrolled away.

## Compaction, and what it must not lose

When history approaches the window limit, **compaction** summarizes it and
re-initializes the loop from the summary. It is necessary for long runs, but
the hard part is choosing what to keep versus discard. Tune for *recall*
first — keep everything that might matter — then tighten precision. The light
form of compaction is clearing stale tool results that are no longer needed.

What compaction must never silently drop:

- transactional and evidentiary facts — amounts, dates, identifiers, quotes,
  exact figures;
- user-stated constraints and decisions;
- the provenance of any claim that will appear in the output.

Progressive summarization is destructive by default — it smooths away exactly
the precise values it should preserve. Keep those facts in a structured block
carried verbatim across compaction and **never compressed**. Do not rely on a
summary for anything you would be wrong to paraphrase.

## Fight lost-in-the-middle structurally

Models weight information at the *boundaries* of the context — the start and
the end — more reliably than the middle. A finding buried mid-context gets
underweighted.

The fix is structural, not a prompt plea. Put a short "key findings" or
"current state" section at the top of long content, with detail under clear
headers below. Do not bury the load-bearing fact in paragraph nine.

## Tool results are a silent context cost

Verbose tool output — forty fields when five were needed, full reasoning
chains, raw dumps — consumes attention budget on *every subsequent turn*, not
just the turn it arrives. Trim results before they enter history: return
structured data scoped to what the next step needs. This is essential for any
multi-turn agent, not an optional optimization. (Tool-side controls for this
are in chapter 4.)

## Provenance: keep evidence attributable

For any agent that synthesizes from sources, a claim without its source is not
usable evidence. Carry, with each finding, a structured mapping: the claim,
the source, the relevant excerpt, and the date.

Two rules:

- **Attribution dies during summarization** unless it is explicitly
  preserved. Every compaction and every synthesis step must carry the
  claim-source mapping through, or the final output becomes plausible and
  uncitable.
- **Do not average away conflicts.** When sources disagree, presenting one
  value as fact destroys information. Surface both with their dates — and
  check whether the "conflict" is actually a *trend* over time, which only the
  dates reveal.

## What ages out, what stays

- **Ages out:** prompting the model to be exhaustively "thorough"; aggressive
  pre-retrieval; heavy instruction density. These patch model behavior that
  capable models have outgrown — re-test them on every upgrade.
- **Stays:** context is finite and attention decays; transactional facts need
  structured persistence; state for long tasks belongs on disk; provenance
  must be carried. These are facts about the world.
