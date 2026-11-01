# ALomancy report: al_loop_47

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-11-01T22:43:36

**Status:** ✖ 2 errors · ⚠ 4 warnings · 27 warning(s) logged

## Headline

| | loop 46 | loop 47 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.353 | 0.373 | +6% |
| Best model test energy MAE (meV/atom) | 207 | 164 | -21% |
| Training structures | 2.06e+03 | 2.08e+03 | +1% |
| Test structures | 107 | 112 | +5% |
| New structures this loop | 54 | 30 | -44% |
| Warnings / errors logged | 50 | 27 | -46% |

Best model: fit_3 of 5 (chosen on the `valid` split).

## Issues & suggestions

- ✖ **remote_jobs_failed**: 21/81 remote jobs (26%) failed (14) or died (7). Check the job logs (results/alomancy.log has each failure's traceback); jobs killed by the scheduler usually need a longer max_time for that phase, or a partition with more memory.
- ✖ **dft_partial_failure**: 15/45 structures sent to DFT (33%) came back without a result. Check the DFT output in results/<loop>/high_accuracy_evaluation; raise high_accuracy_evaluation.max_time / max_go_time if jobs timed out.
- ⚠ **go_not_converged**: 8/30 geometry optimisations (27%) stopped at the step budget (200 steps) before reaching the force ceiling. Raise high_accuracy_evaluation.max_num_of_relax_steps, or raise high_accuracy_evaluation.force_ceiling (now 0.05 eV/Å) so they stop earlier. Non-converged structures are still kept as training data.
- ⚠ **short_bond_excluded**: 3454/3864 structures (89%) were dropped for a bond shorter than 0.5 Å. The model is driving MD into collisions: lower md_kwargs.temperature or md_kwargs.timestep_fs, or add more short-range initialization data (dimers/trimers).
- ⚠ **fewer_candidates**: Only 45 candidates were available for 100 requested structures. Generate more: raise md_kwargs.steps or structure_selection_kwargs.num_of_md_starts.
- ⚠ **few_candidates_generated**: Structure generation produced only 45 usable candidates, less than twice the 100 structures added per loop, so the selector had little to choose from. Raise structure_generation.num_of_structures_to_generate (and md_kwargs.steps or structure_selection_kwargs.num_of_md_starts if MD runs end early).

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

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 4.2%, force 34.8%; test energy 2.8%, force 264.6%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: redundant | Excluded: quality filter | total |
|---|---:|---:|---:|---:|---:|
| IsolatedAtom | 5 | 0 | 0 | 0 | 5 |
| high_sd | 684 | 96 | 28 | 454 | 1262 |
| init_MP | 69 | 4 | 0 | 0 | 73 |
| init_amorphous | 67 | 12 | 8 | 0 | 87 |
| init_dimer | 299 | 0 | 0 | 0 | 299 |
| init_stretch_compress | 390 | 0 | 0 | 398 | 788 |
| init_trimer | 571 | 0 | 0 | 0 | 571 |
| **all** | **2085** | **112** | **36** | **852** | **3085** |

- Redundancy removal: 36 structure(s) flagged in total, 0 of this loop's 30 new.
- DFT geometry-optimisation starting frames: 30 stored this loop, 5 used (the rest excluded by the train/test filters or redundancy removal).
- Quality filters: high_force: 850, formation_energy: 243.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 856 structures probed, tolerance 0.0398, 36 flagged as duplicates (7 plateau(s) found).

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 30 returned of 45 submitted (30 relaxed, 0 single point).
- Geometry optimisation: 22/30 reached the force ceiling (0.05 eV/Å); steps mean 153.6, median 141, max 200 of 200 allowed.
- Average DFT time per relaxation: 353.5 min (median 340.3, max 569.2; 30 structures).
- Largest per-atom force after DFT: median 0.05, max 0.30 eV/Å.
- DFT phase wall-clock: 9.6 h (mean queue time 3 min).

## MLIP training: mace

- Best fit (fit_3) trained for 199 epochs, stage two from epoch 160; weights kept from epoch 171.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: MD

- Runs: 25 (25 completed, 0 seed(s) replaced)
- Frames: 3869 recorded, 365 physical (no bond < 0.5 Å, volume ≤ 3 x the seed's; 345 frame(s) cut after a run expanded past it, seed frames left out), 365 picked as candidates, spread over each run
- Ensemble npt, T = 300 K, timestep 0.3 fs

![MD frames per run](plots/md_frames_per_run.png)

![MD temperature](plots/md_temperature.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 45 candidates scored; 45 selected (100 %; max force std dev ≥ 0.332 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `few_candidates_generated` | fewer than 2x per-loop candidates generated | 1 |
| `fewer_candidates` | fewer candidates than requested | 1 |
| `job_died` | remote job died (killed / lost) | 7 |
| `job_failed` | remote job failed | 14 |
| `jobs_started` | remote jobs started | 5 |
| `md_summary` | MD summary | 1 |
| `md_volume_expanded` | MD runs cut after expanding past max_volume_ratio | 1 |
| `mlip_relax_dropped` | candidates dropped by MLIP relaxation | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_flagged` | redundancy removal passes | 1 |
| `short_bond_excluded` | structures dropped for a short bond | 2 |
