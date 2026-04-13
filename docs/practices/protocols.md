# 小爪工作协议 PROTOCOLS

> 2026-04-13 | 灵感来源：[claw-code 学习](claw-code-study.md)

这是小爪在现有 OpenClaw `spawn → 执行 → announce` 工作流之上加的一层轻量协议。不是平台 API，本质上是**派工模板、回执模板、验收口径、事件记录格式**。

## 一、Task Packet（任务包）

主 agent 派工时，先整理成结构化任务包。

### 必填字段

| 字段 | 说明 |
|---|---|
| `task_id` | 任务编号，YYYYMMDD-短名 |
| `owner` | 谁在派工 |
| `objective` | 一句话说清目标 |
| `in_scope` | 明确要做的事 |
| `out_of_scope` | 明确不要做的事 |
| `deliverables` | 预期交付物 |
| `acceptance` | 验收标准，2~5 条 |
| `constraints` | 限制条件 |

### 模板

```
[Task Packet]
task_id: 20260413-example
owner: main agent
objective: <一句话>
in_scope:
- ...
out_of_scope:
- ...
deliverables:
- ...
acceptance:
- ...
constraints:
- ...
green_contract: G3
```

## 二、Receipt（回执校验）

subagent 启动后第一件事：回执。目标是尽早发现跑错目录、理解错任务。

### 回执内容

```
[Receipt]
task_id: <...>
workspace: <绝对路径>
objective_echo: <用自己的话复述>
deliverables_echo:
- ...
first_step: <准备先做什么>
risk_or_blocker: none
```

### 校验规则

1. 任务对齐：objective_echo ≈ 原 objective
2. 工作区对齐：workspace 在预期目录
3. 交付物对齐：deliverables_echo 覆盖任务包产出
4. 限制对齐：没有遗漏关键约束

通过 → `receipt.ok`；偏差大 → `receipt.reject`，直接重派。

## 三、Failure Taxonomy（故障分类）

| 分类 | 含义 |
|---|---|
| `packet_invalid` | 任务包不完整或互相冲突 |
| `receipt_mismatch` | 回执显示任务理解错了 |
| `workspace_mismatch` | 进入了错误目录/repo |
| `input_missing` | 缺少关键输入 |
| `permission_blocked` | 需要审批/权限 |
| `tool_runtime` | 工具本身报错 |
| `provider_or_context` | 模型/上下文问题 |
| `acceptance_failed` | 做完但没达标 |
| `stale_result` | 结果基于过时上下文 |
| `human_input_required` | 需要人拍板 |

## 四、Event Log（事件流）

写入 `memory/events/YYYY-MM-DD.jsonl`，一行一个 JSON。

### 标准事件

- `task.created` / `task.dispatched`
- `task.receipt.ok` / `task.receipt.reject`
- `task.running` / `task.blocked` / `task.recovered`
- `task.acceptance.green` / `task.finished` / `task.failed`

### 最小字段

`ts` / `task_id` / `role` / `event` / `status` / `summary`

## 五、Green Contract（验收等级）

| 等级 | 含义 |
|---|---|
| **G0** | Receipt OK，还没实质产出 |
| **G1** | Draft Ready，有草稿未自检 |
| **G2** | Self Checked，已最低成本自检 |
| **G3** | Acceptance Passed，验收项逐条满足 |
| **G4** | Human Review Ready，可交 Joey |

### 默认 contract

| 任务类型 | 默认等级 |
|---|---|
| 调研/阅读 | G2 |
| 文档/规范 | G3 |
| 代码修改 | G3 |
| 敏感操作 | G4 |

## 落地顺序

1. 先强制 Task Packet
2. 再强制 Receipt
3. 结果带 Green Level
4. 关键节点写 Event Log
5. 失败用 Taxonomy

> **一句话：先把派工、对齐、验收、记账做规矩。**
