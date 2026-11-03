# 华为智能计算平台中级知识点（AI 生成）

**生成日期：2025-10-09**

> 本文为 AI 生成的补充学习资料，包含命令和配置片段，不能视为官方考纲或已完整复现的实验手册。示例中的主机、设备、文件、应用及模型需自行准备；执行前核对目标版本和前提。
>
> 文字与示例核对日期：2026-10-30。这里只修改已确认的问题及必要的适用条件，不代表所有外部服务和硬件场景均经过实测。

---

## 1.模块 A — 平台与硬件基础（详尽）

### 1.1 BMC / IPMI 常用操作（示例使用 ipmitool）

- 查询 BMC 信息：

```bash
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis status
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sdr elist
```

- 远程开机 / 关机 / 重启：

```bash
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power on
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power off
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password chassis power cycle
```

- 查看远程控制台（SOL）：

```bash
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password sol activate
# 退出：~.
```

- 实务提示：BMC 密码默认需要更改，限制公网可达性；固件升级必须在维护窗口并保留回退方案。

### 1.2 RAID 与 LVM 操作（实战步骤）

- 创建软件 RAID1（仅使用已确认可清空的实验磁盘；下面的建阵列和格式化操作会改变磁盘数据）：

```bash
# 假设 /dev/sdb 和 /dev/sdc
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
mkfs.xfs /dev/md0
mkdir /data
mount /dev/md0 /data
# 写入 /etc/mdadm.conf 保存阵列信息
mdadm --detail --scan >> /etc/mdadm.conf
```

- LVM 示例（独立使用 /dev/sdd，并非接在上一例 /dev/md0 上；同样需要可清空的实验磁盘）：

```bash
pvcreate /dev/sdd
vgcreate vg_data /dev/sdd
lvcreate -n lv_data -L 100G vg_data
mkfs.xfs /dev/vg_data/lv_data
mkdir /mnt/data2
mount /dev/vg_data/lv_data /mnt/data2
```

- 扩容逻辑卷（前提：卷组仍有至少 50G 可用空间；XFS 需已挂载）：

```bash
lvextend -L +50G /dev/vg_data/lv_data
xfs_growfs /mnt/data2    # 如果是 xfs
```

### 1.3 常见硬件故障判断（示例流程）

- 现象：服务器无法开机 → 步骤：
    1. 通过 BMC 查看电源/风扇/温度告警：`ipmitool sdr elist`。
    2. 检查前面板/电源指示灯、错误 LED。
    3. 若 CPU/内存异常，做单条内存替换/插槽交换以排除。
    4. 若使用 Linux md 软件 RAID，可查看 `mdadm --detail /dev/md0`；硬件 RAID 需用对应控制器工具。确认阵列和故障盘后再按厂商流程更换，不能把 mdadm 当作所有 RAID 的通用管理器。

---

## 2.模块 B — 操作系统与系统管理（详尽）

### 2.1 日志与服务排查实战（示例）

- 查看服务日志（近 30 分钟）：

```bash
journalctl -u my-service --since "30 minutes ago" --no-pager
```

- 重启后排查上次启动的内核日志（需要日志仍被保留）：

```bash
journalctl -k -b -1 -n 200 --no-pager
# dmesg 通常只反映当前启动的内核环形缓冲区，不能替代上次启动日志。
dmesg | tail -n 200
```

- 示例：systemctl 无法启动服务

```bash
systemctl start my-service
systemctl status my-service -l
journalctl -u my-service -n 200 --no-pager
# 若权限问题，查看 /etc/systemd/system/my-service.service 的权限与 ExecStart 路径
```

### 2.2 网络排查命令与用法

- 查看 IP/路由：

```bash
ip addr show
ip route show
```

- 查看端口监听：

```bash
ss -tunlp | grep 8080
```

- 抓包定位（捕获到文件再分析）：

```bash
tcpdump -i eth0 host 10.0.0.5 and port 80 -w /tmp/capture.pcap
# 查看抓包（简略）：
tcpdump -r /tmp/capture.pcap -n -tt
```

### 2.3 安全实务（SSH 密钥与限定登陆）

- 生成密钥并部署：

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_exam
ssh-copy-id -i ~/.ssh/id_rsa_exam.pub user@target
# 先用指定的非默认文件名验证密钥登录，保留现有会话。
ssh -i ~/.ssh/id_rsa_exam user@target
```

将以下配置写入服务器的 `/etc/ssh/sshd_config` 或相应配置片段，不是在 Shell 中直接执行：

```sshconfig
PasswordAuthentication no
PermitRootLogin no
```

```bash
# 校验配置后再按目标系统服务名重新加载；部分发行版使用 ssh 而非 sshd。
sudo sshd -t
sudo systemctl reload sshd
```

---

## 3.模块 C — 虚拟化与容器（详尽）

### 3.1 Dockerfile 示例（最佳实践）

```dockerfile
# 多阶段构建示例：构建 Python 应用并只复制运行时
FROM python:3.11-slim as builder
WORKDIR /app
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN python -m compileall -b .

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY --from=builder /app /app
ENV PYTHONUNBUFFERED=1
CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8080", "--workers", "2"]
```

- **前提**：requirements.txt 包含 gunicorn 等依赖，app:app 是可用的 WSGI 应用。复制整个虚拟环境可同时保留包和 gunicorn 启动脚本；仅复制 site-packages 会漏掉控制台入口。原生扩展需要的系统动态库仍需在运行镜像安装，两个阶段应保持兼容的基础环境。

- 构建并运行：

```bash
docker build -t myapp:1.0 .
docker run -d --name myapp -p 8080:8080 myapp:1.0
```

### 3.2 Kubernetes 常用 YAML 示例（Deployment + Service）

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

- **部署前提**：集群节点可获得 myapp:1.0 镜像；仅在本机 docker build 不代表其他节点能拉取。应用需实际提供 /health 且返回成功状态，否则 readiness 检查不会通过。

- 常用命令：

```bash
kubectl apply -f myapp.yaml
kubectl get pods -o wide
kubectl logs deployment/myapp
kubectl describe pod <pod-name>
```

### 3.3 排查常见 Pod 问题

- CrashLoopBackOff 排查流程：
    1. 查看日志：`kubectl logs <pod> --previous`
    2. 查看事件：`kubectl describe pod <pod>`
    3. 检查 liveness／startup 是否连续失败而触发重启；readiness 失败本身只影响就绪状态及服务流量，不会触发容器重启。
    4. `ImagePullBackOff` 是另一类镜像拉取失败状态，不等同于 CrashLoopBackOff：检查镜像名称/标签、私有 registry 凭证（imagePullSecrets）。

---

## 4.模块 D — 数据预处理与工程化（详尽）

### 4.1 Pandas 读大文件（分块）示例

```python
import pandas as pd

chunks = pd.read_csv('large.csv', chunksize=50000)
for i, df in enumerate(chunks):
    # 简单清洗
    df = df.drop_duplicates()  # 仅在当前分块内去重，不保证跨分块全局去重
    df['col'] = df['col'].fillna('unknown')
    # 处理后写入分片文件或数据库
    df.to_parquet(f'out_part_{i}.parquet', index=False)
```

写 Parquet 还需要兼容的 pyarrow／fastparquet 引擎。若需全局去重，应另行维护跨块状态或交给数据库／分布式处理实现，不能把分块内去重当作全量去重。

### 4.2 缺失值处理范例（实务步骤）

- 检查缺失值：

```python
df.isnull().sum()
```

- 简单策略：
    - 少量缺失 → 删除（`df.dropna()`）
    - 数值列 → 使用均值/中位数填充（`df['x'].fillna(df['x'].median())`）
    - 类别列 → 使用 `mode` 或新增类别 `unknown`

用于建模时，填充值等预处理参数应仅由训练集确定，再用于验证／测试集，避免数据泄漏；示例 col 列也需具有与字符串填充值兼容的类型。

---

## 5.模块 E — 模型训练与部署（详尽）

### 5.1 PyTorch 保存与导出到 ONNX（示例）

以下保留 **legacy 导出器**示例，适用于提供 dynamo 参数的 PyTorch 版本（如 2.6 及后续仍支持 legacy 路径的版本）。PyTorch 2.9 起默认 dynamo=True，不能混淆新旧导出器；新导出器按官方说明选取 opset 和依赖，不把 opset 12 当作任意模型的通用设置。

```python
# model: 已训练好的 PyTorch 模型
import torch
model = model.cpu().eval()  # 示例按 CPU 和推理模式导出。
dummy_input = torch.randn(1, 3, 224, 224)
torch.onnx.export(
    model, dummy_input, "model.onnx",
    input_names=['input'], output_names=['output'],
    opset_version=12, dynamo=False,
)
# 更早且不接受 dynamo 参数的版本需删除该参数，并按其文档核对。
```

- 验证 ONNX 模型（使用 onnxruntime）：

```python
import onnxruntime as ort
sess = ort.InferenceSession("model.onnx")
res = sess.run(None, {"input": dummy_input.numpy()})
```

### 5.2 TensorFlow SavedModel 导出（示例）

```python
# Keras 3，使用 TensorFlow 后端并且 model 已构建。
model.export("saved_model_dir")
# 保存 Keras 原生可训练模型则用 model.save("model.keras")。
```

旧版 TensorFlow 后端 Keras 2 中，以下写法可导出 SavedModel，作为历史用法保留；不能原样套用于 Keras 3：

```python
# Keras 2 的历史用法
model.save("saved_model_dir")
```

### 5.3 简单推理服务（FastAPI + onnxruntime 示例）

```python
from fastapi import FastAPI
import onnxruntime as ort
import numpy as np

app = FastAPI()
sess = ort.InferenceSession("model.onnx")

@app.get("/health")
def health():
    return {"status": "ok"}

@app.post("/infer")
def infer(arr: list):
    x = np.array(arr).astype(np.float32)
    out = sess.run(None, {'input': x})
    return {'result': out[0].tolist()}
```

此 FastAPI 示例使用 ASGI 服务，例如 `uvicorn app:app --host 0.0.0.0 --port 8080`（需安装 uvicorn），不能直接套用前面 WSGI 的 Gunicorn 默认入口。输入数组的形状、数据类型和模型路径需与导出模型一致；当前接口是教学片段，不含完整输入校验。

### 5.4 性能优化（实操建议）

- 使用批推理：在客户端合并请求，减少上下文切换。
- 使用 ONNX Runtime + TensorRT/FP16：在 GPU 上加速。
- 转换与量化需区分：FP16 转换不等同于 INT8 动态量化。下面第一条 CLI 针对适配的 Transformer 模型及工具版本，不是所有 ONNX 模型的通用优化命令；性能和精度都需对照评估。

- Transformer 优化／FP16 转换示例：

```bash
python -m onnxruntime.transformers.optimizer --input model.onnx --output model_opt.onnx --float16
```

- INT8 动态量化示例（只对支持的模型／算子和执行提供程序适用）：

```python
from onnxruntime.quantization import quantize_dynamic, QuantType
quantize_dynamic("model.onnx", "model.quant.onnx", weight_type=QuantType.QInt8)
```

---

## 6.模块 F — 监控、故障恢复（详尽）

### 6.1 Prometheus 抓取简单指标（Node exporter + Prometheus scrape config）

- prometheus.yml 示例：

```yaml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['node1:9100','node2:9100']
```

- 常用 PromQL 示例：

```promql
# 最近 5 分钟 CPU 非空闲时间比例，单位为 0—1；不是系统 load average。
1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))
# 按普通用户可用空间计算的文件系统使用比例，高于 85% 的序列。
(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) > 0.85
```

若需给表达式命名并持久记录，应在规则 YAML 中分别配置 record 和 expr，不使用 `指标名 = 表达式` 的查询赋值写法。文件系统查询还应按环境筛选挂载点，排除伪文件系统和容量为零的项。

### 6.2 备份示例（rsync 与数据库）

- 文件镜像同步（rsync 增量传输）：

```bash
rsync -avz --delete /data/ backup.example.com:/backups/data/
# --delete 会删除目标目录中源端已不存在的项；可先加 --dry-run 预览。
```

该示例是镜像同步，不能单凭它保留误删前的历史版本。需要历史恢复点时，应另配快照／版本目录及保留策略，并核验恢复过程。

- MySQL 备份（逻辑备份）：

```bash
mysqldump -u root -p --single-transaction --routines --events --databases mydb > mydb.sql
# 一致性主要针对 InnoDB 等事务表；不保证 MyISAM 等非事务表一致性。
# 导出期间还应避免 ALTER TABLE、DROP TABLE 等会影响一致性或读取的 DDL。
```

---

## 7.模块 G — CI/CD 与自动化（详尽）

### 7.1 GitHub Actions 简单工作流（Docker 构建并推送）

```yaml
name: CI
on:
  push:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v6
      - name: Set image name
        run: echo "IMAGE_NAME=ghcr.io/${GITHUB_REPOSITORY_OWNER,,}/myapp:${GITHUB_SHA}" >> "$GITHUB_ENV"
      - name: Build image
        run: docker build -t "$IMAGE_NAME" .
      - name: Login to registry
        run: printf '%s' "$GHCR_TOKEN" | docker login ghcr.io -u "$GITHUB_ACTOR" --password-stdin
        env:
          GHCR_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: Push
        run: docker push "$IMAGE_NAME"
