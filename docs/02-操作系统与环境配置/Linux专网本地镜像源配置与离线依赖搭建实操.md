# Linux 专网本地镜像源配置与离线依赖搭建实操

## 一、 场景说明与背景
在大型 AI 智算中心交付与专网保密机房中，为了保障数据安全与合规，所有计算节点、存储节点均处于严格的无外网（Air-Gapped）物理隔离状态。而在 GPU 节点部署底层驱动、CUDA 环境、编译工具链及监控组件时，存在大量 deb 包依赖。

通过在局域网跳板机或本地节点构建本地 APT 镜像源，能够实现：
1. 完全离线交付：在无外网环境下无缝使用 apt-get install 自动解析并安装所有依赖包；
2. 版本一致性控制：锁定内核头文件、编译组件版本，避免不同批次机器依赖库不一致引发驱动冲突；
3. 极速内网分发：利用内网千兆/万兆带宽拉取安装包，大幅缩短集群节点初始化耗时。

---

## 二、 离线软件包归类与准备

### 1. 获取全量离线依赖包
在具备外网环境的同构系统测试机上，通过 apt-get download 递归抓取目标软件及其底层依赖。

例如，获取编译环境核心组件：
```bash
# 创建依赖暂存目录
mkdir -p /opt/offline-packages
cd /opt/offline-packages

# 下载目标工具及其完整依赖包（以 build-essential、dkms 为例）
apt-get download $(apt-cache depends --recurse --no-recommends --no-suggests \
  --no-conflicts --no-breaks --no-replaces --no-enhances \
  build-essential dkms | grep "^\w" | sort -u)

```

### 2. 介质灌装并传输至专网服务器

将提取到的全量 .deb 文件打包传入专网服务器，统一存放在专有路径下：

```bash
# 创建本地源主工作目录
sudo mkdir -p /var/local-repo/debs

# 解压或拷贝所有 deb 包至该目录
sudo cp /mnt/usb/packages/*.deb /var/local-repo/debs/

```

---

## 三、 本地 APT 镜像源搭建与索引生成

### 1. 生成 Packages 索引文件

APT 包管理器依赖 Packages.gz 文件来识别软件包之间的依赖拓扑。使用 dpkg-scanpackages 工具（由 dpkg-dev 软件包提供）对包目录进行全量扫描与索引构建：

```bash
# 切换至本地仓库根目录
cd /var/local-repo

# 扫描 debs 目录并生成 gzip 压缩的 Packages 索引
dpkg-scanpackages debs /dev/null | gzip -9c > debs/Packages.gz

```

> 注意：若后续向该目录新增、删减任何 .deb 包，必须重新执行上述扫描命令以刷新索引拓扑。

---

## 四、 客户端 APT 源配置与生效验证

### 1. 备份系统原始软件源

为了避免离线状态下 apt 尝试连接公网源导致解析超时报错，需对默认配置进行归档隔离：

```bash
# 备份并清空原有 sources.list
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
sudo touch /etc/apt/sources.list

```

### 2. 配置本地 file 协议软件源

在 /etc/apt/sources.list.d/ 目录下新增专属的本地源配置文件：

```bash
sudo tee /etc/apt/sources.list.d/local-repo.list <<EOF # ## * --- -9c -y /dev/null /var/local-repo 1. 2. APT EOF GPG HTTP Nginx/Apache Packages Release Sum [trusted="yes]" `[trusted="yes]`" ``` ```bash `debs/`：必须以斜杠结尾，对应存放 `file:/var/local-repo/`：采用标准本地文件路径协议（若作为集群共享源，可结合 apt-get build-essential cd clean deb debs debs/ dpkg-scanpackages file:/var/local-repo/ gcc gzip install is make mismatch` not repository signed`： sudo update | 五、 六、 刷新缓存与实装验证 包但未重新扫描，或者客户端读取了损坏的本地索引缓存。 参数说明： 字段。 或无法找到最新添加的包： 报错：`Hash 报错：`The 排障方案： 排障方案：在源配置中补充 数字签名验证，规避自建本地源因无官方数字证书签名导致的签名校验阻断错误； 文件签名。 暴露为 根因分析：新增了 根因分析：未配置官方 测试安装编译工具链，验证依赖自动解析 清空原有 源）； 现场交付常见报错与排障 索引的相对子目录。 缓存 证书公钥，APT 重新加载本地索引 默认强制校验> debs/Packages.gz
     sudo rm -rf /var/lib/apt/lists/*
     sudo apt-get clean
     sudo apt-get update
     ```
3. 报错：`dpkg-scanpackages: command not found`：
   * 根因分析：极简安装的系统未预装 dpkg-dev 实用程序套件。
   * 排障方案：在有网机器提取 dpkg-dev、libdpkg-perl 相关 deb 包，通过 `sudo dpkg -i *.deb` 手工底层安装该工具后重新构建索引。
