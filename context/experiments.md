# Team Experiment Register - 2026-06-13

This register collates the submitted evidence files under `experiments/team_evidence/`. It is a provenance and triage document for report analysis, not a final statistical analysis. Claims below should be treated as report-ready only where the row has a restored checkpoint evaluation, a clear branch/commit, and enough configuration detail to make the comparison reproducible.

## Evidence sources

| Source file | Owner | Notes |
|---|---:|---|
| `evidence_baron_20260612.md` | Baron | Internal runs from the original coursework branch and later deterministic work. Includes K=2 GRPO/RLOO, K=8 diagnostics, and R5 full K=8. |
| `evidence_fred_20260612.md` | Fred | Internal hard-question mining and hard/medium manifest runs. Includes R8 K=8 hard-medium and R9 staged reward attempt. |
| `evidence_harvey_20260612.md` | Harvey | Internal deterministic RLOO K=2 and K=8 runs. K=8 has strong retained-checkpoint result; K=2 collapses late. |
| `evidence_basia_20260612.md` | Basia/Barbara | External collaborator evidence for KL, length, empty-penalty, and G8+microbatch diagnostics. |
| `evidence_funmi_20260612` | Funmi | External collaborator evidence. File contains repeated pasted audits; the fuller middle section is treated as canonical for variant runs. |
| `evidence_rowan_20260612.md` | Rowan | External collaborator K sweep/reward-reweight evidence. Includes K=16 partial run and pending reward-reweight eval opportunity. |

## Executive summary

The strongest supported internal story is that K=2 training is unstable for this setup, while K=8 materially improves stability and format adherence. The gains in exact GSM8K accuracy are modest on the 64-question greedy evaluation set: Baron R5 GRPO K=8 reaches 35/64 at its best retained checkpoint and 32/64 at the final checkpoint, against a base reference of 31/64. Harvey's deterministic RLOO K=8 reaches 35/64 at its best retained checkpoint but falls to 26/64 final. In contrast, Baron/Harvey RLOO K=2 and Baron GRPO K=2 degrade badly by the end of training, with RLOO K=2 showing severe empty-response collapse.

Fred's hard-question mining is valuable for method coverage and dataset provenance, but the completed K=8 hard-medium run does not beat the base exact score at final evaluation. External collaborators add useful coverage: KL, length, empty-penalty, LoRA, learning-rate, K=4/K=16, and reward-reweight attempts. These should be used as supporting context unless their evaluation protocol and provenance are reconciled with the internal runs.

## Status labels

| Label | Meaning |
|---|---|
| Report-ready | Restored checkpoint eval exists, core config is known, and the result can support a cautious claim. |
| Context only | Useful for diagnostics or broader coverage, but missing a clean comparison, final eval, or complete provenance. |
| Not usable | Training/eval missing, checkpoint missing, branch/config ambiguous, or run failed before producing analysable output. |
| Pending | Specific follow-up could make the run useful. |

## Internal team canonical runs

