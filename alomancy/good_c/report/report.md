# ALomancy report: al_loop_9

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-14T14:02:43

**Status:** ✖ 2 errors · 💡 1 info · 12 warning(s) logged

## Headline

| | loop 8 | loop 9 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.194 | 0.195 | +0% |
| Best model test energy MAE (meV/atom) | 28.7 | 35.5 | +24% |
| Training structures | 5.44e+03 | 5.59e+03 | +3% |
| Test structures | 582 | 601 | +3% |
| New structures this loop | 58 | 40 | -31% |
| Warnings / errors logged | 17 | 12 | -29% |

Best model: fit_0 of 3 (chosen on the `valid` split).

## Issues & suggestions

- ✖ **remote_jobs_failed**: 8/156 remote jobs (5%) failed (7) or died (1). Check the job logs (results/alomancy.log has each failure's traceback); jobs killed by the scheduler usually need a longer max_time for that phase, or a partition with more memory.
- ✖ **dft_partial_failure**: 8/100 structures sent to DFT (8%) came back without a result. Check the DFT output in results/<loop>/high_accuracy_evaluation; raise high_accuracy_evaluation.max_time / max_go_time if jobs timed out.
- 💡 **queue_dominated**: high_accuracy_evaluation spent 51% of its time (15.3 h) queued. Raise max_num_of_concurrent_jobs in the HPC profile or use a less busy partition.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.23 | 3958 | 115 | 100/100 | 100% | 0 | 1 |
| 1 | 0.219 | 4186 | 292 | 96/100 | 99% | 0 | 8 |
| 2 | 0.206 | 4449 | 298 | 99/100 | 98% | 0 | 4 |
| 3 | 0.205 | 4708 | 295 | 98/100 | 100% | 0 | 5 |
| 4 | 0.201 | 4963 | 300 | 100/100 | 99% | 0 | 3 |
| 5 | 0.198 | 5132 | 98 | 100/100 | 100% | 0 | 3 |
| 6 | 0.196 | 5263 | 75 | 92/100 | 100% | 0 | 12 |
| 7 | 0.197 | 5308 | 21 | 32/100 | 100% | 0 | 71 |
| 8 | 0.194 | 5440 | 58 | 86/100 | 99% | 0 | 17 |
| 9 | 0.195 | 5592 | 40 | 92/100 | 96% | 0 | 12 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 0.1%, force 11.4%; test energy 0.2%, force 18.7%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: quality filter | total |
|---|---:|---:|---:|---:|
| CNa_dataset | 3538 | 392 | 662 | 4592 |
| IsolatedAtom | 1 | 0 | 0 | 1 |
| high_sd | 1734 | 205 | 165 | 2104 |
| init_MP | 7 | 1 | 0 | 8 |
| init_amorphous | 26 | 3 | 0 | 29 |
| init_dimer | 99 | 0 | 0 | 99 |
| init_rattle | 5 | 0 | 269 | 274 |
| init_stretch_compress | 120 | 0 | 148 | 268 |
| init_trimer | 50 | 0 | 0 | 50 |
| on_hull | 1 | 0 | 0 | 1 |
| on_hull_expansion | 11 | 0 | 9 | 20 |
| **all** | **5592** | **601** | **1253** | **7446** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 40 new.
- DFT geometry-optimisation starting frames: 144 stored this loop, 131 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 1244, formation_energy: 21.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 5,305 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 92 returned of 100 submitted (92 relaxed, 0 single point).
- Geometry optimisation: 88/92 reached the force ceiling (0.05 eV/Å); steps mean 23.0, median 13, max 200 of 200 allowed.
- Average DFT time per relaxation: 91.2 min (median 52.4, max 732.4; 92 structures).
- Largest per-atom force after DFT: median 0.04, max 2.78 eV/Å.
- DFT phase wall-clock: 30.3 h (mean queue time 918 min).

**Structures evaluated with DFT this loop (final geometries)**

![Structures evaluated with DFT this loop (final geometries)](plots/added_structures.gif)

## MLIP training: mace

- Best fit (fit_0) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 247.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: MD

- Runs: 50 (50 completed, 0 seed(s) replaced)
- Frames: 10000 recorded, 8933 physical (no bond < 0.5 Å, volume ≤ 3 x the seed's; 1017 frame(s) cut after a run expanded past it, seed frames left out), 1000 picked as candidates, spread over each run
- Ensemble npt, T = 500 K, timestep 0.2 fs

![MD frames per run](plots/md_frames_per_run.png)

![MD temperature](plots/md_temperature.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 985 candidates scored; 100 selected (10 %; max force std dev ≥ 0.458 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `blown_up_excluded` | blown-up cells kept from DFT (volume >> vdW volume) | 1 |
| `dft_summary` | DFT batch summary | 1 |
| `job_died` | remote job died (killed / lost) | 1 |
| `job_failed` | remote job failed | 7 |
| `job_resumed` | transient remote error, polling resumed | 1 |
| `jobs_started` | remote jobs started | 4 |
| `md_summary` | MD summary | 1 |
| `md_volume_expanded` | MD runs cut after expanding past max_volume_ratio | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |
