# ALomancy report: al_loop_2

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-06T10:08:49

**Status:** 💡 1 info · 4 warning(s) logged

## Headline

| | loop 1 | loop 2 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.219 | 0.206 | -6% |
| Best model test energy MAE (meV/atom) | 31.5 | 18.1 | -43% |
| Training structures | 4.19e+03 | 4.45e+03 | +6% |
| Test structures | 431 | 460 | +7% |
| New structures this loop | 292 | 298 | +2% |
| Warnings / errors logged | 8 | 4 | -50% |

Best model: fit_2 of 3 (chosen on the `valid` split).

## Issues & suggestions

- 💡 **overfitting**: The best model's test force MAE (0.206 eV/Å) is 2.2x its training MAE (0.0926 eV/Å). The test set covers configurations training doesn't: keep adding data, or raise general.dataset_kwargs.valid_fraction to select models on held-out data.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.23 | 3958 | 115 | 100/100 | 100% | 0 | 1 |
| 1 | 0.219 | 4186 | 292 | 96/100 | 99% | 0 | 8 |
| 2 | 0.206 | 4449 | 298 | 99/100 | 98% | 0 | 4 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

No predictions stored for this loop's best model.

## Training set

| config_type | Train | Test | Excluded: quality filter | total |
|---|---:|---:|---:|---:|
| CNa_dataset | 3538 | 392 | 662 | 4592 |
| IsolatedAtom | 1 | 0 | 0 | 1 |
| high_sd | 591 | 64 | 50 | 705 |
| init_MP | 7 | 1 | 0 | 8 |
| init_amorphous | 26 | 3 | 0 | 29 |
| init_dimer | 99 | 0 | 0 | 99 |
| init_rattle | 5 | 0 | 269 | 274 |
| init_stretch_compress | 120 | 0 | 148 | 268 |
| init_trimer | 50 | 0 | 0 | 50 |
| on_hull | 1 | 0 | 0 | 1 |
| on_hull_expansion | 11 | 0 | 9 | 20 |
| **all** | **4449** | **460** | **1138** | **6047** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 298 new.
- Quality filters: high_force: 1129, formation_energy: 21.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 4,326 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 99 returned of 100 submitted (99 relaxed, 0 single point).
- Geometry optimisation: 97/99 reached the force ceiling (0.05 eV/Å); steps mean 29.1, median 21, max 200 of 200 allowed.
- Average DFT time per relaxation: 85.8 min (median 34.8, max 616.1; 99 structures).
- Largest per-atom force after DFT: median 0.04, max 0.26 eV/Å.
- DFT phase wall-clock: 34.9 h (mean queue time 240 min).

## MLIP training: mace

- Best fit (fit_2) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 249.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: MD

- Runs: 50 (50 completed, 0 seed(s) replaced)
- Candidate frames: 1000 (per completed run: median 200, min 200)
- Ensemble npt, T = 300 K, timestep 0.3 fs

![MD frames per run](plots/md_frames_per_run.png)

![MD temperature](plots/md_temperature.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 829 candidates scored; 100 selected (max force std dev ≥ 0.27 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `blown_up_excluded` |  | 1 |
| `dft_summary` | DFT batch summary | 1 |
| `job_failed` | remote job failed | 1 |
| `jobs_started` | remote jobs started | 4 |
| `md_summary` | MD summary | 1 |
| `md_volume_expanded` |  | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |
