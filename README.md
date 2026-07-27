# 🚀 GPU 集群部署与运维实操 SOP 库

本仓库记录大厂 GPU 智算集群交付、硬件管控、操作系统加固、NVIDIA 驱动部署及自动化运维脚本的标准化操作规程（SOP）。

---

## 📂 文档与 SOP 导航

### 1. 硬件与底层管理
- [BMC 远程管控与服务器初始化](./docs/01-硬件与底层管理/BMC服务器远程配置实操SOP.md)

### 2. 操作系统与系统加固
- [Ubuntu 离线环境软件及依赖包安装指南](./docs/02-操作系统与环境配置/Ubuntu离线环境软件及依赖包安装指南SOP.md)
- [NVIDIA 驱动前置：禁用 nouveau 模块与内核锁版本](./docs/02-操作系统与环境配置/Ubuntu禁用Nouveau开源驱动实操SOP.md)

### 3. 驱动与环境部署
- [NVIDIA GPU 驱动与 CUDA 架构安装 SOP](./docs/03-GPU与高速网络驱动/NVIDIA-GPU驱动与CUDA环境安装SOP.md)
- [InfiniBand (IB) 高速网卡 OFED 驱动部署](./docs/03-GPU与高速网络驱动/Mellanox-IB网卡OFED驱动部署.md)

### 4. 自动化脚本工具
- [Python 自动化日志解析与 Excel 报表生成工具](./docs/04-自动化工具与运维脚本/Python自动化日志解析与Excel报表生成工具SOP.md)

---

## 🛠️ Python 自动化日志解析工具快速使用

### 功能说明
在现场交付中，数百台机器的日志分散分布。本工具用于自动抓取硬件状态/日志报错，自动填充至标准 Excel 交付表格中，大幅提升交付效率。

### 使用方法
```bash
python3 docs/04-自动化工具与运维脚本/parse_log_to_excel.py
