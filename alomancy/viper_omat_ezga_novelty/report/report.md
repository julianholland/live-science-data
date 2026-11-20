# ALomancy report: al_loop_0

Workflow `novelty_selection` · trainer `mace` · generator `ezga` · evaluator `qe` · generated 2026-11-20T05:03:19

**Status:** 💡 1 info · 4 warning(s) logged

## Headline

| | previous | loop 0 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | n/a | 0.0724 |  |
| Best model test energy MAE (meV/atom) | n/a | 33.6 |  |
| Training structures | n/a | 447 |  |
| Test structures | n/a | 28 |  |
| New structures this loop | n/a | 49 |  |
| Warnings / errors logged | n/a | 4 |  |

Best model: fit_0 of 1 (chosen on the `valid` split).

## Issues & suggestions

- 💡 **queue_dominated**: generate_structures spent 99% of its time (15.8 h) queued. Raise max_num_of_concurrent_jobs in the HPC profile or use a less busy partition.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.0724 | 447 | 49 | 49/50 | 100% | 0 | 4 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 1.0%, force 6.3%; test energy 0.3%, force 16.7%.
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
| novelty_selection | 88 | 10 | 0 | 98 |
| **all** | **447** | **28** | **361** | **836** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 49 new.
- DFT geometry-optimisation starting frames: 49 stored this loop, 49 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 361, formation_energy: 17.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 162 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 49 returned of 50 submitted (49 relaxed, 0 single point).
- Geometry optimisation: 49/49 reached the force ceiling (0.05 eV/Å); steps mean 21.8, median 16, max 85 of 200 allowed.
- Average DFT time per relaxation: 13.6 min (median 10.6, max 59.9; 49 structures).
- Largest per-atom force after DFT: median 0.04, max 0.05 eV/Å.
- DFT phase wall-clock: 47.4 h (mean queue time 2714 min).

**Structures evaluated with DFT this loop (final geometries)**

![Structures evaluated with DFT this loop (final geometries)](plots/added_structures.gif)

## MLIP training: mace

- Best fit (fit_0) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 244.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: EZGA

- Seeds: 8, generations run: 11 (max_generations 10, population 100)
- Structures generated: 484, taken as candidates: 484 (newest first, back to generation 0)
- Mutations: mutation_rattle, mutation_random_strain, bounded_mutation_add, bounded_mutation_remove

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: novelty

- 50 of 484 candidates selected against 738 reference structure(s), tolerance 0.877 (descriptor distance).

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `ezga_summary` | EZGA summary | 1 |
| `job_failed` | remote job failed | 1 |
| `job_still_queued` | jobs still queued after their time limit (waiting on) | 50 |
| `jobs_started` | remote jobs started | 2 |
| `novelty_selected` | novelty selection | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |
| `ssh_auth_lost` | ssh login lost (waited for a new login) | 1 |
| `stale_sqlite_sidecars_removed` |  | 1 |
