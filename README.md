# roboparty_train

[![IsaacSim](https://img.shields.io/badge/IsaacSim-6.2.0-silver.svg)](https://docs.omniverse.nvidia.com/isaacsim/latest/overview.html)
[![Isaac Lab](https://img.shields.io/badge/IsaacLab-3.0.0-silver)](https://isaac-sim.github.io/IsaacLab)
[![RSL_RL](https://img.shields.io/badge/RSL_RL-5.5.1-silver)](https://github.com/leggedrobotics/rsl_rl)
[![Python](https://img.shields.io/badge/python-3.12-blue.svg)](https://docs.python.org/3/whatsnew/3.12.html)
[![Linux platform](https://img.shields.io/badge/platform-linux--64-orange.svg)](https://releases.ubuntu.com/22.04/)
[![License](https://img.shields.io/badge/license-BSD--3-yellow.svg)](https://opensource.org/licenses/BSD-3-Clause)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://pre-commit.com/)

[English](README.md) | [中文](README_CN.md)

## Overview

`roboparty_train` provides training, evaluation, and policy export for Roboparty RPO and RP1 robots, built on Isaac Lab and RSL-RL.

The repository contains two submodules:

- `robolab`: Roboparty Isaac Lab extension, environments, scripts, and robot assets.
- `rsl_rl`: RSL-RL fork with AMP and Parkour policy support.

## Features

- RPO and RP1 training environments built on Isaac Lab.
- RSL-RL training, evaluation, and policy export entry points.
- AMP, BeyondMimic, and Parkour workflows.
- AMP and BeyondMimic data conversion from [HHTools](https://github.com/Roboparty/human-humanoid-tools) exports.

## Requirements

- Python 3.12.
- Isaac Lab 3.0.0 GA, with MJWarp (Newton 1.6.1) or Isaac Sim 6.2 for PhysX.
- The included RSL-RL 5.5.1 fork.
- Ubuntu 22.04 x64.

Prepare the Isaac Lab 3.0 source checkout and system prerequisites following the [official installation guide](https://docs.nvidia.com/isaaclab/3.0/isaac-lab/installation). The installation below creates the project's Python environments.

## Clone

Clone the repository and its submodules:

```bash
git clone --recursive https://github.com/Roboparty/roboparty_train.git
cd roboparty_train
```

For an existing checkout:

```bash
git submodule update --init --recursive
```

## Install

Keep the Isaac Lab 3.0.0 source checkout alongside this repository. From the project root:

```bash
git clone --branch v3.0.0 --recursive https://github.com/isaac-sim/IsaacLab.git ../IsaacLab-3.0.0
```

For an existing v3.0.0 checkout, create a symbolic link at `../IsaacLab-3.0.0` instead. Run the installation commands below from the project root.

MJWarp and PhysX use separate Python environments.

MJWarp (default):

```bash
uv sync --locked
source .venv/bin/activate
```

PhysX (Isaac Sim):

```bash
UV_PROJECT_ENVIRONMENT=.venv-physx uv sync --locked --no-default-groups \
    --extra physx --no-install-package usd-exchange
source .venv-physx/bin/activate
```

Keep the PhysX sync options when updating that environment. After activation, run commands with `python` or `isaaclab` directly. Add `--extra video` or `--extra wandb` to the corresponding sync command for video recording or W&B logging.

Verify that the extension is installed by listing the available environments:

```bash
isaaclab list_envs --keyword RP
```

The output should include both `RPO-` and `RP1-` tasks.

## Usage

### Train

Training defaults to MJWarp (`physics=newton_mjwarp`). For PhysX, activate
`.venv-physx` and add `physics=isaacsim_physx`. Kit visualization also uses this environment.

```bash
python robolab/scripts/rsl_rl/train.py --task=RP1-Flat --viz=none --logger=tensorboard --num_envs=4096
```

### Play

Playback defaults to MJWarp. The commands below open a Newton GL window and load the task's latest training checkpoint; use `--checkpoint=/path/to/model.pt` to select a model explicitly. Use `--viz=none` for headless playback. For PhysX, activate `.venv-physx` and add `physics=isaacsim_physx`; replace `--viz=newton_gl` with `--viz=kit` for the Isaac Sim viewport.

```bash
python robolab/scripts/rsl_rl/play.py --task=RP1-Flat --num_envs=1 --checkpoint=latest --viz=newton_gl
```

### Play AMP

```bash
python robolab/scripts/rsl_rl/play.py --task=RPO-AMP-Play --num_envs=1 --checkpoint=latest --viz=newton_gl
```

### Play BeyondMimic

```bash
python robolab/scripts/rsl_rl/play.py --task=RPO-BeyondMimic --num_envs=1 --checkpoint=latest --viz=newton_gl
```

### Play Parkour

```bash
python robolab/scripts/rsl_rl/play.py --task=RPO-Parkour-Play --num_envs=1 --checkpoint=latest --viz=newton_gl
```

Playback exports models to `exported/` in the selected training run. Parkour additionally exports `0-depth_encoder.onnx` and `actor.onnx`; keep their accompanying `.onnx.data` files for deployment.

### Prepare Motion Data

The repository includes the following training data under `robolab/data/motions/`:

| Robot | AMP (`.pkl`) | BeyondMimic (`.npz`) |
| --- | --- | --- |
| RPO | `rpo_amp` | `rpo_bm` |
| RP1 | `rp1_amp` | `rp1_bm` |

Retarget motions in [HHTools](https://github.com/Roboparty/human-humanoid-tools) using the matching robot URDF. Export robot CSV with comments and column headers, or robot PKL, then convert it to training data:

```bash
# AMP: 120 Hz.
python robolab/scripts/tools/hhtools/convert_motion.py --robot rp1 --output_fps 120 \
    -f /path/to/hhtools/walk.csv -o robolab/data/motions/rp1_amp/walk.pkl

# BeyondMimic: 50 Hz.
python robolab/scripts/tools/hhtools/convert_motion.py --robot rp1 --output_fps 50 \
    -f /path/to/hhtools/walk.csv -o robolab/data/motions/rp1_bm/walk.npz
```

Use `--robot rpo` for RPO. Without `--output_fps`, AMP retains the source rate and BeyondMimic defaults to 50 Hz.

## Troubleshooting

- Empty `robolab` or `rsl_rl` directory: run `git submodule update --init --recursive`.
- Missing Isaac Lab imports: activate the Python environment used by Isaac Lab before installing or running scripts.
- Missing task name: run `isaaclab list_envs --keyword RP` and copy the exact task id.
- PhysX training may become abnormally slow in some scenarios.

## References and Thanks

This project builds on the following open-source projects:

- [IsaacLab](https://github.com/isaac-sim/IsaacLab)
- [rsl_rl](https://github.com/leggedrobotics/rsl_rl)
- [legged_gym](https://github.com/leggedrobotics/legged_gym)
- [legged_lab](https://github.com/zitongbai/legged_lab)
- [robot_lab](https://github.com/fan-ziqi/robot_lab)
- [InstinctLab](https://github.com/project-instinct/InstinctLab)