| Owner | Run | Branch / commit | Method | K | Seed | Eval status | Best retained exact | Final exact | Empty signal | Use |
|---|---|---|---|---:|---:|---|---:|---:|---:|---|
| Baron | D1 GRPO debug | `e3ebeef` | GRPO | 2 | 0 | step 50 restored | 30/64 | 30/64 | not flagged | Context only: auth/checkpoint/debug gate. |
| Baron | D2 RLOO debug | inferred `be631b2` | RLOO | 2 | 0 | step 50 restored | 30/64 | 30/64 | not flagged | Context only: RLOO accepted by code path. |
| Baron | R1 GRPO full | `4339ba8` | GRPO | 2 | 0 | retained checkpoints restored | 24/64 at step 2000 | 12/64 at 3364 | 0/64 final | Report-ready negative: K=2 over-optimises format and hurts exact accuracy. |
| Baron | R3 RLOO full | `99cb7f8` | RLOO | 2 | 0 | retained checkpoints restored | 6/64 at step 2500 | 1/64 at 3364 | 58/64 final | Report-ready negative: RLOO K=2 collapses to empty outputs late. |
| Baron | D3 GRPO K=8 debug | `99cb7f8` | GRPO | 8 | 0 | step 50 restored | 31/64 | 31/64 | 0/64 | Context only: K=8 debug passed without OOM/collapse. |
| Baron | D4 GRPO K=8 medium | `99cb7f8` | GRPO | 8 | 0 | retained checkpoints restored | 34/64 at step 500 | 34/64 at step 500 | 0/64 | Context/diagnostic: justified full K=8 run. |
| Baron | R5 GRPO K=8 full | `820fad6` | GRPO | 8 | 0 | all retained checkpoints restored | 35/64 at step 3250 | 32/64 at 3364 | 0/64 | Report-ready positive internal result, but improvement is modest and needs uncertainty. |
| Baron | R6 deterministic K=2 attempt | `57c6409` | GRPO | 2 | 0 | no local eval found | missing | missing | missing | Not usable until eval is found or rerun. |
| Fred | Hard mining base pass | `57c6409` plus mining script | base inference | n/a | 0 | mining complete | n/a | n/a | n/a | Report-ready for manifest provenance: 1846 hard, 1990 medium, 3637 easy from GSM8K train. |
| Fred | R7 hard-medium GRPO | `57c6409`, dirty/untracked mining changes | GRPO | 2 | 0 | trained eval not found | missing | missing | collapse seen in logs | Context only unless trained checkpoint eval is recovered. |
| Fred | R8 hard-medium GRPO | `57c6409` | GRPO | 8 | 0 | final checkpoint restored | 30/64 final | 30/64 at 3452 | 0/64 reported | Report-ready with caveat: hard-medium K=8 final is below base 31/64 exact. |
| Fred | R9 stage 1 format-heavy | `a93ea9f`, dirty reward/launcher changes | RLOO / staged reward | 8 | 0 | stage 1 eval restored | 30/64 at 500 | stage 2 failed | 0/64 reported | Context only: formatting gate passed; stage 2 failed before training. |
| Harvey | R7 RLOO K=2 deterministic | `71aab87` | RLOO | 2 | 0 | retained checkpoint evidence committed | 30/64 at step 500 | 1/64 at 3364 | 58/64 final | Report-ready for RLOO K=2 collapse, but original run root/logs not local. |
| Harvey | R7 RLOO K=8 deterministic | `e3d69a1` | RLOO | 8 | 0 | retained checkpoints restored | 35/64 at step 2000 | 26/64 at 3364 | 0/64 | Report-ready for K=8 stabilising RLOO relative to K=2, with late degradation. |
| Harvey | Older R6 RLOO variants | `8a0f7f2` | RLOO | 2/8 | unclear | no eval found | missing | missing | missing | Not usable. |

## External collaborator runs

