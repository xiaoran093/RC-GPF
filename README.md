# RC-GPF: Calibrating Imperfect Maps for Robust UAV Vision-and-Language Navigation

Code, prior maps, evaluation logs, and table-reproduction scripts for:

**Calibrating Imperfect Maps for Robust UAV Vision-and-Language Navigation**  
Xiaoran Miao, Shiyu Liu, Shang Wu, Jiangfeng She

RC-GPF studies a practical but often hidden failure mode in UAV vision-and-language navigation: agents may receive prior maps, but real maps are rarely clean. They can be misregistered, stale, or partially missing. Instead of trusting a map as a fixed planning input, RC-GPF turns it into calibrated action-level evidence and fuses it with a frozen vision-language policy.

> Project page / code: <https://github.com/xiaoran093/RC-GPF>

## Highlights

- **Robust UAV-VLN with imperfect priors.** RC-GPF targets map corruption that naturally appears in aerial navigation: pose misalignment, outdated obstacles, and recoverable missing structure.
- **Action-conditioned geometric evidence.** Prior maps are projected into forward-action corridors and represented as action-level evidence rather than global dense supervision.
- **Reliability-calibrated fusion.** The method decouples what the map suggests from how much the agent should trust it, using latent prior states and calibrated product-of-experts fusion.
- **Plug-and-play inference.** The policy can remain frozen; RC-GPF operates as a geometry-prior fusion module during evaluation.

## Method Overview

RC-GPF treats a prior map as an uncertain observation about the traversability and usefulness of each candidate action. For every navigation step, the system:

1. Builds action-conditioned geometric evidence from the prior map.
2. Infers a latent reliability state of the prior around the agent.
3. Applies calibrated operators to use, correct, recover, or suppress prior evidence.
4. Fuses the calibrated prior with the vision-language policy through a reliability-constrained product of experts.

This design lets the agent benefit from clean maps while avoiding brittle behavior when the map is corrupted.

## Main Results

Lower NE is better. Higher SR, OSR, and SPL are better.

### Table 1. Main UAV-VLN Results With Clean Prior

| Method | Test-Seen NE (m) | Test-Seen SR (%) | Test-Seen OSR (%) | Test-Seen SPL (%) | Test-Unseen NE (m) | Test-Unseen SR (%) | Test-Unseen OSR (%) | Test-Unseen SPL (%) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Random | 242 | 0.7 | 0.8 | 0.0 | 301 | 0.1 | 0.1 | 0.0 |
| Seq2Seq | 205 | 2.9 | 24.3 | 2.6 | 229 | 2.1 | 20.6 | 1.1 |
| CMA | 161 | 5.4 | 28.1 | 4.8 | 217 | 4.6 | 24.4 | 2.1 |
| See-Point-Fly | - | - | - | - | 191 | 8.2 | 12.7 | 6.3 |
| AerialVLN | 129 | 7.5 | 30.0 | 6.8 | 214 | 7.3 | 28.1 | 4.4 |
| Navid | 153 | 13.0 | 38.2 | 11.6 | 210 | 10.8 | 27.2 | 5.0 |
| NaVila | 132 | 20.3 | 53.5 | 17.8 | 202 | 14.7 | 42.1 | 9.6 |
| OpenFly-Agent | 102 | 24.14 | 56.32 | 20.13 | 185 | 10.89 | 51.24 | 9.96 |
| **Ours w/ clean prior** | **72** | **28.59** | 50.07 | **25.02** | **159** | **13.12** | 50.00 | **11.48** |

### Table 2. Robustness Under Corrupted Priors

| Corrupted Prior | Method | Test-Seen NE | Test-Seen SR | Test-Seen OSR | Test-Seen SPL | Test-Unseen NE | Test-Unseen SR | Test-Unseen OSR | Test-Unseen SPL |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Misregistered | Naive | 97 | 25.72 | 56.11 | 21.63 | 179 | 11.14 | 50.50 | 9.88 |
| Misregistered | **Ours** | **68** | **27.80** | 49.07 | **23.10** | **125** | **11.39** | 46.29 | **10.17** |
| Stale obstacle | Naive | 101 | 25.14 | 55.82 | 20.71 | 186 | 11.14 | 52.23 | 10.12 |
| Stale obstacle | **Ours** | **84** | **31.61** | 55.03 | **26.74** | **140** | **12.38** | 47.03 | **10.77** |
| Recoverable missing | Naive | 101 | 24.78 | 54.89 | 20.44 | 186 | 11.39 | 51.73 | 9.98 |
| Recoverable missing | **Ours** | **59** | **33.69** | 50.57 | **30.20** | **66** | **16.09** | 50.09* | **16.09** |

*Reproducibility note: the migrated raw logs currently recompute `Recoverable missing / Test-Unseen / Ours / OSR` as `16.09`, while the paper-facing table records `50.09`. The verification script flags this single-cell discrepancy so it can be audited before release.

## Reproducing The Two Tables

This repository is organized as a lightweight reproducibility package. It includes the code, configuration files, prior maps, result logs, and table verification scripts needed to audit the reported numbers. It intentionally does **not** redistribute model weights or training data.

### 1. Reproduce Tables From Released Logs

This path is the fastest way to verify the reported table values and does not require GPUs, model checkpoints, or the training set.

```bash
git clone https://github.com/xiaoran093/RC-GPF.git
cd RC-GPF
python tools/verify_rc_gpf_package.py --root .
```

The script recomputes the table metrics from the migrated JSON, CSV, and JSONL evaluation artifacts. It also checks that the package does not contain redistributed weights or training annotations.

Expected result:

```text
OK: migrated logs reproduce the checked result tables.
```

If the script reports the known `Recoverable missing / Test-Unseen / Ours / OSR` warning, compare the printed value with the footnote above. All other values in the two displayed tables are supported by the released logs.

### 2. Result Provenance

The verification script reads the following migrated artifacts:

| Table | Split / Setting | Main source files |
| --- | --- | --- |
| Table 1 | OpenFly-Agent reference | `reports/step2_reference_bundle/step2_reference_seen.json`, `reports/step2_reference_bundle/step2_reference_unseen.json` |
| Table 1 | Ours w/ clean prior | `eval_outputs/clean_enhanced_loop_only_weak_test_seen_20260514_224641/geometry_clean_corrupt_summary.json`, `eval_outputs/clean_enhanced_loop_only_weak_test_unseen_20260514_224641/geometry_clean_corrupt_summary.json` |
| Table 2 | Corrupted prior, test-seen | `reports/step3_main_table_freeze_20260605/main_table_seen_freeze.json` plus copied raw logs under `eval_outputs/` |
| Table 2 | Corrupted prior, test-unseen | `reports/step3_main_table_freeze_20260605/main_table_unseen_freeze.json` plus copied raw logs under `eval_outputs/` |

For a complete inventory of copied code, logs, and prior maps, see `MIGRATION_MANIFEST.md`.

### 3. Fresh Evaluation

Fresh model evaluation requires external assets that are not shipped in this repository:

- OpenFly / UAV-VLN simulator assets and evaluation data.
- Model checkpoints for the base navigation policy.
- The action-affordance or prior-calibration checkpoints referenced by the historical run logs.
- The same software environment used by the OpenFly-Platform evaluation scripts.

The released package does include the prior-map data needed by RC-GPF:

```text
outputs/ortho_prior_mapsv4
outputs/ortho_prior_mapsv4_train7
outputs/ortho_prior_mapsv4_test_seen
outputs/ortho_prior_mapsv4_test_unseen
```

After restoring the external checkpoints and simulator assets, use the preserved scripts in `scripts/`, `tools/`, and the copied `eval_outputs/*/run*.sh` files as the starting point for rerunning the historical jobs. Then run:

```bash
python tools/verify_rc_gpf_package.py --root .
```

to regenerate and audit the final table summaries.

## Repository Layout

```text
RC-GPF/
├── Annotation/                  # Test annotations only; training annotations are not redistributed
├── configs/                     # Evaluation and prior-map configuration files
├── eval_outputs/                # Selected logs needed to reproduce the paper tables
├── my_tools/                    # Project-specific utility scripts
├── outputs/                     # Prior maps used by the experiments
├── paper_figures/               # Reproducible analysis figures
├── reports/                     # Frozen table summaries and provenance bundles
├── scripts/                     # Evaluation and experiment-launch scripts
├── tools/                       # Verification and analysis scripts
├── train/                       # Source code needed by the preserved evaluation scripts
├── MIGRATION_MANIFEST.md        # What was copied and what was intentionally excluded
└── README.md
```

## Reproducibility Policy

This repository is intended for transparent paper auditing:

- **Included:** code, configs, prior maps, selected result logs, frozen summaries, and verification scripts.
- **Excluded:** model weights, checkpoints, training data, and large simulator assets.
- **Primary audit command:** `python tools/verify_rc_gpf_package.py --root .`

## Citation

If this work helps your research, please cite:

```bibtex
@misc{miao2026rcgpf,
  title  = {Calibrating Imperfect Maps for Robust UAV Vision-and-Language Navigation},
  author = {Miao, Xiaoran and Liu, Shiyu and Wu, Shang and She, Jiangfeng},
  year   = {2026},
  note   = {Preprint}
}
```
