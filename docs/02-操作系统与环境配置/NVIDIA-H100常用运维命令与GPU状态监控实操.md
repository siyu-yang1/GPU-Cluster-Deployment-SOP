# NVIDIA H100 常用运维命令与 GPU 状态监控实操

## 一、 场景说明
在以 NVIDIA H100 为核心算力的智算中心集群中，日常巡检、交付验收及训练卡顿排查需频繁通过底层命令行工具对 GPU 状态进行监控。本文档整理了针对 H100 (SXM5 / PCIe) 架构的高频巡检与硬件排错指令。

---

## 二、 基础状态与高频巡检命令

### 1. 实时全卡健康状态巡检
```bash
# 默认每秒刷新一次全卡状态（显存占用、温度、功耗、GPU 利用率）
watch -n 1 nvidia-smi

# 格式化输出关键指标（显卡序号、UUID、温度、已用显存、总显存、当前功耗）
nvidia-smi --query-gpu=index,gpu_name,uuid,temperature.gpu,memory.used,memory.total,power.draw --format=csv

```

### 2. 查看拓扑链路与通信状态 (NVLink / PCIe)

```bash
# 查看所有 GPU 之间的互联拓扑（NVLink 互联矩阵，确认卡间走的是 NVLink 还是 PCIe/SYS）
nvidia-smi topo -m

# 查看 PCIe 协商带宽与链路状态（确认是否协商在 Gen5 x16）
nvidia-smi -q -d PCIE

```

---

## 三、 硬件高阶监控与排障命令

### 1. 显存 ECC 错误排查（重要）

H100 高密计算卡在长时间满载下若出现硬件不稳定，首先需核对 ECC 报错：

```bash
# 查询易失性 (Volatile) 与非易失性 (Aggregate) ECC 错误计数
nvidia-smi -q -d ECC

# 查看被动态隔离下线的坏块页 (Page Retirement)
nvidia-smi -q -d PAGE_RETIREMENT

```

> **排障标准**：
> * 若 `Single Bit ECC`（单比特错误）数量较少，硬件具有自动纠错能力，属于正常范围；
> * 若出现 `Double Bit ECC`（双比特不可纠正错误）或大量 `Pending Page Retirement`，代表显存颗粒存在物理损伤，需立即下线报修换卡。
> 
> 

### 2. 功耗墙与降频状态 (Throttling) 排查

当训练作业吞吐突然下降时，检查显卡是否由于供电不足或散热不良触发了主动降频：

```bash
# 查看 GPU 当前降频原因（Thermal/Power/HW Slowdown）
nvidia-smi -q -d PERFORMANCE

```

### 3. 查看 NVSwitch 与 Fabric 状态

```bash
# 针对 SXM 架构节点，确认各端口 NVLink 状态处于 Active
nvidia-smi nvlink -s

```

---

## 四、 常见异常处理 SOP

1. **GPU 掉卡 (Unable to determine the device handle...)**：
* **现象**：执行 `nvidia-smi` 报错找不到卡，或原本 8 卡仅显示 7 卡。
* **处理**：
1. 执行 `lspci | grep -i nvidia` 查看 PCI 总线上物理硬件是否存在；
2. 检查内核日志：`dmesg -T | grep -i nvrm` 查看是否存在固件崩溃或 PCIe 掉电；
3. 若 PCI 无法识别，通过 BMC 执行冷重启（Chassis Power Reset）；若仍无法识别，需现场报修重插或更换 GPU 板卡。




2. **GPU 处于僵尸进程占用 (Process not found but memory used)**：
* **处理**：执行 `fuser -v /dev/nvidia*` 找出实际占用底层字符设备的残留 PID，使用 `kill -9 <PID>` 强制释放显存。
