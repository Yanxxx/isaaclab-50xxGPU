
[Switch to English (英文)](#en)

<a name="zh"></a>

# TLDR:

下载Dockerfile.base，并将其替换`{ISAACLAB_PATH}/docker/Dockerfile.base`。 Enjoy！


## 修复 Isaac Lab 在新一代 NVIDIA 显卡上的 PyTorch 兼容性问题

### 1. 问题背景

本项目旨在解决官方 Isaac Lab Docker 镜像在**新一代 NVIDIA 显卡**（例如 RTX 40/50 系列）上运行时遇到的兼容性问题。

当使用这些新显卡时，由于 Isaac Sim 基础镜像中预装的 PyTorch 版本过旧，不支持新的 GPU 计算架构（如 `sm_120`），程序会在尝试执行任何 GPU 相关的 PyTorch 操作时崩溃，并抛出以下关键错误：

```
RuntimeError: CUDA error: no kernel image is available for execution on the device
```

### 2. 解决方案

根本的解决方案是，在构建 Docker 镜像的过程中，**强制覆盖**掉基础镜像里预装的旧版 PyTorch，替换为能够支持新显卡架构的最新预览版 (Nightly Build) PyTorch。

这通过修改项目中的 `Dockerfile` 文件来实现。

### 3. 修改步骤

#### 第一步：定位 Dockerfile

在项目根目录下，找到 `Dockerfile` 文件（在您的情况下，即您上传的 `Dockerfile.txt` 文件，它通常也可能位于 `docker/Dockerfile.base`）。

#### 第二步：找到修改位置

用文本编辑器打开此 `Dockerfile`，找到下面这段代码。这段代码是 Isaac Lab 用来安装自身依赖项的，我们的修改将紧跟在它之后。

```dockerfile
# installing Isaac Lab dependencies
# use pip caching to avoid reinstalling large packages
RUN --mount=type=cache,target=${DOCKER_USER_HOME}/.cache/pip \
    ${ISAACLAB_PATH}/isaaclab.sh --install
```

#### 第三步：插入覆盖代码

在上面那段代码的**正下方**，插入以下代码块。它的作用是先卸载旧版 PyTorch，再安装新版。

```dockerfile
# ==================== 开始修改：强制更新 PyTorch ====================
# [说明] 强制覆盖基础镜像里预装的旧版PyTorch，以支持新一代GPU。

# 1. 卸载由基础镜像或上方脚本安装的旧版 torch
RUN ${ISAACLAB_PATH}/_isaac_sim/python.sh -m pip uninstall -y torch torchvision torchaudio

# 2. 安装支持新显卡的最新预览版 torch
#    可以从 PyTorch 官网获取最新的指令: [https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/)
RUN ${ISAACLAB_PATH}/_isaac_sim/python.sh -m pip install --pre torch torchvision torchaudio --index-url [https://download.pytorch.org/whl/nightly/cu124](https://download.pytorch.org/whl/nightly/cu124)
# ==================== 结束修改 ====================
```

### 4. 重新构建 Docker 镜像

保存修改后的 `Dockerfile` 文件。然后回到项目根目录的终端，运行构建命令。

**注意：** 为了确保您的修改能够生效，**必须**使用 `--no-cache` 标志。

```bash
./isaaclab.sh --docker build --no-cache
```

这个过程会需要一些时间，因为它需要重新下载并安装新版本的 PyTorch。

### 5. 运行

构建成功后，您就可以像往常一样启动并进入容器了，此时容器内的 PyTorch 版本已经更新，可以正常支持您的新显卡。

```bash
# 启动容器
./isaaclab.sh --docker start

# 进入容器
./isaaclab.sh --docker shell
```
