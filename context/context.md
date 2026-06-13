# AgentsCW Context

`AgentsCW` is the main report workspace. The sibling `../tpu-2026` repository is the active Part I training/evaluation codebase and should be treated as part of the Section I work.

## Current State

- Report draft: `report/main.tex`.
- I.4 theory section: mostly finished.
- Part II workflow walkthrough: checked.
- I.1-I.3 practical write-up: still WIP; needs tables, figures, concise prose, and final links.
- Part II critique/improvements: drafted; still needs page-limit and source-alignment checks.
- Figure assets present for Part II: `report/figures/lg_diagram.jpg`, `report/figures/adaptive_planning_langgraph.mmd`.
- Expected Part I figures are still missing unless generated later: `baseline_reward_kl.pdf`, `reward_kl_variants.pdf`, `grpo_diagnostic.pdf`.

## File Roles

- `context/coursework.md`: authoritative coursework requirements converted from the brief.
- `context/section_i_context.md`: older, detailed Part I setup and baseline implementation notes.
- `context/section_ii_context.md`: detailed Part II source study and critique candidates.
- `context/report_notes.md`: compact report-ready facts and placeholders.
- `context/plan.md`: current action checklist.
- `context/deep_research.md`: interpretation angles for discussion.
- `context/writing_style.md`: Fred's preferred report prose style.
- `../tpu-2026/agents/*`: live Part I run notes, metrics, decisions, and evidence paths.

## Source Priority

For requirements:

1. `context/coursework.md`
2. `context/coursework.pdf`

For Part I run facts:

1. `../tpu-2026/agents/context.md`
2. `../tpu-2026/agents/report_notes.md`
3. saved logs, metadata, checkpoints, CSVs, and figures
4. `context/section_i_context.md` only for implementation/background details

For Part II:

1. `context/section_ii_context.md`
2. local checkout `external/lg`
3. report draft `report/main.tex`

## Report Constraints

- Submit one PDF.
- I.1-I.3 practical: maximum 3 pages.
- I.4 theory: no page limit, but concise LaTeX.
- Part II: maximum 2 pages excluding the workflow diagram.
- Every URL in the PDF must be clickable.
- I.1-I.3 can discuss shared team experiments; I.4 and Part II are individual.

## Current Part I Direction

The original local KL-sweep idea was superseded. The live evidence now supports a GRPO/RLOO/K diagnostic story:

- K=2 GRPO full baseline completed but underperformed the base model on the 64-prompt greedy eval.
- K=2 RLOO completed but performed worse and developed many empty responses.
- K=8 GRPO debug/medium runs looked healthier, but only make full-run claims if full-run evidence exists.

## Do Not

- Do not treat `report/main.tex` TODOs as facts.
- Do not paste context notes directly into the report.
- Do not claim completed results without saved evidence.
- Do not overwrite the mostly finished I.4 or checked Part II walkthrough without a concrete reason.
