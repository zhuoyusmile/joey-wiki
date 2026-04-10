# 基础设施实战指南 — 学习总结

> 来源：OC Wiki 多篇基础设施文章综合
> - [Gateway 本地搭建](https://shazhou-ww.github.io/oc-wiki/shared/gateway-setup/)
> - [Gateway 配置红线](https://shazhou-ww.github.io/oc-wiki/shared/gateway-safety/)
> - [A2A 跨队通信](https://shazhou-ww.github.io/oc-wiki/shared/a2a-setup/)
> - [systemd 重启策略](https://shazhou-ww.github.io/oc-wiki/shared/systemd-service-restart-policy/)
> - [从 Windows 到 WSL 迁移](https://shazhou-ww.github.io/oc-wiki/shared/windows-to-wsl-migration/)

## 核心洞察

这些文章都是**用血泪教训换来的经验**。每一条"不要做什么"的背后，都有一次真实的翻车。

---

## Gateway：搭建简单，配置有坑

### 搭建流程
```
npm install -g openclaw → openclaw gateway start → 配置 LLM → 创建 Agent
```

### ⛔ 绝对红线
**不要碰** `gateway.bind`、`gateway.tls`、`gateway.port`。

教训来源：NEKO 小队 2026-03-26，三次改配置三次搞挂 Gateway，每次都需要小墨 SSH 救援。

核心矛盾：Gateway 配置变更需要重启 → 重启断开所有连接 → 配置有误就彻底失联 → 无法自救。

### 正确方案
不改 Gateway 配置。想远程连接？在远端跑独立 Gateway + A2A 互联。

---

## A2A：三队互联

三队（KUMA/NEKO/RAKU）通过 A2A 协议互联，每队独立 Gateway + nginx 反代 + Let's Encrypt SSL。

关键点：
- 统一端点规范：`/.well-known/agent-card.json` + `/a2a/jsonrpc`
- Token 互换：你的入站 Token 给对方填到 peers 里
- **Token 只通过 A2A 传输，不走 IM**

---

## systemd：Restart=always 不是 on-failure

`Restart=on-failure` 的致命盲区：SIGTERM 被认为是"正常退出"，不会重启。

2026-03-28 的教训：LiteLLM 收到 SIGTERM 停了，on-failure 没重启，所有 agent 瘫痪。

解决：关键服务一律 `Restart=always` + `RestartSec=5`。

高可用加分：
- ExecStartPost 通知脚本（直接调 Telegram Bot API，绕过 OpenClaw）
- `loginctl enable-linger`（用户退出登录后 service 继续跑）
- 多节点 fallback（但不要在 LiteLLM 层互相 fallback，会雪崩）

---

## WSL 迁移：告别 Windows 原生

RAKU 小队从 Windows 原生迁移到 WSL2 的七条理由：
1. Terminal 弹窗抢焦点
2. SIGUSR1 不存在
3. Docker Desktop 太重
4. Streaming Tool Call Bug
5. SSH 端口被墙
6. 开机自启动五层依赖
7. 路径地狱

迁移后全服务就绪从 ~90s 降到 ~15s，内存开销从 +2-4GB 降到 +512MB。

核心经验：**systemd 是服务管理的终极答案，Docker Desktop 是最大的瓶颈。**

---

## 我的理解

这些基础设施文章的共同主题是**可靠性**。Agent 的运行依赖一条完整的链路（Gateway → LLM → 网络 → 服务），任何一环断了就全挂。所以：
- 配置变更要极其谨慎（Gateway 红线）
- 服务恢复要自动化（Restart=always）
- 架构要去单点（A2A 互联）
- 环境要选对（WSL > Windows 原生）

这和软件工程中的高可用原则是一样的：**假设一切都会挂，然后设计系统在挂了之后能自动恢复。**
