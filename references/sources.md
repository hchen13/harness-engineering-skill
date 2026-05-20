# Sources and Further Reading

This course synthesizes harness-engineering practice as of 2025–2026. The
principles are framework- and API-agnostic; the sources below are where they
were argued in detail.

A standing caveat, and the reason this course exists: material on agent
engineering ages fast. Advice from 2024 — and some from 2025 — assumed model
limitations that no longer hold. Read any source, including this one, by
asking which of its claims are about the *world* and which are about a *model
that has since improved*.

## Foundational

- Rich Sutton, "The Bitter Lesson" (2019) — general methods that scale with
  compute beat hand-built structure over time.
- Anthropic, "Building Effective Agents" (2024) — the workflow-vs-agent
  distinction; simplicity as the default; invest in the agent-computer
  interface.

## Context engineering

- Anthropic, "Effective Context Engineering for AI Agents" (2025) — context as
  a finite resource; "the smallest set of high-signal tokens"; just-in-time
  retrieval.
- Anthropic, "Effective Harnesses for Long-Running Agents" (2025) —
  externalized state; the "shifts" model of multi-window work.
- Manus, "Context Engineering for AI Agents: Lessons from Building Manus"
  (2025) — cache-aware design, recitation, keeping failed attempts in context.

## Tools and orchestration

- Anthropic, "Writing Effective Tools for Agents" (2025) — tools as a
  contract; consolidated high-leverage tools; descriptions as routing logic.
- Anthropic, "Code Execution with MCP" (2025) — the tools-as-code pattern.
- Anthropic, "How We Built Our Multi-Agent Research System" (2025) — when
  multi-agent pays off, and its token economics.
- OpenAI, "A Practical Guide to Building Agents" (2025) — single-agent-first;
  the run loop; guardrails as a layered defense; tool risk rating.
- Cognition, "Don't Build Multi-Agents" (2025) and "Multi-Agents: What's
  Actually Working" (2026) — the parallel-writes problem, and how that view
  itself updated.

## The harness as a discipline

- OpenAI, "Harness Engineering" (2026) and related interviews — scaffolding to
  be removed over time; capability overhang.
- Addy Osmani, "Agent Harness Engineering" (2026) — every module encodes a
  capability assumption; the ceiling moves rather than shrinks.
- The Claude Certified Architect curriculum — agentic loops, tool design,
  deterministic enforcement, context management, evaluation.

## How to use this list

These are starting points, not authorities to defer to. The durable content
of this course is the reasoning, not the citations. When a new model ships,
the right response is not to re-read the sources — it is to re-run the evals
(chapter 10) and see which modules the new model has made unnecessary.
