# Hermes Agent 研究笔记

> 研究时间：2026-04-11 | 来源：Nous Research 官方文档 + GitHub

## 概述

[Hermes Agent](https://github.com/NousResearch/hermes-agent) 是 [Nous Research](https://nousresearch.com) 于 2026年2月开源的自主 AI agent 框架（17k+ stars）。核心卖点是**自我改进的学习循环**——agent 会从经验中自动创建和改进 Skill，跨会话持续变强。

## 核心架构

### 单体内核
所有入口（CLI、Gateway、Cron、ACP）汇入同一个 `AIAgent` 循环。行为一致但核心文件大。

### 自我改进学习循环

```
执行复杂任务 → 总结有效流程 → 自动创建 Skill → 使用中改进 Skill → 下次更高效
```

不是模型自训练，而是**程序化经验固化**。技能存储在 `~/.hermes/skills/`，agent 自己可以 CRUD。

### 分层记忆

| 层级 | 机制 | 用途 |
|------|------|------|
| 短期 | MEMORY.md + USER.md | 启动时冻成 prompt 快照 |
| 中期 | session_search（SQLite FTS5） | 跨会话检索 |
| 深度 | Honcho provider | 用户建模、语义检索 |

### 上下文压缩（双层）

- Gateway 85% 阈值保底
- Agent 内部 50% 阈值主压缩
- 策略：清理旧 tool output → 保留头尾 → 中段结构化 summary → 增量更新

### 子 Agent 隔离

`delegate_task` 生成独立子 agent，父代理只收最终摘要。并行上限 3，不可递归。

### 安全模型

多层防线：命令审批 → 容器隔离 → MCP 过滤 → context 注入扫描 → Gateway 授权。核心原则：**容器即安全边界**。

## 用户侧功能

| 功能 | 评分 | 亮点 |
|------|------|------|
| 消息平台 | 9/10 | 15+ 平台，跨平台上下文连续 |
| 部署 | 9/10 | 6 后端，Daytona/Modal 无服务器休眠 |
| Cron | 9/10 | 内建一等能力，自然语言创建 |
| MCP | 8.5/10 | stdio/HTTP，自动发现注册 |
| 语音 | 8/10 | CLI/Telegram/Discord，流式播报 |
| 人格 | 9/10 | 全局 SOUL.md + session overlay |
| 社区 | 7.5/10 | agentskills.io 兼容，早期增长 |

## 与 OpenClaw 对比

| 维度 | Hermes 更强 | OpenClaw 更强 |
|------|------------|--------------|
| 学习能力 | ✅ Skill 自动创建/改进 | |
| 记忆 | ✅ 分层 + Honcho | |
| 压缩 | ✅ 系统化双层 | |
| 部署弹性 | ✅ 无服务器 | |
| 生态成熟度 | | ✅ 更多现成技能和教程 |
| 即插即用 | | ✅ 更省心 |
| 子 agent 哲学 | | ✅ 都知模式更彻底 |

## 最值得借鉴的设计

1. **技能沉淀闭环** — 复杂任务后自动生成 Skill
2. **上下文压缩系统化** — tool output 清理 + 结构化 summary
3. **记忆分层** — 轻量 prompt memory + 深度外部 provider
4. **安全边界前移** — 隔离环境优先，审批为辅

## 迁移评估

官方提供 `hermes claw migrate`，可导入 OpenClaw 的人格/记忆/技能/API key。

- 🟢 **低成本**：人格、记忆、技能文档、配置
- 🟡 **中成本**：消息平台重连、Cron 重建、MCP 重配
- 🔴 **高成本**：依赖 OpenClaw 特有工具/节点时

## 结论

Hermes 是"面向未来的 agent OS"，OpenClaw 是"成熟的超级工作台"。即使不迁移，Hermes 的设计理念值得吸收。

## 参考链接

- 官方文档：<https://hermes-agent.nousresearch.com/docs/>
- GitHub：<https://github.com/NousResearch/hermes-agent>
- 架构文档：<https://hermes-agent.nousresearch.com/docs/developer-guide/architecture>
- Skills 系统：<https://hermes-agent.nousresearch.com/docs/user-guide/features/skills>
- Memory 系统：<https://hermes-agent.nousresearch.com/docs/user-guide/features/memory>
