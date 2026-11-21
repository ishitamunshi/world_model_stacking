# World-Model Planning for Dexterous Robotic Stacking

**CIS 6280 · World Models · University of Pennsylvania · Final Project (Planning Track)**

This project investigates how the **cost function used to rank imagined action trajectories** affects dexterous robotic stacking. We keep the supplied pretrained policy and world model fixed, use model-predictive planning to evaluate candidate actions, and compare several trajectory-scoring strategies against the original terminal-distance planner.

**Main finding:** On a paired 40-scene development benchmark (one policy sampling seed), **full-trajectory scoring with α = 0.25** achieved **1.275 mean peak stacked boxes**, compared with **1.100** for our reproduced baseline (**+0.175 boxes, +15.9%**). The paired bootstrap 95% confidence interval, **[−0.175, +0.525]**, includes zero, so this is a *promising observed improvement*, not conclusive evidence of generalization.

> **Evaluation status:** 200 of 200 planned episodes completed: 40 scenes × 5 planners, sampler seed 0. The 40-scene set is treated as a **development benchmark**, not an untouched final test set.

## Task and planning pipeline

The simulator contains a dexterous robotic hand and objects with different sizes and properties. The planner aims to build a connected stack in the goal region. The evaluation measures the **maximum connected stack size reached during an episode**, with final connected stack size and other diagnostics as secondary outcomes.

At each planning decision:

1. A frozen policy proposes **16 candidate action sequences**.
2. A frozen world model rolls out predicted latent states for each candidate.
3. A scoring function ranks the predicted trajectories relative to the current visual subgoal.
4. The planner executes the first action block, observes the environment again, and replans.

Our experiments change **Step 3 only**. They do not retrain the world model or policy, and they retain the same simulator and subgoal-switching rule for the controlled cost-function comparisons.

## Methods

Let $D_t = \lVert \hat z_t - g \rVert_2^2$, where $\hat z_t$ is a world-model-predicted latent state and $g$ is the encoded active visual subgoal. Define:

- **Terminal distance:** $D_H$.
- **Full-trajectory distance:** $F = \sum_{t=1}^{H} w_t D_t$, with $w_t = t / \sum_{j=1}^{H} j$. Later predicted states have more weight.
- **Late-4 distance:** $L_4 = \frac{1}{4}\sum_{t=H-3}^{H} D_t$.

All candidate scores are minimized.

| Method | Scoring rule | Purpose |
| --- | --- | --- |
| Original baseline (`baseline`) | $D_H$ | Score the final prediction only |
| Full trajectory (`full_025`) | $0.75D_H + 0.25F$ | Include the entire imagined trajectory |
| Late-4 (`late4_010`) | $0.90D_H + 0.10L_4$ | Concentrate the trajectory penalty near the end |
| Combined A (`combined_a`) | $0.80D_H + 0.10F + 0.10L_4$ | Balance terminal, full, and late costs |
| Combined B (`combined_b`) | $0.70D_H + 0.15F + 0.15L_4$ | Test stronger combined trajectory penalties |

A separate **action-smoothness ablation** tests a terminal-distance cost plus a penalty on successive unnormalized and clipped robot commands. It is documented in `notebooks/action_smoothness.ipynb`, but is not one of the five methods in the 40-scene benchmark. Its preliminary five-scene results did not consistently improve final stacking.

## Results: paired 40-scene benchmark

**Protocol:** First 40 scenes from the supplied `scripted-val100-every80` validation goal file; **policy sampler seed 0**; 16 candidates; `generate=16`; `execute=1`; `tau=0.25`; 1,795 environment control steps. The complete benchmark ran on **Google Colab L4** with **two parallel workers**, and saved each episode independently as JSON.

### Stacking performance

| Planner | Mean peak boxes ↑ | Mean final boxes ↑ | Peak ≥2 | Peak ≥3 | Peak =4 | Mean peak Δ vs. baseline |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| **Baseline** | 1.100 | 1.025 | 22.5% | 5.0% | 2.5% | — |
| **Full trajectory α=0.25** | **1.275** | **1.125** | **30.0%** | **10.0%** | 0.0% | **+0.175** |
| Late-4 α=0.10 | 1.200 | 1.025 | **30.0%** | 2.5% | 2.5% | +0.100 |
| Combined A | 1.200 | 1.100 | 27.5% | 7.5% | 0.0% | +0.100 |
| Combined B | 0.900 | 0.750 | 17.5% | 5.0% | 0.0% | −0.200 |

### Paired uncertainty and scene-level comparisons

Bootstrap intervals use **10,000 paired resamples of scenes**. Better/worse/same counts refer to **peak stacked boxes compared with the baseline on the corresponding scene**.

| Planner | Mean peak Δ | 95% bootstrap CI | Better | Worse | Same |
| --- | ---: | --- | ---: | ---: | ---: |
| Full trajectory α=0.25 | +0.175 | [−0.175, +0.525] | 13 | 7 | 20 |
| Late-4 α=0.10 | +0.100 | [−0.150, +0.375] | 10 | 6 | 24 |
| Combined A | +0.100 | [−0.225, +0.400] | 11 | 8 | 21 |
| Combined B | −0.200 | [−0.500, +0.100] | 6 | 13 | 21 |

**Interpretation:** Full-trajectory α=0.25 has the highest observed mean peak and final stack counts in this benchmark. Neither combined formulation outperformed it. **Every confidence interval includes zero**, so none of the observed mean differences is statistically established at the 95% level by this analysis.

### Additional diagnostics

| Planner | Mean stack loss (peak − final) ↓ | Mean final subgoal index ↑ | Mean longest stall (replans) ↓ |
| --- | ---: | ---: | ---: |
| Baseline | 0.075 | 4.575 | 238.925 |
| Full trajectory α=0.25 | 0.150 | **6.125** | **217.075** |
| Late-4 α=0.10 | 0.175 | 5.025 | 245.825 |
| Combined A | 0.100 | 5.100 | 235.600 |
| Combined B | 0.150 | 4.775 | 255.675 |

The full-trajectory method advanced farther through the visual subgoals and experienced shorter average longest stalls in this evaluation. These diagnostics suggest changes in planner behavior; they do not by themselves establish the causes of success or failure.

## Repository layout

```text
world_model_stacking/
├── README.md
├── notebooks/
│   ├── planning_baseline.ipynb
│   ├── trajectory_scoring.ipynb
│   ├── late_horizon_scoring.ipynb
│   ├── action_smoothness.ipynb
│   └── final_combined_cost_benchmark.ipynb  # Add when ready
└── results/                               # Add exported local results
    ├── multiscene_eval/
    ├── late_horizon_eval/
    ├── smoothness_eval/
    ├── combined_benchmark/
    └── output_videos/
```

**Notebook guide**

| Notebook | Contents |
| --- | --- |
| [`planning_baseline.ipynb`](notebooks/planning_baseline.ipynb) | Starter planner and simulation baseline |
| [`trajectory_scoring.ipynb`](notebooks/trajectory_scoring.ipynb) | Weighted full-trajectory cost and five-scene pilot |
| [`late_horizon_scoring.ipynb`](notebooks/late_horizon_scoring.ipynb) | Late-2, Late-4, and Late-8 scoring comparison |
| [`action_smoothness.ipynb`](notebooks/action_smoothness.ipynb) | Smoothness penalty, candidate-selection diagnostics, and executed-command variation |
| `final_combined_cost_benchmark.ipynb` | Parallel 40-scene, five-method benchmark; add this notebook to `notebooks/` before using the link |

The notebooks are saved artifacts of successive experiments. They may contain earlier exploratory setups and execution outputs; **use the final benchmark notebook and its saved protocol signature** for the reported 40-scene comparison.

## Reproducing the benchmark

1. Open `notebooks/final_combined_cost_benchmark.ipynb` in **Google Colab** after adding it to this repository.
2. Select an available CUDA GPU (the benchmark was run on an **L4**; A100 can also be used if available).
3. Run the setup and mathematical-test cells first. The notebook downloads the official simulator resources, pretrained checkpoints, and visual goals; large model files are **not stored in this repository**.
4. On L4, start with **2 workers**; run the one-scene/two-planner **preflight** before the full batch. Reduce worker count if the preflight encounters GPU out-of-memory errors.
5. Run the **40-scene benchmark**. Each completed episode is saved to Google Drive, and an interrupted run can skip successful episode JSON files when resumed with the same protocol and code fingerprint.
6. Run the analysis cell after all **200/200** episode results are available to generate tables, paired bootstrap confidence intervals, plots, and a Markdown report.

`results/` directory is committed to GitHub, use the corresponding relative subfolders to inspect the saved JSON, CSV, PNG, and selected video outputs without rerunning the simulator.

## Scope and limitations

- The **40-scene benchmark is a development comparison**. The team explored some of these scenes before this evaluation; a new, untouched set is required for a clean generalization claim.
- This benchmark uses **one policy sampler seed (0)**. Additional sampler seeds would help quantify sampling variability.
- The **baseline in this run averaged 1.100 peak boxes**. Differences from other reported baselines should be investigated by matching the precise scene subset, sampler seed, checkpoints, simulator version, and evaluation procedure.
- A larger cost function is not automatically better. Both combined variants failed to outperform full-trajectory α=0.25 in this experiment.
- The project optimizes a **latent-distance planning objective**; latent closeness is not a direct physical measure of box support, contact stability, or robustness to perturbations.

