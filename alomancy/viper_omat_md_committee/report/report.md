# ALomancy report: al_loop_0

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-23T06:36:33

**Status:** 💡 1 info · 3 warning(s) logged

## Headline

| | previous | loop 0 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | n/a | 0.0683 |  |
| Best model test energy MAE (meV/atom) | n/a | 32 |  |
| Training structures | n/a | 442 |  |
| Test structures | n/a | 24 |  |
| New structures this loop | n/a | 49 |  |
| Warnings / errors logged | n/a | 3 |  |

Best model: fit_2 of 3 (chosen on the `valid` split).

## Issues & suggestions

- 💡 **queue_dominated**: high_accuracy_evaluation spent 98% of its time (96.9 h) queued. Raise max_num_of_concurrent_jobs in the HPC profile or use a less busy partition.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.0683 | 442 | 49 | 49/50 | 98% | 0 | 3 |

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
| high_sd | 83 | 6 | 9 | 98 |
| init_MP | 7 | 1 | 0 | 8 |
| init_amorphous | 26 | 3 | 0 | 29 |
| init_dimer | 79 | 0 | 0 | 79 |
| init_rattle | 41 | 1 | 232 | 274 |
| init_stretch_compress | 126 | 13 | 129 | 268 |
| init_trimer | 79 | 0 | 0 | 79 |
| **all** | **442** | **24** | **370** | **836** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 49 new.
- DFT geometry-optimisation starting frames: 49 stored this loop, 40 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 370, formation_energy: 17.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 157 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 49 returned of 50 submitted (49 relaxed, 0 single point).
- Geometry optimisation: 48/49 reached the force ceiling (0.05 eV/Å); steps mean 27.4, median 17, max 200 of 200 allowed.
- Average DFT time per relaxation: 15.3 min (median 10.6, max 102.7; 49 structures).
- Largest per-atom force after DFT: median 0.04, max 1.25 eV/Å.
- DFT phase wall-clock: 99.0 h (mean queue time 5813 min).

**Structures evaluated with DFT this loop (final geometries)**

![Structures evaluated with DFT this loop (final geometries)](plots/added_structures.gif)

## MLIP training: mace

- Best fit (fit_2) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 249.

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

## Selection: committee uncertainty

- 500 candidates scored; 50 selected (10 %; max force std dev ≥ 0.163 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `job_failed` | remote job failed | 1 |
| `job_still_queued` | jobs still queued after their time limit (waiting on) | 167 |
| `jobs_started` | remote jobs started | 3 |
| `md_summary` | MD summary | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |

Other warnings (most frequent first):

- (1x) Only 30 structures available for 50 concurrent structure_generation jobs; reusing structures to fill the requested concurrency. Each reused structure is assigned a distinct MD seed (atoms.info['md_see
