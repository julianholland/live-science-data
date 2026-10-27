# ALomancy report: al_loop_42

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-10-25T13:55:33

**Status:** ✖ 2 errors · ⚠ 4 warnings · 💡 1 info · 31 warning(s) logged

## Headline

| | loop 41 | loop 42 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.469 | 0.333 | -29% |
| Best model test energy MAE (meV/atom) | 152 | 147 | -3% |
| Training structures | 1.9e+03 | 1.93e+03 | +2% |
| Test structures | 86 | 90 | +5% |
| New structures this loop | 27 | 34 | +26% |
| Warnings / errors logged | 30 | 31 | +3% |

Best model: fit_4 of 5 (chosen on the `valid` split).

## Issues & suggestions

- ✖ **remote_jobs_failed**: 26/81 remote jobs (32%) failed (8) or died (18). Check the job logs (results/alomancy.log has each failure's traceback); jobs killed by the scheduler usually need a longer max_time for that phase, or a partition with more memory.
- ✖ **dft_partial_failure**: 11/45 structures sent to DFT (24%) came back without a result. Check the DFT output in results/<loop>/high_accuracy_evaluation; raise high_accuracy_evaluation.max_time / max_go_time if jobs timed out.
- ⚠ **go_not_converged**: 8/34 geometry optimisations (24%) stopped at the step budget (200 steps) before reaching the force ceiling. Raise high_accuracy_evaluation.max_num_of_relax_steps, or raise high_accuracy_evaluation.force_ceiling (now 0.05 eV/Å) so they stop earlier. Non-converged structures are still kept as training data.
- ⚠ **short_bond_excluded**: 190/235 structures (81%) were dropped for a bond shorter than 0.5 Å. The model is driving MD into collisions: lower md_kwargs.temperature or md_kwargs.timestep_fs, or add more short-range initialization data (dimers/trimers).
- ⚠ **fewer_candidates**: Only 45 candidates were available for 100 requested structures. Generate more: raise md_kwargs.steps or structure_selection_kwargs.num_of_md_starts.
- ⚠ **few_candidates_generated**: Structure generation produced only 45 usable candidates, less than twice the 100 structures added per loop, so the selector had little to choose from. Raise structure_generation.num_of_structures_to_generate (and md_kwargs.steps or structure_selection_kwargs.num_of_md_starts if MD runs end early).
- 💡 **go_steps_near_budget**: Geometry optimisations used 161 of 200 allowed steps on average (80%). Consider raising high_accuracy_evaluation.max_num_of_relax_steps before they start hitting the limit.

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

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 2.9%, force 19.0%; test energy 2.0%, force 236.7%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: redundant | Excluded: quality filter | total |
|---|---:|---:|---:|---:|---:|
| IsolatedAtom | 5 | 0 | 0 | 0 | 5 |
| high_sd | 521 | 74 | 44 | 346 | 985 |
| init_MP | 69 | 4 | 0 | 0 | 73 |
| init_amorphous | 71 | 12 | 4 | 0 | 87 |
| init_dimer | 299 | 0 | 0 | 0 | 299 |
| init_stretch_compress | 390 | 0 | 0 | 398 | 788 |
| init_trimer | 571 | 0 | 0 | 0 | 571 |
| **all** | **1926** | **90** | **48** | **744** | **2808** |

- Redundancy removal: 48 structure(s) flagged in total, 0 of this loop's 34 new.
- Quality filters: high_force: 742, formation_energy: 174.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 661 structures probed; the lowest plateau would flag more than 50% of them, so nothing was flagged.

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 34 returned of 45 submitted (34 relaxed, 0 single point).
- Geometry optimisation: 26/34 reached the force ceiling (0.05 eV/Å); steps mean 160.6, median 168, max 200 of 200 allowed.
- Average DFT time per relaxation: 336.8 min (median 336.0, max 506.6; 34 structures).
- Largest per-atom force after DFT: median 0.05, max 0.27 eV/Å.
- DFT phase wall-clock: 22.1 h (mean queue time 82 min).

## MLIP training: mace

- Best fit (fit_4) trained for 199 epochs, stage two from epoch 160; weights kept from epoch 187.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: MD

- Runs: 26 (25 completed, 1 seed(s) replaced)
- Candidate frames: 235 (per completed run: median 5, min 2)
- Ensemble npt, T = 300 K, timestep 0.3 fs

![MD frames per run](plots/md_frames_per_run.png)

![MD temperature](plots/md_temperature.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 45 candidates scored; 45 selected (max force std dev ≥ 0 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `few_candidates_generated` | fewer than 2x per-loop candidates generated | 1 |
| `fewer_candidates` | fewer candidates than requested | 1 |
| `job_died` | remote job died (killed / lost) | 18 |
| `job_failed` | remote job failed | 8 |
| `jobs_started` | remote jobs started | 5 |
| `md_no_steps` | MD runs replaced (no MD step) | 1 |
| `md_summary` | MD summary | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_cap_hit` | redundancy removal skipped (would flag too many) | 1 |
| `short_bond_excluded` | structures dropped for a short bond | 1 |
