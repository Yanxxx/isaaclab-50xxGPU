[阅读中文版本 (Read in Chinese)](README_zh.md)

# Fixing PyTorch Compatibility for Isaac Lab on New-Generation NVIDIA GPUs

## 1. The Problem

This project addresses a compatibility issue when running the official Isaac Lab Docker image on **new-generation NVIDIA GPUs** (e.g., RTX 40-series, RTX 50-series, etc.).

When using this new hardware, the version of PyTorch that comes pre-installed in the base Isaac Sim Docker image is outdated. It lacks support for the newer GPU compute architectures (like `sm_120`), causing the application to crash when any GPU-related PyTorch operation is attempted. This typically results in the following critical error:

```
RuntimeError: CUDA error: no kernel image is available for execution on the device
```

## 2. The Solution

The fundamental solution is to **forcibly overwrite** the outdated PyTorch version during the Docker image build process. We will replace it with a modern, nightly build of PyTorch that includes support for the new GPU architectures.

This is achieved by modifying the project's `Dockerfile`.

## 3. Modification Steps

#### Step 1: Locate the Dockerfile

Find the `Dockerfile` in the root of your project. It might also be located at `docker/Dockerfile.base` depending on the project structure.

#### Step 2: Find the Target Location

Open the `Dockerfile` in a text editor and find the following block of code. This command is used by Isaac Lab to install its own dependencies, and our modification will be inserted immediately after it.

```dockerfile
# installing Isaac Lab dependencies
# use pip caching to avoid reinstalling large packages
RUN --mount=type=cache,target=${DOCKER_USER_HOME}/.cache/pip \
    ${ISAACLAB_PATH}/isaaclab.sh --install
```

#### Step 3: Insert the Override Code

**Directly below** the code block from Step 2, insert the following new commands. These commands will first uninstall the old PyTorch version and then install the new one.

```dockerfile
# ==================== BEGIN MODIFICATION: Force PyTorch Update ====================
# [INFO] Forcibly overwriting the base image's pre-installed PyTorch to support new-generation GPUs.

# 1. Uninstall the old torch version installed by the base image or the script above.
RUN ${ISAACLAB_PATH}/_isaac_sim/python.sh -m pip uninstall -y torch torchvision torchaudio

# 2. Install the latest nightly build of torch that supports the new GPU.
#    You can get the latest command from the official PyTorch website: [https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/)
RUN ${ISAACLAB_PATH}/_isaac_sim/python.sh -m pip install --pre torch torchvision torchaudio --index-url [https://download.pytorch.org/whl/nightly/cu124](https://download.pytorch.org/whl/nightly/cu124)
# ==================== END MODIFICATION ====================
```

After your changes, this section of the `Dockerfile` should look like this:

```dockerfile
# ... (previous content of Dockerfile)

# installing Isaac Lab dependencies
RUN --mount=type=cache,target=${DOCKER_USER_HOME}/.cache/pip \
    ${ISAACLAB_PATH}/isaaclab.sh --install

# ==================== BEGIN MODIFICATION: Force PyTorch Update ====================
# [INFO] Forcibly overwriting the base image's pre-installed PyTorch to support new-generation GPUs.

# 1. Uninstall the old torch version installed by the base image or the script above.
RUN ${ISAACLAB_PATH}/_isaac_sim/python.sh -m pip uninstall -y torch torchvision torchaudio

# 2. Install the latest nightly build of torch that supports the new GPU.
#    You can get the latest command from the official PyTorch website: [https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/)
RUN ${ISAACLAB_PATH}/_isaac_sim/python.sh -m pip install --pre torch torchvision torchaudio --index-url [https://download.pytorch.org/whl/nightly/cu124](https://download.pytorch.org/whl/nightly/cu124)
# ==================== END MODIFICATION ====================

# HACK: Remove install of quadprog dependency
RUN ${ISAACLAB_PATH}/isaaclab.sh -p -m pip uninstall -y quadprog

# ... (subsequent content of Dockerfile)
```

## 4. Rebuild the Docker Image

Save your modified `Dockerfile`. Then, navigate to your project's root directory in your terminal and run the build command.

**Note:** You **must** use the `--no-cache` flag to ensure your changes are applied.

```bash
./isaaclab.sh --docker build --no-cache
```

This process will take some time as it needs to download and install the new PyTorch libraries.

## 5. Run

Once the build is complete, you can start and enter the container as usual. The PyTorch version inside the container will now be up-to-date and compatible with your new GPU.

```bash
# Start the container
./isaaclab.sh --docker start

# Enter the container's shell
./isaaclab.sh --docker shell
```
