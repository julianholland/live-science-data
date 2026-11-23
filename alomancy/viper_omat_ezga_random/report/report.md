# ALomancy report: al_loop_1

Workflow `random_selection` · trainer `mace` · generator `ezga` · evaluator `qe` · generated 2026-11-23T09:16:19

**Status:** 💡 2 infos · 4 warning(s) logged

## Headline

| | loop 0 | loop 1 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.0724 | 0.136 | +88% |
| Best model test energy MAE (meV/atom) | 33.6 | 39.8 | +18% |
| Training structures | 449 | 533 | +19% |
| Test structures | 28 | 38 | +36% |
| New structures this loop | 50 | 48 | -4% |
| Warnings / errors logged | 2 | 4 | +100% |

Best model: fit_0 of 1 (chosen on the `valid` split).

## Issues & suggestions

- 💡 **queue_dominated**: training_plots spent 99% of its time (25.3 h) queued. Raise max_num_of_concurrent_jobs in the HPC profile or use a less busy partition.
- 💡 **overfitting**: The best model's test force MAE (0.136 eV/Å) is 2.7x its training MAE (0.0497 eV/Å). The test set covers configurations training doesn't: keep adding data, or raise general.dataset_kwargs.valid_fraction to select models on held-out data.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.0724 | 449 | 50 | 50/50 | 98% | 0 | 2 |
| 1 | 0.136 | 533 | 48 | 48/50 | 98% | 0 | 4 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 0.6%, force 6.9%; test energy 0.4%, force 21.7%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: quality filter | total |
|---|---:|---:|---:|---:|
| IsolatedAtom | 1 | 0 | 0 | 1 |
| init_MP | 7 | 1 | 0 | 8 |
| init_amorphous | 26 | 3 | 0 | 29 |
| init_dimer | 79 | 0 | 0 | 79 |
| init_rattle | 41 | 1 | 232 | 274 |
| init_stretch_compress | 126 | 13 | 129 | 268 |
| init_trimer | 79 | 0 | 0 | 79 |
| random_selection | 174 | 20 | 0 | 194 |
| **all** | **533** | **38** | **361** | **932** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 48 new.
- DFT geometry-optimisation starting frames: 46 stored this loop, 46 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 361, formation_energy: 17.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 248 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 48 returned of 50 submitted (48 relaxed, 0 single point).
- Geometry optimisation: 47/48 reached the force ceiling (0.05 eV/Å); steps mean 27.2, median 18, max 200 of 200 allowed.
- Average DFT time per relaxation: 17.9 min (median 12.0, max 138.9; 48 structures).
- Largest per-atom force after DFT: median 0.04, max 0.62 eV/Å.
- DFT phase wall-clock: 52.8 h (mean queue time 3046 min).

**Structures evaluated with DFT this loop (final geometries)**

![Structures evaluated with DFT this loop (final geometries)](plots/added_structures.gif)

## MLIP training: mace

- Best fit (fit_0) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 253.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: EZGA

- Seeds: 8, generations run: 11 (max_generations 10, population 100)
- Structures generated: 565, taken as candidates: 500 (newest first, back to generation 4)
- Mutations: mutation_rattle, mutation_random_strain, bounded_mutation_add, bounded_mutation_remove

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `ezga_summary` | EZGA summary | 1 |
| `job_failed` | remote job failed | 2 |
| `job_still_queued` | jobs still queued after their time limit (waiting on) | 50 |
| `jobs_started` | remote jobs started | 3 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |
| `stale_sqlite_sidecars_removed` |  | 1 |
