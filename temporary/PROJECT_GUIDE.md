# DISCOVERSE 项目介绍与使用指南

本文面向首次进入仓库的开发者，说明 DISCOVERSE 的用途、目录结构、安装方式和常用运行入口。项目根目录的 [`README_zh.md`](../README_zh.md) 提供官方介绍；本文更侧重本仓库内的开发和操作路径。

## 1. 项目概览

DISCOVERSE 是一个面向机器人学习的开源仿真框架。它以 MuJoCo 提供物理仿真，并支持 3D Gaussian Splatting（3DGS）高保真渲染，用于构建 Real2Sim2Real 工作流。项目覆盖机械臂、移动机器人、无人机、灵巧手、多机器人协作、传感器仿真、数据生成和策略学习。

核心能力包括：

- MuJoCo 机器人与复杂场景仿真
- 3DGS 高保真视觉渲染
- LiDAR、相机及 ROS/ROS 2 接口
- Airbot Play、MMK2、SkyRover、RM2 等机器人示例
- ACT、Diffusion Policy、RDT 等策略训练与推理入口
- 机器人操作任务的数据采集和自动生成
- MJCF、URDF、Mesh 等模型与场景资源

项目采用 MIT 许可证。Python 要求为 3.8 或更高版本；日常开发推荐 Python 3.10。

## 2. 目录结构

```text
DISCOVERSE/
├── discoverse/              # 核心 Python 包
│   ├── configs/             # 仿真配置
│   ├── envs/                # 通用环境基础设施
│   ├── robots/              # 机器人实现
│   ├── robots_env/          # 机器人环境入口
│   ├── task_base/           # 任务基类
│   ├── universal_manipulation/
│   └── utils/               # 公共工具
├── examples/                # 可直接运行的示例和任务
├── models/
│   ├── meshes/              # 几何网格
│   ├── mjcf/                # MuJoCo XML/MJCF 模型与场景
│   └── urdf/                # URDF 机器人描述
├── policies/                # 策略训练与推理代码
├── scripts/                 # 安装检查、数据生成和模型转换工具
├── submodules/              # 可选功能使用的 Git 子模块
├── data/                    # 任务数据或本地数据集
├── assets/                  # README 图片等静态资源
└── temporary/               # Codex 脚本、测试输出和临时文档
```

`examples/` 按用途划分：

- `robots/`：单机器人和多机器人控制示例
- `tasks_airbot_play/`、`tasks_mmk2/`、`tasks_hand_arm/`：操作任务
- `active_slam/`、`sensor_lidar/`：SLAM 和 LiDAR 示例
- `mocap_ik/`：运动捕捉和逆运动学
- `force_control/`：阻抗控制和力控制
- `ros1/`、`ros2/`：ROS 接口示例
- `universal_tasks/`：通用任务运行入口

## 3. 在 WSL Ubuntu 中进入项目

本工作副本位于 WSL2 的 Ubuntu 文件系统中：

```text
/home/ficooo/DISCOVERSE
```

从 Windows PowerShell 进入 Ubuntu 后再运行 Linux 命令：

```powershell
wsl -d Ubuntu
```

```bash
cd /home/ficooo/DISCOVERSE
```

Windows 也可以通过以下 UNC 路径访问文件：

```text
\\wsl$\Ubuntu\home\ficooo\DISCOVERSE
```

Python、MuJoCo、CUDA 和项目脚本应在同一个 Ubuntu 环境中运行，避免混用 Windows 与 Linux 的 Python 解释器。

## 4. 安装

### 4.1 准备代码和 Git LFS

全新克隆前先安装 Git LFS：

```bash
sudo apt update
sudo apt install -y git-lfs
git lfs install
git clone https://github.com/TATP-233/DISCOVERSE.git
cd DISCOVERSE
```

当前工作副本已经存在时，不需要重新克隆。

### 4.2 创建 Python 环境

使用 Conda：

```bash
conda create -n discoverse python=3.10
conda activate discoverse
python -m pip install --upgrade pip
```

也可以使用项目内虚拟环境：

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

### 4.3 安装核心功能

```bash
pip install -e .
python scripts/check_installation.py
```

`-e` 表示可编辑安装。修改 `discoverse/` 下的源码后，无需重复安装。

### 4.4 按需求安装可选功能

| 需求 | 安装命令 |
| --- | --- |
| LiDAR 仿真 | `pip install -e ".[lidar]"` |
| 3DGS 渲染 | `pip install -e ".[gs]"` |
| XML 场景编辑器 | `pip install -e ".[xml-editor]"` |
| ACT | `pip install -e ".[act_full]"` |
| Diffusion Policy | `pip install -e ".[dp_full]"` |
| RDT | `pip install -e ".[rdt_full]"` |
| ROS 与 RealSense | `pip install -e ".[hardware]"` |
| 开发工具 | `pip install -e ".[dev]"` |
| 全部运行功能 | `pip install -e ".[full]"` |

`full` 会安装较多机器学习和硬件依赖。若只需运行基础 MuJoCo 示例，优先使用核心安装。

### 4.5 初始化子模块

部分可选功能依赖 Git 子模块。先查看状态，再按需初始化：

```bash
python scripts/setup_submodules.py --list
python scripts/setup_submodules.py --module lidar
python scripts/setup_submodules.py --module act
```

自动检测已安装功能：

```bash
python scripts/setup_submodules.py
```

