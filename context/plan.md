# AgentsCW Plan

## Objective

Finish the single-PDF report while preserving the work that is already good: I.4 is mostly finished, and the Part II workflow walkthrough has been checked.

## 1. Preserve Finished Sections

Status: review only.

- Check I.4 for notation consistency, small mathematical errors, and compile issues.
- Keep I.4 concise; avoid broad rewrites.
- Preserve the Part II workflow substance: trigger, mutable tail, fixed prefix, and termination.
- Confirm the exact `cmbagent_lg` commit link is clickable.

## 2. Complete I.1 Baseline Reproduction

Status: WIP.

- Fill baseline run details from `../tpu-2026/agents`, not from old TODOs.
- Include commit, TPU type, steps, split/subset, seed, checkpoint step, and evaluation preset.
- State baseline patches briefly: persistent paths, checkpoint restore, CSV eval, seed/data-cache hygiene.
- Fill base-vs-checkpoint accuracy table.
- Add `report/figures/baseline_reward_kl.pdf`.
- Confirm wall-clock time and final GitLab/W&B/log URLs.

## 3. Complete I.2 Baseline Understanding

Status: WIP.

Use `context/section_i_context.md` for implementation detail, then compress to one page:

- `pi_theta`: LoRA actor with trainable adapters.
- `pi_ref`: frozen base Gemma model.
- `pi_old`: old/sampling actor snapshot for PPO-style ratios.
- Baseline `K=2`.
- Standard group-mean GRPO, not leave-one-out.
- Shaping rewards versus true correctness reward.
- Correctness-only reward can give zero within-group variance early.
- Tunix uses a per-token clipped surrogate; `EPSILON=0.2`.

## 4. Complete I.3 Improvement And Diagnosis

Status: WIP.

- Frame the comparison as K=2 GRPO baseline, K=2 RLOO variant, and K=8 GRPO diagnostics/full run if evidence exists.
- Use the compact metrics in `context/report_notes.md`.
- Add uncertainty if possible, preferably bootstrap CI over the 64 eval prompts.
- Add `report/figures/reward_kl_variants.pdf`.
- Add `report/figures/grpo_diagnostic.pdf`.
- Keep D3/D4 labelled as debug/medium debug unless a full K=8 run completed.
- Make the discussion honest: negative results are acceptable if diagnosed.

## 5. Finalize Part II

Status: drafted.

- Keep the checked workflow walkthrough.
- Ensure problems are grounded in implementation facts.
- Ensure each improvement states change, reason, and tradeoff.
- Check Part II fits in 2 pages excluding the diagram.

## 6. Compile And Polish

Status: pending.

- Replace title-page placeholders: team members, GitLab repo, W&B/log links.
- Remove every `\TODO`.
- Ensure all figure files exist.
- Compile from `AgentsCW/report`.
- Inspect page limits in the produced PDF.

Useful commands:

```bash
latexmk -pdf main.tex
grep -R "\\TODO" -n main.tex
```
