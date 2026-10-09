# AIRBOT Play PTK 叠衣服 Demo

[English](README.md) | [中文](README.zh-CN.md)

本仓库提供双臂 AIRBOT Play 使用 PI0.5 策略完成纯色 M/L 短袖 T 恤叠衣服演示的复现说明。

## Demo 概览

| 工站 | 适用场景 | 要求 |
| --- | --- | --- |
| 电视机抗干扰工站 | 展示策略面对动态视觉背景时的适应能力 | 台面下方安装电视机，并使用 8 mm 透明玻璃覆盖 |
| 白色桌面工站 | 快速部署和常规展示 | 白色桌面不小于 1.7 m × 1.2 m，并使用 6-8 mm 玻璃覆盖 |

![电视机抗干扰叠衣服工站](assets/workstation-overview.png)

## 硬件要求

- 两台配备夹爪的 AIRBOT Play
- 两个腕部相机：720p、2.3 mm 镜头、75 度视场角
- 一个环境相机：1080p、2.7 mm 镜头、120 度视场角
- 纯色 M/L 短袖 T 恤

相机设备号必须按以下顺序配置：环境相机、左腕相机、右腕相机。

### 机械臂与腕部相机安装

按照复现指南准备机械臂、夹爪、线缆和安装配件：

![机械臂组件与安装配件](assets/arm-components.png)

安装支架和夹爪前，确认法兰朝向：

![法兰朝向参考](assets/flange-orientation.png)

按图安装腕部相机支架和夹爪，再检查相机安装状态及线缆走向：

| 安装示意 | 腕部相机安装完成 |
| --- | --- |
| ![腕部相机安装示意](assets/wrist-camera-installation.png) | ![腕部相机安装完成](assets/wrist-camera-assembled.png) |

### 环境相机安装

参考正面和侧面视图检查支架位置，具体尺寸以原始安装指南为准。

| 正面视图 | 侧面视图 |
| --- | --- |
| <img src="assets/environment-camera-front.png" alt="环境相机支架正面视图" width="360"> | <img src="assets/environment-camera-side.png" alt="环境相机支架侧面视图" width="360"> |

| 支架高度参考 | 相机安装高度参考 |
| --- | --- |
| <img src="assets/environment-camera-height.png" alt="环境相机支架高度参考" width="360"> | <img src="assets/camera-mount-height.png" alt="相机安装高度参考" width="360"> |

以上安装图片提取自下方链接中的原始复现 PDF。

## 软件版本基线

本 Demo 已在 AIRBOT Play **5.1.6** 上验证：

- `airbot-configure`
- `airbot_py`
- `airbot_fsm`
- 机械臂服务使用 Conda Python 3.10，策略服务使用 `uv` Python 3.11
- Ubuntu 22.04 或更高版本，以及 NVIDIA RTX 4090（24 GB）或 RTX 5090（32 GB）

请勿将本流程替换为 V5.2 的 `arm_sdk` 或 `airbot-arm` 工作流。

## 快速开始

1. 下载[工站安装说明（PPTX）](https://github.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/raw/refs/heads/feat/add-demo-content/assets/cloth-folding-workstation-installation.pptx)，使用 PowerPoint、WPS 或 LibreOffice Impress 打开，按照说明完成工站安装。
2. 将 [Openpi_RL](https://github.com/Robot-K/Openpi_RL) 克隆至 `$HOME/tv_fold_demo`。
3. 安装 5.1.6 机械臂软件，并在 `Openpi_RL/examples/airbot/robot_config.py` 配置三个相机设备号。
4. 下载模型权重，并在 `Openpi_RL/examples/airbot/cmds/serve_policy.sh` 中设置 `CHECKPOINT_DIR`。
5. 启动两个机械臂服务、策略服务，最后启动 `infer_async.sh`。

```bash
mkdir -p "$HOME/tv_fold_demo"
cd "$HOME/tv_fold_demo"
git clone https://github.com/Robot-K/Openpi_RL.git

uvx --from 'huggingface_hub>=1.0' hf download \
  xiaoleezuishuai/policy-v3-wospatiodelta-iter4-tv2-280000 \
  --local-dir "$HOME/tv_fold_demo/policy_v3_wospatiodelta_iter4_tv2"
```

原始指南中的相机配置如下。截图中的相机编号仅为示例，不是固定设备号；请识别实际设备，并保持环境、左腕、右腕的顺序。

![robot_config.py 中的相机配置参考](assets/camera-configuration.png)

确认 CAN 设备号后，启动机械臂服务：

```bash
airbot_fsm -i can0 -p 50051
airbot_fsm -i can1 -p 50053
```

策略终端中使用 `UV_PYTHON=3.11 bash serve_policy.sh`。启动推理前将 `INTERPOLATE=true`、`DAGGER=false` 写入配置，然后运行 `bash infer_async.sh`。推理快捷键：`Enter` 开始、`D` 舍弃、`Q` 退出。

![INTERPOLATE 与 DAGGER 推理配置](assets/inference-configuration.png)

## 资源

| 资源 | 位置 |
| --- | --- |
| 实现代码 | [Robot-K/Openpi_RL](https://github.com/Robot-K/Openpi_RL) |
| 预训练策略 | [Hugging Face 模型](https://huggingface.co/xiaoleezuishuai/policy-v3-wospatiodelta-iter4-tv2-280000) |
| 可选训练数据 | [Hugging Face 数据集](https://huggingface.co/datasets/xiaoleezuishuai/airbot-fold-cloth-mcap) |
| 完整复现指南 | [PDF](assets/cloth-folding-reproduction-guide.pdf) |
| 工站安装说明 | [下载 PPTX](https://github.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/raw/refs/heads/feat/add-demo-content/assets/cloth-folding-workstation-installation.pptx) |
| AIRBOT Play 5.1.6 发布说明 | [更新日志](https://docs.discover-robotics.com/document/airbot-play/changelog.html#20250623) |
| PI0.5 复现文档 | [官网文档](https://docs.discover-robotics.com/document/airbot-play/hardware-driver/tutorials/model-reproduction/pi0.5.html) |

原始 CAD 包未包含在本仓库。发布设计文件前请先确认其公开分发权限。
