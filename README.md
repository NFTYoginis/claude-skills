# claude-skills

Three small Claude Code skills, each doing one job well rather than one skill trying to do everything.

## [`model-fit`](./model-fit)

Decide which model tier — cheap/fast, balanced, or frontier — fits a task, a subagent, a pipeline stage, or a batch job, using a fixed decision table instead of a fresh judgment call every time. Also flags when a long-horizon/agentic task needs an externally-imposed stop condition regardless of which tier gets picked — tier and stop-condition are two separate decisions, and skipping the second while maximizing the first is the actual risk.

## [`icm-toolkit`](./icm-toolkit)

A fork of Jake Van Clief's [`icm-architect`](https://github.com/RinDig/icm-architect), built on the **Interpretable Context Methodology** ([Van Clief & McDermott, arXiv:2603.16021](https://arxiv.org/abs/2603.16021), MIT-licensed; community: [Clief Notes](https://www.skool.com/cliefnotes)). Keeps everything the original does (Build mode, Restructure mode, the walk test) and adds two new directly-invokable modes:

- **Map** — a fast, no-commitment preview of what an existing folder would look like as an ICM. Read-only, stops at the proposed tree.
- **Assess** — a standalone health-check on a folder that already claims ICM structure. Scores the walk test with concrete evidence, doesn't touch anything.

Full credit for the method and the original skill to Jake Van Clief and to Van Clief & McDermott's paper — see [`icm-toolkit/LICENSE`](./icm-toolkit/LICENSE) and the fork notice at the top of [`icm-toolkit/SKILL.md`](./icm-toolkit/SKILL.md).

## [`icm-fit`](./icm-fit)

The opposite direction from `icm-toolkit`: instead of grading your own folder structure, this grades whether an *external* candidate — a skill, an MCP server, a hook, a CLI, a library, a hosted service — is actually worth pulling into a system built on ICM. Produces a structured verdict (type, token impact, which context layer it would occupy, which existing workflow would absorb it, quality signals, adopt/watch/skip) instead of a vibes paragraph. Its sharpest guardrail: AI-sounding prose in a candidate's docs is a fact about how it was written, never a fact about whether it works — score the artifact, not the writing.

## Install

Copy any subfolder into `~/.claude/skills/<name>/` (or `.claude/skills/<name>/` inside a project). Claude Code picks it up automatically.

## License

MIT. `model-fit` and `icm-fit` are original to this repo (copyright Gabriel Azoulay, 2026 — see [`LICENSE`](./LICENSE)). `icm-toolkit` carries forward Jake Van Clief's own MIT license from the skill it forks — see [`icm-toolkit/LICENSE`](./icm-toolkit/LICENSE).
