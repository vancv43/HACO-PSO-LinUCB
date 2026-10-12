# HACO-PSO-LinUCB: results, tables and run logs

Supporting results for the manuscript **"Enhanced Masked Contextual Bandits for Bank Channel Selection: An Empirical Comparison of a Dual-Model Hybrid with Conventional Metaheuristics"**.

This repository holds the numbers behind every table and figure in the manuscript: per-world JSON logs, the computed summary tables and the figures. Source code is not included; it is available from the corresponding author, as stated in the manuscript.

## Summary of the study

**Problem.** A bank contacts wealth customers many times a year about funds, insurance and bonds. Each contact goes through one of five channels: e-mail, SMS, app push, an Over-The-Top message (OTT) or a relationship manager call (RM call). A push costs almost nothing, while an RM call costs about 150 baht of staff time. The bank only sees the outcome of the channel it actually used.

**Method.** HACO-PSO-LinUCB is a masked, cost-aware LinUCB contextual bandit:

- it never picks a channel that cannot reach the customer;
- it subtracts each channel's cost from the expected value;
- its exploration settings, cost weight and feature set are tuned by HACO-PSO.

HACO-PSO is a dual-model hybrid. Particle swarm optimization handles the numeric settings and ant colony optimization handles the feature bits. It is compared under an equal 60-run budget with random search (RS), Bayesian optimization (TPE), a genetic algorithm (GA), binary PSO (BPSO), ACO and differential evolution (DE).

**Data.** No bank records were used. All experiments run on WealthChannel-Sim, a synthetic dataset whose tables follow the column structure of four bank tables. One world is one simulated bank-year: 1,000 customers, 12 monthly campaigns and five channels. The world number is the random seed. Because the simulator knows each customer's true purchase chance, every policy's test value is exact rather than estimated.

**Design.** The study ran in stages, and each run's plan was frozen before its worlds were generated.

| Stage | Worlds | Purpose |
|---|---|---|
| Stage 1 | 1–30 | Default LinUCB against the bank's rule and practice baselines; ablation; off-policy estimators; age audit |
| Stage 2 | 101–330, three runs of 30 | HACO-PSO tuning in an 11-group feature space, compared with six other tuners; equal-budget tuned baselines |
| Stage 3 | 401–430 | Full search space (4 settings and 60 feature bits); environment A is the current setting, environment B adds a limit of 2 RM calls per day and contact fatigue |
| Post hoc 1 | 401–430 | Ablation of the tuned policy; other tuned linear bandits; the B limits split into call cap and fatigue |
| Post hoc 2 | 501–730 | Symmetric GBM+ and Segment+ baselines; perturbed and nonlinear simulators; tighter and looser limits; doubly robust log-based tuning |

**Main findings.**

1. Default LinUCB beat the bank's current rule in all 30 worlds but lost to the segment rule in every world. It explored too much for a three-month campaign.
2. After tuning, HACO-PSO-LinUCB had the highest net value in Stage 2 (128.0 per 1,000 contacts). It also had the highest accuracy, recall, F1 score and ranking AUC. It beat tuned GBM in 88 of 90 worlds and the tuned segment rule in 86 of 90. The tuned segment rule had the highest precision.
3. In Stage 3, the lead over tuned GBM was 1.7% in environment A and 3.2% in environment B. Against the stronger symmetric baselines of the post hoc analysis, the lead was 2–3% without limits and below 2% under the base call limit. The preset 10% success target was never met.
4. The lead stayed positive with perturbed and nonlinear simulators and with log-based tuning, shrinking to 0.7–1.4% in the hardest settings.
5. The tuned bandit behaved like a greedy linear model that keeps learning. Removing its exploration bonus changed nothing, and online updates added about half a point.
6. HACO-PSO beat random search in three of five comparisons, and DE in both Stage 3 environments. It never beat GA, BPSO or ACO; all differences with them were below one point. HACO-PSO is best described as competitive.

All results are specific to the simulator.

## Figures

The images below are the same figures as in the manuscript. Vector PDF versions are in `figures/`.

### Figure 1: (a) Study workflow and (b) HACO-PSO-LinUCB method

![Figure 1: (a) Study workflow and (b) HACO-PSO-LinUCB method](figures/figure1_workflow_and_method.png)

Panel (a) shows the five stages of the study. A simulator turns the bank schema into synthetic worlds. Features and reachability masks are then built, policies choose channels, and a paired evaluation compares them world by world.

Panel (b) shows the proposed method. HACO-PSO searches settings on the validation months, and the masked LinUCB bandit uses them for every new contact. The test months are scored once, after the setting is fixed.

### Figure 2: HACO-PSO versus other search methods with 95% CIs

<p align="center"><img src="figures/figure2_hacopso_vs_tuners.png" width="520" alt="Figure 2: HACO-PSO versus other search methods with 95% CIs"></p>

Each point is a paired difference in test net value: HACO-PSO minus another tuner, in points per 1,000 contacts. Bars are unadjusted 95% bootstrap CIs. Filled markers show Holm-corrected Wilcoxon p < 0.05 within each column. Data: `tables/stage3/table7_hacopso_vs_tuners.csv`.

### Figure 3: Lead of HACO-PSO-LinUCB over tuned baselines with 95% CIs

