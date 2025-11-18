# 华为等级认证 — 智能计算平台应用开发（中级）详尽知识点
**生成日期：2025-10-09**  
**作者：ChatGPT（GPT-5 Thinking mini）**

> 本文档将每一项知识点展开成可执行的步骤、命令示例和常见配置片段，便于你实操演练与记忆。读到哪项就可以在本地 VM 或云主机上跟着做一遍。
>

---

## 模块 A — 平台与硬件基础（详尽）
### 1. BMC / IPMI 常用操作（示例使用 ipmitool）
+ 查询 BMC 信息：

```plain
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis status
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sdr elist
```

+ 远程开机 / 关机 / 重启：

```plain
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power on
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power off
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power cycle
```

+ 查看远程控制台（SOL）：

```plain
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sol activate
# 退出：~.
```

+ 实务提示：BMC 密码默认需要更改，限制公网可达性；固件升级必须在维护窗口并保留回退方案。

### 2. RAID 与 LVM 操作（实战步骤）
+ 创建软件 RAID1：

```plain
# 假设 /dev/sdb 和 /dev/sdc
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
mkfs.xfs /dev/md0
mkdir /data
mount /dev/md0 /data
# 写入 /etc/mdadm.conf 保存阵列信息
mdadm --detail --scan >> /etc/mdadm.conf
```

+ LVM 示例（在物理卷上创建逻辑卷）：

```plain
pvcreate /dev/sdd
vgcreate vg_data /dev/sdd
lvcreate -n lv_data -L 100G vg_data
mkfs.xfs /dev/vg_data/lv_data
mkdir /mnt/data2
mount /dev/vg_data/lv_data /mnt/data2
```

+ 扩容逻辑卷：

```plain
lvextend -L +50G /dev/vg_data/lv_data
xfs_growfs /mnt/data2    # 如果是 xfs
```

### 3. 常见硬件故障判断（示例流程）
+ 现象：服务器无法开机 → 步骤：
    1. 通过 BMC 查看电源/风扇/温度告警：`ipmitool sdr elist`。
    2. 检查前面板/电源指示灯、错误 LED。
    3. 若 CPU/内存异常，做单条内存替换/插槽交换以排除。
    4. 如果是 RAID 报错，查看 `mdadm --detail /dev/md0` 并替换故障盘。

---

## 模块 B — 操作系统与系统管理（详尽）
### 1. 日志与服务排查实战（示例）
+ 查看服务日志（近 30 分钟）：

```plain
journalctl -u my-service --since "30 minutes ago" --no-pager
```

+ 系统崩溃后查看内核日志：

```plain
journalctl -k | tail -n 200
dmesg | tail -n 200
```

+ 示例：systemctl 无法启动服务

```plain
systemctl start my-service
systemctl status my-service -l
journalctl -u my-service -n 200 --no-pager
# 若权限问题，查看 /etc/systemd/system/my-service.service 的权限与 ExecStart 路径
```

### 2. 网络排查命令与用法
+ 查看 IP/路由：

```plain
ip addr show
ip route show
```

+ 查看端口监听：

```plain
ss -tunlp | grep 8080
```

+ 抓包定位（捕获到文件再分析）：

```plain
tcpdump -i eth0 host 10.0.0.5 and port 80 -w /tmp/capture.pcap
# 查看抓包（简略）：
tcpdump -r /tmp/capture.pcap -n -tt
```

### 3. 安全实务（SSH 密钥与限定登陆）
+ 生成密钥并部署：

```plain
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_exam
ssh-copy-id -i ~/.ssh/id_rsa_exam.pub user@target
# 禁止密码登录（/etc/ssh/sshd_config）
PasswordAuthentication no
PermitRootLogin no
# 重启 sshd
systemctl restart sshd
```

---

## 模块 C — 虚拟化与容器（详尽）
### 1. Dockerfile 示例（最佳实践）
```dockerfile
# 多阶段构建示例：构建 Python 应用并只复制运行时
FROM python:3.11-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN python -m compileall -b .

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /app /app
ENV PYTHONUNBUFFERED=1
CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8080", "--workers", "2"]
```

+ 构建并运行：

```plain
docker build -t myapp:1.0 .
docker run -d --name myapp -p 8080:8080 myapp:1.0
```

### 2. Kubernetes 常用 YAML 示例（Deployment + Service）
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

+ 常用命令：

```plain
kubectl apply -f myapp.yaml
kubectl get pods -o wide
kubectl logs deployment/myapp
kubectl describe pod <pod-name>
```

### 3. 排查常见 Pod 问题
+ CrashLoopBackOff 排查流程：
    1. 查看日志：`kubectl logs <pod> --previous`
    2. 查看事件：`kubectl describe pod <pod>`
    3. 检查探针（readiness/liveness）是否配置过严导致重启。
    4. 若是镜像问题，`ImagePullBackOff`：检查镜像名称/标签、私有 registry 凭证（imagePullSecrets）。

---

## 模块 D — 数据预处理与工程化（详尽）
### 1. Pandas 读大文件（分块）示例
```python
import pandas as pd

chunks = pd.read_csv('large.csv', chunksize=50000)
for i, df in enumerate(chunks):
    # 简单清洗
    df = df.drop_duplicates()
    df['col'] = df['col'].fillna('unknown')
    # 处理后写入分片文件或数据库
    df.to_parquet(f'out_part_{i}.parquet')
```

### 2. 缺失值处理范例（实务步骤）
+ 检查缺失值：

```plain
df.isnull().sum()
```

+ 简单策略：
    - 少量缺失 → 删除（`df.dropna()`）
    - 数值列 → 使用均值/中位数填充（`df['x'].fillna(df['x'].median())`）
    - 类别列 → 使用 `mode` 或新增类别 `unknown`

---

## 模块 E — 模型训练与部署（详尽）
### 1. PyTorch 保存与导出到 ONNX（示例）
```python
# model: 已训练好的 PyTorch 模型
import torch
dummy_input = torch.randn(1, 3, 224, 224)
torch.onnx.export(model, dummy_input, "model.onnx", input_names=['input'], output_names=['output'], opset_version=12)
```

+ 验证 ONNX 模型（使用 onnxruntime）：

```python
import onnxruntime as ort
sess = ort.InferenceSession("model.onnx")
res = sess.run(None, {"input": dummy_input.numpy()})
```

### 2. TensorFlow SavedModel 导出（示例）
```python
# Keras model
model.save("saved_model_dir")
# 使用 TensorFlow Serving 或将其转换为其他格式
```

### 3. 简单推理服务（FastAPI + onnxruntime 示例）
```python
from fastapi import FastAPI
import onnxruntime as ort
import numpy as np

app = FastAPI()
sess = ort.InferenceSession("model.onnx")

@app.post("/infer")
def infer(arr: list):
    x = np.array(arr).astype(np.float32)
    out = sess.run(None, {'input': x})
    return {'result': out[0].tolist()}
```

### 4. 性能优化（实操建议）
+ 使用批推理：在客户端合并请求，减少上下文切换。
+ 使用 ONNX Runtime + TensorRT/FP16：在 GPU 上加速。
+ 量化示例（ONNX Runtime quantization 工具）：

```plain
python -m onnxruntime.tools.transformers.optimizer_cli --input model.onnx --output model_opt.onnx --precision fp16
# 或者使用 quantize_dynamic
from onnxruntime.quantization import quantize_dynamic, QuantType
quantize_dynamic("model.onnx", "model.quant.onnx", weight_type=QuantType.QInt8)
```

---

## 模块 F — 监控、故障恢复（详尽）
### 1. Prometheus 抓取简单指标（Node exporter + Prometheus scrape config）
+ prometheus.yml 示例：

```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node1:9100','node2:9100']
```

+ 常用 PromQL 示例：

```plain
# 最近 5 分钟 CPU 负载平均
avg(rate(node_cpu_seconds_total{mode!="idle"}[5m])) by (instance)
# 磁盘使用率高于 85%
node_filesystem_usage = (node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes
```

### 2. 备份示例（rsync 与数据库）
+ 文件备份（增量 rsync）：

```plain
rsync -avz --delete /data/ backup.example.com:/backups/data/
```

+ MySQL 备份（逻辑备份）：

```plain
mysqldump -u root -p --single-transaction --routines --events --databases mydb > mydb.sql
```

---

## 模块 G — CI/CD 与自动化（详尽）
### 1. GitHub Actions 简单工作流（Docker 构建并推送）
```yaml
name: CI
on:
  push:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build image
        run: docker build -t ghcr.io/${{ github.repository_owner }}/myapp:${{ github.sha }} .
      - name: Login to registry
        run: echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Push
        run: docker push ghcr.io/${{ github.repository_owner }}/myapp:${{ github.sha }}
```

### 2. Ansible 批量替换配置示例
```yaml
- hosts: webservers
  tasks:
    - name: Replace config
      copy:
        src: ./templates/myapp.conf
        dest: /etc/myapp/myapp.conf
        owner: root
        group: root
        mode: '0644'
      notify:
        - restart myapp

  handlers:
    - name: restart myapp
      systemd:
        name: myapp
        state: restarted
```

---

## 模块 H — 安全与合规（详尽）
### 1. 镜像安全扫描（示例工具 Trivy）
```plain
trivy image --severity HIGH,CRITICAL ghcr.io/myorg/myapp:latest
```

+ 报告结果后需要做依赖升级或替换不安全组件。

### 2. 数据脱敏基本操作（示例）
+ 删除或哈希化敏感列：

```python
# pandas 示例
df['email_hash'] = df['email'].apply(lambda x: hashlib.sha256(x.encode()).hexdigest())
df = df.drop(columns=['email'])
```

---

## 常见考题应答模板（实务中速用）
+ **故障排查题**：先写“现象（What）→ 影响范围（Who/How many）→ 立即缓解（Stop-gap）→ 根因排查步骤（Logs/资源/配置）→ 解决方案与预防措施”。
+ **配置题**：给出步骤命令 + 验证命令 + 回滚命令（例如修改 systemd 单元则说明 `systemctl daemon-reload` 与 `systemctl restart`，并给出回滚配置文件的路径与恢复命令）。

---

## 练习任务（跟着做）
1. 使用两块空盘创建 RAID1 -> 在上面创建 LVM -> 挂载并扩容。  
2. 编写 Dockerfile，构建镜像并推送到本地 Registry。  
3. 在 minikube 上部署上面给出的 Deployment，模拟 Pod CrashLoopBackOff（写一个启动脚本退出），并完成排查与修复。  
4. 训练一个小模型，导出为 ONNX，写一个 FastAPI 推理接口并压测（ab 或 wrk）。  
5. 配置 Prometheus 抓取 Node Exporter，搭建 Grafana 面板显示 CPU/内存/磁盘和自定义应用指标。

---



