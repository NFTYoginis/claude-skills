# Map mode and Assess mode — output formats and worked examples

Depth for the two modes SKILL.md only outlines. Read at Map mode step 4 or Assess mode step 3.

Contents: Map mode output shape · Map mode worked example · Assess mode scoring rubric · Assess mode worked example · Shared discipline

## Map mode output shape

A preview, not an audit report — keep it skimmable in under two minutes:

1. **Inventory** — table of area/file, what it is, last touched (if visible), what references it.
2. **Classification** — same table, one more column: role (catalog / contract / factory / product / dead) and a confidence note where the cold read is genuinely unsure.
3. **Form call** — one line naming which of the six forms fits (or which two compose), and why.
4. **Proposed tree** — a fenced code block of the target structure. Every folder in it should trace back to something in the inventory; don't invent stages that aren't there.
5. **Migration map** — old path → new path → role, framed as "if you decide to build this," not as a pending approval.
6. **Open questions** — anything the cold read couldn't resolve (dead links, ambiguous ownership, a form that only half-fits).

Stop after 6. No approval step, no execution.

## Map mode worked example

Run cold against `ai-articles/` (an 11-part article series inside a real working directory). Inventory pass surfaced:

- `NN-brief.md` × 11 — a per-article brief (voice, slug, accent color, structure) — **contract**.
- `NN-images/` × 11 — per-article generated image sets — **product**.
- `NN-preview.html` × 11 — rendered preview per article — **product**.
- `_repo-spec.md` — a master spec for three unrelated downstream repos (content/design/animation starters) — **factory**, but scoped wider than this folder; likely belongs one level up or in its own `_shared/`.
- `_build-index.js`, `_build-series-paste-doc.js` — generator scripts for `INDEX.html` and `series-paste-doc.html` — **factory** (the recipe) whose outputs are **product**.
- `INDEX.html`, `series-index.html`, `series-paste-doc.html` — generated rollups — **product**, and per invariant 9 should never be hand-edited (their generators exist, which is the healthy pattern already in place here).
- No `CLAUDE.md`, no `CONTEXT.md` anywhere in the folder — **catalog is missing entirely**. An agent dropped into this folder cold has to infer the numbering convention and the brief→images→preview relationship from filenames alone.
- One open question: `02-brief.md` links to `_series-brief.md` (voice/structure/give-withhold rules) — that file does not exist in this folder. Either it lives in one of the three starter repos `_repo-spec.md` names, or it's a dead reference. Flagged, not resolved — Map mode doesn't chase it down.

Form call: **Record library** (the record is "one article," each a uniform brief→images→preview unit) with a **Pipeline** factory layer (the two build scripts + the spec) sitting above it as shared tooling.

Proposed tree:

```
ai-articles/
├─ CLAUDE.md                 NEW — routing: what's a record, where the factory lives
├─ CONTEXT.md                NEW — the record schema (brief → images → preview)
├─ _factory/
│  ├─ _repo-spec.md          moved, unchanged
│  ├─ build-index.js         moved, unchanged
│  └─ build-series-paste-doc.js
├─ _index/
│  └─ INDEX.html             generated, unchanged — never hand-edit
├─ records/
│  ├─ 01-token-burn-daily-routine/
│  │  ├─ brief.md            was 01-brief.md
│  │  ├─ images/             was 01-images/
│  │  └─ preview.html        was 01-preview.html
│  └─ … one folder per article, same shape
└─ series/
   ├─ index.html             was series-index.html
   └─ paste-doc.html         was series-paste-doc.html
```

Migration map (excerpt): `01-brief.md` → `records/01-token-burn-daily-routine/brief.md` (contract) · `01-images/` → `records/01-token-burn-daily-routine/images/` (product) · `_repo-spec.md` → `_factory/_repo-spec.md` (factory, but confirm scope — it may belong a level up if the three starter repos use it too) · `_series-brief.md` → **unresolved**, locate or recreate before migrating.

That's the whole Map mode deliverable — no migration happens here.

## Assess mode scoring rubric

