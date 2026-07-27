# InfiniBand (IB) 高速网卡 Mellanox OFED 驱动部署 SOP

在多机多卡 GPU 大模型分布式训练（如 NCCL 跨节点通信）中，InfiniBand (IB) 高速网络是提供高吞吐、低延迟及 RDMA 算力通信的关键。本 SOP 记录 Mellanox/NVIDIA ConnectX 系列网卡 OFED 驱动的标准化安装与验证流程。

---

## 一、 前置检查与环境准备

### 1. 确认 IB 网卡硬件识别
```bash
lspci | grep -i mellanox

```
 * 确认能看到 ConnectX-6 / ConnectX-7 等 Mellanox PCIe 网卡设备。
### 2. 安装编译依赖包
```bash
sudo apt-get update
sudo apt-get install -y build-essential perl m4 debhelper tcl tk gcc make

```
## 二、 MLNX_OFED 驱动安装
### 1. 解压驱动安装包
从 NVIDIA/Mellanox 官网下载对应系统版本的 OFED 驱动（例如 MLNX_OFED_LINUX-23.10-0.5.5.0-ubuntu22.04-x86_64.tgz）：
```bash
tar -zxvf MLNX_OFED_LINUX-*.tgz
cd MLNX_OFED_LINUX-*

```
### 2. 执行安装脚本
针对 GPU 智算节点，使用带有 DKMS 与 RDMA 支持的参数进行安装：
```bash
sudo ./mlnxofedinstall --dkms --add-kernel-support --force

```
### 3. 重启 OFED 驱动模块与服务
```bash
sudo /etc/init.d/openibd restart

```
## 三、 RDMA 与 IB 网卡状态验证
### 1. 查看 IB 端口物理状态
```bash
ibstat

```
 * **交付标准：**
   * State: Active（物理链路已激活）
   * Physical state: LinkUp（物理链路已连接）
   * Rate: 200 或 400（显示正确的 200Gb/s 或 400Gb/s 带宽速率）
### 2. 检查 RDMA 设备
```bash
ibv_devinfo -v

```
 * 确认所有 IB 网卡节点的端口状态均处于 PORT_ACTIVE。
### 3. 跨节点 RDMA 性能测试（可选）
在两台 IB 节点机器之间使用 ib_write_bw 执行带宽高吞吐压测：
 * **服务端：** ib_write_bw -a
 * **客户端：** ib_write_bw -a <服务端_IP>
```

---
