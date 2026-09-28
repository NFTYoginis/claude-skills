---
name: icm-toolkit
description: Design any process, idea, problem, or body of knowledge into an ICM (Interpretable Context Methodology) workspace — folder structure as agent architecture — restructure an existing folder, repo, or vault into one, get a fast no-commitment preview of what a folder would look like as an ICM, or grade an existing folder against ICM conventions. Use when the user wants to (1) turn a recurring workflow into an agent-runnable folder pipeline, (2) organize scattered notes, files, or knowledge into a library one AI agent can walk, (3) map a team or company as connected context ("context map", "second brain", "team brain", "knowledge base for AI"), (4) audit a codebase or mixed folder into a walkable edit map (objects, processes, change-impact) so later agents can change it without slurping the tree, (5) audit or restructure an existing workspace to ICM conventions, (6) says "make this an ICM", "ICM this", "map this repo", "audit this folder", "what would a change hit", "build me a workspace", or "structure this for agents", (7) wants a quick, no-commitment look before deciding anything — "map my folders as an ICM", "just show me what this would look like structured", "show me the tree, don't move anything", "what would this look like as an ICM", "preview this as an ICM", or (8) wants a health-check or grade on a folder that already claims or attempts ICM structure — "audit whether this is actually good ICM", "score my setup", "is this folder well-structured", "grade my ICM", "run the walk test on this", "is this actually walkable for an agent", "diagnose my workspace", "is this over-engineered / under-structured". Trigger whenever the user names ICM or Interpretable Context Methodology, or asks whether a folder, repo, or vault is well-organized for an AI agent to work in — even without those exact phrases.
---

# ICM Toolkit

> **Fork notice.** This skill forks and extends Jake Van Clief's [`icm-architect`](https://github.com/RinDig/icm-architect) skill. Both are built on the **Interpretable Context Methodology** (Van Clief & McDermott, ["Interpretable Context Methodology: Folder Structure as Agent Architecture"](https://arxiv.org/abs/2603.16021), MIT-licensed; community: [Clief Notes](https://www.skool.com/cliefnotes)) — the method, the ten invariants, the six forms, Build mode, Restructure mode, and the walk test below are theirs; full credit to both the paper's authors and the original skill. This fork keeps everything the original does and adds two new directly-invokable modes, Map and Assess (see "Choose a mode"). MIT terms carried over from the original are in `LICENSE`.

Build workspaces where the folder structure does the orchestration. One agent, reading the right files at the right moment, replaces a multi-agent framework: numbered folders carry sequencing, hierarchy carries context scoping, plain markdown files carry state. A human can open any folder and see exactly what state the system is in, because state is just files.

Think of the workspace as a library. The routing files are the catalog: small, stable, they point at everything and store almost nothing. The content lives on the shelves (stage folders, node files, reference material). One librarian — one model — walks the building, and the question decides which shelf gets walked to. Nobody photocopies the library into a backpack; that is what context-stuffing is. The catalog is small on purpose.

Method: Interpretable Context Methodology (Van Clief & McDermott, arXiv:2603.16021, MIT-licensed).

## The invariants

Every ICM, whatever its form, obeys these. When building or restructuring, enforce all ten:

1. **One folder, one job.** Each folder does a single step or holds a single kind of thing, and states its own purpose in a file inside itself. The structure is the documentation.
2. **A small, stable entry file.** `CLAUDE.md` (or `AGENTS.md`) at the root answers "where am I, where does everything live, where do I go for task X" — and nothing else. Target under ~60 lines. It routes; it never holds content.
3. **Numbering encodes order.** `01_`, `02_`, … where sequence matters. Renaming folders reorders the pipeline — that is the point.
4. **Every folder-level contract is explicit.** A `CONTEXT.md` per working folder: what it reads (inputs), what it does (process), what it writes (outputs), what a human checks. See [assets/templates/stage-CONTEXT.md](assets/templates/stage-CONTEXT.md).
5. **Factory vs. product.** Reference material (rules, voice, schemas, templates — stable across runs) lives structurally apart from working artifacts (outputs, drafts — new every run). Configure the factory once; the product is what each run emits.
6. **Every output is an edit surface.** Intermediate outputs are plain files a human can open, edit, and save before the next step reads them. Nothing moves forward until a person has read the last output.
7. **Load only what the step needs.** An agent executing a step reads its contract, its references, and its inputs — not the whole workspace. 2,000–8,000 tokens per step is the healthy range.
8. **Plain text, linkable, queryable.** Markdown + YAML frontmatter. Links (`[[wikilinks]]` or relative paths) make it a graph; frontmatter labels make it queryable. One home per fact — a link beats a copy.
9. **The filesystem is the state machine.** "Status" is derivable by scanning what exists in output folders. Generated indexes (file maps, logs) are rebuilt by script, never hand-edited.
10. **Instantiate by copying.** New unit of work = copy a template folder, not a blank page. Keep templates in a `_templates/` or `_system/` folder.

## Choose a mode

| Mode | Reach for it when | What it produces | Commitment |
|---|---|---|---|
| **Build** | Nothing exists yet — building from a described process, idea, or problem | A scaffolded workspace | Creates files |
| **Restructure** | An existing folder, repo, or vault needs ICM structure *and the person is ready to actually migrate it* | A migrated, validated workspace | Moves/creates files, human-gated |
| **Map** | The person wants to see what an existing folder would look like as an ICM *before deciding anything* | Inventory + classification + a proposed tree | Read-only, stops at the proposal |
| **Assess** | A folder already claims or attempts ICM structure and the person wants a health-check | A scored walk-test diagnosis | Read-only, no changes |

- **Building from a described process, idea, or problem** → Build mode.
- **An existing folder, repo, or vault that needs ICM structure, and the person is ready to migrate it now** → Restructure mode.
- **The person only wants a preview — "just show me," no commitment yet** → Map mode. Don't run the full Restructure pipeline (propose → approve → migrate) when a preview is all that's asked; that's the fastest way to make this skill feel heavier than the person wanted.
- **A folder that's already ICM-ish and the person wants to know if it's *actually good*** → Assess mode. Don't quietly restructure when asked to assess — a grade is not an invitation to fix.
- **A body of work later agents must edit** (code, markdown, or mixed), inside Build or Restructure → System map form. Read [references/system-map.md](references/system-map.md) after picking the form.

Map and Assess answer different questions and are easy to conflate: Map asks *what would this become*; Assess asks *is what already exists here any good*. If the folder has no CLAUDE.md/CONTEXT.md and no ICM pretensions, it's a Map candidate. If it already has an entry file, numbered stages, or a CONTEXT.md and the person is asking whether it *works*, it's an Assess candidate.

## Build mode

**1. Extract the structure from dialogue.** The structure is already in how the person describes the work — don't impose a shape, surface theirs. Ask (a few at a time, not all at once):

- What is the repeating unit of work? (an episode, a client, a report, a person, a team?)
- Walk me through one run, start to finish. Where do you stop and check something before continuing?
- What stays the same every run (voice, rules, brand, schema) vs. what is new every run?
- What does "done" look like — what artifact leaves the workspace?
- Who else touches this, and what do they need to find without asking you?

Their pauses become stage boundaries. Their "I always check X before Y" become human gates. Their "it always has to sound like / follow Z" becomes factory reference material.

**2. Pick the form.** Read [references/forms.md](references/forms.md) and choose:

| Form | Reach for it when |
|---|---|
| **Pipeline** | The same sequence runs repeatedly, producing a deliverable each run |
| **Umbrella** | Several distinct pipelines share one brand/voice/reference layer |
| **Record library** | The unit is a record (person, client, session) that accumulates, not a run |
| **Knowledge bundle** | The product is navigable knowledge itself (a brain, a wiki, a model of something) |
| **Context map** | The subject is an organization — teams, processes, data, and the links between them |
| **System map** | A folder later agents will edit — nouns, movements, and what a change hits. Method: [references/system-map.md](references/system-map.md) |

Real workspaces mix forms (a record library whose records are mini knowledge bundles; a pipeline that emits into a record library). Compose freely — the invariants hold at every level, recursively.

**3. Scaffold the smallest structure that carries the work.** Copy starters from [assets/templates/](assets/templates/) and fill them in. Do not create folders for stages that don't exist yet, empty "misc" buckets, or speculative depth. Three real stages beat seven imagined ones. If the whole job fits in one saved prompt, say so and don't build a workspace at all.

**4. Write the contracts.** Root `CLAUDE.md` (identity + routing table), root `CONTEXT.md` (the pipeline or schema definition), one `CONTEXT.md` per stage/hub folder, `setup/questionnaire.md` if the factory needs configuring per user. Write inputs as explicit file paths, split into working (this run) and reference (every run).

**5. Validate with the walk test** (below).

## Restructure mode

**1. Inventory before touching.** List the tree. For each area note: what it is, when last touched, what refers to it. Never delete or move in this pass.

**2. Find the hidden form.** Ask the owner (or infer and confirm): what is the repeating unit here? Where does work enter and leave? The mess usually contains a real pipeline, library, or map that grew without a skeleton — extract it, don't replace it. Interview the folder the way you'd interview the person.

**3. Classify every file** into one of five roles:
- **Catalog** — identity/routing (becomes or feeds `CLAUDE.md` / index files)
- **Contract** — describes how a step works (becomes a `CONTEXT.md`)
- **Factory** — stable reference (→ `_shared/`, `_system/`, or `references/`)
- **Product** — run-specific artifacts (→ stage `output/` or record folders)
- **Dead** — stale, duplicated, or superseded (→ propose `_archive/`, never silently delete)

**4. Propose before moving.** Present the target tree and a migration map (old path → new path → role). Get approval. This is a human gate in a method built on human gates — honor it.

**5. Migrate.** Move files, write the entry file and contracts, de-duplicate toward one-home-per-fact (leave a link where the copy lived if anything might reference it). Separate method from instance: if the structure will be reused elsewhere, the blank template lives apart from this filled-in deployment.

**6. Validate with the walk test.**

## Map mode

The fast, low-commitment pass. Someone wants to see what their stuff would look like as an ICM without committing to actually restructuring anything yet — "just show me." **The proposed tree is the deliverable, not step 1 of a bigger pipeline.** Do not continue into approval-and-migrate; that's Restructure mode, a separate, later invocation.

Borrows Restructure mode's steps 1–3 logic, then stops before step 4:

**1. Inventory, read-only.** List the tree cold. For each area, note what it is, when it was last touched (if visible from the filesystem), and what else references it. Do not move, rename, or edit anything — Map mode never touches a file.

**2. Find the hidden form.** Infer the repeating unit and which of the six forms (see [references/forms.md](references/forms.md)) the mess already resembles. This is a cold read, not an interview — where confidence is low, say so by area instead of guessing silently or forcing a single form on a mixed folder.

**3. Classify every file** into one of the same five roles as Restructure mode: catalog / contract / factory / product / dead.

**4. Produce the proposed ICM tree** — a target structure in the chosen form(s), plus a short old-path → new-path → role table. Present it as the answer to "what would this look like," not as a plan awaiting a go-ahead.

**Then stop.** Output shape and a worked example: [references/map-and-assess.md](references/map-and-assess.md).

If the person likes what they see and wants it actually built, that's a fresh invocation of Restructure mode (starting from its step 4, propose) using this tree as the starting point — ask, don't assume the green light carried over.

## Assess mode

The walk test, exposed as its own standalone mode — for a folder that already claims or is attempting ICM structure, when the person wants a health-check without being mid-build or mid-restructure. Produces a **scored diagnosis**, not a rewrite.

**1. Walk it cold.** Act as an agent with no memory of this folder. Open the entry file and try to complete one real task from it in order — do not read the whole tree first and then retroactively score it; that defeats the point of a cold walk.

**2. Run every check** from [The walk test](#the-walk-test) below (plus the System map extra checks, only if the folder is that form) and score each one PASS / PARTIAL / FAIL with concrete evidence. "Looks fine" is not evidence. "The entry file is 340 lines and mixes routing with content" is.

**3. Roll it up**: an overall score, a band (walkable / close / not yet walkable), and a short, cheapest-first fix list naming exact files and exact moves. Report format, scoring bands, and a worked example: [references/map-and-assess.md](references/map-and-assess.md).

**4. Stop.** Assess mode never edits the folder, even for an easy fix. If the person wants the fixes made, that's a fresh invocation of Restructure mode (or a targeted edit) using the diagnosis as input.

## The walk test

Validate any ICM — new or restructured — by walking it cold, as an agent with no memory:

- Open the root. Can you answer *where am I* and *where do I go for the current task* within the entry file plus at most two more reads?
- Pick any stage/node. Does its contract name exact input paths, the job, the output, and the human check?
- Can you state pipeline status purely by scanning what exists in `output/` folders (or node frontmatter)?
- Is any routing file carrying content payload? Move the payload to a shelf; leave a pointer.
- Is any fact stored in two places? Pick one home; link from the other.
- Token check: entry file + one contract + its inputs should land in roughly 2k–8k tokens.
- System map only: can a cold agent answer *what is X* and *what else moves if I change X* from `map/CLAUDE.md` plus one card? Extra checks are in [references/system-map.md](references/system-map.md).

If a step fails, fix the structure — not by explaining more, but by moving or splitting files until the walk works.

## Guardrails

- **Don't over-structure.** The ladder runs: chat → saved prompt/skill → folders + one agent. Only climb when the rung below is genuinely automated and repeating. A workspace for a thing done twice is scaffolding, not architecture.
- **Know where ICM loses.** Real-time multi-agent collaboration, high-concurrency multi-user serving, and automated mid-pipeline branching genuinely need framework code. ICM is for sequential, human-reviewed, repeatable work — which is most knowledge work, but not all of it.
- **Anti-patterns seen in the wild:** duplicated entry files that drift (generate one from the other, or make one a pointer); schema documents that mandate names the actual files stopped using (update the schema or the files — pick one); hand-edits to generated indexes; workshop sessions that produce slides instead of structured data (every working session should end in an artifact the structure can hold); patterns declared top-down (one team complaining is a gripe — the same shape appearing three independent times is structure).

## References

- [references/core.md](references/core.md) — the five design principles, the five-layer context hierarchy, naming conventions, token discipline. Read when writing contracts or when a structural call is contested.
- [references/forms.md](references/forms.md) — the six forms in depth: skeletons, moves, failure modes. Read at step 2 of Build mode or step 2 of Restructure mode.
- [references/system-map.md](references/system-map.md) — audit pipeline for the System map form. Read when that form is chosen.
- [references/map-and-assess.md](references/map-and-assess.md) — output templates, scoring rubric, and worked examples for Map mode and Assess mode. Read at step 4 of Map mode or step 3 of Assess mode.
- [assets/templates/](assets/templates/) — copyable starters: `CLAUDE.md`, workspace `CONTEXT.md`, `stage-CONTEXT.md`, `node.md`, `object.md`, `process.md`, `schema.md`, `questionnaire.md`.
