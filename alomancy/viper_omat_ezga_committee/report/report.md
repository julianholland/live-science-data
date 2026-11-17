# ALomancy report: al_loop_0

Workflow `committee_uncertainty` · trainer `mace` · generator `ezga` · evaluator `qe` · generated 2026-11-17T08:26:58

**Status:** ✖ 2 errors · 💡 1 info · 5 warning(s) logged

## Headline

| | previous | loop 0 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | n/a | 0.0683 |  |
| Best model test energy MAE (meV/atom) | n/a | 32 |  |
| Training structures | n/a | 436 |  |
| Test structures | n/a | 27 |  |
| New structures this loop | n/a | 47 |  |
| Warnings / errors logged | n/a | 5 |  |

Best model: fit_2 of 3 (chosen on the `valid` split).

## Issues & suggestions

- ✖ **remote_jobs_failed**: 3/57 remote jobs (5%) failed (3) or died (0). Check the job logs (results/alomancy.log has each failure's traceback); jobs killed by the scheduler usually need a longer max_time for that phase, or a partition with more memory.
- ✖ **dft_partial_failure**: 3/50 structures sent to DFT (6%) came back without a result. Check the DFT output in results/<loop>/high_accuracy_evaluation; raise high_accuracy_evaluation.max_time / max_go_time if jobs timed out.
- 💡 **queue_dominated**: training_plots spent 99% of its time (12.3 h) queued. Raise max_num_of_concurrent_jobs in the HPC profile or use a less busy partition.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.0683 | 436 | 47 | 47/50 | 100% | 0 | 5 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 1.0%, force 6.6%; test energy 0.3%, force 16.6%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: quality filter | total |
|---|---:|---:|---:|---:|
| IsolatedAtom | 1 | 0 | 0 | 1 |
| high_sd | 77 | 9 | 8 | 94 |
| init_MP | 7 | 1 | 0 | 8 |
| init_amorphous | 26 | 3 | 0 | 29 |
| init_dimer | 79 | 0 | 0 | 79 |
| init_rattle | 41 | 1 | 232 | 274 |
| init_stretch_compress | 126 | 13 | 129 | 268 |
| init_trimer | 79 | 0 | 0 | 79 |
| **all** | **436** | **27** | **369** | **832** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 47 new.
- DFT geometry-optimisation starting frames: 47 stored this loop, 39 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 369, formation_energy: 17.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 151 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 47 returned of 50 submitted (47 relaxed, 0 single point).
- Geometry optimisation: 47/47 reached the force ceiling (0.05 eV/Å); steps mean 25.1, median 15, max 167 of 200 allowed.
- Average DFT time per relaxation: 16.2 min (median 10.7, max 96.9; 47 structures).
- Largest per-atom force after DFT: median 0.04, max 0.05 eV/Å.
- DFT phase wall-clock: 3.6 h (mean queue time 98 min).

**Structures evaluated with DFT this loop (final geometries)**

![Structures evaluated with DFT this loop (final geometries)](plots/added_structures.gif)

## MLIP training: mace

- Best fit (fit_2) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 249.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: EZGA

- Seeds: 8, generations run: 11 (max_generations 10, population 100)
- Structures generated: 497, taken as candidates: 497 (newest first, back to generation 0)
- Mutations: mutation_rattle, mutation_random_strain, bounded_mutation_add, bounded_mutation_remove

![Ezga generations](plots/ezga_generations.png)

![Ezga selected by seed type](plots/ezga_selected_by_seed_type.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 497 candidates scored; 50 selected (10 %; max force std dev ≥ 0.29 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `ezga_summary` | EZGA summary | 1 |
| `job_failed` | remote job failed | 3 |
| `jobs_started` | remote jobs started | 4 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |
| `stale_sqlite_sidecars_removed` |  | 1 |