<p align="center"><img src="figures/figure3_lead_vs_tuned_baselines.png" width="520" alt="Figure 3: Lead of HACO-PSO-LinUCB over tuned baselines with 95% CIs"></p>

Each point is the lead of HACO-PSO-LinUCB as a percentage of the baseline's net value. Filled markers show Holm-corrected p < 0.05. Against tuned GBM and the tuned segment rule, the lead is 9–10% before the baselines are tuned and 2–3% after. Against a frozen ridge model tuned the same way, the lead is small and not significant.

## Repository layout

```
figures/   Figures 1–3 of the manuscript, vector PDF and PNG
tables/    computed tables, one folder per stage (CSV / JSON)
logs/      per-world run records, one JSON object per line (JSONL)
```

### Manuscript item → file

| Manuscript | File |
|---|---|
| Figure 1: workflow and proposed method | `figures/figure1_workflow_and_method.pdf` |
| Figure 2: HACO-PSO versus other search methods | `figures/figure2_hacopso_vs_tuners.pdf` (data: `tables/stage3/table7_hacopso_vs_tuners.csv`) |
| Figure 3: lead over tuned baselines | `figures/figure3_lead_vs_tuned_baselines.pdf` (data: `tables/stage2/stage2_equal_budget_vs_tuned_baselines.csv`, `tables/stage3/stage3_paired_tests.json`, `tables/posthoc/table8_posthoc_robustness.json`) |
| Table 1: simulated channels | `tables/stage1/table1_channels_worlds1_30.json` |
| Table 2: simulator parameters | given in full in the manuscript |
| Table 3: Stage 1 net value | `tables/stage1/table3_stage1_net_value.csv`, `table3_stage1_paired_tests.csv` |
| Table 4: Stage 2 results | `tables/stage2/table4_stage2_results.csv` |
| Table 5: Stage 3 results | `tables/stage3/table5_stage3_results.csv`, `stage3_summary_env_A.csv`, `stage3_summary_env_B.csv` |
| Table 6: Stage 3 ablation | `tables/posthoc/table6_stage3_ablation.csv`, `posthoc1_summary.json`, `posthoc1_channels.csv` |
| Table 7: HACO-PSO versus other tuners | `tables/stage3/table7_hacopso_vs_tuners.csv` |
| Table 8: post hoc robustness | `tables/posthoc/table8_posthoc_robustness.json` (Table 8 uses the refit GBM+/Segment+ rows) |
| Table 9: Stage 1 ablation | `tables/stage1/table9_stage1_ablation.csv` |
| Off-policy evaluation and age audit, Stage 1 | `tables/stage1/stage1_off_policy_*.csv`, `stage1_age_audit.csv`, `stage1_learning_curves.csv` |
| Stage 2 runs 1–3, success rule and pooled wins | `tables/stage2/run{1,2,3}_*.csv/json`, `stage2_pooled_90_worlds.json` |
| Stage 2 equal-budget analysis, BO versus RS | `tables/stage2/stage2_equal_budget_*.csv/json` |

### Logs

| Folder | Worlds | Records |
|---|---|---|
| `stage1_default_policies` | 1–30 | test net value of every Stage 1 policy and ablation |
| `stage1_off_policy_evaluation` | 1–30 | estimator values against the true value |
| `stage1_age_audit` | 1–30 | value by age band |
| `stage1_learning_curves` | 1–30 | value over time |
| `stage2_run{1,2,3}_tuners` | 101–130, 201–230, 301–330 | validation and test value of each tuner's chosen setting |
| `stage2_policy_metrics`, `stage2_classification_metrics` | 101–330 | net value, purchase rate, cost, accuracy, precision, recall, F1, AUC |
| `stage2_equal_budget_*` | 101–330 | baselines tuned with the same 60-run budget |
| `stage3_environments_A_B` | 401–430 | every Stage 3 policy in environments A and B |
| `posthoc1_tuned_ablation_and_limits` | 401–430 | first post hoc analysis |
| `posthoc2_robustness` | 501–730 | second post hoc analysis |

Common fields: `seed` / `world_seed` is the world number. `value_per_1000` (or `net_per_1000`) is the expected net value per 1,000 test contacts, in purchase points minus channel cost. `channel_mix` gives the share of contacts per channel. Classification fields are in percent. `run` marks the Stage 2 run.

### Policy labels

Some labels in the Stage 1 logs come from the original study plan and differ from the manuscript:

| Label in files | Name in the manuscript |
|---|---|
| `LinUCB (proposed)` (Stage 1) | Default LinUCB |
| `Linear TS (proposed)` | Linear TS |
| `Segment best-channel rule` | Segment rule |
| `Per-channel GBM, greedy` | Per-channel GBM |
| `Logged business policy` | Logged policy |
| `Uniform random (feasible)` | Uniform |
| `Proposed`, `HACO-PSO` (Stages 2–3) | HACO-PSO-LinUCB |

## Statistics

Paired comparisons use percentile bootstrap intervals over worlds (10,000 resamples) and two-sided Wilcoxon signed-rank tests with Holm correction within each stated family. Intervals are not adjusted for multiple comparisons. Stage 2 runs 2 and 3 and both post hoc analyses are exploratory.