| Owner | Run / W&B ID | Branch / commit | Method | K | Seed | Eval status | Best / final exact | Use |
|---|---|---|---|---:|---:|---|---|---|
| Basia | `8rmv0hgg` KL beta 1e-6 | `kl-control-bk` / `5e7d8f5` | GRPO, beta 1e-6 | 2 | 42 | final restored | final 33/64, base 33/64, format 0/64 | Report-ready external context: accuracy maintained but format collapses. |
| Basia | `oet2tfjd` KL beta 0.32 | `kl-control-bk` / `5e7d8f5`, dirty unknown | GRPO, beta 0.32 | 2 | 42 | final restored | final 0/64 | Report-ready external context: high-beta setting fails completely. |
| Basia | `jcp0b5cy` length penalty G2 | `reward-length-bk` / `f50d731`, dirty unknown | GRPO + length penalty | 2 | 42 | final restored | final 3/64 | Report-ready external context: length penalty alone does not rescue K=2. |
| Basia | `cyay16mj` length penalty G8 bs1 | `reward-length-on-g8-bk` / `89cde30` | GRPO + length penalty | 8 | 42 | step/final evals restored | final 32/64; control step 500 38/64 | Report-ready external context, but microbatch changed so comparison is not clean. |
| Basia | `dsi65u1z` empty penalty G2 | `empty-penalty-bk` / `2dac639`, dirty unknown | GRPO + empty penalty | 2 | 42 | step/final JSONLs committed | final 9/64; step 2000 4/64 | Context/report-ready with caveat: helps empty collapse but still weak exact accuracy. |
| Funmi | `jgs4c6kl` baseline seed42 | `baseline-fls` / `7e696c4` | GRPO | 2 | 42 | retained checkpoints restored | best 18/64 at 2000; final 2/64 | Report-ready external negative baseline for K=2 collapse. |
| Funmi | `hozux9t6` lr1e5 seed42 | `learning-rate-fls` / `99059ce7` | GRPO, LR variant, K provenance uncertain | probably 8 | 42 | retained checkpoints restored | best 20/64; final 19/64 | Context only until K/config provenance is settled. |
| Funmi | `aoz8dtkp` LoRA rank128 alpha128 | `lora-rank128-alpha128-fls` / `99059ce7` | GRPO + LoRA capacity | 8 | 42 | W&B final metrics only | restored exact missing | Not report-ready for accuracy. |
| Funmi | `v5cvlwkm` LoRA rank128 alpha128 G2 | `lora-rank128-alpha128-fls` / `762bb69` | GRPO + LoRA capacity | 2 | 42 | final restored | final 21/64, base 34/64 | Report-ready external context: capacity change does not prevent degradation. |
| Rowan | `dvsaxxyh` K=8 baseline | inferred `n-generations-8`, commit missing | GRPO | 8 | 42 | no local eval/checkpoint | formal eval missing | Context only; may duplicate `kosfzg6t`. |
| Rowan | `kosfzg6t` intended K=16 but actually K=8 | commit missing | GRPO | 8 | 42 | no local eval/checkpoint | formal eval missing | Context only; byte-identical to `dvsaxxyh`. |
| Rowan | `x4j7yhdp` K=4 | `n-generations-4` / `1db3d25` | GRPO | 4 | 42 | checkpoint deleted, no eval | missing | Not usable except W&B reward trend. |
| Rowan | `xqbl406c` K=16 | `n-generations-16` / `17c90d5` | GRPO | 16 | 42 | step 2500 restored before OOM | 36/64 at step 2500 | Partial report-ready external context: promising but failed/OOM before final. |
| Rowan | `smuzmoal` reward reweight | `reward-reweight` / `24583db` plus reward change `37847c1` | GRPO, reweighted reward | 8 | 42 | checkpoints retained; no accuracy eval yet | pending | Pending: high-priority eval candidate. |

## Best-supported comparisons

### Internal comparisons

| Comparison | Evidence | Current conclusion | Caveat |
|---|---|---|---|
| GRPO K=2 vs base | Baron R1 vs base greedy | R1 best retained 24/64 and final 12/64, both below base 31/64. Format improves but exact reasoning degrades. | Single seed, 64 prompts, needs paired uncertainty before final wording. |
| RLOO K=2 vs base | Baron R3 and Harvey R7 K=2 | RLOO K=2 is unstable and collapses late; final exact 1/64 in both Baron/Harvey-style evidence. | Baron and Harvey runs are not identical branches; Harvey same-manifest base missing. |
| GRPO K=8 vs base | Baron R5 | K=8 is much healthier: best 35/64 and final 32/64 against base 31/64, no empty collapse. | Improvement is small; best-checkpoint selection needs to be described honestly. |
| RLOO K=8 vs RLOO K=2 | Harvey R7 K=8 vs K=2 | Increasing K stabilises RLOO: K=8 best 35/64 with no empty collapse, while K=2 final has 58/64 empty. | K=8 final still drops to 26/64, so checkpoint choice matters. |
| Hard/medium mining | Fred mining + R8 | Mining produced a reproducible hard/medium manifest, but R8 final 30/64 does not beat the base 31/64. | R7 trained eval missing; R8 retained-checkpoint eval missing. |

### External coverage comparisons

| Theme | Evidence | What it adds | Caveat |
|---|---|---|---|
| KL sensitivity | Basia beta 1e-6 and 0.32 | Shows regularisation strength can preserve base-level exact with format collapse or destroy accuracy entirely. | Need curves and careful explanation of beta effects before using strongly. |
| Reward shaping | Basia length/empty penalty, Rowan reward reweight pending | Reward changes can change format/empty behaviour but do not automatically improve exact reasoning. | Some dirty status unknown; reward-reweight accuracy still pending. |
| K sweep | Baron/Harvey K=8, Rowan K=16/K=4 | Larger K appears to improve stability up to K=8 and may help at K=16, but K=16 hit OOM. | K=16 is an intermediate failed-run result; K=4 has no eval. |
| LoRA/LR variants | Funmi | Larger LoRA capacity and LR changes do not solve the main issue by themselves. | LR K provenance uncertain; one LoRA run lacks restored exact eval. |

