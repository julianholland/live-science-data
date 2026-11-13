# ALomancy report: al_loop_54

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-13T20:52:00

**Status:** ✖ 2 errors · ⚠ 2 warnings · 💡 1 info · 33 warning(s) logged

## Headline

| | loop 53 | loop 54 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.299 | 0.285 | -5% |
| Best model test energy MAE (meV/atom) | 145 | 135 | -7% |
| Training structures | 2.59e+03 | 2.68e+03 | +3% |
| Test structures | 169 | 180 | +7% |
| New structures this loop | 60 | 28 | -53% |
| Warnings / errors logged | 41 | 33 | -20% |

Best model: fit_4 of 5 (chosen on the `valid` split).

## Issues & suggestions

- ✖ **remote_jobs_failed**: 29/141 remote jobs (21%) failed (23) or died (6). Check the job logs (results/alomancy.log has each failure's traceback); jobs killed by the scheduler usually need a longer max_time for that phase, or a partition with more memory.
- ✖ **dft_partial_failure**: 23/100 structures sent to DFT (23%) came back without a result. Check the DFT output in results/<loop>/high_accuracy_evaluation; raise high_accuracy_evaluation.max_time / max_go_time if jobs timed out.
- ⚠ **go_not_converged**: 9/77 geometry optimisations (12%) stopped at the step budget (200 steps) before reaching the force ceiling. Raise high_accuracy_evaluation.max_num_of_relax_steps, or raise high_accuracy_evaluation.force_ceiling (now 0.05 eV/Å) so they stop earlier. Non-converged structures are still kept as training data.
- ⚠ **short_bond_excluded**: 3610/7852 structures (46%) were dropped for a bond shorter than 0.5 Å. The model is driving MD into collisions: lower md_kwargs.temperature or md_kwargs.timestep_fs, or add more short-range initialization data (dimers/trimers).
- 💡 **queue_dominated**: training_plots spent 58% of its time (5.5 h) queued. Raise max_num_of_concurrent_jobs in the HPC profile or use a less busy partition.

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
| 53 | 0.299 | 2586 | 60 | 67/100 | 84% | 0 | 41 |
| 54 | 0.285 | 2675 | 28 | 77/100 | 88% | 0 | 33 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 2.6%, force 24.7%; test energy 2.2%, force 137.9%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: redundant | Excluded: quality filter | total |
|---|---:|---:|---:|---:|---:|
| IsolatedAtom | 5 | 0 | 0 | 0 | 5 |
| high_sd | 1274 | 164 | 28 | 930 | 2396 |
| init_MP | 69 | 4 | 0 | 0 | 73 |
| init_amorphous | 67 | 12 | 8 | 0 | 87 |
| init_dimer | 299 | 0 | 0 | 0 | 299 |
| init_stretch_compress | 390 | 0 | 0 | 398 | 788 |
| init_trimer | 571 | 0 | 0 | 0 | 571 |
| **all** | **2675** | **180** | **36** | **1328** | **4219** |

- Redundancy removal: 36 structure(s) flagged in total, 0 of this loop's 28 new.
- DFT geometry-optimisation starting frames: 126 stored this loop, 72 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 1326, formation_energy: 382.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 1,446 structures probed, tolerance 0.0398, 36 flagged as duplicates (7 plateau(s) found).

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 77 returned of 100 submitted (77 relaxed, 0 single point).
- Geometry optimisation: 68/77 reached the force ceiling (0.05 eV/Å); steps mean 132.4, median 135, max 200 of 200 allowed.
- Average DFT time per relaxation: 356.1 min (median 361.0, max 578.1; 77 structures).
- Largest per-atom force after DFT: median 0.05, max 0.23 eV/Å.
- DFT phase wall-clock: 26.3 h (mean queue time 553 min).

**Structures evaluated with DFT this loop (final geometries)**

![Structures evaluated with DFT this loop (final geometries)](plots/added_structures.gif)

## MLIP training: mace

- Best fit (fit_4) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 252.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: MD

- Runs: 30 (30 completed, 1 seed(s) replaced)
- Frames: 7263 recorded, 3686 physical (no bond < 0.5 Å, volume ≤ 3 x the seed's; 381 frame(s) cut after a run expanded past it, seed frames left out), 1000 picked as candidates, spread over each run
- Ensemble npt, T = 300 K, timestep 0.1 fs

![MD frames per run](plots/md_frames_per_run.png)

![MD temperature](plots/md_temperature.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 556 candidates scored; 100 selected (18 %; max force std dev ≥ 0.833 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `job_died` | remote job died (killed / lost) | 6 |
| `job_failed` | remote job failed | 23 |
| `jobs_started` | remote jobs started | 5 |
| `md_summary` | MD summary | 1 |
| `md_volume_expanded` | MD runs cut after expanding past max_volume_ratio | 1 |
| `mlip_relax_dropped` | candidates dropped by MLIP relaxation | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_flagged` | redundancy removal passes | 1 |
| `short_bond_excluded` | structures dropped for a short bond | 2 |
