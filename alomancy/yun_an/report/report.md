# ALomancy report: al_loop_50

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-06T12:56:04

**Status:** ✓ no issues · 0 warning(s) logged

## Headline

| | loop 49 | loop 50 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.247 | 0.298 | +21% |
| Best model test energy MAE (meV/atom) | 139 | 141 | +1% |
| Training structures | 2.23e+03 | 2.23e+03 | +0% |
| Test structures | 130 | 130 | +0% |
| New structures this loop | 76 | 0 | -100% |
| Warnings / errors logged | 31 | 0 | -100% |

Best model: fit_1 of 5 (chosen on the `valid` split).

## Issues & suggestions

No issues detected.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 36 | 0.385 | 1860 | 49 | 49/50 | 100% | 3 | 2 |
| 37 | 0.358 | 1894 | 43 | 43/50 | 100% | 2 | 8 |
| 38 | 0.479 | 1929 | 48 | 48/50 | 100% | 4 | 3 |
| 39 | 1.32 | 1972 | 48 | 48/50 | 100% | 0 | 5 |
| 40 | 0.181 | 1698 | 9 | 9/81 | 100% | 3 | 97 |
| 41 | 0.469 | 1896 | 27 | 27/43 | 67% | 2 | 30 |
| 42 | 0.333 | 1926 | 34 | 34/45 | 76% | 0 | 31 |
| 43 | 0.473 | 1971 | 57 | 57/75 | 77% | 3 | 39 |
| 44 | 0.346 | 1997 | 40 | 40/55 | 72% | 5 | 33 |
| 45 | 0.412 | 2007 | 6 | 6/100 | 100% | 0 | 100 |
| 46 | 0.353 | 2055 | 54 | 54/100 | 39% | 0 | 50 |
| 47 | 0.373 | 2085 | 30 | 30/45 | 73% | 0 | 27 |
| 48 | 0.418 | 2152 | 75 | 75/100 | 61% | 0 | 39 |
| 49 | 0.247 | 2232 | 76 | 76/100 | 92% | 0 | 31 |
| 50 | 0.298 | 2232 | 0 | 0 | n/a | 0 | 0 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 3.4%, force 21.6%; test energy 1.6%, force 155.2%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: redundant | Excluded: quality filter | total |
|---|---:|---:|---:|---:|---:|
| IsolatedAtom | 5 | 0 | 0 | 0 | 5 |
| high_sd | 831 | 114 | 28 | 591 | 1564 |
| init_MP | 69 | 4 | 0 | 0 | 73 |
| init_amorphous | 67 | 12 | 8 | 0 | 87 |
| init_dimer | 299 | 0 | 0 | 0 | 299 |
| init_stretch_compress | 390 | 0 | 0 | 398 | 788 |
| init_trimer | 571 | 0 | 0 | 0 | 571 |
| **all** | **2232** | **130** | **36** | **989** | **3387** |

- Redundancy removal: 36 structure(s) flagged in total, 0 of this loop's 0 new.
- Quality filters: high_force: 987, formation_energy: 324.

![Training set composition](plots/training_set_composition.png)

## DFT

No DFT this loop.

## MLIP training: mace

- Best fit (fit_1) trained for 199 epochs, stage two from epoch 160; weights kept from epoch 162.

![Best model training curve](plots/best_model_training_curve.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `jobs_started` | remote jobs started | 1 |
