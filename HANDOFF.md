# LearnOpt Motion Planning 环境配置交接

更新时间：2026-11-02（Asia/Shanghai）。本文按用户要求放在工程根目录，随实际进展更新。

**当前状态：README 基础环境已安装，全部本轮验证通过，quickstart 可以运行。可选 MPC/acados、已有策略部署及 YOPO 数据采集不属于本轮已完成范围。**

## 当前任务与约束

- 为本工程配置运行环境，保证现有系统软件、ROS2 Jazzy 和其他项目的环境不被改动。
- 使用项目独立的 Python 3.13 虚拟环境 `.venv`；依赖与生成文件留在工程本地。
- 不执行系统级 pip 安装，不升级系统 Python，不修改 ROS、显卡驱动、全局 shell 配置或其他项目环境。
- 不修改现有飞行配置、规划器或控制器来掩盖环境问题。
- 首先验证 README 的 quickstart。用户指出权限弹窗过于频繁后，收敛为基础安装和必要验证，暂不扩展安装可选 acados 等组件。

## 已完成

- 阅读 `README.md`、`AGENTS.md`；初始 `git status --short` 为空。
- 确认工程是独立的 Python + MuJoCo 项目，既非 ROS1 也非 ROS2 包。
- 确认项目要求 Python `>=3.13,<3.14`，标准安装命令为 `uv sync --python 3.13`。
- 确认 quickstart 使用 minimum-snap + cascaded，无需 acados 或 RL checkpoint。
- 创建本交接文件。
- 使用已有 `uv 0.11.28`，按原始 `uv.lock` 完成 `uv sync --locked --python 3.13 --managed-python`，安装 77 个包；未修改锁文件和项目依赖声明。
- 下载的 Python 3.13.14 位于 `cache/python/`，独立虚拟环境位于 `.venv/`；uv 缓存位于 `cache/uv/`，使用硬链接安装以节省重复磁盘占用。
- 编译并加载三个可选 Cython 内核：`_collision_broadphase`、`_trajectory_math`、`_aerial_constraints`；输出仅在 `c_generated_code/cython/`。
- 核心依赖已导入验证：MuJoCo 3.12.0、NumPy 2.5.2、SciPy 1.18.1、CasADi 3.8.0、CVXPY 1.9.2、OMPL 2.0.1、Stable-Baselines3 2.8.0。
- PyTorch 2.6.0+cu124 已识别 RTX 3070 Laptop GPU，CUDA 张量矩阵乘法及反向传播通过。CUDA 12.4 运行库随虚拟环境安装，未安装或更改系统 CUDA 工具链与驱动。
- `uv pip check` 通过，77 个已安装包兼容；`uav_ac.main --help` 和 native `status` 通过。
- 运行已有 `tests/unit/test_main.py` 与 `tests/integration/test_mujoco_trajectory_tracking.py`：**22 passed in 5.17s**，未声称运行全量测试。
- 使用原始 quickstart 配置，通过项目 API 完成真实 MuJoCo 物理仿真：1537 个轨迹样本、15370 个物理步、约 15.37 秒仿真时间；终点误差 0.01369 m，平均跟踪误差 0.02422 m，无碰撞。
- 在现有 X11 桌面 `DISPLAY=:1` 实际渲染场景，并打开、同步、自动关闭原生 MuJoCo 查看器。窗口验证与上述完整无窗口轨迹跟踪分开执行。
- 新增本地辅助脚本 `project-env.sh`：对子进程清除 ROS Python/动态库和 Qt 插件路径，禁用用户 site-packages，将缓存留在工程内；不会修改父终端或 shell 启动文件。该脚本是本次新增辅助工具，项目原生入口仍为 `uav_ac.main`。

## 卡住的问题 / 尚未配置的可选功能

