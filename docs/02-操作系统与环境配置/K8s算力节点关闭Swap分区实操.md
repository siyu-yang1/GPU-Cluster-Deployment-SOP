# 如何永久关闭 Swap 分区

在算力节点及 K8s 环境中，必须禁用 Swap 以确保系统性能与集群调度的稳定性。请按照以下标准流程进行操作：

---

### 1. 临时禁用 Swap
使用 `swapoff -a` 命令临时关闭所有 Swap 设备：
```bash
sudo swapoff -a

```
### 2. 禁用 /etc/fstab 中的 Swap 配置
打开系统挂载配置文件 /etc/fstab，查找挂载 Swap 的相关行：
```bash
sudo vim /etc/fstab

```
在对应配置行的行首加上 # 进行注释，保存并退出。
### 3. 重启系统验证持久化
重启服务器以验证配置是否永久生效：
```bash
sudo reboot

```
### 4. 验证禁用状态
**检查项 1：使用 swapon 查看**
```bash
swapon --show

```
 * **结果判定：** 执行后终端无任何输出，说明 Swap 已被成功禁用。
**检查项 2：使用 free 查看内存状态**
```bash
free -h

```
 * **结果判定：** 输出结果中 Swap 一行的 total、used、free 数值全为 0B，即表示 Swap 永久关闭成功。
```
