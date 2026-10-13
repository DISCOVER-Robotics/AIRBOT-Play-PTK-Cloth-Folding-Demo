# AIRBOT Play PTK Cloth Folding Demo

[English](README.md) | [中文](README.zh-CN.md)

This repository documents the reproduction of a dual-arm AIRBOT Play demonstration that folds a plain M/L short-sleeve T-shirt with a PI0.5 policy.

This README follows the seven parts of the original PDF, including installation, downloads, configuration, and execution. Source omissions and path corrections are identified explicitly. No hardware run was performed during this documentation update.

## 1. Workstation options and hardware installation

### 1.1 Workstation options

Two workstation options use the same robot, camera, and policy configuration:

| Workstation | Use case | Requirements |
| --- | --- | --- |
| TV-backed workstation | Demonstrating robustness to dynamic visual backgrounds | A TV below the work surface and an 8 mm transparent glass cover |
| White-table workstation | Fast deployment and standard demonstrations | A white table at least 1.7 m by 1.2 m, with a 6-8 mm glass cover |

### 1.2 TV-backed workstation

The TV workstation uses wood panels, aluminum profiles, brackets, connecting plates, and fasteners. The reference display is a Xiaomi S MiniLED 75-inch low-reflection TV with 1008 dimming zones; a comparable display may be used. The glass surface measures **1650 × 1060 × 8 mm**. The TV sits beneath the glass and plays dynamic or colored content while the robot folds on the glass. [Original TV purchasing reference](https://detail.tmall.com/item.htm?id=1039475756408).

### 1.3 White-table workstation

The white-table option has no TV and no specified table height; its glass is 6–8 mm thick. Both options use the same arms, cameras, model, and relative mounting arrangement. [Original table purchasing reference](https://item.taobao.com/item.htm?id=731805103039).

![TV-backed cloth-folding workstation](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/workstation-overview.png)

### 1.4 Required hardware

- Two AIRBOT Play arms with grippers
- Two 720p wrist cameras, 2.3 mm lens, 75-degree field of view
- One 1080p environment camera, 2.7 mm lens, 120-degree field of view
- Solid-color M/L short-sleeve T-shirt

Also prepare the matching data cables, gripper cables, and power supplies. The source specifies distortion-free wrist and environment cameras.

Connect the cameras in this exact order: environment camera, left wrist camera, right wrist camera.

### 1.5 Arm and wrist camera installation

Prepare the arm, gripper, cables, and mounting hardware shown in the reproduction guide:

![Arm components and mounting hardware](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/arm-components.png)

Check the flange orientation before installing the bracket and gripper:

Position the marking downward as specified in the original guide.

![Flange orientation reference](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/flange-orientation.png)

Install the wrist camera bracket and gripper as illustrated, then check the assembled camera and cable routing:

| Installation | Assembled wrist camera |
| --- | --- |
| ![Wrist camera installation](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/wrist-camera-installation.png) | ![Assembled wrist camera](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/wrist-camera-assembled.png) |

### 1.6 Hardware placement references

Use the front and side views to check the stand placement. Follow the original installation guide for dimensions.

| Front view | Side view |
| --- | --- |
| <img src="https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/environment-camera-front.png" alt="Environment camera stand front view" width="360"> | <img src="https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/environment-camera-side.png" alt="Environment camera stand side view" width="360"> |

| Stand height reference | Camera mounting height reference |
| --- | --- |
| <img src="https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/environment-camera-height.png" alt="Environment camera stand height reference" width="360"> | <img src="https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/camera-mount-height.png" alt="Camera mounting height reference" width="360"> |

These installation images are extracted from the original reproduction PDF linked below.

## Software baseline

The supplied source documents AIRBOT Play **5.1.6** as its software baseline:

- `airbot-configure`
- `airbot_py`
- `airbot_fsm`
- Conda Python 3.10 for the robot service and `uv` Python 3.11 for policy serving
- Ubuntu 22.04 or later and an NVIDIA RTX 4090 (24 GB) or RTX 5090 (32 GB)

Do not replace these components with the V5.2 `arm_sdk` or `airbot-arm` workflow.

## Workflow overview

1. Download [the workstation installation guide (PPTX)](https://github.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/raw/refs/heads/feat/add-demo-content/assets/cloth-folding-workstation-installation.pptx) and open it in PowerPoint, WPS Office, or LibreOffice Impress to install the workstation.
2. Clone [Openpi_RL](https://github.com/Robot-K/Openpi_RL) into `$HOME/tv_fold_demo`.
3. Install the 5.1.6 robot software and configure the three camera device IDs in `Openpi_RL/examples/airbot/robot_config.py`.
4. Download the policy and set `CHECKPOINT_DIR` in `Openpi_RL/scripts/cmds/serve_policy.sh`.
5. Start the two arm services, the policy server, and then `infer_async.sh`.

```bash
mkdir -p "$HOME/tv_fold_demo"
cd "$HOME/tv_fold_demo"
git clone https://github.com/Robot-K/Openpi_RL.git

uvx --from 'huggingface_hub>=1.0' hf download \
  xiaoleezuishuai/policy-v3-wospatiodelta-iter4-tv2-280000 \
  --local-dir "$HOME/tv_fold_demo/policy_v3_wospatiodelta_iter4_tv2"
```

The original guide shows the camera configuration below. Camera indices in the screenshot are examples, not fixed assignments; detect your actual devices and keep the order environment, left wrist, right wrist.

![Camera configuration reference in robot_config.py](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/camera-configuration.png)

Start the arm services after verifying the CAN device assignments:

```bash
airbot_fsm -i can0 -p 50051
airbot_fsm -i can1 -p 50053
```

In the policy terminal, use `UV_PYTHON=3.11 bash serve_policy.sh`. Before inference, set `INTERPOLATE=true` and `DAGGER=false`, then run `bash infer_async.sh`. The inference controls are `Enter` to start, `D` to discard, and `Q` to exit.

![Inference configuration with INTERPOLATE and DAGGER settings](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/inference-configuration.png)

## 2. Project repository and baseline

### 2.1 System requirements

Ubuntu ≥22.04; Conda Python 3.10 for arm services and `uv` Python 3.11 for the policy server. The PDF specifies RTX 4090 24 GB or RTX 5090 32 GB; compatibility with other GPUs is not established by that document.

### 2.2 Install basic tools and uv

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

All checks should return normally. Camera inspection also requires `v4l-utils`: `sudo apt install -y v4l-utils`. This dependency is an addition to the PDF.

### 2.3 Download the repository

```bash
mkdir -p "$HOME/tv_fold_demo"
cd "$HOME/tv_fold_demo"
git clone https://github.com/Robot-K/Openpi_RL.git
```

The PDF does not pin a Git commit. The current upstream default branch is `master` and may change. Record `git -C "$HOME/tv_fold_demo/Openpi_RL" rev-parse HEAD` for reproducibility.

## 3. Data and model downloads

### 3.1 Resources

- [MCAP dataset](https://huggingface.co/datasets/xiaoleezuishuai/airbot-fold-cloth-mcap)
- [Cloth-folding policy](https://huggingface.co/xiaoleezuishuai/policy-v3-wospatiodelta-iter4-tv2-280000)

### 3.2 Download the dataset (optional)

Training data is not required to demonstrate the supplied pretrained policy.

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

### 3.3 Download the model weights

```bash
uvx --from 'huggingface_hub>=1.0' hf download \
  xiaoleezuishuai/policy-v3-wospatiodelta-iter4-tv2-280000 \
  --local-dir "$HOME/tv_fold_demo/policy_v3_wospatiodelta_iter4_tv2"
ls "$HOME/tv_fold_demo/policy_v3_wospatiodelta_iter4_tv2"
```

The PDF recommends clearing incomplete cache files after interrupted downloads. First retry the download; if cleanup is necessary, target only confirmed incomplete cache files and preserve downloaded weights.

## 4. Environment setup and preparation

### 4.1 Create the Conda environment

If Conda already works, skip installation and create `airdc` directly. Do not overwrite an existing Miniconda directory.

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O "$HOME/miniconda.sh"
bash "$HOME/miniconda.sh" -b -p "$HOME/miniconda3"
source "$HOME/miniconda3/etc/profile.d/conda.sh"
conda init bash
```

In a terminal with Conda initialized:

```bash
conda create -n airdc python=3.10 -y
conda activate airdc
python --version
```

### 4.2 Install AIRBOT SDK, AIRDC, and inference client dependencies

#### 4.2.1 Arm software environment

Docker installation commands from the PDF:

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

The PDF says to reboot after configuring user groups but omits the group command. If `docker ps` reports a permission error, follow Docker's official permission setup and log in again, or check using `sudo docker ps`.

Obtain matching drivers and SDK from the [AIRBOT Play 5.1.6 historical release](https://docs.discover-robotics.com/document/airbot-play/changelog.html#v5.1.6). Run from the directory containing the downloaded packages:

```bash
sudo apt-get install ./airbot-configure_5.1.6-1_all.deb
dpkg -l | grep airbot
conda activate airdc
python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple ./airbot_py-5.1.6-py3-none-any.whl
python -m pip list | grep airbot
```

#### 4.2.2 Install data-collection

The PDF references `data-collection.zip`, but its embedded attachment label is not a downloadable URL. This repository does not include the archive. Obtain the original demo companion attachment and extract it to `$HOME/tv_fold_demo/data-collection`, then run:

```bash
cd "$HOME/tv_fold_demo/data-collection"
conda activate airdc
bash install/install.sh
```

The PDF does not separately specify complete Openpi_RL/inference-client dependency installation. Consult the matching code's installation instructions; the command above does not establish that every dependency is installed.

#### 4.2.3 Identify the arm interfaces

Connect the left and right arms separately and record their CAN interfaces:

```bash
ip link | grep can
```

### 4.3 Identify cameras and inspect streams

Connect environment, left wrist, and right wrist cameras. The PDF uses `v4l2-ctl --list-cameras`; standard `v4l2-ctl` uses `--list-devices`, corrected here:

```bash
v4l2-ctl --list-devices
conda activate airdc
cd "$HOME/tv_fold_demo/data-collection"
python3 scripts/multi_capture.py 4 6 8 -ff MJPEG MJPEG MJPEG
```

Replace `4 6 8` with actual camera indices. After checking all streams, edit `$HOME/tv_fold_demo/Openpi_RL/examples/airbot/robot_config.py`:

```python
camera_index: list[int | str] = [4, 6, 8]
```

The order must be environment, left wrist, right wrist. The PDF screenshot and prose use different example indices; neither substitutes for detecting your devices.

## 5. Configure policy serving and inference

The PDF alternates between `scripts/cmds` and `examples/airbot/cmds`. The current upstream repository has the scripts in **`scripts/cmds/`**. Edit the CONFIG section of `$HOME/tv_fold_demo/Openpi_RL/scripts/cmds/serve_policy.sh`:

```bash
CHECKPOINT_DIR="${HOME}/tv_fold_demo/policy_v3_wospatiodelta_iter4_tv2"
PORT=8000
```

Keep a policy configuration matching the model. The PDF does not explicitly specify `POLICY_CONFIG`; compatibility cannot be inferred solely from the weights directory name.

## 6. Start the demo

Clear the arm workspace and keep the emergency stop accessible. Service startup and initialization may move the arms. Use separate terminals for the following processes and replace example CAN interfaces with actual assignments.

### 6.1 Start arm services (terminals 1 and 2)

Left arm:

```bash
conda activate airdc
airbot_fsm -i can0 -p 50051
```

Right arm:

```bash
conda activate airdc
airbot_fsm -i can1 -p 50053
```

### 6.2 Start the policy server (terminal 3)

```bash
cd "$HOME/tv_fold_demo/Openpi_RL/scripts/cmds"
export UV_PYTHON=3.11
bash serve_policy.sh
```

### 6.3 Start inference (terminal 4)

First edit `infer_async.sh` in the same directory:

```bash
INTERPOLATE=true
DAGGER=false
```

`INTERPOLATE` smooths actions; `DAGGER=false` avoids leader-arm connections when no teaching arms are used. Once both arm services and the policy server are running without errors:

```bash
conda activate airdc
cd "$HOME/tv_fold_demo/Openpi_RL/scripts/cmds"
bash infer_async.sh
```

Controls: `Enter` to start, `D` to discard, `Q` to exit. The pretrained policy controls the folding demonstration; normal inference does not require live teleoperation.

## 7. Attachments and resources

The PDF lists `叠衣支架模型.zip`, `叠衣台架安装说明.pptx`, `叠衣支架安装说明.pptx`, and `叠衣支架0904.zip`. The provider confirmed that the two PPTX references are the same guide; this repository keeps one copy. CAD archives are not published. An embedded attachment label in the PDF does not mean that the corresponding file is available in this repository.

| Resource | Location |
| --- | --- |
| Implementation | [Robot-K/Openpi_RL](https://github.com/Robot-K/Openpi_RL) |
| Pre-trained policy | [Hugging Face model](https://huggingface.co/xiaoleezuishuai/policy-v3-wospatiodelta-iter4-tv2-280000) |
| Optional training data | [Hugging Face dataset](https://huggingface.co/datasets/xiaoleezuishuai/airbot-fold-cloth-mcap) |
| Full reproduction guide | [PDF](https://raw.githubusercontent.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/feat/add-demo-content/assets/cloth-folding-reproduction-guide.pdf) |
| Workstation installation guide | [Download PPTX](https://github.com/DISCOVER-Robotics/AIRBOT-Play-PTK-Cloth-Folding-Demo/raw/refs/heads/feat/add-demo-content/assets/cloth-folding-workstation-installation.pptx) |
| AIRBOT Play 5.1.6 release | [Changelog](https://docs.discover-robotics.com/document/airbot-play/changelog.html#v5.1.6) |
| PI0.5 reproduction | [Documentation](https://docs.discover-robotics.com/document/airbot-play/hardware-driver/tutorials/model-reproduction/pi0.5.html) |

The original CAD package is intentionally not included in this repository. Obtain its distribution approval before publishing design files.
