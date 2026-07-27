# BMC 远程管控与服务器初始化实操 SOP

在智算中心 GPU 服务器硬件物理上架与拉线完成后，必须首先通过带外管理网络（BMC/iLO/iDRAC）进行远程管控配置与服务器硬件初始化，为后续操作系统部署及 GPU 驱动安装奠定基础。

---

## 一、 背景与作用

### 1. 什么是 BMC？
BMC（Baseboard Management Controller，板载管理控制器）是服务器主板上的独立芯片，拥有独立的 CPU、内存与网卡，不受主操作系统（如 Linux/Windows）状态影响。

### 2. 核心作用
* **带外远程控制：** 即使服务器操作系统崩溃或断网，仍可通过 HTML5 KVM 界面进行远程开关机、重装系统。
* **硬件健康监控：** 实时监控 GPU 显存温度、CPU 功耗、风扇转速、电源冗余及 PCIe 设备状态。
* **带外日志收集：** 导出 SEL（System Event Log）硬件日志，用于硬件故障（如 GPU 掉卡、内存 ECC 错误）精准排查。

---

## 二、 BMC 初始化配置步骤

### 步骤 1：配置 BMC 独立管理网口 IP
1. 服务器开机按 `DEL` 或 `F2` 进入 BIOS 设置。
2. 导航至 `Server Mgmt` -> `BMC Network Configuration`。
3. 将 IP 配置模式设置为 `Static`（静态 IP），配置规划好的管理网段 IP、子网掩码及网关。
4. 保存并退出 BIOS（按 `F10`）。

### 步骤 2：登录 BMC Web 管理界面
1. 在运维终端浏览器输入配置好的 BMC IP（如 `https://192.168.100.50`）。
2. 输入管理员账号密码登录（默认账户通常为 `admin` / `admin` 或 `root` / `superuser`）。
3. **安全加固：** 首次登录后必须立即修改默认初始密码，并创建专用的交付运维账号。

---

## 三、 服务器 BIOS 与硬件初始化

在 BMC Web 界面中通过 **HTML5 KVM / 远程终端** 连接服务器控制台，进行以下初始化设置：

### 1. BIOS 关键参数优化（针对高算力场景）
* **Performance Mode：** 设置为 `Max Performance`（高性能模式），禁用 CPU 动态降频与节能模式。
* **C-States / P-States：** 禁用 `C-States`，防止 CPU 频繁切换休眠状态导致算力抖动。
* **Virtualization Technology：** 开启 `Intel VT-x` / `AMD-V` 及 `SR-IOV`（支持后续容器/虚拟机网卡直通）。
* **PCIe 64-bit BAR Decoding：** 必须**开启 Above 4G Decoding / Re-BAR**（确保系统能识别多卡大显存 GPU）。

### 2. RAID 阵列构建（系统盘与数据盘）
1. 重启进入 RAID 控制器配置界面（或直接在 BMC Web 的 Storage 模块操作）。
2. **系统盘：** 选用两块 NVMe/SSD 构建 **RAID 1**（镜像冗余，保证 OS 稳定性）。
3. **数据盘/缓存盘：** 多块 NVMe 高速盘构建 **RAID 0** 或 **RAID 10**（追求最高 IOPS 读写性能）。

---

## 四、 验证与交付标准

在 BMC 界面完成以下最终检查：
1. **Sensors / Health Status：** 确认所有 GPU 节点、CPU、电源、风扇显示绿色的 `OK` / `Normal` 状态。
2. **Virtual Media (虚拟介质)：** 验证能够正常挂载 Ubuntu 22.04 LTS / 24.04 LTS 官方 ISO 镜像文件，准备进行操作系统离线安装。
