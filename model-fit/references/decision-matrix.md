# Model Fit — Decision Matrix and Worked Examples

Read the relevant section, not the whole file top to bottom — it's organized
so you can jump straight to the axis or example you need.

## Table of contents

1. [Task type → base tier](#1-task-type--base-tier)
2. [Stakes/reversibility modifier](#2-stakesreversibility-modifier)
3. [Horizon: the axis that isn't about tier at all](#3-horizon-the-axis-that-isnt-about-tier-at-all)
4. [Dependency shape: when fan-out earns its cost](#4-dependency-shape-when-fan-out-earns-its-cost)
5. [Volume and latency](#5-volume-and-latency)
6. [Worked examples](#6-worked-examples)
7. [When the table genuinely doesn't resolve it](#7-when-the-table-genuinely-doesnt-resolve-it)

---

## 1. Task type → base tier

| Signal | Tier | Why |
|---|---|---|
| Grep/search across files, data extraction, reformatting, boilerplate scaffolding, mechanical find-and-replace, rote lookups, simple classification with a clear rubric | **Cheap/fast** | The task has one correct-shaped answer and no real interpretive work; a stronger model spends extra compute confirming what the cheap model already got right. |
| Drafting a first pass, synthesizing several sources into a summary, moderate multi-step reasoning, most ordinary application coding, routine editing | **Balanced** | Enough ambiguity and structure-building that a cheap model produces sloppier drafts needing more correction than the tier cost saves; not yet the kind of call that needs the strongest available judgment. |
| Brand-voice or tone-sensitive writing, a final review before something ships, verifying a high-stakes gate (a compliance check, a security review, a go/no-go), any call that's genuinely ambiguous and consequential at the same time | **Frontier** | These are exactly the cases where the difference between a good and a mediocre model shows up in the output, and where a mistake is expensive enough that the tier premium is cheap insurance by comparison. |

The test that separates "balanced" from "frontier" isn't difficulty in the
puzzle sense — it's whether getting it subtly wrong would be caught later
cheaply (balanced is fine) or would ship as-is and matter (frontier).

## 2. Stakes/reversibility modifier

Ask two questions:

- **Reversible?** Can a mistake be caught and fixed before it costs anything
  — a draft still under review, code still in a branch, a note only you'll
  read?
- **External?** Will the output leave the room — publish to an audience,
  move money, touch production, become something a stranger reads or acts
  on?

If the answer to "external or irreversible" is yes, bump the tier up one
step from wherever task type alone landed you. Mechanical-but-irreversible
work (say, a find-and-replace script that will run directly against a
production database with no review step) moves from cheap to balanced.
Drafting-but-irreversible work (copy that publishes the moment it's
approved, with no further check) moves from balanced to frontier.

If the answer is no — sandboxed, internal, easily undone — task type stands
on its own, and if volume is also high (axis 5), it's fine to let it sit at
or even below the task-type tier.

## 3. Horizon: the axis that isn't about tier at all

Every other axis answers "which tier." This one answers a different
question: "does a stop condition exist yet?" — and it applies on top of
whatever tier the other axes produced, not instead of it.

**Single-shot / bounded turns:** the task will finish in a turn or a small,
predictable number of turns. No special handling needed — tier per task
type and move on.

**Long-horizon / agentic / self-directed:** the task runs across many tool
calls, may spawn its own subagents, or has a fuzzy "done" that the model
itself gets to decide. Here, before the task starts, one of these has to be
pinned down:

- a token or turn budget ("stop after roughly N turns / M tokens and report
  back"),
- a written definition-of-done ("done means X, Y, and Z are true — check
  before declaring finished, not after"),
- or a hard time limit.

The reasoning: even a materially stronger, more capable frontier model does
not reliably self-limit on open-ended work. Given room to run, it tends to
organize itself into its own ad-hoc multi-agent structure — spawning
sub-tasks, delegating pieces to itself — and keeps that going until
something external interrupts it, not because it decided it was done. This
shows up across long-running agentic setups often enough that it's treated
here as the default behavior to plan around, not an edge case. The fix that
actually works is a human-authored stopping point, never "use a smarter
model instead" — a smarter model without a stop condition just runs the same
pattern for longer and at higher cost per step.

Practical consequence: don't treat "I picked the frontier tier" as having
handled the risk of a long-running task. Tier and stop condition are two
separate line items. Skipping the second while maximizing the first is the
actual failure mode this axis exists to catch.

## 4. Dependency shape: when fan-out earns its cost

Two shapes, and they call for opposite structures:

**Independent, parallelizable, high value per piece.** Disjoint sources that
don't need each other's context — e.g., "pull the current public facts on
five unrelated competitors" or "search these ten separate document sets for
a specific detail in each." Here, spinning up several agents genuinely
speeds things up and each branch produces value the others couldn't reach
faster by waiting in a queue.

**Shared context, real sequential dependency.** One piece needs to know what
came out of another — most operator work, most business decisions, most
coding tasks (where a change in one file affects the meaning of another),
most writing (where the ending needs to know what the opening promised).
Splitting this across uncoordinated agents doesn't parallelize the work; it
just produces pieces that don't fit together, and then something (often the
most expensive model in the chain) has to spend effort reconciling them.

The cost that's easy to forget when fan-out looks appealing: running several
agents each re-establishing their own context from scratch is not a cheap
operation. It can plausibly cost several times the tokens of a single
well-tiered agent doing the same work sequentially. That premium is a real
cost, not a rounding error — pay it only when the independence is genuine
and the per-piece payoff justifies it, not by default because parallel
"sounds faster."

## 5. Volume and latency

**High-volume, repeated, or latency-sensitive** (the same prompt run over
tens of thousands of rows; an interactive path where a person is watching a
spinner): weight toward the cheaper/faster tier even if it means giving up
some per-call polish. At volume, the total bill is dominated by how many
times the call runs, not by how much better each individual call could have
been — a marginal per-call quality gain multiplied by 50,000 calls is a very
different number than the same gain on one call.

**One-off, low-volume, latency-tolerant:** there's no volume multiplier to
worry about, so let whichever tier task type (and the stakes modifier)
already produced stand as-is. Don't reach for the cheap tier here "to save
money" — on a single call the savings are negligible and the task-type tier
exists for a reason.

## 6. Worked examples

**"Grep 200 files for a config key and summarize what's inconsistent."**
Task type: mechanical search/extraction → cheap tier. Stakes: internal,
reversible (a wrong summary just gets re-run) → no bump. Horizon: bounded,
single pass → no stop condition needed beyond normal turn limits.
Dependency: the files can be searched independently, but the volume here
(200 files, one grep pass) doesn't clear the bar where fan-out's token
premium pays for itself — a single cheap-tier agent handles this in one
pass faster than coordinating parallel branches would. Volume: one-off ask,
not repeated → doesn't pull further down.
**Verdict: cheap/fast tier, single agent. "Cheap tier: mechanical
search/extraction, reversible, single-shot, one-off — no axis pushes it
up."**

**"Write the final customer-facing copy for our pricing page."**
Task type: this is brand-voice, final-form writing → starts at balanced,
arguably frontier given "final." Stakes: external and effectively
irreversible once published (a stranger reads it, it shapes a purchase
decision) → bump up one tier regardless of where step 1 landed. Horizon:
single-shot (one drafting pass, maybe a revision round) → no special stop
condition beyond the normal one. Dependency: sequential — the page has to
read as one coherent voice, not stitched from independent fragments →
single agent, no fan-out. Volume: one-off page, not batched → no pull
toward cheap.
**Verdict: frontier tier, single agent. "Frontier tier: brand-voice/final
copy, externally visible and effectively irreversible, single-shot, one-off
— the stakes axis alone would justify the top tier even if task type
started lower."**

**"Spin up agents to refactor this whole codebase, go as long as it takes."**
Task type: most of a refactor is ordinary coding → balanced tier for the
bulk of it, with judgment-level review at the point changes get merged
(that review step alone could justify frontier). Stakes: depends whether
this lands in production directly or goes through review — assume it's
merged into a shared codebase eventually, so treat it as bump-worthy at the
merge/review step, even if working branches stay reversible. Horizon: this
is the flag phrase — "go as long as it takes" is explicitly the shape this
axis exists for. **The tier choice is not the fix here.** Before this runs,
pin a turn/token budget or a concrete definition-of-done (e.g., "stop when
these N modules compile and tests pass, then report back for a human
decision on the rest") — without that, this runs long and expensive
regardless of which model tier is chosen, because nothing internal to any
model reliably says "we're done" on its own on an open-ended refactor.
Dependency: refactor work across a codebase is usually NOT
independent — a rename or type change in one file ripples into others — so
default to a single coordinated agent per coherent chunk of the codebase
rather than uncoordinated parallel agents each touching files that affect
each other; only fan out across modules that are genuinely isolated from
each other (e.g., unrelated leaf packages with no shared imports). Volume:
not a repeated-call situation, so no pull toward cheap.
**Verdict: balanced tier for the mechanical refactor work, frontier tier at
the review/merge gate, single coordinated agent (or a few agents only
across genuinely isolated modules) — and, separately and non-negotiably, an
external stop condition set before it starts. "Balanced tier for the
refactor itself, frontier at the merge gate — but the open-ended horizon is
the dominant issue here: set a turn budget or definition-of-done before
this runs, because no tier choice substitutes for that."**

**"Run this same classification prompt over 50,000 rows."**
Task type: classification with a fixed rubric is mechanical once the rubric
is set → cheap tier. Stakes: depends what the classification feeds — if
it's an internal sort, low stakes; if misclassifications directly drive an
external action (e.g., auto-rejecting something a person submitted) that
would bump one tier for the whole batch, or at least for a spot-check
sample. Horizon: bounded — each row is single-shot and the batch as a whole
isn't agentic. Dependency: rows are fully independent of each other, but
this is a batch/throughput problem, not a fan-out-for-speed problem — it
doesn't need multiple coordinating agents, just the cheap tier called
50,000 times (or batched). Volume: this is the textbook high-volume case —
the axis that matters most here. Even a small per-call quality edge from a
pricier tier gets multiplied by 50,000 and becomes a large number, while
the cheap tier's per-call weakness stays small in absolute terms if the
rubric is clear.
**Verdict: cheap/fast tier, no fan-out (a throughput/batching concern, not a
parallel-agents concern), and consider a periodic frontier-tier spot-check
sample rather than upgrading the whole batch. "Cheap tier: mechanical
classification, high volume dominates the cost calculus, single-shot per
row — volume is the deciding axis, not task type headroom."**

## 7. When the table genuinely doesn't resolve it

A few cases sit between rows on purpose:

- **A task that's mechanical in form but the stakes are severe** (e.g., a
  find-and-replace across a legal contract that will be signed as-is): let
  the stakes axis win over task type. Stakes modifiers exist specifically
  to override a task-type tier that would otherwise underrate a risky
  mechanical action.
- **A single call that's both high-volume AND high-stakes** (e.g., 50,000
  automated messages sent directly to customers with no review): volume
  pulls down, stakes pushes up — resolve by keeping the per-call tier cheap
  but adding a stakes-driven safeguard outside the model choice itself (a
  sampling review, a staged rollout, a human check on the first batch)
  rather than paying the frontier-tier cost on all 50,000 calls.
- **A task whose horizon is unclear at the start** ("keep going until it
  looks good"): treat this as long-horizon by default and require a stop
  condition before starting — the cost of wrongly imposing a budget on a
  task that would have finished quickly anyway is far lower than the cost
  of an ungoverned agentic run.

If a case still doesn't resolve after applying these, that's the signal
this is one of the genuinely rare situations worth spending a real judgment
call on — not a sign the table is broken.
