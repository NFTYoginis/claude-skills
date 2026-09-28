# icm-toolkit

A fork of Jake Van Clief's [`icm-architect`](https://github.com/RinDig/icm-architect) skill. Both are built on **Interpretable Context Methodology (ICM)** — folder structure as agent architecture — and both are MIT-licensed; credit for the method and the original Build/Restructure modes goes to Van Clief & McDermott and to the original skill. This fork keeps everything the original does and adds two new directly-invokable modes: **Map** and **Assess**.

ICM replaces orchestration code with structure: numbered folders carry sequencing, hierarchy carries context scoping, plain markdown files carry state. One agent, reading the right files at the right moment, does the work of a multi-agent framework — and a human can open any folder and see exactly what state the system is in.

The workspace is a library. The routing files are the catalog: small, stable, they point at everything and store almost nothing. One librarian — one model — walks the building, and the question decides which shelf gets walked to.

- Paper: [Interpretable Context Methodology: Folder Structure as Agent Architecture](https://arxiv.org/abs/2603.16021) (Van Clief & McDermott)
- Original skill: [icm-architect](https://github.com/RinDig/icm-architect) (Jake Van Clief)
- Community: [Clief Notes](https://www.skool.com/cliefnotes)

## What it does

Four modes:

- **Build** — extracts the structure already present in how you describe your work (the stages, the human gates, what's stable vs. per-run), picks one of six proven forms, and scaffolds the smallest workspace that carries it.
- **Restructure** — audits an existing folder, classifies every file (catalog / contract / factory / product / dead), proposes a migration map for approval, then migrates and validates.
- **Map** *(new in this fork)* — a fast, no-commitment preview. Runs the same inventory-and-classify pass as Restructure, produces the proposed ICM tree, and stops — no migration, no approval pipeline. For "just show me what this would look like."
- **Assess** *(new in this fork)* — the walk test exposed as its own standalone mode. Cold-walks a folder that already claims ICM structure and produces a scored PASS/PARTIAL/FAIL diagnosis with a cheapest-first fix list. For a health-check on something already built, without being mid-build or mid-restructure.

Six forms, one skeleton: **Pipeline** (production line), **Umbrella** (portfolio of pipelines), **Record library** (people/clients/sessions), **Knowledge bundle** (a navigable brain), **Context map** (an organization as a graph), **System map** (a folder later agents will edit — nouns, movements, change-impact). They compose and recurse.

Build, Restructure, and Assess all validate with the **walk test**: an agent with no memory must orient, act, and report status from the files alone.

## Install

**Claude Code:** copy this folder to `~/.claude/skills/icm-toolkit/` (or `.claude/skills/icm-toolkit/` inside a project), then ask Claude to "ICM this" / "map my folders as an ICM" / "grade my setup" / "structure this for agents".

**Claude apps:** upload `icm-toolkit.skill` (build it with the skill-creator packager, or zip this folder's contents) via Settings → Capabilities.

## Layout

```
icm-toolkit/
├─ SKILL.md              the method: invariants, four modes (Build/Restructure/Map/Assess), walk test
├─ references/
│  ├─ core.md            five principles, five-layer hierarchy, naming, token discipline
│  ├─ forms.md           the six forms in depth: skeletons, moves, failure modes
│  ├─ system-map.md      audit pipeline for the System map form
│  └─ map-and-assess.md  output templates, scoring rubric, worked examples for Map/Assess
└─ assets/templates/     copyable starters: CLAUDE.md, CONTEXT.md, stage contract,
                         node card, object/process cards, schema, questionnaire
```

MIT licensed, like the protocol it serves. See `LICENSE` (original copyright retained).
