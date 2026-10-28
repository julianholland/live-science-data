# ALomancy report: al_loop_43

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-10-28T05:12:34

**Status:** ✖ 2 errors · ⚠ 4 warnings · 💡 1 info · 39 warning(s) logged

## Headline

| | loop 42 | loop 43 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.333 | 0.473 | +42% |
| Best model test energy MAE (meV/atom) | 147 | 189 | +28% |
| Training structures | 1.93e+03 | 1.97e+03 | +2% |
| Test structures | 90 | 96 | +7% |
| New structures this loop | 34 | 57 | +68% |
| Warnings / errors logged | 31 | 39 | +26% |

Best model: fit_2 of 5 (chosen on the `valid` split).

## Issues & suggestions

- ✖ **remote_jobs_failed**: 30/112 remote jobs (27%) failed (15) or died (15). Check the job logs (results/alomancy.log has each failure's traceback); jobs killed by the scheduler usually need a longer max_time for that phase, or a partition with more memory.
- ✖ **dft_partial_failure**: 18/75 structures sent to DFT (24%) came back without a result. Check the DFT output in results/<loop>/high_accuracy_evaluation; raise high_accuracy_evaluation.max_time / max_go_time if jobs timed out.
- ⚠ **go_not_converged**: 13/57 geometry optimisations (23%) stopped at the step budget (200 steps) before reaching the force ceiling. Raise high_accuracy_evaluation.max_num_of_relax_steps, or raise high_accuracy_evaluation.force_ceiling (now 0.05 eV/Å) so they stop earlier. Non-converged structures are still kept as training data.
- ⚠ **short_bond_excluded**: 142/217 structures (65%) were dropped for a bond shorter than 0.5 Å. The model is driving MD into collisions: lower md_kwargs.temperature or md_kwargs.timestep_fs, or add more short-range initialization data (dimers/trimers).
- ⚠ **fewer_candidates**: Only 75 candidates were available for 100 requested structures. Generate more: raise md_kwargs.steps or structure_selection_kwargs.num_of_md_starts.
- ⚠ **few_candidates_generated**: Structure generation produced only 75 usable candidates, less than twice the 100 structures added per loop, so the selector had little to choose from. Raise structure_generation.num_of_structures_to_generate (and md_kwargs.steps or structure_selection_kwargs.num_of_md_starts if MD runs end early).
- 💡 **overfitting**: The best model's test force MAE (0.473 eV/Å) is 2.7x its training MAE (0.178 eV/Å). The test set covers configurations training doesn't: keep adding data, or raise general.dataset_kwargs.valid_fraction to select models on held-out data.

<details><summary>Suggestions file notes</summary>

- suggestions entry `selection_redundant_shortfall` has no trigger (typo?)

</details>

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

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 4.0%, force 18.5%; test energy 2.3%, force 238.8%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: redundant | Excluded: quality filter | total |
|---|---:|---:|---:|---:|---:|
| IsolatedAtom | 5 | 0 | 0 | 0 | 5 |
| high_sd | 569 | 80 | 47 | 346 | 1042 |
| init_MP | 69 | 4 | 0 | 0 | 73 |
| init_amorphous | 68 | 12 | 7 | 0 | 87 |
| init_dimer | 299 | 0 | 0 | 0 | 299 |
| init_stretch_compress | 390 | 0 | 0 | 398 | 788 |
| init_trimer | 571 | 0 | 0 | 0 | 571 |
| **all** | **1971** | **96** | **54** | **744** | **2865** |

- Redundancy removal: 54 structure(s) flagged in total, 3 of this loop's 57 new.
- Quality filters: high_force: 742, formation_energy: 174.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 712 structures probed, tolerance 0.0398, 6 flagged as duplicates (9 plateau(s) found).

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 57 returned of 75 submitted (57 relaxed, 0 single point).
- Geometry optimisation: 44/57 reached the force ceiling (0.05 eV/Å); steps mean 143.9, median 143, max 200 of 200 allowed.
- Average DFT time per relaxation: 294.6 min (median 300.0, max 580.8; 57 structures).
- Largest per-atom force after DFT: median 0.05, max 0.49 eV/Å.
- DFT phase wall-clock: 54.6 h (mean queue time 516 min).

## MLIP training: mace

- Best fit (fit_2) trained for 199 epochs, stage two from epoch 160; weights kept from epoch 162.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: MD

- Runs: 27 (25 completed, 2 seed(s) replaced)
- Candidate frames: 217 (per completed run: median 5, min 2)
- Ensemble npt, T = 300 K, timestep 0.3 fs

![MD frames per run](plots/md_frames_per_run.png)

![MD temperature](plots/md_temperature.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 75 candidates scored; 75 selected (max force std dev ≥ 0 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `few_candidates_generated` | fewer than 2x per-loop candidates generated | 1 |
| `fewer_candidates` | fewer candidates than requested | 1 |
| `job_died` | remote job died (killed / lost) | 15 |
| `job_failed` | remote job failed | 15 |
| `job_resumed` | transient remote error, polling resumed | 5 |
| `jobs_started` | remote jobs started | 5 |
| `md_no_steps` | MD runs replaced (no MD step) | 1 |
| `md_summary` | MD summary | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_flagged` | redundancy removal passes | 1 |
| `short_bond_excluded` | structures dropped for a short bond | 1 |