```

示例使用 GitHub 托管的当前 Ubuntu runner；自托管 runner 需满足 checkout v6 的版本要求。GHCR 还受组织及已有包的访问设置约束；镜像命名空间需使用合法的小写名称，生产使用应核对并锁定动作提交版本。

### 7.2 Ansible 批量替换配置示例

```yaml
- hosts: webservers
  become: true  # 修改 /etc 及重启系统服务需相应提权配置。
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

## 8.模块 H — 安全与合规（详尽）

### 8.1 镜像安全扫描（示例工具 Trivy）

```bash
trivy image --severity HIGH,CRITICAL ghcr.io/myorg/myapp:latest
```

- 报告结果后需要做依赖升级或替换不安全组件。

### 8.2 数据脱敏基本操作（示例）

- 删除或哈希化敏感列（哈希不自动等于匿名化，仍需评估关联和字典匹配风险）：

```python
# pandas 示例；df 已存在。
import pandas as pd
import hashlib
df['email_hash'] = df['email'].map(
    lambda x: hashlib.sha256(str(x).encode('utf-8')).hexdigest()
    if pd.notna(x) else None
)
df = df.drop(columns=['email'])
```

---

## 9.常见考题应答模板（实务中速用）

- **故障排查题**：先写“现象（What）→ 影响范围（Who/How many）→ 立即缓解（Stop-gap）→ 根因排查步骤（Logs/资源/配置）→ 解决方案与预防措施”。
- **配置题**：给出步骤命令 + 验证命令 + 回滚命令（例如修改 systemd 单元则说明 `systemctl daemon-reload` 与 `systemctl restart`，并给出回滚配置文件的路径与恢复命令）。

---

## 10.练习任务（跟着做）

1. 使用两块空盘创建 RAID1 -> 在上面创建 LVM -> 挂载并扩容。
2. 编写 Dockerfile，构建镜像并推送到本地 Registry。
3. 在 minikube 上部署上面给出的 Deployment，模拟 Pod CrashLoopBackOff（写一个启动脚本退出），并完成排查与修复。
4. 训练一个小模型，导出为 ONNX，写一个 FastAPI 推理接口并压测（ab 或 wrk）。
5. 配置 Prometheus 抓取 Node Exporter，搭建 Grafana 面板显示 CPU/内存/磁盘和自定义应用指标。

