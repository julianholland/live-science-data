# ALomancy report: al_loop_4

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-08T05:13:49

**Status:** 💡 1 info · 3 warning(s) logged

## Headline

| | loop 3 | loop 4 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.205 | 0.201 | -2% |
| Best model test energy MAE (meV/atom) | 21.9 | 20.9 | -5% |
| Training structures | 4.71e+03 | 4.96e+03 | +5% |
| Test structures | 490 | 520 | +6% |
| New structures this loop | 295 | 300 | +2% |
| Warnings / errors logged | 5 | 3 | -40% |

Best model: fit_1 of 3 (chosen on the `valid` split).

## Issues & suggestions

- 💡 **overfitting**: The best model's test force MAE (0.201 eV/Å) is 2.4x its training MAE (0.0851 eV/Å). The test set covers configurations training doesn't: keep adding data, or raise general.dataset_kwargs.valid_fraction to select models on held-out data.

## Trends

| loop | test F-MAE (eV/Å) | train | new | DFT returned | GO converged | new redundant | warnings |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.23 | 3958 | 115 | 100/100 | 100% | 0 | 1 |
| 1 | 0.219 | 4186 | 292 | 96/100 | 99% | 0 | 8 |
| 2 | 0.206 | 4449 | 298 | 99/100 | 98% | 0 | 4 |
| 3 | 0.205 | 4708 | 295 | 98/100 | 100% | 0 | 5 |
| 4 | 0.201 | 4963 | 300 | 100/100 | 99% | 0 | 3 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

No predictions stored for this loop's best model.

## Training set

| config_type | Train | Test | Excluded: quality filter | total |
|---|---:|---:|---:|---:|
| CNa_dataset | 3538 | 392 | 662 | 4592 |
| IsolatedAtom | 1 | 0 | 0 | 1 |
| high_sd | 1105 | 124 | 71 | 1300 |
| init_MP | 7 | 1 | 0 | 8 |
| init_amorphous | 26 | 3 | 0 | 29 |
| init_dimer | 99 | 0 | 0 | 99 |
| init_rattle | 5 | 0 | 269 | 274 |
| init_stretch_compress | 120 | 0 | 148 | 268 |
| init_trimer | 50 | 0 | 0 | 50 |
| on_hull | 1 | 0 | 0 | 1 |
| on_hull_expansion | 11 | 0 | 9 | 20 |
| **all** | **4963** | **520** | **1159** | **6642** |

- Redundancy removal: 0 structure(s) flagged in total, 0 of this loop's 300 new.
- Quality filters: high_force: 1150, formation_energy: 21.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 4,676 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 100 returned of 100 submitted (100 relaxed, 0 single point).
- Geometry optimisation: 99/100 reached the force ceiling (0.05 eV/Å); steps mean 28.4, median 16, max 200 of 200 allowed.
- Average DFT time per relaxation: 52.5 min (median 11.3, max 376.1; 100 structures).
- Largest per-atom force after DFT: median 0.04, max 2.36 eV/Å.
- DFT phase wall-clock: 6.4 h (mean queue time 3 min).

## MLIP training: mace

- Best fit (fit_1) trained for 299 epochs, stage two from epoch 240; weights kept from epoch 244.

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

- 824 candidates scored; 100 selected (max force std dev ≥ 0.253 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `blown_up_excluded` |  | 1 |
| `dft_summary` | DFT batch summary | 1 |
| `jobs_started` | remote jobs started | 4 |
| `md_summary` | MD summary | 1 |
| `md_volume_expanded` |  | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |
