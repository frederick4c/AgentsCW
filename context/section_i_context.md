# Section I Context: GRPO Finetuning of Gemma 3 on a TPU

This is a factual context and planning file for coursework Part I, sections
I.1--I.3. It is deliberately fuller than the final 3-page practical report
section should be. Do not paste this file into the report verbatim; use it as a
runbook, evidence checklist, and source of implementation facts.

## Coursework Requirements

Part I practical work asks for three things:

1. **I.1 Reproducing the baseline, 10 marks**
   - Run `scripts/train.py` end-to-end on the assigned `v6e-1` TPU with the
     default `scripts/config.py`.
   - Run `scripts/evaluate.py` on the resulting checkpoint.
   - Report:
     - exact repository commit;
     - wall-clock time;
     - number of GRPO steps actually completed;
     - base-model GSM8K accuracy;
     - LoRA-finetuned checkpoint GSM8K accuracy;
     - held-out split and fixed evaluation seed;
     - baseline mean reward curve versus GRPO step;
     - baseline KL curve, `KL(pi_theta || pi_ref)`, versus GRPO step.
   - If the baseline does not run as shipped, add a short "baseline patches"
     note with the minimal fix.

2. **I.2 Understanding the baseline, 10 marks**
   - In at most one report page, map the baseline to lecture notation:
     - `pi_theta`, `pi_old`, and `pi_ref`;
     - group size `K`;
     - advantage estimator and whether it is standard or leave-one-out;
     - reward shaping versus true correctness reward;
     - clipped PPO-style surrogate location and clip range `epsilon`.

3. **I.3 Improving on the baseline, 30 marks**
   - Make at least one principled modification motivated by lecture theory.
   - Run a controlled comparison against the baseline.
   - Typical expected evidence is baseline plus two further complete training
     runs.
   - Report:
     - a table comparing base, baseline GRPO, and improved checkpoint(s);
     - seeds and an uncertainty measure, such as second seed or bootstrap CI;
     - mean reward and KL curves for all runs on shared axes;
     - one diagnostic plot for a GRPO failure mode;
     - a discussion connecting empirical findings to theory, including honest
       negative results.

## Repository Snapshot

Working TPU repository:

- Path: `../tpu-2026`
- Role: the repo to modify, run, evaluate, and eventually port to GitLab.
- Branch: `main`
- Current commit: `324abbe4b4e229ea812223856393547db4fbb53e`
- Current short log head: `324abbe Add chat.py interactive REPL for trained checkpoints; document in README`
- Remote observed locally: `https://github.com/barongracias/tpu-2026.git`
- Current tracked status observed during planning: clean.

Original/provenance clone:

- Path: `external/tpu-2026-old`
- Role: reference/provenance only, not the primary working repo.
- Current commit: `324abbe4b4e229ea812223856393547db4fbb53e`
- Remote observed locally: `https://github.com/borisbolliet/tpu-2026.git`
- Local modifications observed:
  - `scripts/chat.py`
  - `scripts/evaluate.py`
  - `scripts/model.py`
  - `scripts/rewards.py`
  - `scripts/train.py`
- The local modifications are import-path edits of the form:
  - from local imports such as `from config import ...`
  - to package imports such as `from external.tpu.scripts.config import ...`
- Treat these old-clone edits as provenance context only unless deliberately
  porting or comparing them.

Report scaffold:

- Path: `report/main.tex`
- Existing placeholders already map to this plan:
  - `I.1 Reproducing the baseline`
  - `I.2 Understanding the baseline`
  - `I.3 Improving on the baseline`
  - expected figures:
    - `report/figures/baseline_reward_kl.pdf`
    - `report/figures/reward_kl_variants.pdf`
    - `report/figures/grpo_diagnostic.pdf`

## Baseline Implementation Facts

Important source files in `../tpu-2026/scripts`:

- `config.py`: single source of truth for model, data, LoRA, GRPO, optimizer,
  checkpointing, generation, and logging settings.
- `train.py`: builds the mesh, base model, LoRA actor, tokenizer, dataset,
  optimizer, Tunix `RLCluster`, `GRPOConfig`, and `GRPOLearner`; then calls
  `trainer.train(train_ds, val_ds)`.
- `evaluate.py`: builds the model/sampler and reports exact numeric accuracy,
  partial accuracy within 10 percent, and format accuracy.
- `rewards.py`: reward functions and regex parsing.
- `data.py`: GSM8K loading, prompt template, fixed shuffle seed, train/val/test
  construction.
- `model.py`: Gemma 3 loading, mesh creation, LoRA wrapping, tokenizer loading.
- `scripts/README.md`: operational notes for tmux, W&B, TensorBoard, resumes,
  evaluation, checkpoint pitfalls, and hyperparameter intuition.

Default config facts from `scripts/config.py`:

- Model: `google/gemma-3-1b-it`
- Tokenizer: `gs://gemma-data/tokenizers/tokenizer_gemma3.model`
- Data source default: `DATA_SOURCE=os.environ.get("DATA_SOURCE", "tfds")`
- Train data directory: `./data/train`
- Test data directory: `./data/test`
- Train fraction: `TRAIN_FRACTION=0.9`
- LoRA rank: `RANK=64`
- LoRA alpha: `ALPHA=64.0`
- Rollout prompt length: `MAX_PROMPT_LENGTH=256`
- Rollout generation length: `TOTAL_GENERATION_STEPS=768`
- Rollout temperature: `TEMPERATURE=0.9`
- Group size: `NUM_GENERATIONS=2`, so `K=2`
- GRPO inner iterations: `NUM_ITERATIONS=1`
- KL coefficient: `BETA=0.08`
- PPO-style clip range: `EPSILON=0.2`
- Train micro batch size: `TRAIN_MICRO_BATCH_SIZE=1`
- Number of batches: `NUM_BATCHES=3738`
- Number of test batches: `NUM_TEST_BATCHES=64`
- Evaluation during training: `EVAL_EVERY_N_STEPS=64`
- Epochs: `NUM_EPOCHS=1`
- Maximum GRPO steps:
  - `MAX_STEPS = int(NUM_BATCHES * NUM_ITERATIONS * TRAIN_FRACTION * NUM_EPOCHS)`
  - with defaults, `MAX_STEPS = int(3738 * 1 * 0.9 * 1) = 3364`
- Learning rate: `LEARNING_RATE=3e-6`
- Weight decay: `WEIGHT_DECAY=0.1`
- Gradient clipping: `MAX_GRAD_NORM=0.1`
- Checkpoint directory: `/tmp/content/ckpts/`
- Intermediate checkpoint directory: `/tmp/content/intermediate_ckpt/`
- TensorBoard directory: `/tmp/content/tmp/tensorboard/grpo`
- Checkpoint save interval: `SAVE_INTERVAL_STEPS=500`
- Max checkpoints to keep: `MAX_TO_KEEP=4`
- W&B project default: `tunix`
- W&B entity default: `milindsarkaryt-iiser-mohali`

Data and evaluation facts:

- `data.py` shuffles GSM8K with `seed=42`.
- `build_train_val_test` batches train data, slices the first `NUM_BATCHES`,
  then uses the first `int(len(full) * TRAIN_FRACTION)` batches for training
  and the remainder for validation.
- Test data is `get_dataset(test_dir, "test", source).batch(1)[:64]` by
  default, so the current evaluation subset is 64 GSM8K test examples unless
  this is changed.
- `evaluate.py --preset greedy` is the deterministic baseline evaluation path.
- `evaluate.py` passes `seed=p` during generation, with `p=0` for the default
  `num_passes=1`.
- The report should state the evaluation preset, source, split/subset, and seed
  explicitly.

## I.1 Baseline Reproduction Runbook

Use the working repo `../tpu-2026` for all commands. On the TPU VM, use tmux so
the run is not tied to a fragile SSH shell.

Pre-run evidence to record:

- Commit:
  - `git rev-parse HEAD`
  - expected before modifications: `324abbe4b4e229ea812223856393547db4fbb53e`
- Status:
  - `git status --short`
  - save output in notes; ideally clean for the baseline.
- Config:
  - copy or commit `scripts/config.py` as used for the baseline.
- Environment:
  - TPU type: `v6e-1`
  - zone/name/project from TPU setup, if useful for reproducibility.
- Log identity:
  - W&B run URL or TensorBoard event path.
  - Any checkpoint directory override, if changed.

Training command options:

```bash
cd ~/tpu-2026/scripts
source ~/venvs/tunix/bin/activate
python -u train.py
```

or, preferred:

```bash
cd ~/tpu-2026/scripts
./run_tmux.sh
```

Important operational notes:

- `scripts/README.md` says to always train inside `tmux`.
- `train.py` initializes W&B before constructing `RLCluster`, which works around
  a known Tunix hang.
- `/tmp` is volatile, while the default `CKPT_DIR` is `/tmp/content/ckpts/`;
  copy checkpoints to persistent storage or change `CKPT_DIR` before long runs
  if needed.
- Tunix writes TensorBoard scalar events to
  `/tmp/content/tmp/tensorboard/grpo`.
- A full baseline run is expected by the coursework to take roughly 5--9 hours
  on `v6e-1`.

Post-run evidence to record:

- Wall-clock start and finish timestamps.
- Wall-clock duration.
- Final printed step count or last checkpoint step.
- Whether `MAX_STEPS=3364` completed.
- Checkpoint path and step used for evaluation.
- W&B run URL and/or TensorBoard event file location.
- Any failure, interruption, resume, or baseline patch.

Baseline evaluation:

```bash
cd ~/tpu-2026/scripts
source ~/venvs/tunix/bin/activate
python evaluate.py --preset greedy
```

Use this for:

- base model evaluation before restoring/training, or by using a no-restore
  path if implemented for evaluation;
- LoRA checkpoint evaluation after training.

Current `evaluate.py` builds a LoRA-wrapped model but does not expose an obvious
checkpoint-restore CLI in the inspected code. Before running final evaluation,
verify how the trained checkpoint is restored for evaluation. Options:

- inspect whether Tunix/Orbax restore is implicitly triggered by the trainer or
  local environment;
- add a minimal, documented restore path to `evaluate.py` if needed;
- use `chat.py --no-restore` only as an interactive sanity check, not as the
  final evaluation table unless it is adapted into a batch evaluator.

If a restore patch is required, record it under "baseline patches" with:

- affected file;
- exact commit containing the patch;
- why it was needed;
- confirmation that the patch did not alter the baseline training objective.

Report artifacts for I.1:

- Table: baseline run details.
- Table: base model and LoRA checkpoint GSM8K accuracy.
- Figure: `baseline_reward_kl.pdf`, with mean reward and KL versus GRPO step.
- Short baseline-patches paragraph.

Fields still unknown before runs:

- `TO_FILL_AFTER_RUN`: baseline W&B/TensorBoard run URL.
- `TO_FILL_AFTER_RUN`: wall-clock time.
- `TO_FILL_AFTER_RUN`: actual GRPO steps completed.
- `TO_FILL_AFTER_RUN`: base model accuracy.
- `TO_FILL_AFTER_RUN`: baseline LoRA checkpoint accuracy.
- `TO_FILL_AFTER_RUN`: checkpoint step/path evaluated.

## I.2 Understanding The Baseline

### Policies in the LoRA setup

In the baseline:

- `pi_ref` is the frozen base Gemma model loaded by `load_base_model`.
- `pi_theta` is the actor model: the same base model wrapped with trainable
  LoRA adapters by `get_lora_model`.
- `pi_old` is the sampling/old policy snapshot used for PPO-style ratios during
  GRPO updates.

Trainable parameters:

- The frozen base weights are not trained.
- The LoRA adapter parameters are trainable.
- `pi_theta` carries the trainable adapter state.
- `pi_ref` is fixed.
- `pi_old` is the previous/sampling snapshot of the actor used to compute ratios
  for the surrogate objective.

Baseline code anchors:

- `model.py`: `load_base_model` loads the frozen base model.
- `model.py`: `get_lora_model` applies LoRA via `qwix.LoraProvider`.
- `train.py`: `RLCluster(actor=lora, reference=base, ...)`.

### Group size and advantage estimator

Baseline group size:

- `K = NUM_GENERATIONS = 2`.

Tunix implementation source:

- `https://raw.githubusercontent.com/google/tunix/main/tunix/rl/grpo/grpo_learner.py`

Relevant Tunix facts from that source:

- `GRPOConfig` defaults to `advantage_estimator="grpo"` and
  `policy_loss_fn="grpo"`.
- `GRPOLearner._generate_and_compute_advantage` obtains the estimator through
  `function_registry.get_advantage_estimator(self.algo_config.advantage_estimator)`.
- `compute_advantages` reshapes rewards by `num_generations`, computes grouped
  reward means and grouped reward standard deviations, repeats them back across
  the group, and returns:

```text
(rewards - mean_grouped_rewards) / (std_grouped_rewards + 1e-4)
```

- The grouped standard deviation uses `ddof=1`.
- This is the standard group-mean GRPO advantage, not leave-one-out.

### Reward terms

Reward functions in `scripts/rewards.py`:

- `match_format_exactly`: full template match, `+3` or `0`.
- `match_format_approximately`: shaping reward based on expected tag counts and
  position, up to `+2.5`, with penalties for missing/extra tags.
- `check_answer`: correctness-oriented reward for exact, stripped, close, wrong,
  or unparsable bracketed answers.
- `check_numbers`: fallback numeric extraction after `<answer>`, with `+1.5`
  for exact numeric match.

Classification for the report:

- Shaping rewards:
  - `match_format_exactly`
  - `match_format_approximately`
  - arguably `check_numbers` as a fallback parseability/numeric extraction aid,
    though it also partially overlaps with correctness.
- True correctness reward:
  - `check_answer`, especially exact numeric answer inside the required template.

Discussion point for I.2(c):

- With correctness alone early in training, many completions will be unparsable
  or wrong, so all `K` siblings for a prompt may receive the same reward.
- When all rewards in a group are equal, within-group reward variance
  `sigma_r` is near zero and centered advantages are zero or numerically
  unstable.
- Shaping rewards create early within-group variance by rewarding partial format
  compliance, giving GRPO a learning signal before exact math correctness is
  common.
- The risk is reward hacking: format reward rises while true correctness stays
  flat.

### Clipped surrogate

Tunix implementation source:

- `https://raw.githubusercontent.com/google/tunix/main/tunix/rl/grpo/grpo_learner.py`

Relevant implementation facts:

- The registered policy loss is `grpo_loss_fn`.
- It computes current per-token log probabilities for the actor.
- If `old_per_token_logps` are absent, it uses a stopped-gradient copy of
  current log probabilities; otherwise it uses stored old-policy log
  probabilities.
- It computes an importance ratio through:
  - `seq_importance_ratio = per_token_logps - old_per_token_logps`
  - `coef_1 = exp(seq_importance_ratio)`
  - `coef_2 = clip(coef_1, 1 - epsilon, 1 + epsilon_high)`
- It applies the clipped surrogate per token:
  - `per_token_loss = -minimum(coef_1 * advantage, coef_2 * advantage)`
- It adds the reference KL penalty if `beta != 0`:
  - `per_token_loss = per_token_loss + beta * kl`
- It logs:
  - `kl`
  - `pg_clipfrac`

Mapping to baseline config:

- `epsilon` is `EPSILON=0.2` in `scripts/config.py`.
- `beta` is `BETA=0.08` in `scripts/config.py`.

Deviation from simple lecture form:

- The implementation operates on per-token log probabilities and aggregates
  token-level losses, rather than using only a single sequence-level ratio in
  the simplest lecture notation.
- With baseline `NUM_ITERATIONS=1`, old-policy log probabilities may be
  stop-gradient current log probabilities in the inspected Tunix path, because
  explicit old actor inference is only computed when `num_iterations > 1`.

## I.3 Experiment Plan

The team has not decided the final I.3 direction yet. Use the default below
unless the team explicitly changes direction before committing TPU time.

### Default: KL coefficient sweep

Rationale:

- The coursework asks for a principled modification grounded in theory.
- `beta` directly controls the reference KL penalty, which is central to GRPO's
  trust-region/stability story.
- It is low-risk because it only changes `BETA` in `scripts/config.py`.
- It gives clear predictions:
  - smaller `beta` should permit faster drift from the base model and possibly
    higher reward/accuracy, but risks larger KL and reward hacking;
  - larger `beta` should keep KL lower and outputs more base-like, but may
    suppress learning.

Controlled runs:

| Run | Name | Change | Prediction |
| --- | --- | --- | --- |
| 1 | Baseline | `BETA=0.08` | Reference point |
| 2 | Lower KL anchor | `BETA=0.04` | Higher reward/accuracy if baseline under-updates; higher KL |
| 3 | Stronger KL anchor | `BETA=0.16` | Lower KL; possibly slower learning or lower accuracy |

Controls:

- Same repo except the explicit `BETA` change.
- Same data source, preferably `tfds`.
- Same train/test split and `seed=42` dataset shuffle.
- Same `MAX_STEPS=3364`, or clearly normalise by step if interrupted.
- Same LoRA rank, learning rate, generation length, group size, temperature,
  top-k/top-p, batch size, and evaluation preset.
- Same evaluation subset: default GSM8K test subset of 64 batches/examples
  unless changed deliberately before all runs.

Evidence to collect for every run:

- Run name.
- Commit hash.
- Config diff from baseline.
- W&B run URL and/or TensorBoard event path.
- Wall-clock time.
- Completed GRPO steps.
- Final or chosen checkpoint step.
- GSM8K exact accuracy.
- Partial accuracy.
- Format accuracy.
- Mean reward curve.
- KL curve.
- Completion length curve if available.
- `pg_clipfrac` curve if available.

Recommended diagnostic plot:

- First choice: response/completion length over training for each run, because
  Tunix logs `completions/mean_length`, `completions/max_length`, and
  `completions/min_length`.
- Alternative: `pg_clipfrac` over training, because Tunix logs it in
  `grpo_loss_fn`.
- Alternative if extra instrumentation is added: degenerate groups where all
  `K` rewards are equal, or distribution of `A_hat`.

Success criteria:

- The report does not need the variant to beat the baseline.
- A good outcome is a controlled observation that matches or contradicts a
  theory prediction and is diagnosed clearly.
- The main empirical comparison should be accuracy versus KL behavior, not only
  reward versus step.

Fields still unknown before runs:

- `TO_FILL_AFTER_RUN`: actual selected I.3 direction.
- `TO_FILL_AFTER_RUN`: run IDs and URLs.
- `TO_FILL_AFTER_RUN`: final accuracies.
- `TO_FILL_AFTER_RUN`: uncertainty estimates.
- `TO_FILL_AFTER_RUN`: final diagnostic plot choice.

### Decision matrix for alternatives

Switch to reward shaping if baseline logs show:

- format reward rises but correctness stays flat;
- completion lengths grow without accuracy gains;
- format accuracy dominates exact answer accuracy.

Reward-shaping options:

- down-weight `match_format_exactly` and/or `match_format_approximately`;
- add a length penalty;
- make approximate format matching stricter;
- separate correctness and shaping metrics in logging if not already visible.

Theoretical angle:

- shaping increases early reward variance but can bias optimisation toward proxy
  behavior.
- length penalties can prevent reward hacking through verbose template filling,
  but may penalize legitimate reasoning.

Switch to group-size `K` work if TPU budget permits and the team wants a more
direct estimator-variance story:

- compare `NUM_GENERATIONS=2` against a larger `K`, such as `4`;
- expect lower advantage noise but higher rollout cost;
- normalise compute carefully, because larger `K` changes time per step.

Theoretical angle:

- group-relative baselines reduce prompt-level reward offsets;
- larger groups can improve reward statistics but reduce number of optimizer
  steps under a fixed wall-clock budget.

## Interfaces And Artifacts

No public code APIs need to change for the planning file itself.

Expected submitted or linked artifacts:

- GitLab repository link for final code submission.
- Training logs:
  - W&B run(s), TensorBoard logs, JSONL, or equivalent.
  - Must be included in or accessible from the GitLab repository.
- Checkpoints:
  - checkpoint path;
  - exact checkpoint step evaluated.
- Evaluation outputs:
  - base model;
  - baseline checkpoint;
  - each variant checkpoint;
  - same held-out GSM8K split/subset and seed.
- Figures for `report/main.tex`:
  - `report/figures/baseline_reward_kl.pdf`
  - `report/figures/reward_kl_variants.pdf`
  - `report/figures/grpo_diagnostic.pdf`
- Tables for `report/main.tex`:
  - baseline run details;
  - baseline accuracy;
  - controlled comparison with bootstrap CI or second seed if available.

Suggested run ledger:

| Field | Baseline | Beta 0.04 | Beta 0.16 |
| --- | --- | --- | --- |
| Commit | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` |
| Config diff | none | `BETA=0.04` | `BETA=0.16` |
| W&B/TB URL | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` |
| Start time | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` |
| End time | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` |
| Steps | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` |
| Checkpoint evaluated | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` |
| Exact accuracy | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` |
| Partial accuracy | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` |
| Format accuracy | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` | `TO_FILL_AFTER_RUN` |

## Plotting And Uncertainty Plan

Training curves:

- Extract scalar curves from W&B or TensorBoard.
- Plot baseline and variants on shared axes.
- Use GRPO step on the x-axis.
- Include at least:
  - `rewards/score/mean`;
  - `kl`.
- If possible, include `pg_clipfrac` in the diagnostic or appendix notes.

Uncertainty options:

- Preferred: second evaluation seed or second complete run for the same variant,
  if TPU budget allows.
- Practical fallback: bootstrap confidence interval over the fixed held-out
  evaluation prompts.
- If using the default 64-example test subset, state that the CI is over those
  64 prompts, not the full GSM8K test set.

Bootstrap CI sketch:

- Store one boolean exact-correct value per evaluated prompt.
- Resample prompt indices with replacement many times, such as 10,000 bootstrap
  samples.
- Compute accuracy for each bootstrap sample.
- Report the 2.5 and 97.5 percentiles as a 95 percent CI.

## Final Report Mapping

Use this file to fill the existing `report/main.tex` placeholders:

- I.1 run details table:
  - commit, TPU type, commands, wall-clock time, steps, split, seed.
- I.1 baseline patches:
  - "ran as shipped" or the minimal patch description.
- I.1 accuracy table:
  - base model and LoRA GRPO checkpoint.
- I.1 figure:
  - baseline mean reward and KL.
- I.2 paragraphs:
  - use the implementation facts above, compressed to at most one page.
- I.3 modification and motivation:
  - explain KL sweep or chosen alternative.
- I.3 controlled comparison:
  - same data, same eval split, same compute or normalised per step.
- I.3 table:
  - base, baseline GRPO, beta-low, beta-high, plus uncertainty.
- I.3 figures:
  - reward/KL variants;
  - one diagnostic.
- I.3 discussion:
  - connect observed KL, reward, and accuracy to the theory prediction.

Keep Part I I.1--I.3 to at most 3 pages in the final PDF, excluding references
and appendix. This context file can be much longer.

## Assumptions And Defaults

- `context/section_i_context.md` is a local planning/context file and is not
  necessarily submitted.
- `context/` is currently ignored by `.gitignore`.
- The working repo for experiments is `../tpu-2026`.
- The old clone `external/tpu-2026-old` is provenance only.
- The initial I.3 recommendation is a `BETA` sweep because it is low-risk,
  controlled, and directly grounded in GRPO theory.
- Unknown results are marked with `TO_FILL_AFTER_RUN`; do not replace them
  until the actual run/evaluation evidence exists.
- Tunix source reference for I.2 implementation claims:
  - `https://raw.githubusercontent.com/google/tunix/main/tunix/rl/grpo/grpo_learner.py`
