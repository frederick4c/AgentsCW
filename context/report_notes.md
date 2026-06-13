# Report Notes

This file is for compact facts that can be lifted into `report/main.tex` after being turned into prose. Detailed reasoning belongs in `deep_research.md`; detailed source study belongs in `section_i_context.md` or `section_ii_context.md`.

## Draft Status

- I.1: scaffold/TODOs remain.
- I.2: scaffold/TODOs remain.
- I.3: scaffold/TODOs remain.
- I.4: worked LaTeX answers are mostly finished.
- Part II: diagram, draft walkthrough, and drafted critique/improvements are present.
- References/appendix: still need cleanup.

## Report-Ready Part I Metrics

Use exact evidence paths from `../tpu-2026/agents` before finalising.

| Eval | Exact | Partial | Format | Empty |
| --- | ---: | ---: | ---: | ---: |
| Base greedy | 31/64 (48.44%) | 31/64 (48.44%) | 1/64 (1.56%) | 0/64 |
| R1 GRPO K=2 step 2000 | 24/64 (37.50%) | 26/64 (40.62%) | 28/64 (43.75%) | 0/64 |
| R1 GRPO K=2 step 3364 | 12/64 (18.75%) | 12/64 (18.75%) | 23/64 (35.94%) | 0/64 |
| R3 RLOO K=2 step 2500 | 6/64 (9.38%) | 7/64 (10.94%) | 16/64 (25.00%) | 45/64 |
| R3 RLOO K=2 step 3364 | 1/64 (1.56%) | 1/64 (1.56%) | 5/64 (7.81%) | 58/64 |
| D3 GRPO K=8 step 50 | 31/64 (48.44%) | 32/64 (50.00%) | 8/64 (12.50%) | 0/64 |
| D4 GRPO K=8 step 500 | 34/64 (53.12%) | 35/64 (54.69%) | 54/64 (84.38%) | 0/64 |

D3 and D4 are diagnostics, not full controlled runs. Do not present D4 as the final improved checkpoint unless later full-run evidence supports that story.

## Required Part I Fill-Ins

- GitLab repository URL.
- W&B/TensorBoard/log URLs.
- Team member names.
- Exact R1 commit and checkpoint evaluated.
- R1 wall-clock duration, if available.
- Whether final table uses R1 final checkpoint, R1 best retained checkpoint, or both.
- Bootstrap CI or other uncertainty measure.

## Part II Facts

- Studied commit: `d7d0592a714e4cc01c97f1b77afdd57a208b18db`.
- Commit URL: `https://github.com/borisbolliet/cmbagent_lg/tree/d7d0592a714e4cc01c97f1b77afdd57a208b18db`.
- Diagram file: `report/figures/lg_diagram.jpg`.
- Mermaid source: `report/figures/adaptive_planning_langgraph.mmd`.
- Current critique triad: LLM-only trigger; failed steps halt outer loop; unvalidated tail rewrites.
- Current improvement triad: structured trigger policy; failure-aware replanning; validated/budgeted tail replacement with better persistence.

## Figure Needs

- `report/figures/baseline_reward_kl.pdf`: baseline mean reward and KL.
- `report/figures/reward_kl_variants.pdf`: shared-axis reward/KL comparison.
- `report/figures/grpo_diagnostic.pdf`: likely empty responses, response length, KL-vs-accuracy, or format-vs-exact accuracy.

## Final Cleanup Checklist

- Remove every `\TODO`.
- Replace placeholder URL macros.
- Ensure all report URLs are clickable.
- Compile and inspect page limits.
- Keep appendices non-essential.
