# AIRBOT Play PTK 叠衣服 Demo

[English](README.md) | [中文](README.zh-CN.md)

本仓库提供双臂 AIRBOT Play 使用 PI0.5 策略完成纯色 M/L 短袖 T 恤叠衣服演示的复现说明。

本文按原始 PDF 的七部分整理，保留安装、下载、配置和运行步骤。路径差异及原文未提供的信息单独注明；本次未做真机运行验证。

## 一、工站方案与硬件安装

### 1.1 两种工站方案

| 工站 | 适用场景 | 要求 |
| --- | --- | --- |
| 电视机抗干扰工站 | 展示策略面对动态视觉背景时的适应能力 | 台面下方安装电视机，并使用 8 mm 透明玻璃覆盖 |
| 白色桌面工站 | 快速部署和常规展示 | 白色桌面不小于 1.7 m × 1.2 m，并使用 6-8 mm 玻璃覆盖 |

### 1.2 方案 A：电视机抗干扰版

电视机方案使用木板、铝型材、角码、连接板与紧固件搭建台架；参考电视为小米 S MiniLED 75 英寸低反屏、1008 分区，也可选择尺寸和显示效果相近的设备。玻璃板尺寸为 **1650 × 1060 × 8 mm**。电视机位于台面下方，衣服在透明玻璃上操作，用于展示动态、彩色背景下的适应能力。[原文电视采购参考](https://detail.tmall.com/item.htm?id=1039475756408)。

### 1.3 方案 B：白色桌面简易版

白色桌面方案不安装电视机，桌面高度不限，玻璃厚度为 6–8 mm；两种方案的机械臂、相机、模型及相对安装关系相同。[原文桌子采购参考](https://item.taobao.com/item.htm?id=731805103039)。

![电视机抗干扰叠衣服工站](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/workstation-overview.png)

### 1.4 方案所需硬件

- 两台配备夹爪的 AIRBOT Play
- 两个腕部相机：720p、2.3 mm 镜头、75 度视场角
- 一个环境相机：1080p、2.7 mm 镜头、120 度视场角
- 纯色 M/L 短袖 T 恤

还需配套数据线、夹爪线和电源；原文要求腕部及环境相机均无畸变。

相机设备号必须按以下顺序配置：环境相机、左腕相机、右腕相机。

### 1.5 机械臂与腕部相机安装

按照复现指南准备机械臂、夹爪、线缆和安装配件：

![机械臂组件与安装配件](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/arm-components.png)

安装支架和夹爪前，确认法兰朝向：

按原文，将标志向下摆放后再安装。

![法兰朝向参考](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/flange-orientation.png)

按图安装腕部相机支架和夹爪，再检查相机安装状态及线缆走向：

| 安装示意 | 腕部相机安装完成 |
| --- | --- |
| ![腕部相机安装示意](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/wrist-camera-installation.png) | ![腕部相机安装完成](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/wrist-camera-assembled.png) |

### 1.6 硬件安装位置参考

参考正面和侧面视图检查支架位置，具体尺寸以原始安装指南为准。

| 正面视图 | 侧面视图 |
| --- | --- |
| <img src="https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/environment-camera-front.png" alt="环境相机支架正面视图" width="360"> | <img src="https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/environment-camera-side.png" alt="环境相机支架侧面视图" width="360"> |

| 支架高度参考 | 相机安装高度参考 |
| --- | --- |
| <img src="https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/environment-camera-height.png" alt="环境相机支架高度参考" width="360"> | <img src="https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/camera-mount-height.png" alt="相机安装高度参考" width="360"> |

以上安装图片提取自下方链接中的原始复现 PDF。

## 软件版本基线

原始资料使用 AIRBOT Play **5.1.6**，此处记录其版本基线：

- `airbot-configure`
- `airbot_py`
- `airbot_fsm`
- 机械臂服务使用 Conda Python 3.10，策略服务使用 `uv` Python 3.11
- Ubuntu 22.04 或更高版本，以及 NVIDIA RTX 4090（24 GB）或 RTX 5090（32 GB）

请勿将本流程替换为 V5.2 的 `arm_sdk` 或 `airbot-arm` 工作流。

## 流程概览

1. 下载[工站安装说明（PPTX）](https://github.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/raw/refs/heads/feat/add-demo-content/assets/cloth-folding-workstation-installation.pptx)，使用 PowerPoint、WPS 或 LibreOffice Impress 打开，按照说明完成工站安装。
2. 将 [Openpi_RL](https://github.com/Robot-K/Openpi_RL) 克隆至 `$HOME/tv_fold_demo`。
3. 安装 5.1.6 机械臂软件，并在 `Openpi_RL/examples/airbot/robot_config.py` 配置三个相机设备号。
4. 下载模型权重，并在 `Openpi_RL/scripts/cmds/serve_policy.sh` 中设置 `CHECKPOINT_DIR`。
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

![robot_config.py 中的相机配置参考](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/camera-configuration.png)

确认 CAN 设备号后，启动机械臂服务：

```bash
airbot_fsm -i can0 -p 50051
airbot_fsm -i can1 -p 50053
```

策略终端中使用 `UV_PYTHON=3.11 bash serve_policy.sh`。启动推理前将 `INTERPOLATE=true`、`DAGGER=false` 写入配置，然后运行 `bash infer_async.sh`。推理快捷键：`Enter` 开始、`D` 舍弃、`Q` 退出。

![INTERPOLATE 与 DAGGER 推理配置](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/inference-configuration.png)

## 二、项目仓库与版本基线

### 2.1 系统要求

Ubuntu ≥22.04；机械臂服务使用 Conda Python 3.10，模型服务使用 `uv` Python 3.11。原文指定 RTX 4090 24 GB 或 RTX 5090 32 GB，未说明其他 GPU 的兼容性。

### 2.2 安装基础工具与 uv

```bash
sudo apt update
sudo apt install -y git git-lfs curl wget ca-certificates build-essential cmake unzip
sudo apt install -y iproute2 udev kmod usbutils libcanberra-gtk-module libcanberra-gtk3-module
git lfs install
curl --proto '=https' --tlsv1.2 -LsSf https://astral.sh/uv/install.sh | sh
export PATH="$HOME/.local/bin:$PATH"
uv --version
git --version
lsusb --version | head -n 1
nvidia-smi
```

上述检查均应正常输出。相机检查还需 `v4l-utils`：`sudo apt install -y v4l-utils`，这是对 PDF 缺少该依赖的补充。

### 2.3 下载仓库

```bash
mkdir -p "$HOME/tv_fold_demo"
cd "$HOME/tv_fold_demo"
git clone https://github.com/Robot-K/Openpi_RL.git
```

PDF 未指定 Git commit。上游当前默认分支为 `master`，可能继续变化；部署时记录 `git -C "$HOME/tv_fold_demo/Openpi_RL" rev-parse HEAD`，以便追溯。

## 三、数据与模型下载

### 3.1 资源地址

- [MCAP 数据集](https://huggingface.co/datasets/xiaoleezuishuai/airbot-fold-cloth-mcap)
- [叠衣服模型](https://huggingface.co/xiaoleezuishuai/policy-v3-wospatiodelta-iter4-tv2-280000)

### 3.2 下载数据集（按需）

使用预训练模型演示时无需先下载训练数据。

```bash
uvx --from 'huggingface_hub>=1.0' hf download \
  xiaoleezuishuai/airbot-fold-cloth-mcap \
  --repo-type dataset \
  --include 'fold_cloth_v3/*' \
  --include 'fold_cloth_v3_dagger/*' \
  --include 'fold_cloth_v4/*' \
  --include 'fold_cloth_v4_dagger/*' \
  --local-dir "$HOME/tv_fold_demo/mcap_data"
ls "$HOME/tv_fold_demo/mcap_data"
```

### 3.3 下载模型权重

```bash
uvx --from 'huggingface_hub>=1.0' hf download \
  xiaoleezuishuai/policy-v3-wospatiodelta-iter4-tv2-280000 \
  --local-dir "$HOME/tv_fold_demo/policy_v3_wospatiodelta_iter4_tv2"
ls "$HOME/tv_fold_demo/policy_v3_wospatiodelta_iter4_tv2"
```

原文提示网络中断时清理未完成缓存后重试。可先重跑下载命令；如需清理，请只处理确认未完成的缓存，不要删除已下载模型。

## 四、环境搭建与运行准备

### 4.1 创建 Conda 环境

已有可用 Conda 时跳过安装部分，直接创建 `airdc` 环境；不要覆盖现有 Miniconda 目录。

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O "$HOME/miniconda.sh"
bash "$HOME/miniconda.sh" -b -p "$HOME/miniconda3"
source "$HOME/miniconda3/etc/profile.d/conda.sh"
conda init bash
```

在已加载 Conda 的终端执行：

```bash
conda create -n airdc python=3.10 -y
conda activate airdc
python --version
```

### 4.2 安装 AIRBOT SDK、AIRDC 与推理客户端依赖

#### 4.2.1 机械臂软件环境

原文 Docker 安装命令如下：

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
docker ps
```

原文要求用户组配置后重启 Ubuntu，但没有提供用户组命令。如果 `docker ps` 无权限，请按 Docker 官方安装说明配置权限并重新登录，或使用 `sudo docker ps` 检查。

从 [AIRBOT Play 5.1.6 历史版本](https://docs.discover-robotics.com/document/airbot-play/changelog.html#v5.1.6) 取得匹配的驱动和 SDK。在下载文件所在目录执行：

```bash
sudo apt-get install ./airbot-configure_5.1.6-1_all.deb
dpkg -l | grep airbot
conda activate airdc
python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple ./airbot_py-5.1.6-py3-none-any.whl
python -m pip list | grep airbot
```

#### 4.2.2 安装 data-collection

PDF 引用 `data-collection.zip`，但 PDF 中的嵌入附件不是可直接下载的地址。本仓库未包含该压缩包；请取得与该 Demo 配套的原始附件，解压至 `$HOME/tv_fold_demo/data-collection` 后执行：

```bash
cd "$HOME/tv_fold_demo/data-collection"
conda activate airdc
bash install/install.sh
```

原 PDF 没有单独列出完整的 Openpi_RL/推理客户端依赖安装步骤；还需核对配套代码的安装说明，不能将上述安装脚本等同于所有依赖已安装完成。

#### 4.2.3 确认机械臂设备号

分别连接左右臂，记录各自的 CAN 接口：

```bash
ip link | grep can
```

### 4.3 相机识别与画面检查

连接环境、左腕、右腕相机。原文写 `v4l2-ctl --list-cameras`；标准 `v4l2-ctl` 使用 `--list-devices`，此处修正为：

```bash
v4l2-ctl --list-devices
conda activate airdc
cd "$HOME/tv_fold_demo/data-collection"
python3 scripts/multi_capture.py 4 6 8 -ff MJPEG MJPEG MJPEG
```

`4 6 8` 仅为示例，必须替换为实际相机编号。检查三路画面后，编辑 `$HOME/tv_fold_demo/Openpi_RL/examples/airbot/robot_config.py`：

```python
camera_index: list[int | str] = [4, 6, 8]
```

顺序必须是环境、左腕、右腕。原文截图中的编号与文字示例不同，均不能替代实际设备识别。

## 五、配置模型服务与推理脚本

原 PDF 同时出现 `scripts/cmds` 和 `examples/airbot/cmds` 两种路径。已核对当前上游仓库，脚本位于 **`scripts/cmds/`**。编辑 `$HOME/tv_fold_demo/Openpi_RL/scripts/cmds/serve_policy.sh` 的 CONFIG 区：

```bash
CHECKPOINT_DIR="${HOME}/tv_fold_demo/policy_v3_wospatiodelta_iter4_tv2"
PORT=8000
```

保留与模型匹配的策略配置。原 PDF 未明确 `POLICY_CONFIG` 的值，不能只凭权重目录名称推断兼容性。

## 六、启动 Demo

先清空机械臂工作区并确保急停可用。服务启动及初始化可能使机械臂运动。以下命令分别在独立终端执行，CAN 接口必须使用实际值。

### 6.1 启动机械臂（终端 1、2）

左臂：

```bash
conda activate airdc
airbot_fsm -i can0 -p 50051
```

右臂：

```bash
conda activate airdc
airbot_fsm -i can1 -p 50053
```

### 6.2 启动模型服务（终端 3）

```bash
cd "$HOME/tv_fold_demo/Openpi_RL/scripts/cmds"
export UV_PYTHON=3.11
bash serve_policy.sh
```

### 6.3 启动推理（终端 4）

先编辑同目录的 `infer_async.sh`：

```bash
INTERPOLATE=true
DAGGER=false
```

`INTERPOLATE` 启用动作插值平滑；无示教臂时 `DAGGER=false` 避免连接 leader 端口。确认机械臂和模型服务启动且没有错误后运行：

```bash
conda activate airdc
cd "$HOME/tv_fold_demo/Openpi_RL/scripts/cmds"
bash infer_async.sh
```

快捷键：`Enter` 开始、`D` 舍弃、`Q` 退出。这是预训练策略控制的叠衣服演示；正常推理不要求实时遥操作。

## 七、附件与资源

原 PDF 列出 `叠衣支架模型.zip`、`叠衣台架安装说明.pptx`、`叠衣支架安装说明.pptx` 和 `叠衣支架0904.zip`。按提供者确认，两份 PPTX 为同一份安装说明，本仓库仅保留一份。CAD 压缩包未公开上传；PDF 内嵌附件名称不代表仓库已包含对应文件。

| 资源 | 位置 |
| --- | --- |
| 实现代码 | [Robot-K/Openpi_RL](https://github.com/Robot-K/Openpi_RL) |
| 预训练策略 | [Hugging Face 模型](https://huggingface.co/xiaoleezuishuai/policy-v3-wospatiodelta-iter4-tv2-280000) |
| 可选训练数据 | [Hugging Face 数据集](https://huggingface.co/datasets/xiaoleezuishuai/airbot-fold-cloth-mcap) |
| 完整复现指南 | [PDF](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/cloth-folding-reproduction-guide.pdf) |
| 工站安装说明 | [下载 PPTX](https://github.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/raw/refs/heads/feat/add-demo-content/assets/cloth-folding-workstation-installation.pptx) |
| AIRBOT Play 5.1.6 发布说明 | [更新日志](https://docs.discover-robotics.com/document/airbot-play/changelog.html#v5.1.6) |
| PI0.5 复现文档 | [官网文档](https://docs.discover-robotics.com/document/airbot-play/hardware-driver/tutorials/model-reproduction/pi0.5.html) |

原始 CAD 包未包含在本仓库。发布设计文件前请先确认其公开分发权限。
