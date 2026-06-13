# Team Artefact Actions — Fred and Harvey (2026-06-12)

Written by Baron's workflow after a full audit of the submission repo, this fork, and the W&B project. Each item states WHAT to provide, WHY it is needed, and WHERE to put it. The coursework PDF requires the report's accuracy table to carry uncertainty (bootstrap CI over the 64 eval prompts), shared-axis reward/KL curves for baseline and variants, and one diagnostic plot, and it requires all training/evaluation logs to be accessible from the linked repository. None of that can be computed from W&B scalar summaries alone — it needs per-prompt eval CSVs, run metadata, and scalar exports collected in one place.

Status note: nothing here is being treated as "missing permanently". Runs that exist but are not yet evaluated/collated are marked PENDING. Provide the items below (or say explicitly that an item does not exist) and the final I.3 analysis will include your runs; without them, the report can only cite your runs as unverified W&B records, which wastes completed TPU work.

## Where to put artefacts

Per run, copy LIGHTWEIGHT evidence (no checkpoint trees) into:

```
$HOME/tpu-runs/part-i/report_diagnostics/<run-id>/
```

then push a short note under `experiments/variants/` on your branch with the exact paths, and tell Baron. Eval CSVs are typically <100 KB each; metadata JSON is tiny. If you can, also copy the CSVs/JSON into the repo under `experiments/evidence/<run-id>/` and push — that makes them reachable from the submission repo directly.

## Common deliverable per run (applies to every run listed below)

1. `git rev-parse HEAD` of the EXACT launch commit, plus branch name.
   Why: the report must state the exact commit per run; branch refs alone do not prove provenance (several branches point at the same commit).
2. `ckpts/run_metadata.json` from the run root (or an explicit statement that the run predates the metadata writer).
   Why: records estimator, K, seed, max steps, data dirs, commit — the controlled-comparison evidence.
3. Per-prompt greedy eval CSVs for: the base model (`--no-restore`) AND every evaluated retained checkpoint, all against the SHARED held-out manifest (GSM8K test split, 64 prompts, `EVAL_SEED=0`; on deterministic-platform code use `EVAL_MANIFEST`, equivalent to `gsm8k_test_seed0_n64.jsonl`).
   Why: paired bootstrap CIs and cross-run comparability are computed per prompt; aggregate counts are not enough. The base CSV from your own environment validates comparability with Baron's base 31/64 reference.
4. A per-checkpoint summary table: step, exact, partial, format, empty-response count.
   Why: best-vs-final checkpoint selection is a central part of the analysis; empty-response count is the key collapse diagnostic.
5. TensorBoard scalar export (CSV or the event file path) for: `rewards/*/mean`, `actor/*/kl`, completion length.
   Why: the report needs reward and KL curves on shared axes; W&B step ordering is unreliable for these runs (known step-order warnings), so TensorBoard is the source of truth.
6. W&B run URL(s).
7. Approximate wall-clock (start/finish timestamps from `logs/train.log`).
   Why: the PDF requires wall-clock time to be reported for runs used in the report.

## HARVEY — specific items (all PENDING VERIFICATION)

### H1. R6 RLOO K=8 (`R6-rloo-k8-full-s0-20260609_212314`) and R6 RLOO K=2 (`R6-rloo-k2-full-s0-20260609_220042`)
- Both trained to step 3364 with retained checkpoints every 250 steps; NO eval CSVs and NO `run_metadata.json` existed at the last audit. Provide the common deliverables, especially retained-checkpoint evals with the shared manifest.
- Provenance caveat to confirm, not hide: launch commit was `8a0f7f2` on branch `harvey`, which is NOT based on `deterministic-platform` (no `MODEL_REVISION` pin, no `EVAL_SEED`/`EVAL_MANIFEST`, no metadata writer). State this explicitly in your note; the report can still use the runs with a disclosed caveat, or prefer R7 below.

### H2. R7 deterministic RLOO reruns (launched 2026-06-11 from `harvey-grpo-k8-rerun`)
- K=2: `R7-rloo-k2-det-harvey-full-s0-20260611_102009` (Harvey VM, commit `71aab87`).
- K=8: `R7-rloo-k8-det-harvey-full-s0-20260611_105132` (shared Boris VM, commit `e3d69a1`).
- When they finish: confirm completion, then run retained-checkpoint evals + base eval with `EVAL_MANIFEST`/`EVAL_SEED=0` and provide the common deliverables.
- Why these matter most: they are the clean, deterministic RLOO K=2 vs K=8 comparison. The report's current RLOO conclusion rests only on Baron's R3 (RLOO K=2, which collapsed to empty responses); your R7 results will either confirm or overturn that conclusion, and the TPU window closes Monday 15 June.

### H3. Hard-mined RLOO — yes/no answer required
- No W&B run matching RLOO + hard/hardmedium was found. If you ran or are preparing one: provide the W&B run id, launch commit, the hard-example manifest path, and the mining rule used (ideally reuse Fred's `scripts/mine_hard_examples.py` and manifest for comparability). If not: state "not run" so the report does not wait on it.

### H4. Branch hygiene
- `origin/harvey` (`8a0f7f2`) deletes `agents/` and `experiments/` relative to deterministic-platform. Do not delete anything; just note which branch each run actually launched from. The rerun branch `harvey-grpo-k8-rerun` already documents this well — keep using it.

## FRED — specific items

### F1. Hard-example manifests (the main gap)
- Your code is pushed (`origin/fred` @ `a93ea9f`: `scripts/mine_hard_examples.py`, run scripts, `TRAIN_MANIFEST` support) — good. But the actual manifests are still only on your VM:
  - `/home/fredlawrence/tpu-runs/part-i/manifests/gsm8k_train_base_hard_medium.jsonl`
  - `/home/fredlawrence/tpu-runs/part-i/manifests/gsm8k_test_seed0_n64.jsonl`
- Push both JSONLs (they are small) under `experiments/evidence/manifests/` on your branch, with a short note stating: mining model + revision, decoding preset, seeds, pass/fail selection rule, and counts.
- Why: a curriculum claim is unverifiable without the exact training-set manifest; the PDF requires cited data to be reachable.

### F2. R8 hard-medium GRPO K=8 (`xnn69ylg` / `R8-grpo-k8-hardmedium`)
- Provide the common deliverables. You already have `eval/r8_step3452_greedy.csv` (30/64 exact at final step 3452) — copy it over, plus base CSV and any retained-checkpoint evals. If only the final step was evaluated, evaluate a few earlier retained checkpoints (e.g. 500/1500/2500) so checkpoint selection is comparable with R5.

### F3. R7 hard-medium GRPO K=2 (`3v8bx0iz`) — documented collapse
- This run collapsed (empty completions from ~step 518). It is still report-usable as an honest negative result. Provide: the TensorBoard export showing the collapse, and ideally one salvage eval at `ckpts/actor/250` to show the pre-collapse state. Your own notes already flag that the two `best_greedy.csv` files are byte-identical base (`--no-restore`) evals — do not relabel them; keep them as base references.

### F4. Known mixup to avoid repeating
- `r7_best_greedy.csv` and R8 `best_greedy.csv` were produced with `requested_ckpt_dir=/tmp/content/ckpts/` and no restore. Any new eval must pass `--ckpt-dir $RUN_ROOT/ckpts --step <n>` and verify `restored_step` is non-empty in the CSV metadata before naming the file as a trained eval.

## Deadline

TPU window ends Monday 15 June 2026. Eval passes are cheap (~minutes per checkpoint); prioritise H2 evals and F1 manifests. If an item will not be ready by 14 June evening, say so explicitly so the report freezes its claims on Baron-verified evidence only.
