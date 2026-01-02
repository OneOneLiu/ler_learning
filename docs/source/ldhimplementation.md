# LeRobot 学习笔记

## 1. Docker 镜像构建 ✅

在项目根目录执行：

```bash
cd /home/daohui/catkin_ws/src/ler_learning
docker build -f docker/Dockerfile.root -t lerobot-gpu:latest .
```

状态：已完成 ✅

## 2. 运行 Docker 容器

### 方式一：纯容器环境（推荐学习） ✅

```bash
docker run -it --rm --gpus all \
  --name lerobot_dev \
  -v ~/.cache/huggingface:/home/user_lerobot/.cache/huggingface \
  -v /home/daohui/lerobot_outputs:/outputs \
  --ipc=host \
  --network host \
  lerobot-gpu:latest bash
```

使用容器内的完整环境，立即可用。

### 方式二：开发模式（可访问宿主机代码） ✨

```bash
docker run -it --rm --gpus all \
  --name lerobot_dev \
  -v /home/daohui/catkin_ws/src/ler_learning:/workspace \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  -v /home/daohui/lerobot_outputs:/outputs \
  --ipc=host \
  --network host \
  --volume="/home/$USER/.ssh_docker:/root/.ssh/" \
  --volume="/home/$USER/.gitconfig:/root/.gitconfig:rw" \
  lerobot-gpu:latest bash
```

- `/lerobot` - 容器内的环境（可运行）
- `/workspace` - 宿主机代码（可编辑）
- 两者互不干扰！

## 3. 验证环境 ✅

进入容器后执行：

```bash
# 检查 LeRobot 安装
lerobot-info

# 检查 GPU
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}'); print(f'GPU count: {torch.cuda.device_count()}')"

# 检查环境
which python
```

**验证结果：**
- LeRobot 版本: 0.4.3 ✅
- PyTorch: 2.7.1+cu126 ✅
- GPU: NVIDIA RTX 6000 Ada Generation ✅
- CUDA 可用: True ✅
- GPU 数量: 1 ✅

**注意：** 如果需要用 pip，使用 `python -m pip` 代替 `pip` 命令。

## 4. 学习路径

### Step 1: 了解数据集
```python
from lerobot.datasets.lerobot_dataset import LeRobotDataset

# 加载一个示例数据集
dataset = LeRobotDataset("lerobot/pusht")
print(f"数据集大小: {len(dataset)}")
print(f"第一个样本: {dataset[0].keys()}")
```

### Step 2: 运行第一个训练示例（ACT）
```bash
cd /lerobot/examples/tutorial/act
python act_training_example.py
```

### Step 3: 使用命令行训练
```bash
lerobot-train \
  --policy=act \
  --dataset.repo_id=lerobot/pusht \
  --training.offline_steps=1000
```

### Step 4: 评估模型
```bash
lerobot-eval \
  --policy.path=lerobot/act_pusht \
  --env.type=pusht \
  --eval.n_episodes=10
```

## 常见问题

### Git 权限问题

**问题：** 容器内无法 commit
**原因：** `/lerobot` 目录权限问题

**解决方案：**
1. **推荐：在宿主机使用 Git**
   ```bash
   # 在宿主机
   cd /home/daohui/catkin_ws/src/ler_learning
   git add .
   git commit -m "message"
   ```

2. 在容器内使用 `/workspace`（挂载目录）
   ```bash
   # 在容器内
   cd /workspace
   git config --global user.name "Your Name"
   git config --global user.email "your@email.com"
   git add .
   git commit -m "message"
   ```

3. 修复 `/lerobot` 权限
   ```bash
   # 在容器内
   git config --global --add safe.directory /lerobot
   ```

## 下一步计划

- [x] 构建 Docker 镜像
- [x] 运行容器并验证环境
- [ ] 探索数据集
- [ ] 运行 ACT 训练示例
- [ ] 理解 ACT 算法原理
- [ ] 尝试其他算法（Diffusion, VQ-BeT）
- [ ] 在模拟环境中评估模型