## Candidate report claims

The following are the safest claims to draft around, subject to final paired/bootstrap analysis:

1. In this implementation, K=2 training can make the model worse despite improving format-related reward. Baron R1 is the cleanest GRPO example: exact accuracy falls from base 31/64 to 24/64 at the best retained checkpoint and 12/64 at the final checkpoint.
2. RLOO with K=2 is particularly unstable in the collected runs. Baron R3 and Harvey K=2 both end near 1/64 exact with many empty outputs.
3. Increasing generations to K=8 is the strongest practical stabiliser observed internally. Baron R5 GRPO K=8 has no empty outputs and reaches 35/64 exact at its best retained checkpoint; Harvey RLOO K=8 reaches the same best exact score.
4. The exact-accuracy improvement from K=8 remains modest on the 64-prompt evaluation. The final Baron R5 checkpoint is 32/64 vs base 31/64; the best retained checkpoint is 35/64. The report should present uncertainty and avoid overstating the effect.
5. Hard-question mining produced a clean hard/medium training subset, but Fred R8 does not yet show a final exact-accuracy improvement over base. It is useful as an attempted improvement and negative result.
6. External collaborator runs broaden the failure-mode map: KL strength, reward shaping, LoRA capacity, learning rate, and larger K all affect stability, but none is a simple guaranteed fix.

## Claims to avoid or qualify

- Do not say K=8 definitively improves GSM8K accuracy without uncertainty. The effect is small and checkpoint-dependent.
- Do not compare external-team percentages as if every run used the same code, branch, seed, checkpoint policy, and eval manifest unless those are reconciled.
- Do not treat Rowan K=16 as a completed full run. It failed/OOMed after step 2991; only the step-2500 checkpoint has an evaluated accuracy.
- Do not use Fred R7 accuracy until a trained-checkpoint eval is recovered. The copied CSV appears to be base/mixup rather than R7-trained.
- Do not use Funmi `lr1e5_seed42` as a pure learning-rate ablation until the K/config provenance is resolved.
- Do not use `baron_k8` as a separate branch provenance claim unless the exact pushed branch/run link is verified.

## Fred verification addendum - 2026-06-13

Updated verification against `../tpu-2026` commit `4db577b` found Fred's evidence bundle at `../tpu-2026/agents/evidence_fred_20260612.md` and `../tpu-2026/experiments/evidence/`. The fuller local note is at `context/experiments/verification_fred_20260613.md`, but `context/` is currently ignored for new files by `.gitignore`.

- R7/R8 provenance should be treated as `57c6409add0d81bbdb32ca7f4b3e176b4e044068` plus dirty/untracked hard-mining and manifest-training changes later committed on `fred`/`origin/fred` as `a93ea9f387b8c5083607263daf157929482c50cb`. Clean `57c6409` alone did not contain `TRAIN_MANIFEST`/manifest-training support.
- Hard-mining counts are now verified via `../tpu-2026`: 7,473 GSM8K train examples processed; 3,637 easy, 1,990 medium, 1,846 hard, 0 skipped; hard+medium manifest has 3,836 rows and shared eval manifest has 64 rows.
- R8 is confirmed only as a final trained checkpoint eval, not a retained-checkpoint sweep: run root `/home/fredlawrence/tpu-runs/part-i/R8-grpo-k8-hardmedium-20260610_131135`, W&B id `xnn69ylg`, restored step `3452`, exact `30/64`, partial `32/64`, format `60/64`, empty `0/64`.
- R7 is confirmed as a training-collapse diagnostic, not an accuracy row: run root `/home/fredlawrence/tpu-runs/part-i/R7-grpo-k2-hardmedium-20260610_102859`, W&B id `3v8bx0iz`, K=2 GRPO on the hard/medium manifest, final checkpoint `3452`, first zero-length train completion at step `457`, persistent zero-length run from step `518`, and no restored trained eval found.
- The `r7_best_greedy.csv` and R8 `best_greedy.csv` files are base/no-restore evals with `31/64` exact and must not be labelled trained R7/R8.
- R9 is now verified as a completed two-stage RLOO K=8 run, not a failed stage-2 attempt: run root `/home/fredlawrence/tpu-runs/part-i/R9-rloo-k8-format-answer-full-20260612_131553`, branch/commit `fred` / `a93ea9f387b8c5083607263daf157929482c50cb`, stage 1 W&B `2f7wikq5`, stage 2 W&B `izp0f1u1`; best retained exact `37/64` at steps `2500`/`3000`, final exact `34/64` at step `3364`, base `31/64`. Use with checkpoint-selection and uncertainty caveats.

