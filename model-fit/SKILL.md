---
name: model-fit
description: Decide which model tier — cheap/fast, balanced, or frontier — fits a task, a subagent you're about to spawn, a pipeline stage, or a batch job, using a fixed decision table rather than a fresh judgment call each time. Use this whenever picking a model for a task or subagent, asking "which model should I use for X" or "is this worth the expensive model," about to dispatch or delegate work without having chosen a model, or scoping a multi-step / long-running / agentic task before kicking it off. Also use it to set an external stop condition before any long-horizon or agentic run, and to tier each stage of a mixed pipeline separately instead of picking one model for the whole thing.
---

# Model Fit

Match the model tier to the shape of the task — no more, no less. A cheap, fast
model on mechanical work is not cutting corners; it's the correct call. A
frontier model on the same work is waste, not rigor. The reverse mistake is
just as common: using the cheap tier on a judgment call, or trusting the
frontier tier to keep a long agentic task in check by itself. It won't — see
the horizon axis below.

## Why a lookup table, not a judgment call

Don't ask an LLM to decide which LLM to use. That decision costs a call
in itself, and it only earns that cost back if a fixed lookup would have
gotten the answer wrong often enough to matter. In practice, task shape
predicts the right tier well enough that a table beats a fresh judgment call
almost every time — it's instant, free, and (this matters more than it
sounds) auditable: anyone can read the one-line reason and check it, where an
adaptive judgment call is just another opaque model output. Reach for
judgment only when a task genuinely straddles two rows of the table below.

## The five axes, in one table

Work through these in order. The first axis sets a starting tier; the rest
only ever push it up, down, or split it apart — they don't override it.

| Axis | Question | Effect |
|---|---|---|
| 1. Task type | Mechanical, or judgment? | Sets the base tier (cheap / balanced / frontier) |
| 2. Stakes | Reversible and private, or irreversible/external? | Bump UP one tier if irreversible or externally visible |
| 3. Horizon | Single-shot, or long-running/agentic? | Doesn't change the tier — forces a stop condition to exist |
| 4. Dependency shape | Independent and parallelizable, or shared-context/sequential? | Decides single-agent vs. multi-agent fan-out |
| 5. Volume/latency | One-off, or high-volume/repeated/latency-sensitive? | Pulls toward the cheaper tier when volume is high |

Full matrix, the reasoning behind each axis, and worked examples live in
[references/decision-matrix.md](references/decision-matrix.md) — read it the
first time you use this skill, or whenever a case doesn't cleanly match the
summary below.

## The procedure

**1. Pick the base tier from task type (axis 1).**
- Mechanical / search / extraction / formatting / boilerplate / rote lookup →
  **cheap/fast** tier.
- Drafting, synthesis, moderate reasoning, most ordinary coding →
  **balanced** tier.
- Judgment calls, brand voice, final review, high-stakes gate verification,
  anything ambiguous-and-consequential → **frontier** tier.

**2. Apply the stakes modifier (axis 2).**
If the output is irreversible or will be seen by someone outside the
immediate work session — published content, money movement, a production
deploy, a decision a stranger will act on — bump the tier up by one step from
whatever step 1 gave you. A reversible draft, an internal note, or sandboxed
code doesn't need the bump; it can stay at the task-type tier, or even drop
a notch if it's also high-volume (axis 5).

**3. Check the horizon (axis 3) — separately from the tier.**
This is the axis most people conflate with tier choice, and it's the one
that matters most. If the task is single-shot or runs in a small, bounded
number of turns, steps 1–2 are the whole answer. If it's long-horizon,
agentic, or self-directed (the model will use tools repeatedly, spawn its
own subagents, or run until "done" is a judgment call it makes itself), then
**regardless of which tier you picked**, an external stop condition —
a token or turn budget, a written definition-of-done, a time limit — has to
exist before the task starts. This is not a nice-to-have: even a materially
stronger frontier model will reliably organize itself into ad-hoc
multi-agent sub-structure on long tasks and keep going until something
outside the model stops it. A frontier model with no imposed stop condition
is not the safe choice for a long task — it's the one most likely to run
long and expensive. Tier and stop-condition are two separate decisions; do
both.

**4. Check dependency shape (axis 4).**
If the work is genuinely independent and parallelizable — disjoint sources,
no shared context needed between pieces — fan-out to multiple agents can be
worth it despite the real cost: a multi-agent fan-out can burn several times
the tokens of a single well-tiered agent, because each branch re-establishes
context on its own. That premium is worth paying only when the pieces truly
don't depend on each other and the per-piece value is high (e.g., broad
research across unrelated sources). If the work shares context or has real
sequential dependencies — most operator, business, coding, and writing
tasks — a single agent at the right tier beats splitting it across agents
that can't see each other's work.

**5. Check volume/latency (axis 5).**
High-volume, repeated, or latency-sensitive work (the same short call run
over thousands of rows, an interactive path a person is waiting on) should
weight toward the cheaper/faster tier even at some per-call quality cost,
because volume dominates total cost long before per-call quality does.
One-off, low-volume, latency-tolerant work has no reason to trade quality
for cost — let the task-type tier from step 1–2 stand.

**6. Translate the tier into an actual model choice** — see below, this is
context-dependent and shouldn't be hardcoded.

**7. Always state the one-line reason.**
Name the axis (or axes) that drove the call, not just the verdict. This is
what makes a recommendation auditable instead of a black box a human has to
either trust blindly or redo from scratch. Format:

> `<tier>` tier: `<task type>`, `<reversible/irreversible>` output,
> `<single-shot/long-horizon>`, `<volume>` — `<which axis pushed it, if any>`.

Example: "balanced tier: drafting task, reversible output, single-shot, low
volume — no axis pushes it up or down."

## Translating a tier into a concrete model

The tier is portable; the model name isn't. Pick the translation that
matches where you're actually running:

- **Spawning a subagent inside Claude Code's own Agent/Task tool:** the
  tool's `model` parameter already speaks in tiers — map cheap/fast →
  `haiku`, balanced → `sonnet`, frontier → `opus` (or whatever that live
  tool schema calls its own tiers; don't assume the names above are
  universal, check the tool definition in front of you).
- **Writing actual API integration code** (a script or app calling the
  Anthropic Messages API directly): don't reach for a model-ID or pricing
  table here — that data goes stale fast, and if this environment has a
  `claude-api` skill, it already owns keeping it current. Defer to it
  explicitly rather than guessing a version string.
- **A different provider or an unfamiliar model lineup:** tiers are
  relative, not absolute. Pick that lineup's cheapest model still capable
  of the task type for the cheap tier, and its most capable model for
  judgment/high-stakes work. Don't assume Anthropic's specific tier names
  (Haiku/Sonnet/Opus) carry over.

## Mixed-shape pipelines

A pipeline with stages of different shapes gets a tier **per stage**, not one
blended tier for the whole thing. This is the common case, not an edge case:
"scrape 500 pages, then write the summary, then have someone sign off on the
public version" is three different shapes back to back (cheap → balanced →
frontier), and picking one tier for all three either overpays on the scrape
or underpays on the sign-off. Walk each stage through the procedure above
independently and report a tier (with its one-line reason) per stage.
