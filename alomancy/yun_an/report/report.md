# ALomancy report: al_loop_52

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-10T15:19:36

**Status:** ✖ 2 errors · ⚠ 1 warning · 18 warning(s) logged

## Headline

| | loop 51 | loop 52 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.341 | 0.284 | -17% |
| Best model test energy MAE (meV/atom) | 126 | 126 | -0% |
| Training structures | 2.41e+03 | 2.51e+03 | +4% |
| Test structures | 152 | 162 | +7% |
| New structures this loop | 92 | 62 | -33% |
| Warnings / errors logged | 17 | 18 | +6% |

Best model: fit_2 of 5 (chosen on the `valid` split).

## Issues & suggestions

- ✖ **remote_jobs_failed**: 14/128 remote jobs (11%) failed (6) or died (8). Check the job logs (results/alomancy.log has each failure's traceback); jobs killed by the scheduler usually need a longer max_time for that phase, or a partition with more memory.
- ✖ **dft_partial_failure**: 6/100 structures sent to DFT (6%) came back without a result. Check the DFT output in results/<loop>/high_accuracy_evaluation; raise high_accuracy_evaluation.max_time / max_go_time if jobs timed out.
- ⚠ **short_bond_excluded**: 3817/5734 structures (67%) were dropped for a bond shorter than 0.5 Å. The model is driving MD into collisions: lower md_kwargs.temperature or md_kwargs.timestep_fs, or add more short-range initialization data (dimers/trimers).

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
| 50 | 0.298 | 2323 | 86 | 86/100 | 85% | 0 | 28 |
| 51 | 0.341 | 2409 | 92 | 92/100 | 85% | 0 | 17 |
| 52 | 0.284 | 2512 | 62 | 94/100 | 100% | 0 | 18 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 2.6%, force 20.2%; test energy 1.6%, force 140.8%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: redundant | Excluded: quality filter | total |
|---|---:|---:|---:|---:|---:|
| IsolatedAtom | 5 | 0 | 0 | 0 | 5 |
| high_sd | 1111 | 146 | 28 | 823 | 2108 |
| init_MP | 69 | 4 | 0 | 0 | 73 |
| init_amorphous | 67 | 12 | 8 | 0 | 87 |
| init_dimer | 299 | 0 | 0 | 0 | 299 |
| init_stretch_compress | 390 | 0 | 0 | 398 | 788 |
| init_trimer | 571 | 0 | 0 | 0 | 571 |
| **all** | **2512** | **162** | **36** | **1221** | **3931** |

- Redundancy removal: 36 structure(s) flagged in total, 0 of this loop's 62 new.
- DFT geometry-optimisation starting frames: 126 stored this loop, 51 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 1219, formation_energy: 362.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 1,283 structures probed, tolerance 0.0398, 36 flagged as duplicates (8 plateau(s) found).

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 94 returned of 100 submitted (94 relaxed, 0 single point).
- Geometry optimisation: 94/94 reached the force ceiling (0.05 eV/Å); steps mean 104.4, median 97, max 200 of 200 allowed.
- Average DFT time per relaxation: 231.9 min (median 230.3, max 475.3; 94 structures).
- Largest per-atom force after DFT: median 0.05, max 0.05 eV/Å.
- DFT phase wall-clock: 8.2 h (mean queue time 24 min).

**Structures evaluated with DFT this loop (final geometries)**

![Structures evaluated with DFT this loop (final geometries)](plots/added_structures.gif)

## MLIP training: mace

- Best fit (fit_2) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 245.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: MD

- Runs: 30 (30 completed, 1 seed(s) replaced)
- Frames: 4781 recorded, 1681 physical (no bond < 0.5 Å, volume ≤ 3 x the seed's; 17 frame(s) cut after a run expanded past it, seed frames left out), 1000 picked as candidates, spread over each run
- Ensemble npt, T = 300 K, timestep 0.1 fs

![MD frames per run](plots/md_frames_per_run.png)

![MD temperature](plots/md_temperature.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 236 candidates scored; 100 selected (42 %; max force std dev ≥ 0.239 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `job_died` | remote job died (killed / lost) | 8 |
| `job_failed` | remote job failed | 6 |
| `jobs_started` | remote jobs started | 5 |
| `md_summary` | MD summary | 1 |
| `md_volume_expanded` | MD runs cut after expanding past max_volume_ratio | 1 |
| `mlip_relax_dropped` | candidates dropped by MLIP relaxation | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_flagged` | redundancy removal passes | 1 |
| `short_bond_excluded` | structures dropped for a short bond | 2 |
