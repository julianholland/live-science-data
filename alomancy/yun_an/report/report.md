# ALomancy report: al_loop_44

Workflow `committee_uncertainty` · trainer `mace` · generator `md` · evaluator `qe` · generated 2026-10-30T05:42:37

**Status:** ✖ 2 errors · ⚠ 4 warnings · 33 warning(s) logged

## Headline

| | loop 43 | loop 44 | change |
|---|---:|---:|---:|
| Best model test force MAE (eV/Å) | 0.473 | 0.346 | -27% |
| Best model test energy MAE (meV/atom) | 189 | 144 | -24% |
| Training structures | 1.97e+03 | 2e+03 | +1% |
| Test structures | 96 | 100 | +4% |
| New structures this loop | 57 | 40 | -30% |
| Warnings / errors logged | 39 | 33 | -15% |

Best model: fit_3 of 5 (chosen on the `valid` split).

## Issues & suggestions

- ✖ **remote_jobs_failed**: 29/92 remote jobs (32%) failed (8) or died (21). Check the job logs (results/alomancy.log has each failure's traceback); jobs killed by the scheduler usually need a longer max_time for that phase, or a partition with more memory.
- ✖ **dft_partial_failure**: 15/55 structures sent to DFT (27%) came back without a result. Check the DFT output in results/<loop>/high_accuracy_evaluation; raise high_accuracy_evaluation.max_time / max_go_time if jobs timed out.
- ⚠ **go_not_converged**: 11/40 geometry optimisations (28%) stopped at the step budget (200 steps) before reaching the force ceiling. Raise high_accuracy_evaluation.max_num_of_relax_steps, or raise high_accuracy_evaluation.force_ceiling (now 0.05 eV/Å) so they stop earlier. Non-converged structures are still kept as training data.
- ⚠ **short_bond_excluded**: 225/280 structures (80%) were dropped for a bond shorter than 0.5 Å. The model is driving MD into collisions: lower md_kwargs.temperature or md_kwargs.timestep_fs, or add more short-range initialization data (dimers/trimers).
- ⚠ **fewer_candidates**: Only 55 candidates were available for 100 requested structures. Generate more: raise md_kwargs.steps or structure_selection_kwargs.num_of_md_starts.
- ⚠ **few_candidates_generated**: Structure generation produced only 55 usable candidates, less than twice the 100 structures added per loop, so the selector had little to choose from. Raise structure_generation.num_of_structures_to_generate (and md_kwargs.steps or structure_selection_kwargs.num_of_md_starts if MD runs end early).

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
| 44 | 0.346 | 1997 | 40 | 40/55 | 72% | 5 | 33 |

![Best-model MAE per loop](plots/training_al_loop_mae_plot.png)

![Time per phase and loop](plots/timing_combined.png)

## Best model

![Best model parity](plots/best_model_parity.png)

Median relative error: train energy 2.3%, force 20.5%; test energy 1.8%, force 242.6%.
Energies with |formation energy| < 0.05 eV/atom and atoms with |F| < 0.1 eV/Å are left out; force error is |ΔF|/|F| per atom.

![Best model percentage error](plots/best_model_percent_error.png)

## Training set

| config_type | Train | Test | Excluded: redundant | Excluded: quality filter | total |
|---|---:|---:|---:|---:|---:|
| IsolatedAtom | 5 | 0 | 0 | 0 | 5 |
| high_sd | 596 | 84 | 56 | 346 | 1082 |
| init_MP | 69 | 4 | 0 | 0 | 73 |
| init_amorphous | 67 | 12 | 8 | 0 | 87 |
| init_dimer | 299 | 0 | 0 | 0 | 299 |
| init_stretch_compress | 390 | 0 | 0 | 398 | 788 |
| init_trimer | 571 | 0 | 0 | 0 | 571 |
| **all** | **1997** | **100** | **64** | **744** | **2905** |

- Redundancy removal: 64 structure(s) flagged in total, 5 of this loop's 40 new.
- Quality filters: high_force: 742, formation_energy: 174.

![Training set composition](plots/training_set_composition.png)

### Redundancy removal

Redundancy removal flags near-duplicate training structures, so the model isn't trained on many copies of the same geometry. Each structure is described by `char_vec_128`: all of its interatomic distances, sorted and resampled to 128 values (`global_descriptor/atomic_distance_descriptor.make_char_vec`). Two structures count as duplicates when the Euclidean distance between their descriptors is below a tolerance: in database order, each structure still in the training set is kept unless it is within the tolerance of one already kept. The plot sweeps that tolerance and counts how many structures are kept at each value. **We are looking for plateaus** (shaded): ranges where the count doesn't change, so the result doesn't depend on the exact tolerance. The orange line (right axis) is the number of structures removed between consecutive tolerance steps, the gradient of the count: zero on a plateau. The start of the lowest plateau at least `general.redundancy_kwargs.min_plateau_width` of the swept range wide is used (dashed line). Nothing is flagged when no plateau qualifies, or when it would flag more than `max_flagged_fraction` of the structures; `general.redundancy_kwargs.tolerance` sets the tolerance directly.

- This loop: 742 structures probed, tolerance 0.0398, 10 flagged as duplicates (8 plateau(s) found).

![Redundancy tolerance probe](plots/redundancy_tolerance_probe.png)

## DFT

- Structures: 40 returned of 55 submitted (40 relaxed, 0 single point).
- Geometry optimisation: 29/40 reached the force ceiling (0.05 eV/Å); steps mean 136.9, median 142, max 200 of 200 allowed.
- Average DFT time per relaxation: 277.1 min (median 287.4, max 581.5; 40 structures).
- Largest per-atom force after DFT: median 0.05, max 0.74 eV/Å.
- DFT phase wall-clock: 36.1 h (mean queue time 338 min).

## MLIP training: mace

- Best fit (fit_3) trained for 199 epochs, stage two from epoch 160; weights kept from epoch 191.

![Best model training curve](plots/best_model_training_curve.png)

## Structure generation: MD

- Runs: 27 (25 completed, 3 seed(s) replaced)
- Candidate frames: 280 (per completed run: median 9, min 2)
- Ensemble npt, T = 300 K, timestep 0.3 fs

![MD frames per run](plots/md_frames_per_run.png)

![MD temperature](plots/md_temperature.png)

## High-accuracy evaluation: QE

![DFT geometry steps](plots/dft_geometry_steps.png)

![DFT wall time](plots/dft_wall_time.png)

## Selection: committee uncertainty

- 55 candidates scored; 55 selected (max force std dev ≥ 0 eV/Å).

![Committee force std dev](plots/committee_force_std_dev.png)

## Warnings and events

| event | meaning | count |
|---|---|---:|
| `dft_summary` | DFT batch summary | 1 |
| `few_candidates_generated` | fewer than 2x per-loop candidates generated | 1 |
| `fewer_candidates` | fewer candidates than requested | 1 |
| `job_died` | remote job died (killed / lost) | 21 |
| `job_failed` | remote job failed | 8 |
| `jobs_started` | remote jobs started | 5 |
| `md_no_steps` | MD runs replaced (no MD step) | 1 |
| `md_summary` | MD summary | 1 |
| `quality_filtered` | train/test quality filter passes | 2 |
| `redundancy_flagged` | redundancy removal passes | 1 |
| `short_bond_excluded` | structures dropped for a short bond | 1 |
