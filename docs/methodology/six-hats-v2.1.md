# 4 帽 + 都知 v2.1 升级方案

> 版本：v2.1（草案）
> 日期：2026-05-02
> 作者：小爪
> 上一版：v2.0（2026-04-09 飞书 docx，4 帽 + 都知，门下 approve）
> 升级依据：
> - PTFA 论文精读（[精读底稿](ptfa-paper-deep-read.md) / [报告 + 心得](ptfa-takeaways.md)）
> - 9 模型自评互评测试（2026-05-02 飞书话题）
> - LLM 性格研究综述（[`llm-personality-research.md`](llm-personality-research.md)）

---

## 0. 升级摘要（一页纸）

### v2.0 → v2.1 主要变化

| 维度 | v2.0 | v2.1 |
|---|---|---|
| 角色总数 | 4 帽 + 都知（5）| 4 帽 + 都知-facilitator + 都知-judge（6） |
| 阶段控制 | prompt 提示 | **硬编码 phase state machine**（4 阶段） |
| 帽子发言权 | 都知调度 | **每帽 self-veto + 都知二次仲裁**（每轮 ≤ N 顶发言） |
| 红帽 functional core | 删除（嵌入黄/黑）| **追问 underlying preference 能力**塞进都知-facilitator |
| 白帽工具 | 凭 LLM 知识 | **必须接 web_search**，输出带 source URL |
| 评测 | 凭主观 / 没硬指标 | **5 个客观 metric** + 独立 reviewer agent post-mortem |
| 模型分配 | 凭直觉 | **基于 9 模型自评互评 + SycEval 数据**指定 |

### 一句话表达

> v2.1 = v2.0 + **决策权与主持权分离** + **阶段硬切换** + **白帽接工具** + **post-mortem 自动复盘** + **基于实测数据的模型分配**。

---

## 1. 角色定义（v2.1）

### 1.1 4 顶帽子（保留 v2.0 设计）

| 帽 | 职责 | 主力模型 | 备选 |
|---|---|---|---|
| **白帽** | 客观事实、数据、信息缺口；**必须接 web_search 工具**；输出带 source URL | Claude Opus 4.6-1m（长 context = 装事实的桶） | Qwen / 混元 / Gemini 3.1 Pro |
| **黄帽** | 价值、收益、可行性；正向论证 | Claude Sonnet 4.6（互评 5 票黄帽） | GPT-5.4 |
| **黑帽** | 风险、批判、伦理否决（v2.0 已合并橙帽） | Claude Opus 4.7（互评 5 票黑帽，自他双认） | Sonnet 4.6 |
| **绿帽** | 创意、替代方案、远距离联想；保留**追问 underlying preference**职能（兼任） | Gemini 3.1 Pro（自他共识绿帽） | DeepSeek / GPT-5.5 |

### 1.2 都知拆成两个角色（v2.1 新增）

#### 都知-facilitator（调度官）
**职责**：
- Turn-taking：决定每轮哪几顶帽子真的发言（基于 self-veto + priority）
- 限制每轮发言上限（默认 N=2，避免 4 帽炸场）
- **追问 underlying preference**（红帽 functional core 的归宿）
- 检测沉默 / 偏题 / 单帽 dominate 并干预
- **没有「终止讨论」权限**

**主力模型**：GPT-5.5（自评 + B 都说蓝帽，调度切换强）
**备选**：Claude Opus 4.7

#### 都知-judge（拍板人）
**职责**：
- 独立判断"是否进入下一阶段"
- 独立判断"最终结论是否成熟"
- **有权强制终止讨论**
- 不参与主持，只在每阶段末被调用 1 次

**主力模型**：Claude Opus 4.7（黑帽稳、批判力强、不易被讨好）
**备选**：Claude Opus 4.6-1m

**铁律**：facilitator 和 judge **必须不同 prompt**（建议不同模型实例），决策权与主持权严格分离。

### 1.3 红帽的归宿

v2.0 删红帽是对的（9 模型自评互评 9/9 一致认为最不适合红帽），但红帽的 **functional core**（追问情感 / 偏好来源）有用，归并到：
- **都知-facilitator**：开场 + 阶段切换时主动追问"你为什么这么想？这背后的偏好是什么？"
- **绿帽**：当某 agent 给出"我觉得 A 比 B 好"但没理由时，**强制反问**

---

## 2. Phase State Machine（v2.1 核心新增）

### 2.1 四阶段硬切换

```
[diverge] → [critique] → [converge] → [finalize]
   ↑           ↑            ↑            ↑
  绿/白      黑/白        黄/黑/judge   judge
```

