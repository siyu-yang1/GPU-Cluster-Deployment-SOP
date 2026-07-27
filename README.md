# 🚀 GPU 智算中心现场交付与运维 SOP 实战指南

本项目基于智算中心（如宁夏国家算力枢纽节点）现场交付与运维实操经验沉淀，涵盖服务器硬件上架、BMC 远程管理、断网/离线环境部署、NVIDIA GPU / IB 网卡驱动安装、硬件满载压测及 Python 自动化报表提效全流程。

---

## 📌 项目亮点与核心能力

- 🛡️ **内网/离线交付实操：** 总结 Ubuntu 离线依赖包打包与离线 `dpkg` 部署套路，解决政企绝密机房断网部署痛点。
- ⚡ **环境踩坑避雷：** 详细记录禁用 `nouveau` 开源驱动、锁死内核更新、关闭 Swap 分区等 GPU 驱动部署前置 SOP。
- 📊 **代码自动化提效：** 提供自研 Python 自动化日志解析工具，实现多节点原始日志自动提取并导出结构化 Excel 报表。
- 🔧 **标准化压测：** 熟练掌握 `fio`（存储读写）、`stress-ng`（CPU/内存）、`gpu-burn`（GPU 满载）硬件验收规范。

---

## 📂 文档与 SOP 导航

### 1. 硬件与底层管理
- [BMC 远程管控与服务器初始化](./docs/01-硬件与BMC远程管理/BMC服务器远程配置实操.md)

### 2. 操作系统与系统加固
- [Ubuntu 离线环境软件及依赖包安装指南](./docs/02-操作系统与环境配置/Ubuntu2204离线依赖包安装指南.md)
- [NVIDIA 驱动前置：禁用 nouveau 模块与内核锁版本](./docs/02-操作系统与环境配置/永久禁用nouveau开源驱动与屏蔽内核更新.md)
- [关闭 Swap 分区实操规范](./docs/02-操作系统与环境配置/K8s算力节点关闭Swap分区实操.md)

### 3. 驱动与环境部署
- [NVIDIA GPU 驱动与 CUDA 架构安装 SOP](./docs/03-GPU与高速网络驱动/NVIDIA-GPU驱动与CUDA环境安装SOP.md)
- [InfiniBand (IB) 高速网卡 OFED 驱动部署](./docs/03-GPU与高速网络驱动/Mellanox-IB网卡OFED驱动部署.md)

### 4. 自动化脚本工具
- [Python 自动化日志解析与 Excel 报表生成工具](./docs/05-自动化提效工具/parse_log_to_excel.py)

---

## 🛠️ Python 自动化日志解析工具快速使用

### 功能说明
在现场交付中，几百台机器的日志离散分布。本工具用于自动抓取硬件状态/日志报错，自动填充至标准 Excel 交付表格中，大幅提升交付效率。

### 使用方法
```bash
# 1. 克隆项目
git clone [https://github.com/siyu-yang1/GPU-Cluster-Deployment-SOP.git](https://github.com/siyu-yang1/GPU-Cluster-Deployment-SOP.git)

# 2. 运行脚本解析日志
python3 parse_log.py
