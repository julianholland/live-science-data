# ALomancy report: al_loop_5

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-08T07:28:11

**Status:** 💡 1 info · 0 warning(s) logged

## Headline

| | loop 4 | loop 5 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.201 | 0.198 | -2% |
| Best model test energy MAE (meV/atom) | 20.9 | 26.5 | +27% |
| Training structures | 4.96e+03 | 4.96e+03 | +0% |
| Test structures | 520 | 520 | +0% |
| New structures this loop | 300 | 0 | -100% |
| Warnings / errors logged | 3 | 0 | -100% |

Best model: fit_2 of 3 (chosen on the `valid` split).

## Issues & suggestions

- 💡 **overfitting**: The best model's test force MAE (0.198 eV/Å) is 2.7x its training MAE (0.0726 eV/Å). The test set covers configurations training doesn't: keep adding data, or raise general.dataset_kwargs.valid_fraction to select models on held-out data.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.23 | 3958 | 115 | 100/100 | 100% | 0 | 1 |
| 1 | 0.219 | 4186 | 292 | 96/100 | 99% | 0 | 8 |
| 2 | 0.206 | 4449 | 298 | 99/100 | 98% | 0 | 4 |
| 3 | 0.205 | 4708 | 295 | 98/100 | 100% | 0 | 5 |
| 4 | 0.201 | 4963 | 300 | 100/100 | 99% | 0 | 3 |
| 5 | 0.198 | 4963 | 0 | 0 | n/a | 0 | 0 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

No predictions stored for this loop's best model.

## Training set

| config_type | Train | Test | Excluded: quality filter | total |
|---|---:|---:|---:|---:|
| CNa_dataset | 3538 | 392 | 662 | 4592 |
| IsolatedAtom | 1 | 0 | 0 | 1 |
| high_sd | 1105 | 124 | 71 | 1300 |
| init_MP | 7 | 1 | 0 | 8 |
| init_amorphous | 26 | 3 | 0 | 29 |
| init_dimer | 99 | 0 | 0 | 99 |
| init_rattle | 5 | 0 | 269 | 274 |
| init_stretch_compress | 120 | 0 | 148 | 268 |
| init_trimer | 50 | 0 | 0 | 50 |
| on_hull | 1 | 0 | 0 | 1 |
| on_hull_expansion | 11 | 0 | 9 | 20 |
| **all** | **4963** | **520** | **1159** | **6642** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 0 new.
- Quality filters: high_force: 1150, formation_energy: 21.

![Training set composition](plots/training_set_composition.png)

## DFT

No DFT this loop.

## MLIP training: mace

- Best fit (fit_2) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 296.

![Best model training curve](plots/best_model_training_curve.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `jobs_started` | remote jobs started | 1 |
