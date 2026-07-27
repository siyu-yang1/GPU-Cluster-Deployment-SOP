# Ubuntu 系统禁用 Nouveau 开源驱动实操 SOP

在部署 NVIDIA 官方 GPU 驱动或 CUDA 架构前，必须优先禁用 Linux 系统默认加载的开源驱动 `nouveau`，否则会导致官方驱动安装失败、设备识别异常或内核崩溃。

---

## 一、 背景介绍

### 1. 什么是 Nouveau？
`nouveau` 是由第三方开源社区维护的 NVIDIA 图形驱动程序，旨在为 Linux 系统提供基础的显示支持。

### 2. 为什么部署 GPU 集群必须禁用它？
* **驱动冲突：** `nouveau` 会常驻内核并独占 GPU 设备句柄，导致 NVIDIA 官方闭源驱动（如 `nvidia-driver`）无法正常加载内核模块。
* **功能受限：** `nouveau` 不支持 CUDA 计算、NVLink 高速互联以及高性能 GPU 虚拟化，无法满足智算中心的高性能计算需求。

---

## 二、 禁用 Nouveau 完整操作步骤

### 步骤 1：检查 Nouveau 是否正在运行
在终端中执行以下命令：
```bash
lsmod | grep nouveau

```
 * **结果判定：** 若终端有内容输出，说明 nouveau 驱动当前正在运行，必须按照后续步骤进行禁用。
### 步骤 2：修改黑名单配置文件
新建并编辑黑名单配置文件 /etc/modprobe.d/blacklist-nouveau.conf：
```bash
sudo vim /etc/modprobe.d/blacklist-nouveau.conf

```
在文件中添加以下两行配置：
```text
blacklist nouveau
options nouveau modeset=0

```
保存并退出（Vim 中依次按 Esc -> 输入 :wq -> Enter）。
### 步骤 3：更新 Initial RAM Filesystem (initramfs)
修改配置后，必须更新内核引导镜像以应用黑名单：
```bash
sudo update-initramfs -u

```
### 步骤 4：重启服务器
重启系统使内核重新加载配置：
```bash
sudo reboot

```
## 三、 验证禁用状态
系统重启完成后，重新登录终端并执行验证命令：
```bash
lsmod | grep nouveau

```
 * **成功标准：** 执行后终端**没有任何输出**，即表示 nouveau 驱动已成功彻底禁用，可以开始进行 NVIDIA 官方驱动的安装。
```

---
