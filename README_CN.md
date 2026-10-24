# roboparty_train

[![IsaacSim](https://img.shields.io/badge/IsaacSim-6.2.0-silver.svg)](https://docs.omniverse.nvidia.com/isaacsim/latest/overview.html)
[![Isaac Lab](https://img.shields.io/badge/IsaacLab-3.0.0-silver)](https://isaac-sim.github.io/IsaacLab)
[![RSL_RL](https://img.shields.io/badge/RSL_RL-5.5.1-silver)](https://github.com/leggedrobotics/rsl_rl)
[![Python](https://img.shields.io/badge/python-3.12-blue.svg)](https://docs.python.org/3/whatsnew/3.12.html)
[![Linux platform](https://img.shields.io/badge/platform-linux--64-orange.svg)](https://releases.ubuntu.com/22.04/)
[![License](https://img.shields.io/badge/license-BSD--3-yellow.svg)](https://opensource.org/licenses/BSD-3-Clause)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://pre-commit.com/)

[English](README.md) | [中文](README_CN.md)

## 概述

`roboparty_train` 基于 Isaac Lab 和 RSL-RL，提供 Roboparty RPO、RP1 机器人的策略训练、评估和导出功能。

仓库包含两个 submodule：

- `robolab`：Roboparty 的 Isaac Lab 扩展、环境、脚本和机器人资产。
- `rsl_rl`：支持 AMP 和 Parkour 策略的 RSL-RL fork。

## 主要特性

- 基于 Isaac Lab 的 RPO 和 RP1 训练环境。
- RSL-RL 训练、评估和策略导出入口。
- AMP、BeyondMimic 和 Parkour 工作流。
- 将 [HHTools](https://github.com/Roboparty/human-humanoid-tools) 导出结果转换为 AMP 和 BeyondMimic 训练数据。

## 环境要求

- Python 3.12。
- Isaac Lab 3.0.0 GA，使用 MJWarp（Newton 1.6.1）或 Isaac Sim 6.2 的 PhysX 后端。
- 仓库附带的 RSL-RL 5.5.1 fork。
- Ubuntu 22.04 x64。

请按照 Isaac Lab 3.0 的[官方安装指南](https://docs.nvidia.com/isaaclab/3.0/isaac-lab/installation)准备源码和系统依赖。下面的安装步骤会创建项目使用的 Python 环境。

## 克隆仓库

克隆仓库及子模块：

```bash
git clone --recursive https://github.com/Roboparty/roboparty_train.git
cd roboparty_train
```

已有仓库可执行：

```bash
git submodule update --init --recursive
```

## 安装

Isaac Lab 3.0.0 源码与本项目放在同级目录。在项目根目录执行：

```bash
git clone --branch v3.0.0 --recursive https://github.com/isaac-sim/IsaacLab.git ../IsaacLab-3.0.0
```

已有 v3.0.0 源码时，可在 `../IsaacLab-3.0.0` 创建指向该源码的符号链接。以下安装命令均在项目根目录执行。

MJWarp 和 PhysX 使用独立的 Python 环境。

MJWarp（默认）：

```bash
uv sync --locked
source .venv/bin/activate
```

PhysX（Isaac Sim）：

```bash
UV_PROJECT_ENVIRONMENT=.venv-physx uv sync --locked --no-default-groups \
    --extra physx --no-install-package usd-exchange
source .venv-physx/bin/activate
```

更新 PhysX 环境时保留上述同步选项，激活后直接使用 `python` 或 `isaaclab` 运行命令。需要视频录制或 W&B 日志时，在对应同步命令后添加 `--extra video` 或 `--extra wandb`。

通过列出环境确认扩展安装成功：

```bash
isaaclab list_envs --keyword RP
```

输出应同时包含 `RPO-` 和 `RP1-` 任务。

## 用法

### 训练

训练默认使用 MJWarp（`physics=newton_mjwarp`）。使用 PhysX 时，激活 `.venv-physx` 并添加 `physics=isaacsim_physx`；使用 Kit 可视化也需在此环境中运行。

```bash
python robolab/scripts/rsl_rl/train.py --task=RP1-Flat --viz=none --logger=tensorboard --num_envs=4096
```

### 测试

回放默认使用 MJWarp。以下命令通过 Newton GL 打开窗口，并加载对应任务最新的训练 checkpoint；也可用 `--checkpoint=/path/to/model.pt` 指定模型。无界面运行使用 `--viz=none`。使用 PhysX 时，激活 `.venv-physx` 并添加 `physics=isaacsim_physx`；使用 Isaac Sim 窗口时将 `--viz=newton_gl` 改为 `--viz=kit`。

```bash
python robolab/scripts/rsl_rl/play.py --task=RP1-Flat --num_envs=1 --checkpoint=latest --viz=newton_gl
```

### 测试 AMP

```bash
python robolab/scripts/rsl_rl/play.py --task=RPO-AMP-Play --num_envs=1 --checkpoint=latest --viz=newton_gl
```

### 测试 BeyondMimic

```bash
python robolab/scripts/rsl_rl/play.py --task=RPO-BeyondMimic --num_envs=1 --checkpoint=latest --viz=newton_gl
```

### 测试 Parkour

```bash
python robolab/scripts/rsl_rl/play.py --task=RPO-Parkour-Play --num_envs=1 --checkpoint=latest --viz=newton_gl
```

回放自动将模型导出到对应训练目录的 `exported/`。Parkour 额外生成 `0-depth_encoder.onnx` 和 `actor.onnx`；部署时保留配套的 `.onnx.data` 文件。

### 准备动作数据

仓库在 `robolab/data/motions/` 中预置了以下训练数据：

| 机器人 | AMP（`.pkl`） | BeyondMimic（`.npz`） |
| --- | --- | --- |
| RPO | `rpo_amp` | `rpo_bm` |
| RP1 | `rp1_amp` | `rp1_bm` |

在 [HHTools](https://github.com/Roboparty/human-humanoid-tools) 中使用对应机器人的 URDF 完成重定向，导出机器人 CSV（保留注释和列名）或 PKL，再转换为训练数据：

```bash
# AMP：120 Hz。
python robolab/scripts/tools/hhtools/convert_motion.py --robot rp1 --output_fps 120 \
    -f /path/to/hhtools/walk.csv -o robolab/data/motions/rp1_amp/walk.pkl

# BeyondMimic：50 Hz。
python robolab/scripts/tools/hhtools/convert_motion.py --robot rp1 --output_fps 50 \
    -f /path/to/hhtools/walk.csv -o robolab/data/motions/rp1_bm/walk.npz
```

RPO 使用 `--robot rpo`。省略 `--output_fps` 时，AMP 保留源帧率，BeyondMimic 默认输出 50 Hz。

## 常见问题

- `robolab` 或 `rsl_rl` 是空目录：执行 `git submodule update --init --recursive`。
- 找不到 Isaac Lab 相关 import：请先激活 Isaac Lab 使用的 Python 环境，再安装或运行脚本。
- 找不到 task 名称：执行 `isaaclab list_envs --keyword RP`，复制实际输出的 task id。
- PhysX 后端在部分场景下可能出现训练速度异常缓慢的情况。

## 参考和致谢

本项目基于以下开源项目构建：

- [IsaacLab](https://github.com/isaac-sim/IsaacLab)
- [rsl_rl](https://github.com/leggedrobotics/rsl_rl)
- [legged_gym](https://github.com/leggedrobotics/legged_gym)
- [legged_lab](https://github.com/zitongbai/legged_lab)
- [robot_lab](https://github.com/fan-ziqi/robot_lab)
- [InstinctLab](https://github.com/project-instinct/InstinctLab)
