
# 智算中心离线环境关键驱动与组件安装指南 (OFED / NVIDIA / FabricManager)

## 一、 场景说明与核心逻辑
在智算中心隔离网络（Air-Gapped）环境下，无法直接通过外网源进行在线拉取。高密 GPU 节点的核心软件栈（Mellanox 高速网卡驱动、NVIDIA 显卡驱动、NVSwitch FabricManager）必须采用离线交付方式。

离线交付核心三步法：
1. 外网同构机下载：精准匹配系统版本与内核版本，抓取离线包与依赖；
2. 介质分发/导入：通过内部跳板机、SCP 传输或专用介质分发至离线节点；
3. 本地加载与底层安装：处理图形桌面占用、内核头文件编译环境与相关服务闭环。

---

## 二、 前置软件包准备与下载（联网跳板机执行）

### 1. Mellanox OFED 高速网卡驱动下载
针对 Ubuntu 22.04 LTS 与 ConnectX-6 (CX6) / ConnectX-7 (CX7) 网卡：
```bash
wget [https://content.mellanox.com/ofed/MLNX_OFED-23.10-3.2.2.0/MLNX_OFED_LINUX-23.10-3.2.2.0-ubuntu22.04-x86_64.iso](https://content.mellanox.com/ofed/MLNX_OFED-23.10-3.2.2.0/MLNX_OFED_LINUX-23.10-3.2.2.0-ubuntu22.04-x86_64.iso)

```

> 注：在以太网 RoCE 组网场景下，CX6 / CX7 网卡若默认识别为 IB 链路，需使用 mstconfig 工具将其链路类型由 InfiniBand (IB) 切换为 Ethernet 模式。

### 2. NVIDIA Tesla GPU 驱动下载

针对数据中心计算卡（如 Tesla/Hopper 架构，以 535.183.06 版本为例）：

```bash
wget [https://cn.download.nvidia.com/tesla/535.183.06/NVIDIA-Linux-x86_64-535.183.06.run](https://cn.download.nvidia.com/tesla/535.183.06/NVIDIA-Linux-x86_64-535.183.06.run)

```

### 3. NVIDIA FabricManager 离线包准备

针对搭载 NVSwitch 的高密多卡（如 HGX / SXM）架构节点，必须准备与 GPU 驱动版本严格一致的 FabricManager 安装包：

```bash
# 提取与驱动版本 (535.183.06) 严格匹配的离线包
apt-get download nvidia-fabricmanager-535

```

---

## 三、 介质分发与拷贝

将下载完成的 .iso、.run 及依赖包传输至目标专网服务器：

```bash
# 通过内网跳板机 SCP 拷贝
scp MLNX_OFED_LINUX-*.iso NVIDIA-Linux-*.run user@<目标节点IP>:/data/packages/

```

---

## 四、 离线安装与配置全流程（离线节点执行）

### 1. Mellanox OFED 网卡驱动离线安装

#### (1) 挂载 ISO 镜像文件

```bash
sudo mkdir -p /mnt/ofed
sudo mount -o loop MLNX_OFED_LINUX-23.10-3.2.2.0-ubuntu22.04-x86_64.iso /mnt/ofed

```

#### (2) 触发安装脚本与内核适配

```bash
cd /mnt/ofed

# 标准一键安装
sudo ./mlnxofedinstall

# 【重要踩坑排障】若提示内核模块不匹配或缺少内核源码支持，追加 --add-kernel-support 参数重新编译打包
sudo ./mlnxofedinstall --add-kernel-support

```

#### (3) 重启驱动模块或重启节点

```bash
# 重启 openibd 驱动服务或整机重启生效
sudo /etc/init.d/openibd restart

```

---

### 2. NVIDIA GPU 驱动离线规范化安装

#### (1) 卸载/关闭系统图形界面服务（关键前置步骤）

若系统默认运行了 X-Window 或桌面显示服务，NVIDIA 安装程序将直接检测报错并中断安装。需将其切换至纯多用户文本模式：

```bash
# 切换系统运行目标为文本模式 (multi-user)
sudo systemctl set-default multi-user.target
sudo reboot

```

#### (2) 执行显卡驱动静默/引导安装

```bash
# 赋予驱动文件可执行权限
chmod +x NVIDIA-Linux-x86_64-535.183.06.run

# 执行安装（建议跳过 OpenGL、禁用 Nouveau 已提前完成）
sudo ./NVIDIA-Linux-x86_64-535.183.06.run --no-opengl-files

```

#### (3) 安装 FabricManager 并拉起系统服务

```bash
# 安装匹配的 FabricManager deb 包
sudo dpkg -i nvidia-fabricmanager-535_*.deb

# 启动并设置开机自启
sudo systemctl enable --now nvidia-fabricmanager
sudo systemctl status nvidia-fabricmanager

```

#### (4) 恢复图形界面模式（按需配置）

若特定现场需要保留桌面 GUI 环境，可在底层驱动固化后恢复 target 设置：

```bash
sudo systemctl set-default graphical.target
sudo reboot

```

#### (5) 驱动状态与设备可用性验证

```bash
nvidia-smi

```

> 验证确认所有物理 GPU 均正常枚举识别，无掉卡、温度功耗正常，且 CUDA Driver Version 正常显现。

---

## 五、 现场交付核心踩坑点与排障总结

1. 底层依赖缺失导致安装中断：
* 现象：提示缺少编译环境 gcc、make、build-essential 等。
* 排障：在联网环境使用 apt-get download 将相关依赖包全量抓取，通过 dpkg -i *.deb 先行离线补全依赖。


2. Linux 内核头文件不匹配：
* 现象：编译内核模块报错，提示找不到对应内核目录 /lib/modules/.../build。
* 排障：运行 uname -r 核对当前内核版本，在离线节点必须安装与当前内核精确对应的头文件包：
sudo dpkg -i linux-headers-$(uname -r)_*.deb


3. 图形界面未完全杀除 (X server is running)：
* 现象：运行 .run 驱动时提示错误退出。
* 排障：必须执行 systemctl set-default multi-user.target 重启进入无头终端，或手动停用 systemctl stop gdm3 / lightdm。



```

```
