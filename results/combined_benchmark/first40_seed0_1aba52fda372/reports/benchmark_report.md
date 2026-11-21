# Combined-Cost Planning: 40-Scene Benchmark

**Status:** Complete 40-scene paired evaluation
**Paired scene/sampler tuples:** 40 / 40
**Sampler seeds:** [0]
**Protocol/code fingerprint:** `1aba52fda372`
**Frozen components:** pretrained policy, world model, original subgoal manager,
16 candidates, generate=16, execute=1, tau=0.25, horizon=1795.

| Method | Mean peak | ≥1 % | ≥2 % | ≥3 % | 4 % | Mean final | Δ vs baseline | 95% bootstrap CI | Better/Worse/Same |
|---|---:|---:|---:|---:|---:|---:|---:|---|---|
| baseline | 1.100 | 80.0 | 22.5 | 5.0 | 2.5 | 1.025 | +0.000 | — | 0/0/40 |
| full_025 | 1.275 | 87.5 | 30.0 | 10.0 | 0.0 | 1.125 | +0.175 | [-0.175, +0.525] | 13/7/20 |
| late4_010 | 1.200 | 85.0 | 30.0 | 2.5 | 2.5 | 1.025 | +0.100 | [-0.150, +0.375] | 10/6/24 |
| combined_a | 1.200 | 85.0 | 27.5 | 7.5 | 0.0 | 1.100 | +0.100 | [-0.225, +0.400] | 11/8/21 |
| combined_b | 0.900 | 67.5 | 17.5 | 5.0 | 0.0 | 0.750 | -0.200 | [-0.500, +0.100] | 6/13/21 |

## Interpretation notes
- Bootstrap confidence intervals are paired by scene and sampling seed.
- The 40 scenes are a shared development benchmark, not held-out test evidence.
- Do not select weights on held-out scenes; freeze the method before final validation.
- The official baseline is evaluated fresh with the same A100 and exact scenario seeds.
- Differences may vary across hardware precision and policy sampler seeds.

## Figures
- `mean_peak_final.png`
- `success_rates.png`
- `subgoal_progress.png`
