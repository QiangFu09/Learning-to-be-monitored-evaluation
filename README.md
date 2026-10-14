# Learning to Be Monitored: Evaluation

An intervention-evaluation toolkit derived from [MonitorBench](https://github.com/ASTRAL-Group/MonitorBench). It packages the twelve task datasets, corrected target-outcome verification, CoT-only monitoring, strict output handling, and instance-level causal-recall bound estimators.

Evaluation is maintained as a standalone repository. Training code is maintained separately; compatible model checkpoints are supplied through the adapter interface described below.

This is a **release candidate**. CPU analysis and packaging tests are separate from GPU inference validation. The portable orchestration still needs a fresh GPU canary before publication; it is not a claim of a clean-environment end-to-end reproduction.

The repository contains code and evaluation assets, not experiment results or model weights. Seeds, checkpoint packages, labels and output locations are supplied by the caller. Training methods are not hard-coded into the analysis.

## Supported profiles

| Role | Profiles |
|---|---|
| Task model | `qwen35_4b`, `qwen35_9b`, `mistral_small32_24b` |
| Monitor | `qwen3_32b`, `llama33_70b`, `gptoss_120b` |

Exact model revisions, decoding and serving settings are in [profiles.json](src/monitorability_eval/assets/profiles.json). Eight task rollouts are generated per arm and four monitor verdicts per usable intervention rollout. The task model produces explicit `<reasoning>` / `<answer>` blocks; Qwen task profiles disable native thinking. Monitor-specific reasoning and serialization remain part of each pinned profile.

Do not change the monitor identity, prompts, parser or sampling between a base and its comparison models. The same task/Y bundle is reusable across monitors; only Z is regenerated.

## Install and inspect

CPU analysis has no third-party dependencies and requires Python 3.10 or later. GPU profiles target the Python 3.12 runtime described in [RUNTIME.md](docs/RUNTIME.md).

```bash
uv venv .venv
uv pip install --python .venv/bin/python -e .
.venv/bin/python -m monitorability_eval inspect
.venv/bin/python -B -m unittest discover -s tests -v
```

Install the `runtime` extra only in a dedicated inference environment, after reviewing its package pins and your compute policy. Installation/downloads are not performed by the evaluator.

## Evaluation flow

Set `SEED`, `VERIFIER_SEED`, `MODEL`, `TASK`, `OUTPUT`, and `CHECKPOINT` to your chosen values. Keep the verifier seed identical across comparisons, even when evaluating multiple rollout seeds. Plan first; task and monitor commands do not run inference without `--execute`.

```bash
python -m monitorability_eval task --model "$MODEL" --task "$TASK" --seed "$SEED" --verifier-seed "$VERIFIER_SEED" --label base --output "$OUTPUT/base-task"
python -m monitorability_eval task --model "$MODEL" --task "$TASK" --seed "$SEED" --verifier-seed "$VERIFIER_SEED" --label candidate --adapter "$CHECKPOINT" --output "$OUTPUT/candidate-task"
```

For execution, add `--execute` inside an appropriately sized compute allocation. Start with a separate `--sample-limit` canary for base and candidate; diagnostic cells are excluded from formal analysis by default. Evaluate each requested task explicitly, using distinct output directories.

Once task/Y generation completes:

```bash
python -m monitorability_eval monitor --source "$OUTPUT/base-task" --monitor "$MONITOR" --seed "$SEED" --output "$OUTPUT/base-monitor"
python -m monitorability_eval monitor --source "$OUTPUT/candidate-task" --monitor "$MONITOR" --seed "$SEED" --output "$OUTPUT/candidate-monitor"
```

Again, add `--execute` only in the appropriate allocation. A successful output is sealed with file hashes. Existing destinations are never overwritten. Corrupted, incomplete, or incompatible source bundles are refused.

## Analysis

Use the same analysis command for all support modes. This one-task example requires complete base/candidate coverage; omit `--tasks` only when supplying every task in the twelve-task set.

```bash
python -m monitorability_eval analyze --bundles "$OUTPUT/base-monitor" "$OUTPUT/candidate-monitor" --base base --candidates candidate --seeds "$SEED" --tasks "$TASK" --support native
```

`--support common` selects the pairwise base/candidate estimable-ID intersection. `--support trajectory_common` intersects base with every declared candidate. Intersections are always within one task, monitor and evaluation seed. Supply multiple `--seeds` or `--candidates` explicitly; all requested cells must exist. Analyze different monitors separately.

Outputs include lower recall bound, upper recall bound (`TPR_mc`), `TNR_X1`, both instance-first composites, deltas, support IDs/counts, task/seed aggregates, NA reasons and invalid/drop diagnostics. Native deltas are not matched-instance effects. See [METRICS.md](docs/METRICS.md).

## Checkpoints and provenance

Model releases belong in a separate model repository, with immutable revision links from this codebase once published. See [CHECKPOINT_RELEASE.md](docs/CHECKPOINT_RELEASE.md) for the proposed package and release checks. No download link is presented as available before an actual upload and verification.

The evaluator accepts an unmerged adapter package rather than guessing how to load arbitrary weight directories. See [ADAPTERS.md](docs/ADAPTERS.md). Do not provide teacher-private information to the student during evaluation.

## Attribution and publication checks

Read [NOTICE.md](NOTICE.md), [UPSTREAM_CHANGES.md](docs/UPSTREAM_CHANGES.md) and [DATASETS.md](docs/DATASETS.md). Upstream code attribution is preserved. Dataset redistribution and model licenses require separate review before public release; an upstream code license is not blanket permission for every bundled asset.
