# ALomancy report: al_loop_1

Workflow `novelty_selection` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-04T04:29:59

**Status:** ⚠ 1 warning · 0 warning(s) logged

## Headline

| | loop 0 | loop 1 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.416 | 0.416 | -0% |
| Best model test energy MAE (meV/atom) | 38.4 | 33.7 | -12% |
| Training structures | 3.97e+03 | 4.03e+03 | +1% |
| Test structures | 410 | 414 | +1% |
| New structures this loop | 123 | 100 | -19% |
| Warnings / errors logged | 5 | 0 | -100% |

Best model: fit_0 of 1 (chosen on the `valid` split).

## Issues & suggestions

- ⚠ **new_structures_force_filtered**: 38/100 new structures (38%) were excluded from training by general.train_filter.max_force (5.0 eV/Å). Lower high_accuracy_evaluation.force_ceiling (now 0.05) so DFT relaxes them further, or make generation gentler (lower md_kwargs.temperature or md_kwargs.timestep_fs).

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.416 | 3970 | 123 | 8/10 | 100% | 0 | 5 |
| 1 | 0.416 | 4028 | 100 | 0 | n/a | 0 | 0 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 0.3%, force 42.7%; test energy 0.3%, force 43.3%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: quality filter | total |
|---|---:|---:|---:|---:|
| CNa_dataset | 3538 | 392 | 662 | 4592 |
| IsolatedAtom | 1 | 0 | 0 | 1 |
| high_sd | 158 | 16 | 41 | 215 |
| init_MP | 7 | 1 | 0 | 8 |
| init_amorphous | 26 | 3 | 0 | 29 |
| init_dimer | 99 | 0 | 0 | 99 |
| init_rattle | 5 | 0 | 269 | 274 |
| init_stretch_compress | 120 | 0 | 148 | 268 |
| init_trimer | 50 | 0 | 0 | 50 |
| novelty_selection | 12 | 2 | 2 | 16 |
| on_hull | 1 | 0 | 0 | 1 |
| on_hull_expansion | 11 | 0 | 9 | 20 |
| **all** | **4028** | **414** | **1131** | **5573** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 100 new.
- Quality filters: high_force: 1122, formation_energy: 21.

![Training set composition](plots/training_set_composition.png)

## DFT

No DFT this loop.

## MLIP training: mace

- Best fit (fit_0) trained for 49 epochs, stage two from epoch 40; weights kept from epoch 49.

![Best model training curve](plots/best_model_training_curve.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `jobs_started` | remote jobs started | 1 |
