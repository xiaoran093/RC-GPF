# RC-GPF

Official implementation of **Calibrating Imperfect Maps for Robust UAV
Vision-and-Language Navigation**.

RC-GPF calibrates clean, misregistered, stale, and partially missing geometric
priors before fusing them with a frozen UAV vision-language navigation policy.
The code uses the environments, OpenFly-Agent, annotations, and evaluation
protocol of [OpenFly-Platform](https://github.com/SHAILAB-IPEC/OpenFly-Platform).

## 1. Method Overview

At each navigation step, RC-GPF extracts action-conditioned geometric evidence
from the local prior map. According to the evaluated prior condition, it applies
score correction or local map recovery and then combines the calibrated map
score with the OpenFly-Agent policy score.

The experiments cover:

- clean prior;
- misregistered prior;
- stale-obstacle prior;
- recoverable-missing prior.

## 2. Repository Layout

```text
RC-GPF/
|-- Annotation/                  # OpenFly test-seen/test-unseen episodes
|-- configs/                     # Evaluation and environment configs
|-- scripts/repro/               # Map collection and evaluation launchers
|-- train/                       # Policy loading and simulator evaluation
|-- tools/                       # Result aggregation
|-- outputs/                     # Locally generated prior maps
`-- reproduction_outputs/        # Fresh evaluation results
```

Model weights, simulator packages, prior maps, training data, and historical
logs are not included in this repository.

## 3. Reproduction

### 3.1 Prepare OpenFly

Install [OpenFly-Platform](https://github.com/SHAILAB-IPEC/OpenFly-Platform)
and download its AirSim/Unreal Engine scenes and OpenFly-Agent. Use the same
Python environment for RC-GPF:

```bash
conda activate openfly
cd /path/to/RC-GPF
export PYTHON_BIN="$(command -v python)"
pip install -r requirements.txt
```

Place or link the external files at the following paths:

```text
openfly-agent-7b/
runtime/base_policy/config.json
runtime/base_policy/dataset_statistics.json
runtime/base_policy/checkpoints/base_policy.pt
runtime/clean_prior_policy/config.json
runtime/clean_prior_policy/dataset_statistics.json
runtime/clean_prior_policy/checkpoints/clean_prior_policy.pt
checkpoints/action_affordance_denoiser.pt
envs/airsim/
envs/ue/
```

Link the assets from a local OpenFly checkout:

```bash
SOURCE_ROOT=/path/to/OpenFly-Platform \
  bash scripts/repro/setup_local_assets.sh
```

### 3.2 Collect Prior Maps

Prior maps are generated from the OpenFly simulator scenes:

```bash
# All seven test-seen environments and one test-unseen environment
bash scripts/repro/collect_prior_maps.sh all
```

Optional subsets:

```bash
bash scripts/repro/collect_prior_maps.sh seen
bash scripts/repro/collect_prior_maps.sh unseen
bash scripts/repro/collect_prior_maps.sh env_airsim_16 env_ue_smallcity
```

Generated maps are saved under `outputs/ortho_prior_mapsv4/`. Existing complete
maps are skipped; use `FORCE=1` to regenerate them.

### 3.3 Preflight

```bash
python scripts/repro/preflight.py
```

### 3.4 Run Evaluations

Each command runs both test-seen and test-unseen. Cases are executed
sequentially.

```bash
# Clean prior: 2 evaluations
bash scripts/repro/run_experiments.sh clean

# Three corrupted priors + Naive: 6 evaluations
bash scripts/repro/run_experiments.sh naive

# Three corrupted priors + Ours: 6 evaluations
bash scripts/repro/run_experiments.sh ours
```

Run one or two corruption types by listing them after `naive` or `ours`:

```bash
bash scripts/repro/run_experiments.sh naive misregistered stale
bash scripts/repro/run_experiments.sh ours recoverable-missing
```

Supported names are `misregistered`, `stale`, and `recoverable-missing`. Use
`--plan` to print the selected cases without starting a simulator.

To place all 14 evaluations in one directory:

```bash
export RESULT_ROOT=/absolute/path/to/reproduction_outputs/paper_main
bash scripts/repro/run_experiments.sh clean
RESUME=1 bash scripts/repro/run_experiments.sh naive
RESUME=1 bash scripts/repro/run_experiments.sh ours
```

The aggregated metrics are written to `fresh_evaluation_summary.md` and
`fresh_evaluation_summary.json` under `RESULT_ROOT`.

## Acknowledgements

We thank the OpenFly authors for releasing the aerial VLN benchmark,
OpenFly-Agent, simulation environments, and evaluation toolchain. Please cite
[OpenFly](https://arxiv.org/abs/2502.18041) when using this repository:

```bibtex
@article{OpenFly,
  author       = {Yunpeng Gao and Chenhui Li and Zhongrui You and Junli Liu and
                  Zhen Li and Pengan Chen and Qizhi Chen and Zhonghan Tang and
                  Liansheng Wang and Penghui Yang and Yiwen Tang and Yuhang Tang and
                  Shuai Liang and Songyi Zhu and Ziqin Xiong and Yifei Su and
                  Xinyi Ye and Jianan Li and Yan Ding and Dong Wang and Zhigang Wang and
                  Bin Zhao and Xuelong Li},
  title        = {OpenFly: A Comprehensive Platform for Aerial Vision-Language Navigation},
  journal      = {CoRR},
  volume       = {abs/2502.18041},
  year         = {2025}
}
```
