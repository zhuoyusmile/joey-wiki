# claw-code 学习报告

> 2026-04-13 | 来源：[ultraworkers/claw-code](https://github.com/ultraworkers/claw-code)

## 项目概述

claw-code 是 Claude Code 的 Rust 实现，~77k 行代码，9 个 crate。它不只是一个 CLI 工具，而是一个**可被 agent 驱动的 coding harness**。

核心理念：人类给方向，agent 负责规划、执行、测试、恢复、汇报。

## 三条核心设计原则

### 1. 把隐式经验变成显式结构

| 隐式经验 | claw-code 的显式结构 |
|---|---|
| "worker 没起来" | WorkerStatus 6 态状态机 |
| "常见的坑" | Recovery Recipes |
| "测试过了" | Green Contract 四级验收 |
| "会话太长了" | Session Compaction + 预算触发 |

### 2. 把出了问题才看日志变成平时就有事件流

- **Lane Events**：`lane.started` / `lane.blocked` / `lane.green` / `lane.failed` 等标准化事件
- **Failure Taxonomy**：12 类统一故障分类，事件系统、策略引擎、恢复系统共用
- **Degraded Report**：MCP/插件部分失败时，汇报"还能做什么"而不是只报错

### 3. 把 prompt 工程变成控制面工程

- **Context Preflight**：发请求前估算 token，超预算先压缩再发
- **Session Compaction**：按 token 预算触发，保留最近原文 + 历史摘要合并
- **System Prompt 分层**：稳定骨架 + 动态边界，`SYSTEM_PROMPT_DYNAMIC_BOUNDARY`
- **Policy Engine**：把自动化策略从 if/else 泥团抽成声明式规则

## 架构总览

```
rust/
├── api/            # 模型提供商抽象（Anthropic/OpenAI/xAI）+ 流式 + 鉴权 + preflight
├── runtime/        # 核心运行时：会话、权限、配置、MCP、worker、恢复、策略
├── tools/          # 内置工具 + 工具 registry + 全局 registry（team/task/worker）
├── commands/       # slash command 定义与解析
├── plugins/        # 插件系统 + hook + 生命周期管理
├── rusty-claude-cli/  # 主 CLI 入口 (claw)
├── telemetry/      # 埋点、trace、usage
├── mock-anthropic-service/  # 本地 mock provider（端到端测试）
└── compat-harness/ # 兼容/清单抽取
```

## 最值得学的模块

### Worker Boot 状态机 (`runtime/src/worker_boot.rs`)

把 worker 启动过程显式化：

```
Spawning → TrustRequired → ReadyForPrompt → Running → Finished/Failed
```

还能检测 prompt misdelivery（发错地方/发错任务）并自动恢复。

### Session Compaction (`runtime/src/compact.rs`)

- 按估算 token 预算触发，不是按轮数
- 旧历史压成 synthetic system message
- 保留最近 N 条原文
- 专门修了 ToolUse/ToolResult 边界问题（孤儿 tool message 会让 provider 报 400）

### Policy Engine (`runtime/src/policy_engine.rs`)

声明式规则层：

```
if GreenAt(workspace) + ScopedDiff + ReviewPassed → MergeToDev
if StaleBranch → MergeForward
if StartupBlocked → RecoverOnce → Escalate
```

### Recovery Recipes (`runtime/src/recovery_recipes.rs`)

把常见失败映射到恢复步骤：

- Trust prompt 未解决 → 自动接受
- Prompt 发错 → 重定向到 agent
- 分支过时 → rebase + clean build
- MCP 握手失败 → 重试
- Provider 失败 → 重启 worker

### Green Contract (`runtime/src/green_contract.rs`)

渐进式验收：TargetedTests → Package → Workspace → MergeReady

## 对我们的启发

### 差距分析

| 领域 | claw-code | 我们 |
|---|---|---|
| Context 管理 | compaction + preflight + cache 观测 | 靠平台默认，没主动管理 |
| Subagent 管理 | 6 态状态机 + task packet + receipt | spawn 完等 announce |
| 状态与事件 | typed events + failure taxonomy | 没有显式状态机 |
| 故障恢复 | recovery recipes + 自动重试 | 出问题重新起 agent |

### 我们的 P0 落地（详见 [工作协议](protocols.md)）

1. **结构化 Task Packet** — 派工有 contract
2. **Receipt 回执校验** — subagent 先报到再开干
3. **统一 Failure Taxonomy** — 10 类故障分类
4. **轻量 Event Log** — JSONL 事件流
5. **Green Contract** — G0~G4 五级验收

### 不值得现在学的

- 完整 lane/PR/merge 自动化（我们人在环太强）
- 一比一等价 provider cache 机制
- 过细的 Rust 式类型层级
- 全量插件生态（先把生命周期可见性做好）

## 参考链接

- 仓库：[ultraworkers/claw-code](https://github.com/ultraworkers/claw-code)
- 哲学：[PHILOSOPHY.md](https://github.com/ultraworkers/claw-code/blob/main/PHILOSOPHY.md)
- 路线图：[ROADMAP.md](https://github.com/ultraworkers/claw-code/blob/main/ROADMAP.md)
