# ICM Fit — dimension detail

Full menus, edge cases, and a worked example for each of the six dimensions in `SKILL.md`. Read the relevant section when a candidate doesn't obviously fit a category, or read the worked example at the bottom to see the whole rubric applied end to end.

Contents: Type menu in depth · Token impact estimation · Layer fit in depth · Workflow integration menu · Component match, expanded · Quality signals, expanded · Two worked examples

---

## 1. Type, in depth

| Category | Recognize it by | Default integration cost |
|---|---|---|
| Skill | A packaged instruction set (e.g. a `SKILL.md` + bundled resources) that only loads when triggered | Low — additive, removable, no always-on footprint |
| MCP server | Exposes tools/resources over Model Context Protocol; needs a running process or remote endpoint | Low-to-moderate — clean interface, but now something has to stay running and authenticated |
| Hook | Fires on an agent-harness event (before/after a tool call, session start/stop, etc.) | Low-to-moderate — powerful because it's invisible; that's also the risk (see Quality/Risk notes) |
| Standalone CLI | Runs outside the agent entirely, invoked via shell | Low — if it does the job, it never has to touch context at all (see Layer: None) |
| Library / SDK | Code a maintained project imports and calls | Moderate-to-high — now living inside code someone maintains, with its own upgrade cadence and breaking changes |
| Pattern / prompt (no code) | A documented technique, not an artifact — "always ask X before Y," a prompt template | Low to adopt, easy to silently stop following — durability is the real risk, not integration |
| Hosted service | Third-party API requiring an account, credentials, usually billing | High — the heaviest tail: uptime dependency, data portability risk, ongoing cost, a login someone has to keep alive |

Mixed cases are common — an MCP server that's really a thin wrapper around a hosted service inherits the hosted-service cost, not the MCP-server cost. Classify by what actually has to keep working, not by the wrapper.

---

## 2. Token / context impact — estimating magnitude

Anchor the estimate against something concrete instead of guessing:

- **Small** — a few hundred tokens' difference per typical use (e.g., a slightly shorter reference file than what it replaces).
- **Meaningful** — a full reference load avoided or added on a routine that runs often (daily/every-run), or a multi-thousand-token difference on an occasional task.
- **Dramatic** — something previously re-explained or re-derived every session moves to a one-time load or disappears from context entirely (moved to a CLI/hook that runs outside the model), or the reverse: a candidate wants to inject a large always-on block that used to not exist at all.

The question to actually ask: **"What would the agent otherwise be holding in context to get this same result, and how often does that happen?"** A tool used once a month at a moderate cost matters far less than one invoked on every single run, even at a smaller per-use cost. Weight by frequency, not just per-use size.

---

## 3. Layer fit, in depth

This mirrors the same layer hierarchy an ICM-structured workspace uses for its own content (see `icm-toolkit` if present) — entry file at the top, factory reference material doing most of the everyday work, product/output at the bottom, plus a "none" for things that never enter the context window at all.

- **Entry-file layer.** Reserved for identity and routing — "where am I, where do I go." A candidate has no business here unless it is genuinely something every single task needs to know exists, which is almost never true of a third-party tool. If someone's pitch for a candidate is "put a line about it in the root file so nobody forgets," that instinct is the WATCH/SKIP signal itself — a tool that only survives by being nagged into memory at the top level usually isn't earning its place.
- **Routing layer.** A pointer living in a project or sandbox-level routing file, loaded whenever that area of work is opened. Reasonable for something used constantly *within* that specific area, rarely outside it.
- **Contract layer.** The candidate becomes a named action inside a stage's contract — "this step calls tool X" — without the tool's own documentation being loaded into context at all (the contract just says how it's used, the same way a contract says "run the linter" without inlining the linter's manual). This is the natural home for CLIs, hooks, and MCP tools that do a job rather than inform one.
- **Factory / reference layer.** Loaded only when a task actually needs it. This is where documentation, style guides, API references, and most "read this before doing X" material belongs — and it's where most good candidates land, because it costs nothing on the sessions that don't need it.
- **Product layer.** The candidate produces per-run artifacts (a generated file, a report) rather than being consulted as reference. It doesn't sit in context on an ongoing basis; it shows up in the output of a run.
- **None.** The candidate runs entirely outside the agent's context window — invoked via shell, hit as a webhook, fired by a hook script — and the agent only ever sees its result, not its internals. This is often the *cheapest* possible integration, and worth actively preferring when the job doesn't require the model to understand how the tool works internally.

