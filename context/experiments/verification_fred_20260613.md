# Fred verification note - 2026-06-13

Scope: checked `context/context.md`, `context/report_notes.md`, `context/experiments.md`, local `context/team_actions_fred_harvey_20260612.md`, and relevant local `../tpu-2026` notes/scripts/branches.

Update after `../tpu-2026` commit `4db577b` (`report evidence and verification`): Fred evidence is now present in the sibling code/evidence repo at `../tpu-2026/agents/evidence_fred_20260612.md` and `../tpu-2026/experiments/evidence/`. It is still not present under `AgentsCW/experiments/team_evidence/` in this checkout.

## 1. Confirmed rows

- Fred R7 hard-medium GRPO: confirmed as GRPO, K=2, seed 0, run root `/home/fredlawrence/tpu-runs/part-i/R7-grpo-k2-hardmedium-20260610_102859`, W&B run id `3v8bx0iz`, `TRAIN_MANIFEST=/home/fredlawrence/tpu-runs/part-i/manifests/gsm8k_train_base_hard_medium.jsonl`, `SAVE_INTERVAL_STEPS=250`, `MAX_TO_KEEP=20`, final checkpoint `ckpts/actor/3452`, and operational completion. Local notes say the run collapsed to immediate-EOS/empty completions: first zero-length train completion at step `457`, persistent zero-length run from step `518`, final train/eval mean length `0`, and final train `actor/train/grad_norm=0`, `actor/train/kl=0`, `actor/train/loss=0`.
- Fred R7 eval caveat: confirmed that `/home/fredlawrence/tpu-runs/part-i/R7-grpo-k2-hardmedium-20260610_102859/eval/r7_best_greedy.csv` is base/no-restore, not trained R7. It has `31/64` exact, `31/64` partial, `1/64` format, `0/64` empty, and must not be reported as trained accuracy.
- Fred hard-mining base pass: confirmed from `../tpu-2026/experiments/evidence/hard_mining_base_gsm8k_train_20260609_220117/summary.md`. Processed 7,473 GSM8K train examples: 3,637 easy, 1,990 medium, 1,846 hard, 0 skipped. Hard+medium training manifest has 3,836 rows at `../tpu-2026/experiments/evidence/manifests/gsm8k_train_base_hard_medium.jsonl`; shared eval manifest has 64 rows at `../tpu-2026/experiments/evidence/manifests/gsm8k_test_seed0_n64.jsonl`.
- Fred R8 hard-medium GRPO: confirmed as GRPO, K=8, seed 0, run root `/home/fredlawrence/tpu-runs/part-i/R8-grpo-k8-hardmedium-20260610_131135`, W&B run id `xnn69ylg`, same hard-medium train manifest, and final trained eval at step `3452`.
- Fred R8 final trained metrics: confirmed from local `../tpu-2026/agents/context.md` and `agents/plan.md`: `/home/fredlawrence/tpu-runs/part-i/R8-grpo-k8-hardmedium-20260610_131135/eval/r8_step3452_greedy.csv`, restored `ckpts/actor` at step `3452`, exact `30/64` (`46.88%`), partial `32/64` (`50.00%`), format `60/64` (`93.75%`), empty `0/64`, mean response length about `180.5` words.
- Fred R8 base/no-restore eval caveat: confirmed that `/home/fredlawrence/tpu-runs/part-i/R8-grpo-k8-hardmedium-20260610_131135/eval/best_greedy.csv` is byte-identical to the R7 base/no-restore CSV, has empty `restored_step`, and should not be reported as trained R8.
- Fred R9 two-stage RLOO K=8: confirmed from `../tpu-2026/agents/evidence_fred_20260612.md` and `../tpu-2026/experiments/evidence/R9-rloo-k8-format-answer-full-20260612_131553/`. Branch/commit `fred` / `a93ea9f387b8c5083607263daf157929482c50cb`; stage 1 W&B `2f7wikq5`, stage 2 W&B `izp0f1u1`; stage 1 `REWARD_PROFILE=format_heavy`, step 500; stage 2 `REWARD_PROFILE=answer_heavy`, final step 3364. Stage 2 best retained exact is tied at steps `2500` and `3000`, both `37/64` (`57.81%`); final step `3364` is `34/64` (`53.12%`), partial `36/64`, format `56/64`, empty `0/64`.

## 2. Corrections needed

- The register should not list Fred R7/R8 as clean `57c6409` provenance without a caveat. Commit `57c6409add0d81bbdb32ca7f4b3e176b4e044068` does not contain the `TRAIN_MANIFEST`/manifest-training support needed by `scripts/run_r8_grpo_k8_hardmedium.sh`. The relevant code was later committed on `fred`/`origin/fred` as `a93ea9f387b8c5083607263daf157929482c50cb`. Since R7/R8 ran on 2026-06-10 and `a93ea9f` was committed on 2026-06-11, safest wording is: based on `57c6409` plus dirty/untracked hard-mining and manifest-training changes later committed as `a93ea9f`; exact launch diff not proven locally.
- Fred R8 `Best retained exact` should not imply a retained-checkpoint sweep. Local evidence only confirms the final trained checkpoint eval at step `3452`, exact `30/64`. Earlier retained checkpoints such as 500/1500/2500 remain unevaluated or not locally evidenced.
- Fred hard-mining counts are now locally verified in `../tpu-2026`, but the register should cite that sibling evidence path explicitly if used.
- Fred R9 should be upgraded from context-only/stage-failed wording. Current evidence says the two-stage RLOO K=8 run completed stage 2 and has restored checkpoint evals at steps `500`, `1000`, `2000`, `2500`, `3000`, and `3364`. Keep a caveat that reward-profile/launcher changes were dirty/uncommitted at launch and paired/bootstrap uncertainty is still needed.
- The register's base comparisons for R8 and R9 are valid for the shared held-out `gsm8k_test_seed0_n64.jsonl` sample, using base/no-restore `31/64` CSVs now present under `../tpu-2026/experiments/evidence/`.

## 3. Missing artefacts

- Still absent from `AgentsCW`: `experiments/team_evidence/evidence_fred_20260612.md`. The available evidence is in the sibling `../tpu-2026` repo.
- Still missing for R7: a restored trained-checkpoint eval CSV. The likely salvage eval should restore `--ckpt-dir /home/fredlawrence/tpu-runs/part-i/R7-grpo-k2-hardmedium-20260610_102859/ckpts --step 250`.
- Still missing for R8: retained-checkpoint evals such as steps 500/1500/2500 if a best-checkpoint sweep is desired. Only final step `3452` is locally evidenced.
- Still missing for all Fred runs: exact TPU type/zone for the personal `federico-tpu` VM and direct W&B remote-status confirmation. The audit relies on local logs/metadata and W&B ids/URLs recorded there.
- Still needed before strong performance claims: paired/bootstrap uncertainty on the 64-prompt eval sample.

## 4. Report-ready claims from my runs

- Hard-mining can now be used as report-ready manifest provenance from `../tpu-2026`: 7,473 train examples processed; 3,836 hard+medium selected; 64-prompt held-out eval manifest present.
- R8 can be used cautiously as a negative/stability result if the report cites it as final-checkpoint-only: K=8 GRPO on the hard/medium manifest avoided the R7 empty-output collapse at final step `3452`, with exact `30/64` versus same held-out base/no-restore `31/64`, and format `60/64`.
- R7 can be used as a training-collapse diagnostic, not as an accuracy row, unless a trained checkpoint eval is recovered. The supported claim is that K=2 GRPO on the hard/medium manifest collapsed to empty/immediate-EOS completions despite operational training completion.
- R9 can be used cautiously as a completed two-stage RLOO K=8 result: best retained exact `37/64` at steps `2500`/`3000`; final exact `34/64` at step `3364`; base `31/64`. State the checkpoint-selection caveat and avoid strong claims until uncertainty is computed.

## 5. Claims to avoid or qualify

- Do not claim a trained R7 exact-accuracy result from `r7_best_greedy.csv`; it is base/no-restore.
- Do not claim R8 improved exact accuracy over base; final trained R8 is `30/64`, below the base/no-restore `31/64` on the held-out eval.
- Do not call R8's `30/64` a best checkpoint. It is the only confirmed trained checkpoint eval locally.
- Do not present the Fred rows as clean `57c6409` runs without noting dirty/untracked manifest-training changes later committed as `a93ea9f`.
- Do not present R9 as a failed stage-2 run; this is now superseded by `../tpu-2026` evidence showing stage 2 completed and was evaluated.
- Do not claim R9 definitively improves general performance beyond this 64-prompt sample until paired/bootstrap uncertainty is computed.

## 6. Recommended next action

Update `context/experiments.md` Fred rows to reflect the new `../tpu-2026` evidence: keep R7 as collapse/no trained accuracy, keep R8 final-only at `30/64`, mark hard-mining counts as verified via sibling evidence, and upgrade R9 to completed two-stage RLOO K=8 with best retained `37/64` and final `34/64`. Next useful work is a restored R7 step-250 eval, optional R8 retained-checkpoint evals, and bootstrap/paired uncertainty for R9 and any table rows used in the report.
