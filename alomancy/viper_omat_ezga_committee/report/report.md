# ALomancy report: al_loop_0

Workflow `committee_uncertainty` · trainer `mace` · generator `ezga` · evaluator `qe` · generated 2026-11-14T09:45:32

**Status:** 💡 2 infos · 3 warning(s) logged

## Headline

| | previous | loop 0 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | n/a | 0.193 |  |
| Best model test energy MAE (meV/atom) | n/a | 40.4 |  |
| Training structures | n/a | 456 |  |
| Test structures | n/a | 14 |  |
| New structures this loop | n/a | 48 |  |
| Warnings / errors logged | n/a | 3 |  |

Best model: fit_1 of 3 (chosen on the `valid` split).

## Issues & suggestions

- 💡 **queue_dominated**: high_accuracy_evaluation spent 78% of its time (5.6 h) queued. Raise max_num_of_concurrent_jobs in the HPC profile or use a less busy partition.
- 💡 **overfitting**: The best model's test force MAE (0.193 eV/Å) is 2.1x its training MAE (0.0904 eV/Å). The test set covers configurations training doesn't: keep adding data, or raise general.dataset_kwargs.valid_fraction to select models on held-out data.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.193 | 456 | 48 | 48/50 | 100% | 0 | 3 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 1.9%, force 7.1%; test energy 0.3%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: quality filter | total |
|---|---:|---:|---:|---:|
| IsolatedAtom | 1 | 0 | 0 | 1 |
| high_sd | 86 | 10 | 0 | 96 |
| init_MP | 7 | 1 | 0 | 8 |
| init_amorphous | 26 | 3 | 0 | 29 |
| init_dimer | 99 | 0 | 0 | 99 |
| init_rattle | 46 | 0 | 228 | 274 |
| init_stretch_compress | 141 | 0 | 127 | 268 |
| init_trimer | 50 | 0 | 0 | 50 |
| **all** | **456** | **14** | **355** | **825** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 48 new.
- DFT geometry-optimisation starting frames: 48 stored this loop, 48 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 355, formation_energy: 12.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 119 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 48 returned of 50 submitted (48 relaxed, 0 single point).
- Geometry optimisation: 48/48 reached the force ceiling (0.05 eV/Å); steps mean 26.3, median 18, max 107 of 200 allowed.
- Average DFT time per relaxation: 11.5 min (median 7.6, max 43.0; 48 structures).
- Largest per-atom force after DFT: median 0.04, max 0.05 eV/Å.
- DFT phase wall-clock: 7.2 h (mean queue time 335 min).

**Structures evaluated with DFT this loop (final geometries)**

![Structures evaluated with DFT this loop (final geometries)](plots/added_structures.gif)

## MLIP training: mace

- Best fit (fit_1) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 269.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: EZGA

- Seeds: 8, generations run: 10 (max_generations 10, population 100)
- Structures generated: 406, taken as candidates: 406 (newest first, back to generation 0)
- Mutations: mutation_rattle, mutation_random_strain, bounded_mutation_add, bounded_mutation_remove

![Ezga generations](plots/ezga_generations.png)

![Ezga selected by seed type](plots/ezga_selected_by_seed_type.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 406 candidates scored; 50 selected (12 %; max force std dev ≥ 0.809 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `ezga_summary` | EZGA summary | 1 |
| `job_failed` | remote job failed | 2 |
| `jobs_started` | remote jobs started | 4 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |
