# Ansible 批量配置管理与多服务器基础部署实践

## 一、 场景说明与自动化诉求
在智算中心进行集群交付时，通常面临数十台甚至上百台 GPU 计算节点的批量初始化任务。若采用人工逐台 SSH 登录执行命令，存在重复劳动耗时严重、人工操作容易产生遗漏、集群环境基线难以统一等痛点。

通过引入基于 Agentless（无代理）架构的自动化配置工具 Ansible，仅需在一台管控跳板机上编写剧本（Playbook），即可实现对全量节点进行高效并发的基础环境加固、内核优化、目录创建及基础工具链分发。

---

## 二、 基础清单 (Inventory) 规划与免密配置

### 1. 节点主机清单配置 (hosts.ini)
在管控节点定义目标主机的分组、IP 及 SSH 端口等元数据：
```ini
[gpu_nodes]
node-01 ansible_host=192.168.10.101
node-02 ansible_host=192.168.10.102
node-03 ansible_host=192.168.10.103
node-04 ansible_host=192.168.10.104

[gpu_nodes:vars]
ansible_user=root
ansible_port=22
ansible_python_interpreter=/usr/bin/python3

2. 批量分发 SSH 公钥实现免密认证
Bash
# 生成跳板机密钥对
ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa

# 批量测试连通性
ansible gpu_nodes -i hosts.ini -m ping
三、 集群基准初始化剧本 (cluster_init.yml)
以下剧本实现了算力节点上线前的四项核心基线操作：

内核参数与网络栈优化；

禁用 Swap 分区（保障 K8s 算力调度）；

禁用 Nouveau 开源显卡驱动；

规范化创建交付与存储工作目录。

YAML
---
- name: GPU 计算节点基础环境批量初始化与加固
  hosts: gpu_nodes
  gather_facts: true
  tasks:
    - name: 1. 临时关闭 Swap 分区
      command: swapoff -a
      when: ansible_swaptotal_mb > 0

    - name: 2. 永久注释 /etc/fstab 中的 Swap 挂载项
      replace:
        path: /etc/fstab
        regexp: '^(\s*[^#\s]+\s+none\s+swap\s+.*)$'
        replace: '# \1'

    - name: 3. 优化系统内核参数与网络栈
      sysctl:
        name: "{{ item.key }}"
        value: "{{ item.value }}"
        state: present
        reload: yes
        sysctl_file: /etc/sysctl.d/99-gpu-tuning.conf
      loop:
        - { key: 'vm.max_map_count', value: '262144' }
        - { key: 'net.core.somaxconn', value: '65535' }
        - { key: 'net.ipv4.tcp_max_syn_backlog', value: '65535' }
        - { key: 'fs.file-max', value: '2097152' }

    - name: 4. 永久禁用 Nouveau 开源显卡驱动
      copy:
        dest: /etc/modprobe.d/blacklist-nouveau.conf
        content: |
          blacklist nouveau
          options nouveau modeset=0
        owner: root
        group: root
        mode: '0644'

    - name: 5. 规范化创建智算中心数据与驱动目录
      file:
        path: "{{ item }}"
        state: directory
        owner: root
        group: root
        mode: '0755'
      loop:
        - /data/packages
        - /data/logs/inspection
        - /data/docker-compose
四、 剧本语法校验与执行
1. 语法检查与模拟运行 (Dry Run)
在正式下发至生产机器前，先行执行语法校验与模拟演练，确认逻辑无误：

Bash
# 检查剧本语法
ansible-playbook -i hosts.ini cluster_init.yml --syntax-check

# 模拟试运行（不会实际修改远端配置）
ansible-playbook -i hosts.ini cluster_init.yml -C
2. 正式并发执行部署
Bash
# 执行正式部署，并发度设置为 10
ansible-playbook -i hosts.ini cluster_init.yml -f 10
五、 现场交付成效与价值
标准化防差错：杜绝了人工配置参数敲错或节点间基线不统一引发的隐蔽故障；

交付效率倍增：将数十台计算节点的系统初始化耗时从原本半天压缩到 3 分钟以内批量下发完毕；

版本受控可追溯：剧本纳入 Git 代码版本管控，实现基础设施即代码（IaC）的运维规范落地。