---

### 10.1 练习验收与课程回查

> 以下是对现有练习的验收补充，属于学习设计；没有实际运行记录时，不能写成“部署成功”或“验证通过”。

| 练习 | 最少需要保留的结果 | 回查笔记 |
| --- | --- | --- |
| 环境与容器 | 依赖版本、实际启动入口、导入／启动结果 | [平台搭建](./1.人工智能与平台搭建.md)、[平台管理](./2.平台管理.md) |
| 数据采集与处理 | 批次／位点、重放结果、输入输出差异解释 | [数据采集](./3.数据采集.md)、[数据处理](./5.数据处理.md) |
| 存储与恢复 | 恢复点、所需备份链、应用可读性及实测时间 | [数据存储](./4.数据存储.md)、[备份恢复](./6.数据备份与恢复.md) |
| 模型导出与接口 | 固定输入的导出前后差异、异常输入及性能记录 | [算法建模](./7.机器学习基础算法建模.md)、[模型测试](./8.人工智能开发模型测试.md) |

一条可复习的完整路线是：**确认环境 → 固定数据与规则 → 训练基线 → 独立评估 → 导出与接口核验 → 留存版本和恢复记录**。实际部署仍需按所选产品和版本调整；这些步骤不替代前文具体命令的前提说明。

## 11.核对资料

- [Docker 多阶段构建](https://docs.docker.com/build/building/multi-stage/)、[Kubernetes 探针](https://kubernetes.io/docs/concepts/workloads/pods/probes/) —— 运行依赖和探针行为。
- [Keras 3 迁移](https://keras.io/guides/migrating_to_keras_3/)、[PyTorch ONNX 导出](https://docs.pytorch.org/docs/stable/onnx) —— 新旧导出接口。
- [ONNX Runtime 量化](https://onnxruntime.ai/docs/performance/model-optimizations/quantization.html)、[Transformer 优化器](https://onnxruntime.ai/docs/performance/transformers-optimization.html) —— 量化方式、模型和算子适用范围。
- [Prometheus 规则](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/)、[Node 指标查询示例](https://grafana.com/docs/grafana-cloud/platform/knowledge-graph/advanced-configuration/enable-prom-metrics-collection/infrastructure/node/) —— 表达式和指标含义。
- [rsync 手册](https://www.samba.org/ftp/rsync/rsync.1)、[MySQL mysqldump](https://dev.mysql.com/doc/refman/8.4/en/mysqldump.html)、[journalctl](https://www.freedesktop.org/software/systemd/man/latest/journalctl.html) —— 同步、备份一致性与日志范围。
- [GitHub 镜像发布](https://docs.github.com/en/actions/tutorials/publish-packages/publish-docker-images)、[checkout 说明](https://github.com/actions/checkout)、[pandas Parquet](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to_parquet.html) —— 权限、runner 兼容及数据引擎依赖。

## 12.相关笔记

- [MOC-智能计算平台应用开发](./MOC.md) —— 返回方向索引，查看学习路线与主题分组。
- [1.人工智能与平台搭建](./1.人工智能与平台搭建.md) —— 基础：人工智能概念、开发环境与框架。
- [2.平台管理](./2.平台管理.md) —— 基础：集群、资源与系统管理。
- [3.数据采集](./3.数据采集.md) —— 数据：采集方式、工具及流程优化。
- [4.数据存储](./4.数据存储.md) —— 数据：文件系统、数据库与复制策略。
- [5.数据处理](./5.数据处理.md) —— 数据：ETL、Kettle 实践与标注分类。
- [6.数据备份与恢复](./6.数据备份与恢复.md) —— 运维：备份类型、快照、重删与恢复。
- [7.机器学习基础算法建模](./7.机器学习基础算法建模.md) —— 模型：机器学习任务与基础算法。
- [8.人工智能开发模型测试](./8.人工智能开发模型测试.md) —— 模型：开发生命周期、数据检查与测试阶段。
