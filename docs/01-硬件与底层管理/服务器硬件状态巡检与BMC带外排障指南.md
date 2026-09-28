# 服务器硬件状态巡检与 BMC 带外排障指南

## 一、 核心硬件物理状态检查规范
在智算中心机房交付与日常巡检中，首先需确认物理服务器指示灯及各部件状态：
1. **电源模块 (PSU)**：确认双冗余电源指示灯常绿；若出现琥珀色/橙色常亮或闪烁，需排查市电供电或模块故障。
2. **硬盘背板与盘符状态**：
   * 确认各盘位无黄色告警灯（Amber）；
   * 通过 RAID 阵列卡控制台或系统工具确认各磁盘状态处于 Online 状态，排查 Rebuilding 或 Failed 状态。
3. **内存与 CPU 状态**：
   * 开机自检通过系统控制台确认内存识别容量与物理插槽一致；
   * 检查是否出现不可纠正 ECC 错误导致的通道降速或屏蔽。

---

## 二、 BMC / IPMI 远程带外管理实操

### 1. 登录与基础配置
* 通过专用带外网络（Mgmt 网口）配置静态 IP 地址，使用浏览器访问 BMC Web 管理后台。
* 在无 GUI 环境下，通过终端使用 `ipmitool` 开展带外控制。

### 2. 常用 IPMI 带外运维指令
```bash
# 查看 BMC 带外固件版本及基本信息
ipmitool mc info

# 查看服务器底板所有传感器数据（温度、风扇转速、电压）
ipmitool sdr list

# 查看系统硬件事件告警日志 (SEL - System Event Log)
ipmitool sel elist

# 发生异常硬件误报时清空日志缓存
ipmitool sel clear

# 查看当前电源状态
ipmitool chassis power status

# 远程冷重启 / 开机 / 关机
ipmitool chassis power reset
ipmitool chassis power on
ipmitool chassis power off
