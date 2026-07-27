# Ubuntu 离线环境软件及依赖包安装指南 SOP

在政企保密机房、智算中心隔离网段等**无公网连接（Air-Gapped）**的真实生产环境中，传统的 `apt update` 和 `apt install` 将无法直接使用。本指南详细记录如何在大厂运维实战中打通离线依赖包的打包、传输与离线安装全流程。

---

## 一、 整体解决方案思路

在离线环境中安装软件，核心流程分为 **有网环境（准备阶段）** 和 **无网环境（部署阶段）**：

```text
 [有网环境/开发机]                          [无网环境/智算节点]
1. 下载 .deb 软件及递归依赖包  ─── U盘/堡垒机 ───> 1. 解压依赖资源包
2. 打包为 tar.gz 压缩包      ─── 传输 ───────> 2. 使用 dpkg 批量离线安装

```
## 二、 步骤 1：在有网环境打成离线依赖包
寻找一台与目标服务器 **Ubuntu 系统版本完全一致**（如同为 Ubuntu 22.04 LTS）的有网机器操作：
### 1. 清空本地 apt 缓存
```bash
sudo apt-get clean

```
### 2. 仅下载软件及其所有关联依赖包（不安装）
以安装 build-essential（编译环境）和 net-tools 为例：
```bash
# 使用 --download-only 参数，依赖包会自动下载到 /var/cache/apt/archives/
sudo apt-get install --download-only -y build-essential net-tools

```
### 3. 创建独立打包目录并汇总包文件
```bash
# 创建离线包目录
mkdir -p ~/offline-packages

# 将下载的所有 .deb 依赖包复制到该目录下
cp /var/cache/apt/archives/*.deb ~/offline-packages/

# 将目录压缩打包
tar -zcvf offline-packages.tar.gz -C ~/ offline-packages

```
## 三、 步骤 2：离线服务器部署与安装
将 offline-packages.tar.gz 压缩包通过 U 盘、跳板机（堡垒机）或 SFTP 传输至目标离线 GPU 服务器上：
### 1. 解压离线包
```bash
tar -zxvf offline-packages.tar.gz
cd offline-packages

```
### 2. 使用 dpkg 批量强行安装依赖
```bash
# -R 参数表示递归安装目录下所有 .deb 文件
sudo dpkg -i -R ./*.deb

```
## 四、 踩坑避雷与进阶技巧
### 1. 遇到 dpkg: dependency problems 依赖缺失报错？
 * **原因：** 某些底层 C 库在目标机上版本过低或缺失。
 * **解决：** 在有网机器上改用 apt-rdepends 工具递归导出全量依赖树：
   ```bash
   sudo apt-install apt-rdepends
   apt-rdepends <package_name> | build-essential
   
   ```
### 2. 搭建本地 Apt 离线源/镜像源（适用于大规模节点部署）
如果集群内有几十上百台机器，逐台拷贝 .deb 包效率较低，推荐在局域网内搭建内部 HTTP 源：
```bash
# 扫描当前目录下的 deb 包生成 Packages 索引文件
dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz

```
在其他机器的 /etc/apt/sources.list 中配置该局域网内网 IP 即可实现内网快速 apt install。
## 五、 验证与交付标准
执行以下命令验证离线软件是否安装成功：
```bash
# 验证编译工具链是否就绪
gcc --version
make --version

# 验证基础网络工具是否可正常使用
ifconfig

```
 * **结果判定：** 能够正常输出版本号及网络网卡信息，即表示离线环境软件及依赖包部署成功。
```


```
