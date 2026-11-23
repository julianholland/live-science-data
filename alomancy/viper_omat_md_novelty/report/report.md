# ALomancy report: al_loop_1

Workflow `novelty_selection` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-23T09:10:41

**Status:** 💡 1 info · 4 warning(s) logged

## Headline

| | loop 0 | loop 1 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.0724 | 0.111 | +53% |
| Best model test energy MAE (meV/atom) | 33.6 | 31.7 | -6% |
| Training structures | 443 | 529 | +19% |
| Test structures | 28 | 38 | +36% |
| New structures this loop | 48 | 19 | -60% |
| Warnings / errors logged | 7 | 4 | -43% |

Best model: fit_0 of 1 (chosen on the `valid` split).

## Issues & suggestions

- 💡 **queue_dominated**: high_accuracy_evaluation spent 58% of its time (1.4 h) queued. Raise max_num_of_concurrent_jobs in the HPC profile or use a less busy partition.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.0724 | 443 | 48 | 48/50 | 98% | 0 | 7 |
| 1 | 0.111 | 529 | 19 | 48/50 | 100% | 0 | 4 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 0.6%, force 6.2%; test energy 0.2%, force 8.0%.
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
| novelty_selection | 170 | 20 | 2 | 192 |
| **all** | **529** | **38** | **363** | **930** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 19 new.
- DFT geometry-optimisation starting frames: 77 stored this loop, 77 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 363, formation_energy: 17.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 244 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 48 returned of 50 submitted (48 relaxed, 0 single point).
- Geometry optimisation: 48/48 reached the force ceiling (0.05 eV/Å); steps mean 18.3, median 19, max 33 of 200 allowed.
- Average DFT time per relaxation: 11.6 min (median 10.4, max 28.1; 48 structures).
- Largest per-atom force after DFT: median 0.04, max 0.05 eV/Å.
- DFT phase wall-clock: 2.3 h (mean queue time 82 min).

**Structures evaluated with DFT this loop (final geometries)**

![Structures evaluated with DFT this loop (final geometries)](plots/added_structures.gif)

## MLIP training: mace

- Best fit (fit_0) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 241.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: MD

- Runs: 50 (50 completed, 0 seed(s) replaced)
- Frames: 5000 recorded, 4950 physical (no bond < 0.5 Å, volume ≤ 3 x the seed's, seed frames left out), 500 picked as candidates, spread over each run
- Ensemble npt, T = 500 K, timestep 0.2 fs

![MD frames per run](plots/md_frames_per_run.png)

![MD temperature](plots/md_temperature.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: novelty

- 50 of 500 candidates selected against 834 reference structure(s), tolerance 0.561 (descriptor distance).

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `job_failed` | remote job failed | 2 |
| `jobs_started` | remote jobs started | 3 |
| `md_summary` | MD summary | 1 |
| `novelty_inexact` | novelty tolerance not exact | 1 |
| `novelty_selected` | novelty selection | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |
