---
name: icm-fit
description: Grade whether an external candidate — a Claude Code skill, MCP server, hook, standalone CLI, library/SDK, hosted service, or a bare pattern/prompt someone found on GitHub, Hacker News, X, or a Slack recommendation — is actually worth pulling INTO a system built on ICM (Interpretable Context Methodology: context loaded in layers from always-on down to loaded-on-demand). Produces a structured verdict — type, token impact with rough magnitude, which layer it would occupy, which existing workflow/contract would absorb it, quality signals, and an adopt / watch / skip call — instead of a vibes paragraph. Use this whenever someone pastes a repo link, package name, MCP server, or skill and asks "should we use this", "is this worth adding", "what do you think of this tool", "found this, could be useful for us", "worth integrating?", or is otherwise weighing whether to adopt a third-party tool or dependency — even if they never say "evaluate" or "rubric". This is the OPPOSITE direction from icm-toolkit: icm-toolkit grades whether your OWN folder structure follows ICM conventions; icm-fit grades whether to bring something NEW in from outside. Never use icm-fit to audit your own workspace, and never use icm-toolkit to vet a third-party tool — if the target of the judgment is "us," reach for icm-toolkit instead.
---

# ICM Fit

A rubric for one recurring judgment: **is this candidate worth the integration tax, given the system it would join?** Apply it to any skill, MCP server, hook, CLI, library, service, or pattern someone is proposing to adopt.

This exists because the hit rate on "useful to a specific ICM-structured system" is low even for genuinely good tools. Most candidates are built for nobody's system in particular — a well-made general-purpose framework can still be a net loss for a system whose whole design bet is a small, stable entry point and context loaded only when a task needs it. The rubric is a filter against integration tax, not a checklist to fill in performatively. **Bias toward SKIP when uncertain — attention to maintain the integration is the scarce resource, not the supply of candidates.**

Skipping a dimension is fine once an earlier answer is a clear no — but say *which* dimension got skipped and *why*, so the verdict reads as a shortcut, not an omission.

## Before scoring anything: the guardrail

**AI-sounding prose in a candidate's README or docs is a fact about how it was written, never a fact about whether it works.** Inflated-significance language, rule-of-three cadences, em-dash overuse, a certain vocabulary cluster — all of that describes authorship style, not artifact quality. Score the tool, not the writing:

- Does it do what it claims when you can check?
- Do cited benchmarks or numbers resolve to something real?
- Is the repository actually alive — not "does it read like a person wrote every word"?

The only things that count as real evidence *against* a candidate are objective and checkable: fabricated or unverifiable benchmarks, dead links, citations that don't resolve or can't be true, claims the actual code contradicts. A gorgeously human-written README on a broken tool is still broken. A flatly AI-drafted README on a tool that genuinely works is still a tool that works. Never downgrade or skip a candidate on prose style alone — that's a tell about the writer, not the artifact.

## The six dimensions

Work through these in order. Full menus, worked examples, and edge cases for each are in [references/dimensions.md](references/dimensions.md) — read it before scoring anything you're unsure how to classify.

### 1. Type

Pin the candidate to exactly one category — it drives everything else, because integration cost is mostly a function of category, not quality:

`skill` · `MCP server` · `hook` · `standalone CLI` · `library / SDK` · `pattern or prompt (no code)` · `hosted service`

Skills, hooks, and CLIs slot in and out cleanly — deleting them leaves no trace. Libraries/SDKs embed into code you maintain. Hosted services carry the heaviest tail: accounts, billing, a dependency on someone else's uptime, and data that may not come back out in a usable shape.

### 2. Token / context impact

Compare the candidate's typical-use cost against doing the equivalent task today, and give a **rough magnitude**, not just a direction:

- **Saves tokens** — moves work to a cheaper or on-demand surface (a skill that only loads when triggered, a CLI that runs outside the context window entirely, a reference file replacing something previously re-explained every session)
- **Neutral** — same cost, different ergonomics
- **Costs tokens** — adds always-loaded context, expensive round-trip tool calls, or heavy scaffolding the model has to hold in mind

"Saves tokens" with no magnitude is not an assessment — say small / meaningful / dramatic, and say relative to what.

### 3. Which layer it would occupy

Use the same layer vocabulary as the system's own ICM structure — entry file, routing, stage contracts, factory reference material, per-run product:

| Layer | What lives there | Bar for a new candidate |
|---|---|---|
| **Entry-file layer** (root `CLAUDE.md`/`AGENTS.md`, always loaded) | Identity + routing only | Extremely high — every line here taxes every single session forever |
| **Routing layer** (root/sandbox `CONTEXT.md`) | Pointers to where things live | High — loaded whenever that area is opened, still meant to stay small |
| **Contract layer** (a stage's `CONTEXT.md` — what it reads, does, writes) | A step in a process | Moderate — this is where a candidate becomes *an action a stage takes*, e.g. "step 3 calls this CLI" |
| **Factory / reference layer** (`references/`, `_shared/`, loaded on demand) | Stable material read when a task needs it | **This is where most good candidates belong** |
| **Product layer** (per-run output) | This run's artifacts | Candidate is generating output, not being loaded as reference |
| **None** | — | Runs entirely outside the agent — a CLI invoked once via shell, a service hit by a script — and never occupies a context window at all |

If a candidate seems to want the entry-file layer, treat that as a warning, not a compliment — it means someone is proposing to tax every future session for this one thing. Push it down a layer and see if it still works there; it usually does.

### 4. Workflow integration

Name the existing recurring routine this would slot into — a scheduled maintenance pass, a pipeline stage that runs every time a unit of work moves through, a setup/onboarding step, a review or verification pass — or say **"new routine required."**

A new routine is a real cost: someone has to learn a new trigger, remember it exists, and keep remembering it after the novelty wears off. That cost is worth paying sometimes. It is not free, and "new routine required" should raise the bar for the rest of the verdict rather than being a throwaway line.

### 5. Component / contract match

Name the **specific** stage, contract, pipeline step, or role that would actually pick this up and use it regularly — not a category of user, a named one: "the step-2 draft contract, which already calls a similar CLI" beats "content creators."

**"Could be useful for anyone / any project / any task" is a red flag, not a selling point.** Generic-fit candidates almost never earn their integration cost, because nobody in particular is responsible for remembering they exist. If nothing in the system would predictably reach for this on its own, that alone is often enough to end at SKIP.

### 6. Quality signals

Cheap public proxies — weigh each lightly, never let one carry the verdict alone:

- **Recency** — quiet for 12+ months is a yellow flag for a tool that needs active maintenance (an MCP server, a CLI with a moving API surface); it's a non-issue for a stable reference (a static ruleset, a finished spec).
- **Popularity** (stars, downloads, forks) — an adoption proxy, not a quality proxy. Near-worthless signal for a niche tool nobody but you would need.
- **License** — check compatibility with how it'll actually be used; flag copyleft or non-commercial clauses if they'd bind.
- **Dependency weight** — a heavy toolchain (its own package manager, a runtime you don't already run) raises integration cost independent of what the tool does.
- **Maintainer signal** — solo hobby project vs. company-backed vs. active community fork. Solo-maintainer tools carry real mortality risk; that's a fact worth naming, not a moral judgment on the maintainer.

## The verdict

Close with a structured call, not a paragraph of vibes:

```
Type: <category>
Token impact: <saves/neutral/costs> — <magnitude + what it's relative to>
Layer: <entry-file / routing / contract / factory-reference / product / none>
Workflow: <existing routine it slots into, or "new routine required">
Component match: <specific stage/contract/role — or "no clear owner">
Quality signals: <2-4 short bullets, only the ones that moved the needle>
Verdict: ADOPT / WATCH / SKIP — <one sentence naming the dominant factor>
Skipped dimensions: <none, or which ones and why — e.g. "stopped after
  component match returned no clear owner; token/quality not scored">
```

- **ADOPT** — clear layer, clear owner, cost is small or the payoff is large enough to justify it.
- **WATCH** — real potential but missing one load-bearing piece (no current owner, too new to judge maintenance, license unclear) — log it, don't integrate it yet.
- **SKIP** — the default when a dimension comes back weak or generic. Not an insult to the tool; it's a statement that this system in particular shouldn't carry it right now.

Worth one closing sentence on lock-in and alternatives when it's not obvious: does adopting this make it harder to walk back later, and is there already something in the system doing most of this job? Neither needs its own formal dimension — fold it into the verdict sentence when it matters, skip it silently when it doesn't.
