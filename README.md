# RC-GPF

## 1. Method Overview

The method targets a practical UAV-VLN setting where an agent can use external geometric priors, but the priors may be imperfect. In real aerial navigation, maps can be misregistered, stale, or partially missing. Directly injecting such priors into a navigation policy can improve clean-map performance but may also make the agent brittle under corruption.

RC-GPF treats the prior map as uncertain action-level evidence rather than a fixed oracle. At each navigation step, it:

1. Builds action-conditioned geometric evidence from the prior map.
2. Estimates whether the local prior is clean, misregistered, stale, or recoverably missing.
3. Applies calibrated operators to use, correct, recover, or suppress the prior evidence.
4. Fuses the calibrated prior with the base vision-language policy through reliability-aware product-of-experts fusion.

## 2. Repository Layout

```text
RC-GPF/
├── Annotation/
│   ├── seen.json
│   └── unseen.json
├── configs/
│   └── Evaluation and prior-map configs
├── eval_outputs/
│   └── Historical reference logs and fresh reproduction outputs
├── my_tools/
│   └── Project-specific helper scripts
├── outputs/
│   ├── ortho_prior_mapsv4
│   ├── ortho_prior_mapsv4_train7
│   ├── ortho_prior_mapsv4_test_seen
│   └── ortho_prior_mapsv4_test_unseen
├── paper_figures/
│   └── Reproducible analysis figures
├── reports/
│   ├── step2_reference_bundle
│   └── step3_main_table_freeze_20260605
├── scripts/
│   ├── Experiment launch scripts
│   └── repro/run_screenshot_results.sh
├── tools/
│   ├── summarize_screenshot_results.py
│   └── verify_rc_gpf_package.py
├── train/
│   └── Source code required by the preserved evaluation scripts
├── MIGRATION_MANIFEST.md
└── README.md
```


## 3. Reproducing Results

Step 2: enter the repository.

```bash
git clone https://github.com/xiaoran093/RC-GPF.git
cd RC-GPF
```


Step 3: rerun the clean-prior and corrupted-prior experiments.

```bash
bash scripts/repro/run_screenshot_results.sh
```

By default, fresh outputs are written to:

```text
eval_outputs/repro_screenshot_results_<timestamp>/
```

To choose the output directory manually:

```bash
RESULT_ROOT=eval_outputs/repro_screenshot_results_manual \
  bash scripts/repro/run_screenshot_results.sh
```

