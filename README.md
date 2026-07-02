# RC-GPF

## 1. Method Overview

**RC-GPF** is the code and reproduction package for **Calibrating Imperfect Maps for Robust UAV Vision-and-Language Navigation**.

The method targets a practical UAV-VLN setting where an agent can use external geometric priors, but the priors may be imperfect. In real aerial navigation, maps can be misregistered, stale, or partially missing. Directly injecting such priors into a navigation policy can improve clean-map performance but may also make the agent brittle under corruption.

RC-GPF treats the prior map as uncertain action-level evidence rather than a fixed oracle. At each navigation step, it:

1. Builds action-conditioned geometric evidence from the prior map.
2. Estimates whether the local prior is clean, misregistered, stale, or recoverably missing.
3. Applies calibrated operators to use, correct, recover, or suppress the prior evidence.
4. Fuses the calibrated prior with the base vision-language policy through reliability-aware product-of-experts fusion.

This repository keeps the code, configs, prior maps, and scripts needed to rerun the reported experiments. Model weights, training data, and simulator assets are intentionally not included.

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

The included prior-map data is under `outputs/`. New reproduction runs are written under `eval_outputs/`.

## 3. Reproducing The Screenshot Results

The screenshot results should be reproduced by rerunning evaluation, not by directly counting the copied historical logs. The copied logs are only kept as reference artifacts.

Step 1: restore the external assets that are not included in this repository.

```text
openfly-agent-7b/
runs/local_lora_baseline_tuning_20260501_231110/lr1e-5_const_stop0p2_warm100_r16/baseline_no_map_lora_stopreg/n0+b1+x7/checkpoints/step-000400-epoch-00-loss=0.0031.pt
runs/local_lora_map_influence_phase2_gated_20260430_205653/main/gf_lam0p03_gate0p003_lr2e6_s100/residual_geometry_full_lora_maponly/n0+b1+x7/checkpoints/step-000100-epoch-00-loss=0.0509.pt
reports/action_affordance_denoiser_train7_veto/model/action_affordance_denoiser.pt
```

You can either place the files at these paths or create symbolic links to your local copies. The prior maps used by the experiments are already included under `outputs/`.

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

