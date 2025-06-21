[Read in English](README.md)

# 修复 Isaac Lab 在新一代 NVIDIA 显卡上的 PyTorch 兼容性问题

## 1. 问题背景

本项目旨在解决官方 Isaac Lab Docker 镜像在**新一代 NVIDIA 显卡**（例如 RTX 40/50 系列）上运行时遇到的兼容性问题。

当使用这些新显卡时，由于 Isaac Sim 基础镜像中预装的 PyTorch 版本过旧，不支持新的 GPU 计算架构（如 `sm_120`），程序会在尝试执行任何 GPU 相关的 PyTorch 操作时崩溃，并抛出以下关键错误：

```
RuntimeError: CUDA error: no kernel image is available for execution on the device
```

## 2. 解决方案

根本的解决方案是，在构建 Docker 镜像的过程中，**强制覆盖**掉基础镜像里预装的旧版 PyTorch，替换为能够支持新显卡架构的最新预览版 (Nightly Build) PyTorch。

这通过修改项目中的 `Dockerfile` 文件来实现。

## 3. 修改步骤

####
