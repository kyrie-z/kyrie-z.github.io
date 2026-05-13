---
layout: post
title: "Cloudflare 代理节点搭建"
date: 2026-05-11
categories: [Linux, 网络]
tags: [Cloudflare, VPN]
nav_category: 网络
nav_order: 1
---

> 使用 Cloudflare Pages 部署 `edgetunnel` 搭建代理节点的方案，是近年来非常流行的一种 **Serverless（无服务器）** 翻墙技术。
>
> **核心思想**：利用 Cloudflare 遍布全球的边缘计算能力，将代理服务端代码运行在离你最近的 CDN 节点上。

---

## 一、名词解释

### 1.1 Cloudflare（基础设施：全球分发网络）

> 官网：[https://dash.cloudflare.com/](https://dash.cloudflare.com/)

Cloudflare (CF) 是全球最大的 CDN（内容分发网络）和网络安全公司之一。选择它搭建代理有以下三个关键特点：

| 特性 | 说明 |
|------|------|
| **Anycast 任播网络** | CF 在全球拥有数百个数据中心。用户访问时，请求自动路由到物理距离最近的节点 |
| **边缘计算 (Workers/Pages)** | 传统 CDN 只分发静态文件，而 CF 允许在边缘节点直接运行代码（JavaScript/WebAssembly），即本次利用的 `Workers` 和 `Pages` 功能 |
| **反向代理护盾** | 对后端服务器而言，CF 是反向代理。用户只能看到 CF 的 IP，无法获取真实服务器 IP |

### 1.2 Edgetunnel（程序逻辑：协议转换网关）

> GitHub：[https://github.com/cmliu/edgetunnel](https://github.com/cmliu/edgetunnel)

`edgetunnel` 是运行在 Cloudflare 边缘环境（Workers 或 Pages）中的**脚本代码**。

| 特性 | 说明 |
|------|------|
| **逻辑核心** | 利用 Node.js 的流处理（Stream）或 `fetch` API，解析 **VLESS** 协议 |
| **白嫖服务器** | 通常代理需要一台 VPS，但 `edgetunnel` 让 Cloudflare 边缘节点充当服务器 |
| **协议转换** | 在边缘节点监听 WebSocket 连接，拆解代理请求并重新发起请求 |

---

## 二、工作原理

本架构可拆解为三个核心原理：

### 2.1 运行载体：边缘计算

传统 VPN 需要购买拥有公网 IP 的 VPS 服务器，而 Cloudflare Pages 提供了 **Edge Computing（边缘计算）** 能力：

- 代码不运行在固定服务器上，而是分发到全球数百个数据中心
- `edgetunnel` 本质是用 JavaScript（或 WebAssembly）编写的反向代理程序
- Cloudflare 的 V8 引擎提供运行环境

### 2.2 传输协议：VLESS + WebSocket (WSS)

Cloudflare CDN 节点默认只处理标准 HTTP/HTTPS 流量，代理数据需进行伪装封装：

| 协议 | 作用 |
|------|------|
| **WebSocket (WS)** | CF 支持将 HTTP 连接升级为 WebSocket 长连接。代理流量包裹在 WS 数据帧中，表面看像在实时通信 |
| **VLESS 协议** | Xray-core 提出的轻量级代理协议。`edgetunnel` 解析 VLESS，验证 UUID（密码）并提取目标地址 |

### 2.3 网络拓扑与流量转发

这是最关键的"偷天换日"过程。当你访问 YouTube 时，流量路径如下：

```
┌─────────────────────────────────────────────────────────────────────────┐
│  用户设备 (Client)                                                       │
│  ↓ 封装: TLS [ WS [ VLESS [ Data ] ] ]                                  │
├─────────────────────────────────────────────────────────────────────────┤
│  防火墙 (GFW)                                                            │
│  ↓ 检测 SNI 和协议特征 → 识别为标准 HTTPS 访问，放行                      │
├─────────────────────────────────────────────────────────────────────────┤
│  CF Pages (边缘节点)                                                     │
│  ↓ 1. TLS 解密  2. 建立 WebSocket 连接  3. Edgetunnel 校验 VLESS UUID    │
├─────────────────────────────────────────────────────────────────────────┤
│  目标网站 (Target)                                                       │
│  ← 以机房身份发起 Raw TCP/HTTP 请求                                      │
│  → 返回原始网页数据                                                       │
├─────────────────────────────────────────────────────────────────────────┤
│  CF Pages (边缘节点)                                                     │
│  ↓ 重新进行 VLESS + WS + TLS 封装                                        │
├─────────────────────────────────────────────────────────────────────────┤
│  用户设备 (Client)                                                       │
│  ← 通过建立好的隧道回传数据 → 客户端解包后呈现内容                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 三、方案优缺点

### ✅ 优势

| 优势 | 说明 |
|------|------|
| **完全免费** | 只要流量不极其夸张，Cloudflare Pages 免费额度（每天 10 万次请求）对个人绝对够用 |
| **抗封锁能力强** | Cloudflare IP 数量庞大且属于商业 CDN IP，GFW 难以全面封杀（易误杀正常外贸业务） |
| **免维护服务器** | 无需操心 Linux 系统安全更新、DDoS 防护等运维问题 |

### ⚠️ 劣势/风险

| 劣势 | 说明 |
|------|------|
| **速度受限与优选 IP 依赖** | 如不通过脚本寻找并绑定"优选 IP"，国内直连 Cloudflare 节点延迟往往很高 |
| **Cloudflare 封号风险** | `edgetunnel` 本质是将 CF 的 Web 服务滥用成大流量隧道代理，违反 ToS。单账户月流量达几百 GB ~ TB 级别极易被封禁 |

---

## 四、部署步骤

### 4.1 准备工作

| 项目 | 链接 |
|------|------|
| Cloudflare 平台 | [https://dash.cloudflare.com/](https://dash.cloudflare.com/) |
| edgetunnel 源码 | [点击下载 ZIP](https://github.com/cmliu/edgetunnel/archive/refs/heads/main.zip) |

### 4.2 必选步骤

#### Step 1：创建 KV 存储空间

进入 Cloudflare 控制台：

```
左侧栏 → 存储和数据库 → Workers KV → Create Instance
```

填写实例名称（如 `helloworld-kv`），然后创建。

> 💡 **KV 是什么**：存储数据库，用于存放 `edgetunnel` 运行所需的配置数据。

---

#### Step 2：部署 Pages 脚本

```
左侧栏 → 计算 → Workers 和 Pages → 创建应用程序
→ 选择 "Pages" → 拖放文件上传
→ 项目名（如 helloworld）
→ 上传提前下载的 edgetunnel 压缩包
→ 部署站点
```

---

#### Step 3：设置管理员密码

部署完成后：

```
点击 "继续处理站点" → 设置 → 环境变量
→ "制作" 为生产环境定义变量 → 添加变量
```

| 变量名称 | 值 |
|----------|-----|
| `ADMIN` | 你的管理员密码 |

点击 **保存**。

---

#### Step 4：绑定 KV 命名空间

```
Workers 和 Pages → 点击项目名 → 设置 → 绑定
→ + 添加 → KV 命名空间
```

| 变量名称 | KV 命名空间 |
|----------|-------------|
| `KV` | 刚创建的 KV 实例 |

点击 **保存**，然后**重新部署**。

---

#### Step 5：进入 edgetunnel 后台

部署完成后，在 `Workers 和 Pages / helloworld` 页面找到 **生产域**，访问地址：

```
https://helloworld-a0f.pages.dev/login
```

输入 `ADMIN` 密码登录，即可在 **获取节点链接** 中复制节点。

---

#### Step 6：速度优化（重要）

> ⚠️ 默认网络质量较差，建议进行以下优化

登录后台后，点击 **"我是高手！我就要折腾！"**，进行如下配置：

**① 基础优化**

- **详细配置信息** → 勾选 "启动 0-RTT"
- **Encrypted Client Hello** → 勾选 "启动"

> 💡 **ECH 说明**：WS + TLS 节点通信内容虽已加密，但 TLS 握手阶段仍会暴露 SNI（节点伪装域名 HOST）。ECH 将 SNI 也加密，外部观察到的域名统一显示为 `cloudflare-ech.com`。

**② 订阅转换配置**

```
订阅转换配置 → 选择：
"识别多地区、CloudFlareCDN负载均衡CF节点专用CM_Online_Full_multiMode_CF"
```

**③ 添加优选 IP**

进入 [bestcf](https://bestcf.pages.dev/) 获取推荐 IP，然后在后台配置：

```
优选订阅生成 → 选择 "自定义订阅（支持汇聚订阅）"
→ 填入优选地址
```

推荐优选订阅地址：

```
https://bestcf.pages.dev/domain/all.txt
https://bestcf.pages.dev/vps789/top20.txt
https://bestcf.pages.dev/gslege/Cfxyz.txt
https://bestcf.pages.dev/gslege/SG.txt
https://bestcf.pages.dev/gslege/JP.txt
https://bestcf.pages.dev/gslege/US.txt
https://raw.githubusercontent.com/HandsomeMJZ/cfip/refs/heads/main/best_ips.txt
```

---

#### Step 7：获取节点链接

展开 **获取节点链接**，根据客户端类型（v2rayN / sing-box / Clash 等）获取对应订阅链接。

---

### 4.3 可选步骤：自定义域名

> 使用 [dnshe](https://www.dnshe.com/) 注册免费域名（送 5 个永久免费域名，根域名选择 `ccwu.cc` 或 `cc.cd`）

#### Step 1：创建域名

登入 [dnshe](https://www.dnshe.com/)：

```
Free Domains → 注册新域名
```

获取新域名（如 `zilong78.cc.cd`）。

---

#### Step 2：申请 DNS 服务器

登入 [Cloudflare](https://dash.cloudflare.com/)：

```
账户主页 → 域 → 添加域名
→ 粘贴新域名（zilong78.cc.cd）
→ 继续 → 选择 Free 计划
→ 继续前往激活
→ 复制 CF 分配的两个 DNS 服务器地址
```

---

#### Step 3：绑定 DNS 服务器

**① 在 dnshe 中配置**

```
新域名 → DNS 服务器 → 添加 CF 的两个 DNS 服务器
→ 保存配置
```

**② 在 Cloudflare 中确认**

```
回到 CF 页面 → 点击 "我已更新名称服务器"
→ 等待约 2 分钟完成注册
```

---

#### Step 4：配置自定义域名

**① 添加自定义域**

```
Workers 和 Pages / helloworld → 自定义域
→ 添加域名（如 zilong78.cc.cd）
→ 确认新 DNS 记录（此时暂停）
```

**② 添加 DNS 记录**

```
CF → 域 → 选择刚创建的域名
→ DNS 记录 → 添加记录
→ 填写上一步显示的 "确认新 DNS 记录" 信息
→ 保存
```

**③ 激活域名**

```
回到 "确认新 DNS 记录" 界面
→ 点击 "激活域"
→ 等待约 2 分钟完成
```

---

#### Step 5：更新节点伪装域名

通过自定义域名访问 edgetunnel 后台：

```
https://zilong78.cc.cd/login
```

更新 **"节点伪装域名"** 为你的自定义域名。

---

## 五、自定义域名的好处

使用自定义域名替换默认的 `*.pages.dev` 域名，有以下优势：

| 好处 | 说明 |
|------|------|
| **规避域名封锁** | `pages.dev` 域名在某些地区已被针对性干扰，自定义域名可有效规避 |
| **提升隐蔽性** | 自定义域名看起来像普通个人网站，降低被识别为代理服务的风险 |
| **防止关联封禁** | 即使单个域名被封，只需更换新域名即可恢复，不影响 Cloudflare 账户 |
| **灵活管理** | 可随时更换域名、切换 DNS 解析，拥有完全控制权 |
| **支持更多功能** | 自定义域名可配合 ECH 加密，进一步隐藏 SNI 信息 |

> 💡 **建议**：即使是免费域名，也比默认的 `pages.dev` 域名更稳定可靠。

---

## 参考链接

- [Cloudflare 官网](https://dash.cloudflare.com/)
- [edgetunnel 项目](https://github.com/cmliu/edgetunnel)
- [dnshe 免费域名](https://www.dnshe.com/)
- [bestcf 优选 IP](https://bestcf.pages.dev/)
