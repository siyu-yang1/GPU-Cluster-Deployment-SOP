# Docker 容器调用 GPU 环境配置指南 (NVIDIA Container Toolkit)

## 一、 场景说明
在智算中心生产环境中，AI 模型训练与推理任务均以容器化（Docker / K8s）方式运行。默认情况下，Docker 容器无法直接访问宿主机的物理 GPU。必须通过安装并配置 NVIDIA Container Toolkit (nvidia-docker2)，将物理 GPU 设备与驱动库挂载进容器命名空间。

---

## 二、 前置依赖检查
在配置前，必须确保宿主机已满足以下条件：
1. NVIDIA 官方物理显卡驱动已正常安装并成功运行（`nvidia-smi` 正常显示）；
2. 宿主机 Docker 引擎已正常拉起（`docker --version`）。

---

## 三、 NVIDIA Container Toolkit 离线安装与配置

### 1. 软件源与依赖包安装
在专网离线环境中，提前在同构联网机下载以下组件的 deb 安装包，传输至宿主机并安装：
* `libnvidia-container1`
* `libnvidia-container-tools`
* `nvidia-container-toolkit-base`
* `nvidia-container-toolkit`

```bash
# 离线环境批量安装底层运行时工具
sudo dpkg -i libnvidia-container*.deb nvidia-container-toolkit*.deb

```

### 2. 配置 Docker 默认运行时引擎

将 NVIDIA 运行时注册至 Docker 守护进程配置文件 `/etc/docker/daemon.json`：

```bash
# 自动配置 Docker 守护进程
sudo nvidia-ctk runtime configure --runtime=docker

# 重启 Docker 服务生效
sudo systemctl restart docker

```

生成的 `/etc/docker/daemon.json` 核心配置如下：

```json
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}

```

---

## 四、 容器调用 GPU 验证测试

### 1. 单卡与全卡挂载启动测试

```bash
# 测试 1：调用宿主机所有可用 GPU 并运行 nvidia-smi 验证
docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi

# 测试 2：指定分配物理 GPU 0 和 GPU 1 给容器
docker run --rm --gpus '"device=0,1"' nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi

```

### 2. 在 Docker Compose 中配置 GPU 资源分配

在微服务编排交付中，标准的 `docker-compose.yml` GPU 分配规范：

```yaml
version: '3.8'

services:
  ai-inference-service:
    image: ai-inference:latest
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all          # 挂载所有 GPU，或指定具体数量如 1、2
              capabilities: [gpu]

```

---

## 五、 现场常见报错与排障

1. **报错：`docker: Error response from daemon: could not select device driver "" with capabilities: [[gpu]]**`：
* **根因**：Docker 未能识别 nvidia runtime，或配置后未重启 Docker 服务。
* **排障**：检查 `/etc/docker/daemon.json` 格式是否正确，并执行 `sudo systemctl daemon-reload && sudo systemctl restart docker`。


2. **报错：`nvidia-container-cli: initialization error: nvml error: driver/library version mismatch**`：
* **根因**：系统后台静默升级了内核或显卡驱动模块，导致正在运行的内核驱动与宿主机 NVML 动态链接库版本不一致。
* **排障**：无需重装，直接对宿主机执行冷重启 `sudo reboot` 重新加载一致的驱动模块即可。
```