Six checks, taken directly from [The walk test](../SKILL.md#the-walk-test):

| # | Check | Pass bar |
|---|---|---|
| 1 | Orientation | Entry file + at most 2 more reads answers "where am I / where do I go for this task" |
| 2 | Contract | The relevant `CONTEXT.md` names exact input paths, the job, the output, and one human check |
| 3 | Status by scanning | What's done/pending is derivable from what exists on disk — no separate tracker needed to know |
| 4 | Catalog carries no payload | Routing files (entry file, root `CONTEXT.md`) hold pointers only, no content |
| 5 | One home per fact | No rule, schema, or fact is duplicated across two files without one being a link |
| 6 | Token check | entry file + one contract + its inputs lands in ~2k–8k tokens |

Add the two System map extra checks from [references/system-map.md](system-map.md#walk-test-this-form) only when assessing that form (map one hop from the subject's entry file; `effects/CONTEXT.md` names a real hit/no-hit).

Score each PASS / PARTIAL / FAIL with one line of evidence — a line count, a token estimate, a named file doing two jobs. Never soften a FAIL into a PARTIAL to be polite; never pad a PASS with restated praise.

**Roll-up:**
- **Score** — count PASS as 1, PARTIAL as 0.5, FAIL as 0, out of 6 (or 8 with System map checks).
- **Band** — 6/6 (or 8/8): walkable as-is. 4–5: close, name the one or two gaps. ≤3: not yet walkable — treat as a Restructure candidate, not a tuning pass.
- **Top fixes, cheapest first** — usually 1–3 items, each naming the exact file and the exact move (split, point instead of restate, delete the duplicate, write the missing contract).

## Assess mode worked example

Run cold against the root of a real, in-use operator workspace (`CLAUDE.md`, `STATUS.md`, `CONTEXT.md`, `_config/`, `briefs/` at the top level — a working HQ folder that has never been run through this skill but converged on ICM-like shape on its own). Specifics of the workspace's own content are omitted below; only structural facts are evidence.

| # | Check | Result | Evidence |
|---|---|---|---|
| 1 | Orientation | **PASS** | The entry file is almost entirely a routing table and links (a "read this, then this, then this" chain); a cold agent reaches the right sub-file for a named task in 2 reads. |
| 2 | Contract | **PARTIAL** | Each worker sub-folder does carry its own `CONTEXT.md` — but its sections are "Voice & Audience / Reference Exemplars / What to Load for Each Task," not the canonical Inputs/Process/Outputs/Human-check shape. It routes and informs; it doesn't declare exact input paths or a single stated human check. Contract-as-concept exists; contract-as-template doesn't. |
| 3 | Status by scanning | **PASS** | A rolling root `STATUS.md` plus a `STATUS.md` in every worker sub-folder derive current state from what's logged, not from a hidden tracker; a maintenance script keeps rollover lock-safe and current. |
| 4 | Catalog carries no payload | **PARTIAL** | The root entry file mostly routes, but its guardrail section embeds several full-sentence policy statements directly rather than only pointing to the config file that already exists for that purpose — a small amount of payload sitting in the catalog. |
| 5 | One home per fact | **PASS** | Cross-references ("details: X," "ruling chain is Y") consistently point rather than restate; no duplicated policy text found in the sampled files. |
| 6 | Token check | **FAIL** | Root entry file is ~63 lines but ~7.2KB (roughly 1.8k tokens at 4 bytes/token) — already over the L0 budget of 300–800 tokens on its own, because the line-count invariant and the byte/token budget are tracking different things here. Add one worker `CONTEXT.md` (also ~7KB, ~1.9k tokens) before any actual input file loads, and entry + one contract alone is already ~3.7k tokens — one real reference file away from blowing the 8k ceiling. |

**Score: 3.5/6 — close, not walkable at the token budget.** The shape is genuinely ICM-convergent (routing chain, per-room contracts, script-enforced status) — this reads as a workspace that arrived at the method independently, not one built from it.

**Top fixes, cheapest first:**
1. Move the inline policy sentences out of the root entry file into the `_config/` file they already point to elsewhere for other rules — check 4, one edit.
2. Trim or split the root entry file itself; a 7KB "small, stable" file is the single biggest token-check violation — check 6.
3. Template the per-room `STATUS.md` files toward the stage-`CONTEXT.md` shape (Inputs/Process/Outputs/Human check) so contract-completeness stops depending on which room wrote it — check 2.

## Shared discipline

Both modes are cold reads: don't pre-load the whole tree "to be thorough" before writing the inventory or the score — that's the slurp this method exists to prevent, and it also lets bias creep in (a model that's already read everything tends to grade generously). Sample enough to classify honestly, cite what you saw, and say where confidence is low rather than smoothing it over.