初始化所有子模块：

```bash
python scripts/setup_submodules.py --all
```

## 5. 验证环境

运行项目提供的检查脚本：

```bash
python scripts/check_installation.py --verbose
python scripts/check_mujoco_install.py
```

还可以进行最小导入检查：

```bash
python -c "import discoverse, mujoco; print(discoverse.__version__)"
```

若导入的是错误环境中的包，先确认解释器路径：

```bash
which python
python -m pip show discoverse
```

## 6. 常用运行示例

所有命令均从项目根目录执行。

### 6.1 启动基础机器人环境

```bash
python discoverse/robots_env/airbot_play_base.py
python discoverse/robots_env/mmk2_base.py
```

### 6.2 运行操作任务

```bash
python examples/tasks_airbot_play/place_coffeecup.py
python examples/tasks_airbot_play/stack_block.py
python examples/tasks_mmk2/kiwi_pick.py
python examples/tasks_mmk2/cabinet_door_open.py
```

### 6.3 运行机器人示例

```bash
python examples/robots/leap_hand_env.py
python examples/robots/rm2_car.py
python examples/robots/skyrover.py
python examples/robots/cooperative_control.py
```

### 6.4 逆运动学与力控制

```bash
python examples/mocap_ik/mocap_ik_manipulator.py --robot airbot_play
python examples/mocap_ik/mocap_ik_mmk2.py
python examples/force_control/impedance_control.py
```

### 6.5 SLAM、LiDAR 与 ROS

```bash
python examples/active_slam/camera_view.py
python examples/sensor_lidar/mmk2_lidar_ros2.py
python examples/ros2/mmk2_ros2.py
```

ROS 示例需要先启动对应版本的 ROS 环境，并安装项目的 `ros` 可选依赖。

## 7. 数据生成与策略

任务数据生成入口位于 `scripts/tasks_data_gen.py`，各具体任务通常也能直接运行并生成轨迹。开始批量任务前，先查看脚本参数：

```bash
python scripts/tasks_data_gen.py --help
```

策略的统一入口为：

```bash
python policies/train.py --help
python policies/infer.py --help
```

ACT、Diffusion Policy、RDT 和其他策略的实现分别位于 `policies/` 的对应子目录。训练前应确认数据格式、配置文件、GPU/CUDA 版本和子模块状态。

## 8. 模型与高保真渲染

MuJoCo 场景主要存放在 `models/mjcf/`，URDF 文件存放在 `models/urdf/`，网格资源存放在 `models/meshes/`。需要 3DGS 渲染时，安装 `gs` 依赖；大型 3DGS 模型默认不进入 Git 仓库，并在首次需要时从模型仓库下载。

国内网络环境可以设置 Hugging Face 镜像：

```bash
export HF_ENDPOINT=https://hf-mirror.com
```

检查 CUDA 和 GPU 是否可用：

```bash
nvidia-smi
python -c "import torch; print(torch.cuda.is_available())"
```

## 9. WSL 图形界面注意事项

MuJoCo 和部分示例会创建图形窗口。Windows 11 通常通过 WSLg 直接显示；可以检查：

```bash
echo "$DISPLAY"
echo "$WAYLAND_DISPLAY"
```

无图形界面的服务器可使用 EGL 或其他离屏渲染方式。常见设置为：

```bash
export MUJOCO_GL=egl
```

具体渲染后端取决于显卡驱动、CUDA、容器和远程桌面环境。出现 OpenGL/EGL 错误时，应先运行 `scripts/check_mujoco_install.py`，再检查驱动和环境变量。

## 10. 开发建议

1. 从项目根目录运行脚本，避免相对路径失效。
2. 先安装最小依赖，再为目标功能补充可选依赖。
3. 修改核心模块后运行安装检查和相关示例。
4. 不要提交 `data/`、3DGS 大模型、虚拟环境、日志和临时输出。
5. 一次提交只包含同一目的的改动；提交前检查 `git diff` 和 `git status`。
6. 临时脚本和验证输出放入 `temporary/`，成熟后再移动到正式目录。

## 11. 常见问题

### `ModuleNotFoundError: discoverse`

确认已激活正确环境，并从仓库根目录执行：

```bash
pip install -e .
python -m pip show discoverse
```

### 子模块目录为空

```bash
python scripts/setup_submodules.py --list
python scripts/setup_submodules.py --module <功能名>
```

### 模型或数据缺失

先确认 Git LFS 已启用：

```bash
git lfs install
git lfs pull
```

3DGS 模型和部分数据不由普通 Git 文件管理，需要按根目录 README 的说明单独下载。

### WSL 中无法显示窗口

确认正在 WSL Ubuntu 内运行 Linux Python，并检查 `DISPLAY`、WSLg 和 GPU 驱动。只需验证逻辑或渲染输出时，可以尝试 `MUJOCO_GL=egl` 离屏模式。

## 12. 进一步阅读

- [`README_zh.md`](../README_zh.md)：官方中文说明
- [`README.md`](../README.md)：官方英文说明
- [`pyproject.toml`](../pyproject.toml)：依赖组和打包配置
- [`scripts/setup_submodules.py`](../scripts/setup_submodules.py)：子模块映射和初始化方式
- [`scripts/check_installation.py`](../scripts/check_installation.py)：环境检查范围
- [项目主页](https://air-discoverse.github.io/)
- [论文](https://arxiv.org/abs/2507.21981)