| 阶段 | 允许发言的帽子 | 主导帽子 | 切换触发 |
|---|---|---|---|
| **diverge**（发散）| 白 + 绿 + 都知-fac | 绿帽 | judge 判断"已收集足够备选方案" |
| **critique**（批判）| 黑 + 白 + 都知-fac | 黑帽 | judge 判断"风险已被充分评估" |
| **converge**（收敛）| 黄 + 黑 + 都知-fac | 黄帽 | judge 判断"备选已收窄到 ≤2 个" |
| **finalize**（拍板）| 都知-judge only | judge | judge 输出最终结论或 escalate |

### 2.2 实现要求（铁律）

1. **不靠 prompt 控阶段**：不能在系统 prompt 里写"先发散后收敛"——LLM 会 ignore（PTFA 实证）
2. **每阶段 = 独立的 LLM 调用 / 独立的 task**：不是一个 prompt 里走完
3. **绿帽在 critique / converge 阶段必须 mute**：发散 agent 在收敛阶段是噪声源
4. **judge 只在阶段末调用 1 次**：不是每轮调用，避免误判
5. **judge 可以 escalate 给 Joey**：当多次循环 diverge → critique 还无法收敛时

---

## 3. 发言机制（self-veto + 都知仲裁）

### 3.1 self-veto（抄 PTFA）

每个被允许发言的帽子在每轮先输出：
```json
{
  "should_speak": true | false,
  "priority": 0-10,
  "content": "..."  // 仅 should_speak=true 时填
}
```

`should_speak: false` 即过滤，不进入下一步。

### 3.2 都知仲裁（v2.1 新增）

都知-facilitator 收到所有帽子的意向后：
1. 过滤掉 `should_speak: false` 的
2. 按 priority 降序，取前 **N 顶**（默认 N=2，可配置）
3. 应用 cooldown：上轮发言过的帽子 priority -3
4. 输出最终发言列表

这一层避免 PTFA 里"绿/蓝刷屏淹没其他帽子"的问题。

---

## 4. 工具与输出规范

### 4.1 白帽必须接工具

- **必接**：`web_search`、文档检索（如有）、`memory_search`
- 输出格式必须含：
  ```markdown
  事实陈述 [Source: <URL or memory:path#line>]
  ```
- 没有 source 的事实陈述 → **黑帽有权直接 reject**，要求白帽补充

### 4.2 黄/黑/绿无强制工具

但建议：
- 黑帽：可调 `web_search` 验证白帽事实
- 绿帽：可调 `web_search` 找类比 / 案例

### 4.3 都知-judge 只看输出，不再调用其他帽子

judge 的输入 = 当前阶段所有帽子发言的 trace。不允许 judge 自己再去 web_search 或追问，否则相当于又当主持又当判官。

---

## 5. Post-mortem 自动复盘

每次完整 run 结束后，由独立 **reviewer agent**（建议 Sonnet 4.6 或 GPT-5.5）跑一次 post-mortem，**强制检查 5 项**：

| # | 检查项 | 失败定义 | 说明 |
|---|---|---|---|
| 1 | 共识是否被打散 | converge 阶段已达成的备选在 finalize 前被新提议推翻 | PTFA 真实事故 |
| 2 | 帽子 dominance | 任一帽子发言占比 > 40% | 防话痨刷屏 |
| 3 | 提问被忽视率 | 直接提问无回应数 / 总直接提问数 > 20% | 防 PTFA 那种"忽视真人提问"病 |
| 4 | 阶段切换准确率 | judge 切换决定 vs 实际状态（用 trace 反推）一致率 | 防误切换 |
| 5 | 白帽事实 source 覆盖率 | 带 source 的事实陈述 / 总事实陈述 > 80% | 强制工具使用 |

任一项 fail → 该 run 标记为「不合格 run」，进入 v2.1 backlog 优化目标。

---

## 6. 客观 Metric（v2.1 新增）

每个 run 自动从 trace 算：

1. **共识达成时间**：进入 finalize 阶段所用轮数 / 秒数
2. **共识稳定性**：finalize 后是否被新提议打散（0/1）
3. **每帽子有效贡献率**：被采纳进最终结论的发言数 / 该帽子总发言数
4. **直接提问被回应率**：见上 §5 表第 3 项
5. **阶段切换准确率**：见上 §5 表第 4 项

这些指标比 Likert 问卷强，可以量化对比 v2.0 vs v2.1 的实际改进。

---

## 7. 模型分配（基于实测数据）

### 7.1 决策依据