---

## 4. Workflow integration menu

Generic examples of "existing routine" — substitute the system's actual named routines when scoring a real candidate:

- A recurring pipeline stage that runs every time a unit of work passes through
- A scheduled maintenance sweep (daily / weekly / monthly)
- A setup or onboarding step that runs once per new project/worker/client
- A review or verification pass before something ships
- An ad hoc research or lookup habit (lower bar to justify, since there's no standing cost — but also lower value, since nothing forces anyone to remember it)
- **New routine required** — genuinely new trigger, new habit, new thing to remember

A candidate that only fits "new routine required" isn't automatically a SKIP — some tools are worth building a new habit around — but the bar goes up, because the real cost of a new routine shows up months later when everyone has forgotten it exists, not on day one when it's novel and everyone's excited about it.

---

## 5. Component match, expanded

The sharpest test in the whole rubric: **can you name the one component, stage, or role that would reach for this without being reminded?**

Good answers are specific enough that a stranger reading just the verdict would know exactly who's responsible: "the deploy-verification stage, which already shells out to similar CLIs" or "the weekly-maintenance routine, replacing a manual step it currently does by hand." Bad answers are answers that could be copy-pasted onto a review of a completely different candidate without changing a word: "could help with research," "useful for any content work," "developers would like this."

If the honest answer is "no clear owner yet, but it seems generally good" — that's not a failed search, that's the finding. Write it down as the finding and let it drive the verdict toward WATCH or SKIP rather than searching harder for a home that isn't there.

---

## 6. Quality signals, expanded

None of these should individually move a verdict from ADOPT to SKIP or back — they're corroborating, not decisive, and they're easy to check, which is exactly why they're easy to over-trust.

- **Recency.** Ask what kind of tool this is before penalizing silence. An MCP server wrapping a fast-moving API going quiet for a year is a real risk (the API has probably moved under it). A finished, self-contained reference (a style guide, a completed spec, a small single-purpose CLI with no external dependencies) going quiet for a year just means it's done, not abandoned.
- **Popularity.** Useful only as a floor-check ("has literally anyone else used this and lived to tell about it") for unfamiliar categories like hosted services or libraries with security surface. Worthless as a signal for a niche tool solving a specific, uncommon problem — those are supposed to have small numbers.
- **License.** The one signal that can be a hard blocker on its own — a license genuinely incompatible with how the tool would be used ends the evaluation regardless of how good everything else looks.
- **Dependency weight.** Look at what it drags in, not just what it claims to do. A CLI that needs its own language runtime and package manager installed is a heavier commitment than its feature list suggests.
- **Maintainer signal.** Solo-dev tools aren't disqualified — plenty of the best small tools are solo-maintained — but name the mortality risk explicitly in the verdict rather than silently assuming continuity. Company-backed doesn't automatically mean safer either; companies abandon products and change licenses too.

---

## Worked example 1 — a solo-maintained MCP server for a niche API

*Candidate: an MCP server, maintained by one developer, wrapping a niche third-party analytics API. Last commit 4 months ago, ~40 GitHub stars, MIT license, requires an API key and a running Node process.*

```
Type: MCP server (wrapping a hosted service — inherits that service's uptime/cost tail)
Token impact: Saves tokens on the specific task (avoids manually explaining the API's
  schema every time it's queried) — magnitude: meaningful, but only on the handful of
  sessions that touch this analytics data at all.
Layer: Contract layer — a named action a specific reporting stage would call; its own
  docs never need to load into context, only its existence in that stage's contract.
Workflow: Slots into an existing recurring reporting routine (if one exists) rather than
  requiring a new one — assuming that routine currently does this analytics pull by hand.
Component match: The reporting/analytics stage specifically — not a generic fit. Clear
  owner if that stage exists; if it doesn't, this candidate has no home yet.
Quality signals: Solo maintainer (real mortality risk, named not penalized) · 4 months
  quiet is a mild yellow flag for something wrapping a live third-party API that can
  change under it · MIT license, no blocker · star count too low to mean anything either
  way for a niche tool like this.
Verdict: WATCH — good layer fit and a plausible specific owner, but only if the reporting
  routine it would serve actually exists yet; log it and revisit at the next reporting
  cycle rather than wiring it in speculatively today.
Skipped dimensions: none.
```

*Sanity check: this is the right shape of verdict for a "maybe, not yet" case — it doesn't inflate a genuinely narrow, contract-layer tool into something bigger than it is, and it doesn't punish it for being solo-maintained or quiet, both of which are proportionate for what it is.*

---

## Worked example 2 — a hypothetical multi-agent orchestration framework

*Candidate: a GitHub repo implementing a general-purpose multi-agent orchestration framework — message bus, agent registry, supervisor/worker pattern, its own YAML-based agent-definition DSL. Actively maintained by a small company, 3k stars, Apache-2.0, requires adopting its runtime and rewriting existing agent logic into its DSL.*

```
Type: Library / SDK (a full runtime + its own DSL — the heaviest end of that category,
  bordering on a framework replacing the system's existing architecture outright)
Token impact: Costs tokens and then some — the DSL itself becomes something every
  future session has to understand and reason inside of; this isn't a context-loading
  cost so much as a "the model must now think in a new formalism" cost. Magnitude: large.
Layer: Would want to sit at, or above, the entry-file layer, since the DSL would govern
  how every stage is defined — that is itself the warning sign called out in the layer
  section: a candidate whose pitch requires top-level, always-relevant placement.
Workflow: New routine required across the board — every existing stage would need to be
  re-expressed in the framework's terms before it worked at all. Not a slot-in; a rewrite.
Component match: "Could replace the whole orchestration layer" is the textbook version of
  the red flag — a claim to be useful for everything is a claim to have no specific owner.
Quality signals: not worth spending more time on — the dimensions above already closed it.
Verdict: SKIP — the system's whole design bet is a small stable entry point and layered,
  on-demand context; this candidate's core value proposition is a framework doing
  coordination in code, which is precisely the trade this kind of system is built to avoid.
  Real-time multi-agent message-passing at genuine concurrency is a case where that trade
  goes the other way — but nothing in the candidate's pitch here establishes that this
  system actually has that need.
Skipped dimensions: quality signals scored but not detailed — recency/stars/maintainer
  were all fine, and none of that changes a verdict this dimension-1-through-5 already
  settled.
```

*Sanity check: this is a case where the honest, well-reasoned answer is SKIP even though the tool itself is probably good engineering — the rubric is doing its job by separating "is this well-built" (plausibly yes) from "does this system in particular want it" (no), which is exactly the judgment icm-fit exists to make instead of a generic quality review.*

---

## Worked example 3 — a simple CSV-to-JSON CLI

*Candidate: a small, dependency-free CLI that converts CSV to JSON. Single maintainer, last commit 2 years ago, no open issues, ~200 stars, MIT.*

```
Type: Standalone CLI
Token impact: Saves tokens, small magnitude — replaces the model writing an inline
  conversion script from scratch each time this specific need comes up, which is not
  frequent enough for the saving to be more than small.
Layer: None — runs entirely outside the agent via shell; the model only ever needs to
  see its output, never its internals or documentation.
Workflow: Ad hoc — no standing routine currently does CSV-to-JSON conversion often
  enough to call this "slotting into" anything; it would be reached for occasionally.
Component match: No specific stage owns this today; it would be a generically handy
  utility invoked whenever the need happens to come up, by whichever task hits it.
Quality signals: 2 years quiet is a non-issue here — this is a finished, narrow tool
  with no external API to drift out from under it, not an actively-maintained-dependent
  one · dependency-free, which is the main risk this category usually carries · license
  fine.
Verdict: WATCH, leaning SKIP as a formal "adopt" — there's nothing wrong with it and
  nothing to integrate: it doesn't need a place in the system at all, since a one-off
  shell invocation whenever CSV-to-JSON comes up costs nothing and requires no standing
  commitment. Formal adoption would be solving a problem that doesn't exist; use it ad
  hoc the day it's needed and skip giving it a permanent home.
Skipped dimensions: none, but note the finding itself — some genuinely fine candidates
  don't need a verdict above WATCH because there's no integration to evaluate in the
  first place, just an occasional shell command.
```

*Sanity check: this correctly resists the temptation to manufacture a bigger verdict than the candidate deserves — a tiny, solid, dependency-free CLI doesn't need "ADOPT" ceremony; it needs "yes, use it when the need comes up, don't build anything around it."*