- **基础 quickstart 无阻塞问题。**
- `configs/flight/flight.yaml` 虽通过配置校验，但使用 MPC，需要另装 acados 和 Python 接口；本轮未完成该可选安装，也未验证 MPC 求解器生成。仅查询到官方稳定版 v0.6.0；不要据此认为 acados 已安装。
- `flight_gate_racing.yaml` 缺少 `.zip` 模型；`yopo.yaml` 缺少 whole-body checkpoint；`yopo_uav.yaml` 缺少有效的 UAV checkpoint 元数据。这些模型/数据不随仓库提供，安装 Python 依赖不会自动生成。
- YOPO 官方数据采集器另需 CUDA toolkit、Eigen3/PCL/OpenCV/yaml-cpp 开发依赖；本轮未安装这些可选开发组件。PyTorch 的 CUDA 运行库不能替代 `nvcc` 工具链。
- 工具沙箱出现 `bwrap: loopback: Failed RTM_NEWADDR: Operation not permitted`，部分命令因此需要外部执行审批；不是项目的安装要求。前期拆分检查过细造成重复弹窗，后续应合并必要操作，避免无关扩展。
- 安装后系统所在分区剩余约 **6.1 GB**，使用率 97%。本轮未删除其他项目或用户文件；进行后续大型训练/数据采集前需先规划存储。

## 下一步

1. 使用下方启动命令进行交互体验；基础环境无需重新安装。
2. 如后续明确需要默认 MPC，再把 acados 源码、构建与安装前缀放在工程 `cache/acados/`，接口仅安装进 `.venv`；完成真实求解器生成/求解验证，不改系统库搜索配置。
3. 如需要 RL/YOPO 部署，准备匹配的 checkpoint 和相邻元数据，再验证相应原始配置；不要把随机测试权重当作可用策略交付。
4. 如需要 YOPO 官方 CUDA 数据采集，单独规划工具链和磁盘空间。

## 已知注意事项 / 踩过的坑

- 不能把 Jazzy 的 Python 环境当成本工程虚拟环境；应显式使用 `.venv/bin/python`。
- 默认 `configs/flight/flight.yaml` 使用 MPC；首次验证应使用 `configs/flight/flight_quickstart.yaml`。
- `--help`、配置校验或无窗口计算通过，均不代表 MuJoCo 窗口已成功打开。
- 工程没有 `--headless` CLI 参数；需要使用现有测试/API 做无窗口检查。
- 不删除 acados 缓存，不向其他人索取生成的二进制，不使用系统级安装来解决本地依赖。
- 系统 Python 为 3.12.3，ROS2 Jazzy 的 Python 路径同样是 3.12；当前终端还加载了多个其他工作区的 `PYTHONPATH` / `LD_LIBRARY_PATH`。显式使用本地隔离脚本可避免这些路径进入本工程进程。
- 仓库意外跟踪了 `uav_ac/rl/__pycache__/__init__.cpython-313.pyc`。导入会产生非源码差异；已恢复本次生成的该缓存差异，并在辅助脚本设置 `PYTHONDONTWRITEBYTECODE=1`，防止再次改写。
- `.venv` 的 Python 依赖 `cache/python/` 内的解释器；不能只删该目录后继续使用现有虚拟环境。
- `project-env.sh` 和环境产物被现有 `.gitignore` 忽略；重新克隆时不会自动包含它们。本次仅新增的交接 Markdown 需要出现在 Git 未跟踪列表中。

## 启动方式（环境已验证）

在仓库根目录执行：

```bash
./project-env.sh
```

不带参数时，辅助脚本执行项目原生的 quickstart 命令；保持终端当前 ROS 环境不变。也可以显式传入命令：

```bash
./project-env.sh .venv/bin/python -m uav_ac.main --config configs/flight/flight_quickstart.yaml
./project-env.sh .venv/bin/python -m uav_ac.main --help
./project-env.sh .venv/bin/python -m uav_ac.planning.native status
```

以后确需同步原锁定依赖时（通常不需要重复执行）：

```bash
./project-env.sh /home/lm/.local/bin/uv sync --locked --python 3.13 --managed-python
```

避免在已额外安装 acados Python 接口后直接执行精确 `uv sync`：acados 不在项目依赖声明内，精确同步可能移除额外安装包；届时应记录接口安装并使用合适的同步方式。

## 验证记录和文件位置

- 总报告：`runs/environment/verification-20261102T142213/report.json`，`all_checks_passed: true`。
- 各项标准输出：同目录 `*.log`；依赖版本快照：`installed-packages.log`。
- 场景渲染：同目录 `mujoco-render.png`。
- 本次集中验证脚本：`runs/environment/verify_environment.py`；会新建带时间戳的输出目录，并短暂打开后自动关闭查看器。
- 安装与本轮验证没有执行 sudo、apt 安装、系统 pip 安装，也没有修改 `.bashrc` / `.zshrc`、ROS 安装、驱动、其他工作区或原始飞行配置。