- **9 模型自评互评测试**（2026-05-02）：A·Opus 4.7 → 黑帽（5 票）、B·Opus 4.6-1m → 白帽（5 票）、D·Sonnet 4.6 → 黄帽（5 票）、E·Gemini 3.1 → 绿帽（4 票）
- **SycEval 数据**：Claude 系列最坚持立场（适合 judge / 黑帽），Gemini 最易让步（适合 diverge 阶段的发散绿帽）
- **Anthropic Claude's Character**：Claude 系列在批判性 + 元认知上有 prior

### 7.2 班子表

| 角色 | 主力 | 备选 | 依据 |
|---|---|---|---|
| 白帽 | Opus 4.6-1m | Qwen / 混元 / Gemini 3.1 | 长 context + 实测 5 票 |
| 黄帽 | Sonnet 4.6 | GPT-5.4 | 实测 5 票黄 + 表达流畅 |
| 黑帽 | Opus 4.7 | Sonnet 4.6 | 实测 5 票黑 + SycEval 最坚持 |
| 绿帽 | Gemini 3.1 Pro | DeepSeek / GPT-5.5 | 自他共识 + SycEval 易让步 = 易发散 |
| 都知-facilitator | GPT-5.5 | Opus 4.7 | 自评 + 互评共识蓝帽 |
| 都知-judge | Opus 4.7 | Opus 4.6-1m | SycEval 最坚持 = 不易被绿帽说动 |

### 7.3 红帽不分配模型

红帽 functional core 已嵌入都知-facilitator 和绿帽 prompt，无独立模型实例。

---

## 8. v2.0 → v2.1 迁移清单

实战中若要从 v2.0 升到 v2.1，按顺序：

- [ ] **A**：把 v2.0「都知」prompt 拆成 facilitator + judge 两份独立 prompt
- [ ] **B**：实现 phase state machine（4 阶段，硬编码切换，不靠 prompt）
- [ ] **C**：每帽子加 self-veto 输出格式（`should_speak / priority / content`）
- [ ] **D**：白帽 prompt 加「必须接 web_search + 带 source URL」要求；写校验逻辑
- [ ] **E**：把红帽 functional core（追问 underlying preference）加进 facilitator + 绿帽 prompt
- [ ] **F**：写 post-mortem reviewer agent prompt + 5 项检查实现
- [ ] **G**：写 trace 自动算 5 个 metric 的脚本
- [ ] **H**：按 §7.2 配置主力模型

建议优先级：A > B > D > C > F/G > E > H

---

## 9. 仍未解决的问题（v2.1 backlog）

1. **judge 的 escalate 阈值**：多少次循环算"无法收敛"，需要请 Joey 介入？
2. **是否需要"二审 judge"**：单 judge 万一固执怎么办？是否引入双 judge 投票？
3. **跨 run 学习**：不合格 run 的失败模式如何沉淀进下版 prompt？需要长期记录。
4. **成本控制**：6 角色 × N 轮 × M token 可能很贵。需要 lite/standard/full 三档预算。
5. **真实任务验证**：v2.1 还没在小说项目 / 决策项目里跑过。先要选个 pilot task。

---

## 10. 配套素材索引

| 素材 | 路径 | 用途 |
|---|---|---|
| v2.0 原始方法论 | `memory/2026-04-09.md` "六顶帽子项目"段 + 飞书 docx | 历史基线 |
| PTFA 论文精读 | [`ptfa-paper-deep-read.md`](ptfa-paper-deep-read.md) | 升级依据 |
| PTFA 报告 + 心得 | [`ptfa-takeaways.md`](ptfa-takeaways.md) | Joey 视角总结 |
| LLM 性格研究综述 | [`llm-personality-research.md`](llm-personality-research.md) | 14 篇真实文献 |
| 9 模型自评互评测试 | 2026-05-02 飞书话题（待整理） | 模型分配数据依据 |
| AI 幻觉与多 agent 编排 | [`ai-hallucination.md`](ai-hallucination.md) | 防偏差参考 |

---

## 附录：v2.1 与 PTFA 的关系

| 维度 | PTFA | v2.0 | **v2.1** |
|---|---|---|---|
| 帽子数 | 6 | 4 + 都知 | 4 + 2 都知 |
| facilitator/judge | 合一（蓝帽兼任）| 合一（都知兼任）| **拆开** |
| 阶段控制 | prompt only（失控）| prompt only | **硬编码** |
| 发言调度 | 每帽 self-veto | 都知调度 | **self-veto + 都知仲裁** |
| 工具 | 无 | 无 | **白帽强制 web_search** |
| 评测 | Likert 问卷 | 凭感觉 | **5 项 metric + post-mortem** |
| 红帽 | 自评+追问 | 删 | **删但保留 functional core** |
| 模型分配 | 单一 OpenAI | 凭感觉 | **基于实测 + SycEval 数据** |

**v2.1 是 PTFA 失败模式的工程对策版本**。
