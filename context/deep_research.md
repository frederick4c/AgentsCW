# Interpretation Notes

These notes are for explaining results, not for storing exact run facts. Use `report_notes.md` for compact metrics and `../tpu-2026/agents` for evidence paths.

## Part I Thesis

The practical section can be framed as an honest controlled negative result plus a stability diagnostic:

- K=2 GRPO was operationally reproduced but did not beat the base model on the fixed 64-prompt greedy eval.
- K=2 RLOO was theoretically motivated but collapsed more severely, with many empty responses.
- K=8 GRPO diagnostics suggest larger groups may stabilise early training, but full-run claims require full-run evidence.

## Why K=2 Was Fragile

With `K=2`, group-relative advantages are based on only two sibling rewards. This makes the signal sensitive to reward ties, reward parsing, and shaping terms. If both siblings get similar rewards, the centred advantage is weak; if formatting reward dominates correctness, the model can optimise tags while losing arithmetic quality.

R1 fits this pattern: format compliance improved, exact accuracy fell below base, and the best retained checkpoint was earlier than the final checkpoint.

## Why RLOO Was Still Worth Trying

RLOO is a principled leave-one-out baseline comparison for small groups. It preserves pairwise reward-difference information differently from standard GRPO, so it was a reasonable Section I.3 variant.

The observed R3 result should be written as a contradicted prediction: the implementation ran and restored, but generation quality collapsed. That makes empty-response rate and qualitative output checks essential diagnostics.

## Failure Modes To Discuss

Reward-format over-optimisation:

- formatting improves while numeric correctness worsens;
- reward curves alone can look healthier than external greedy accuracy.

Empty-response collapse:

- R3 has many empty responses by retained checkpoints;
- aggregate rewards or KL curves may not expose this enough on their own.

KL/clip mismatch:

- R1 KL rose substantially before the final checkpoint;
- `pg_clipfrac` stayed at `0.0`, so clipping did not flag the degradation.

## Good Diagnostic Figure Candidates

- Empty responses by checkpoint for R1/R3/D4.
- Exact accuracy versus format accuracy by checkpoint.
- KL versus exact accuracy for R1/R3.
- Mean response length by checkpoint, especially to show R3 collapse.

## Part II Thesis

The adaptive-planning branch is genuinely closed-loop because execution feedback can rewrite future steps. Its weakness is the control policy: adaptation is an LLM judgement with little explicit objective, limited history, no post-replan validation, and no outer recovery path after step failure.

This supports the current Part II critique and improvements without needing more source detail in the short handoff notes. Use `section_ii_context.md` for code anchors.
