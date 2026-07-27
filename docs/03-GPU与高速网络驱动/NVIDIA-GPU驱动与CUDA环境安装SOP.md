# NVIDIA GPU 驱动与 CUDA 架构安装 SOP

在智算中心及高算力集群部署中，NVIDIA 驱动与 CUDA Toolkit 的标准化安装是决定 GPU 能否正常识别、发挥算力性能的核心前提。本 SOP 适用于 Ubuntu 22.04 LTS / 24.04 LTS 环境下的标准离线/在线部署。

---

## 一、 环境准备与前置检查

在安装 NVIDIA 驱动前，必须确保已完成以下准备：
1. **禁用 Nouveau 开源驱动：** 确保 `lsmod | grep nouveau` 无任何输出（若未禁用请参阅禁用 SOP）。
2. **确认 GPU 硬件状态：**
   ```bash
   # 检查系统是否正常识别到 NVIDIA PCIe 设备
   lspci | grep -i nvidia

```
## 二、 NVIDIA GPU 驱动安装
建议优先使用官方 .run 离线安装包（版本兼容性最好，不依赖系统 apt 源）：
### 1. 下载驱动 run 包
从 NVIDIA 官方驱动库获取对应显卡型号（如 H100/A100/L40S/RTX 4090）的 Linux 驱动：
```bash
# 以 535/550 系列数据中心驱动为例
chmod +x NVIDIA-Linux-x86_64-535.129.03.run

```
### 2. 执行静默/交互式安装
```bash
sudo ./NVIDIA-Linux-x86_64-535.129.03.run \
  --no-opengl-files \
  --no-nouveau-check \
  --dkms \
  -s

```
 * **参数说明：**
   * --no-opengl-files：不安装 NVIDIA Desktop 相关的 OpenGL 库，避免导致 GUI 显示异常。
   * --dkms：启用 DKMS 机制，确保后续 Linux 内核小升级时自动重编译驱动模块。
### 3. 验证驱动状态
安装完成后，执行：
```bash
nvidia-smi

```
 * **交付标准：** 能够正常输出 GPU 列表、显存容量、温度及 Driver Version 驱动版本号。
## 三、 CUDA Toolkit 架构部署
### 1. 安装 CUDA Toolkit (以 CUDA 12.2 为例)
下载对应的 .run 安装包：
```bash
chmod +x cuda_12.2.0_535.54.03_linux.run
sudo ./cuda_12.2.0_535.54.03_linux.run

```
> **注意：** 在安装界面的组件选择中，**取消勾选 [ ] Driver**（因为上面已经单独安装了驱动），仅保留 CUDA Toolkit、CUDA Samples 等组件。
> 
### 2. 配置环境变量
在 /etc/profile 或用户 ~/.bashrc 末尾写入环境变量：
```bash
export PATH=/usr/local/cuda-12.2/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-12.2/lib64:$LD_LIBRARY_PATH

```
刷新使之生效：
```bash
source ~/.bashrc

```
### 3. 验证 CUDA 环境
```bash
nvcc -V

```
 * **结果判定：** 终端成功输出 nvcc: NVIDIA (R) Cuda compiler driver 及正确的 CUDA 版本信息。
```

---