## Missing artefacts and follow-ups

### High priority for the final report

1. Compute paired/bootstrap uncertainty for the main internal comparison: base vs Baron R5 step 3250 and step 3364, plus ideally Baron R1/R3 and Harvey K8.
2. Decide fixed-step vs best-checkpoint reporting. If using best checkpoint, state that checkpoint selection used the retained validation/eval sweep and report final checkpoint separately.
3. Export or collect the scalar curves needed for figures: reward components, KL, completion length, empty counts if available, and exact/format eval by checkpoint.
4. Build one clean results table for the report with only report-ready rows.
5. Verify whether external collaborator results can be cited in the team report and how to phrase them without implying ownership of their runs.

### Owner-specific gaps

| Owner | Gaps |
|---|---|
| Baron | Bootstrap CIs for R5; TensorBoard scalar export for R5; clarify `baron_k8` branch provenance; locate R6 eval or mark unusable. |
| Fred | Recover R7 trained eval if possible; eval R8 retained checkpoints; export TensorBoard scalars; record R9 stage-2 failure clearly. |
| Harvey | Same-manifest base eval for RLOO K=2/K=8; K=8 scalar export; original K=2 run root/logs if still available. |
| Basia | Provide missing baseline `jgs4c6kl` and G8-rerun `4i8lcitv` JSONLs if used for bootstrap; resolve dirty statuses. |
| Funmi | Resolve K provenance for `lr1e5_seed42`; recover restored exact eval for `aoz8dtkp`; provide metadata/events if available. |
| Rowan | Evaluate `smuzmoal` retained checkpoints; verify if K=16 checkpoint/eval files still exist; avoid using K=4 for accuracy. |

## Recommended report table subset

For the main report, keep the table compact and favour the internally controlled evidence:

| Row | Why include |
|---|---|
| Base greedy reference, 31/64 | Anchor for all internal claims. |
| Baron R1 GRPO K=2 best/final | Shows format optimisation can hurt exact reasoning. |
| Baron R3 RLOO K=2 best/final | Shows severe RLOO K=2 empty collapse. |
| Baron R5 GRPO K=8 best/final | Main positive internal result. |
| Harvey RLOO K=8 best/final | Cross-check that K=8 also stabilises RLOO. |
| Fred R8 hard-medium final | Negative result for hard-question mining. |
| Optional external rows: Basia KL/length, Rowan K=16 step2500, Funmi baseline | Use only if space allows and caveats are clear. |

## Figure candidates

1. Accuracy and format by checkpoint for Baron R1, Baron R5, Harvey RLOO K=2, and Harvey RLOO K=8.
2. Empty-response count by checkpoint for RLOO K=2 vs K=8.
3. Reward-component/KL traces for Baron R1 vs R5, if scalar exports are available.
4. Hard-mining composition bar: easy/medium/hard counts from Fred's base mining pass.
5. Optional external coverage heatmap: intervention vs final/best exact score, with caveat markers for failed or incomplete runs.

## Immediate next analysis task

Prepare a `report/data/` or `experiments/analysis/` table from the report-ready rows only, then compute paired/bootstrap confidence intervals for the base-vs-R5 comparison and the RLOO K=2-vs-K=8 comparison. The current register is enough to choose which rows belong in that table; it is not yet a substitute for the final numeric analysis.
