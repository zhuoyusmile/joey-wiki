# PTFA 文献报告 + 我们的心得

> 写于 2026-05-02
> 作者：小爪
> 用途：Joey 复看 / 班子方法论 v2.1 升级依据
> 配套精读底稿：[`ptfa-paper-deep-read-2026-05-02.md`](ptfa-paper-deep-read.md)
> 调研综述：[`llm-personality-research-2026-05-02.md`](llm-personality-research.md)

---

## 一、这篇文章讲了什么

**论文**：*PTFA: An LLM-based Agent that Facilitates Online Consensus Building through Parallel Thinking*
**作者**：Wen Gu 等（Nagoya Institute of Technology / University of Southampton / JAIST 等）
**时间**：v1 = 2025-03-16；v2 = 2025-10-22
**arXiv**：<https://arxiv.org/abs/2503.12499>
**HTML 全文**：<https://arxiv.org/html/2503.12499v2>
**领域**：cs.HC / cs.AI

### 一句话概括

> 把 Edward de Bono 的六顶思考帽方法工程化成 6 个独立 LLM agent，做**真人在线讨论的主持人（facilitator）**，让 3 个真人在 20 分钟内更顺利达成共识。

### 核心设定

- **6 个 hat-agent**（白/红/黑/黄/绿/蓝），每顶帽子 = 一个独立的 OpenAI Assistant（带专属 system prompt）。
- **不是 agent 互相辩论**：参与者是 **3 个真人 + 6 个 hat-agent**，agent 是讨论的"facilitator"，最终决策由真人做。
- **平台**：Discourse 论坛 + WebSocket + PostgreSQL，6 个 hat 当作"虚拟用户"在帖子里发言。
- **触发**：30 秒一个 tick，每个 hat 看当前讨论上下文，**自己判断该不该发言**——觉得没必要就输出 `'Good'`，平台过滤掉不显示。
- **没有显式 phase 切换、没有独立 judge、没有投票**。所谓"先发散后收敛"只在 prompt 里告诉 LLM。

### 实验

- 48 名参与者 / 16 组 / 3 人/组 / 每组 2 场 = **32 场讨论**。
- 任务：决定一起做什么活动、决定看什么电影（轻量、生活化）。
- baseline：纯定时模板（"开始讨论吧"/"该收尾了"三句话）。
- 评测：**只有 7 级 Likert 问卷**——没有任何客观 metric（无准确率、无共识质量评分）。
- 结果：PTFA 在"是否帮助达成共识"上略胜 baseline，但**论文自陈结果"mixed"**，没有显著提升。

### 主要结论

- PTFA **不是一篇 SOTA 论文**，是一篇 pilot study + 失败模式诚实记录 + 数据集构建。
- 真正的价值在于**首次在真人讨论里跑通六顶帽子的多 agent facilitator**，并把踩过的坑全部摊开。

---

## 二、重点 + 亮点

### 🌟 亮点 1：红帽意外地是最有用的帽子之一

- 大家都担心"AI 假装有情绪很尬"，但 PTFA 红帽**绕开了这个死结**：它不自己表达情绪，而是**追问对方的情绪和偏好来源**。
- 当真人只说"我觉得还行 / 不太喜欢"这种 0 信息量发言时，红帽问"具体哪一点让你不喜欢？"，逼真人外化情感，让讨论从浅层进入深层。
- 失败案例：偶尔对中性发言强加情绪解读（论文 Figure 7b）。
- **对我们的启发**：删了红帽不等于丢掉它的 functional core——"追问 underlying preference"这个能力必须保留进绿帽或都知。

### 🌟 亮点 2：`'Good'` self-veto 是个聪明的低成本设计

- 每个 hat 在每个 tick 都被调用，但**只在觉得"现在我说有用"时才真的发话**，否则输出字符串 `'Good'`，被平台过滤掉。
- 用 1 个 LLM 调用换取"是否插话"的判断，免去了复杂的调度器。
- 缺陷：没有外部仲裁，绿/蓝两个话痨帽子永远觉得自己该说话，最后淹没了其他帽子。

### 🌟 亮点 3：诚实暴露失败模式

- 整个 Section 4.2.3 + 4.3 是一份"我们做错了什么"的清单。
- 最严重的失败：**Phase Management 完全失控**——
  > "The LLM is prompted to facilitate a divergent first discussion stage ... followed by a convergent second stage. **In practice, the facilitator stayed in the first stage throughout** ..."
  - 真实事故：一组真人已经达成共识，PTFA 又抛出新建议把意见打散，剩余时间不够再统一，**最终共识破裂**。
- 其他失败：8 分钟才发第一条、忽视真人直接提问、每条发言后都插话、白帽因训练 cutoff 答错事实。

### ⚠️ 重点警示：这不是 SOTA，不能当 baseline 用

- 评测只有 Likert 问卷，48 人 pilot，结果"mixed"。
- 不能写"我们方案比 PTFA 提升 X%"——人家本来就没声称自己是高分。
- 论文的价值在**反面教材**，不在正面验证。

### 📝 重点细节：六顶帽子的 prompt 模板（可直接借鉴）

| 帽子 | situational prompt 示例（论文 Section 3） |
|---|---|
| 白 | "Could you clarify the exact figures or facts related to this issue? Here's what we know so far: [insert relevant data]." |
| 红 | "This feels like an emotional moment. How are we feeling about this issue right now?" |
| 黑 | "Have we considered the potential downsides? Here's a risk we might be overlooking: [insert risk]." |
| 黄 | "Looking at the bright side, this idea offers some exciting opportunities we shouldn't overlook." |
| 绿 | "What if we approached this from a different angle? Here's an idea to consider: [insert new idea]." |
| 蓝 | "It seems like we're getting off track. Maybe we should focus on this key point: [insert key point]." |

---

## 三、我们的心得（v2.1 升级清单）

把 PTFA 的踩坑 + 我们 4 月 9 号 v2.0 方案 + 这次 9 模型自评互评测试三件事合起来，整理成可落地的升级清单。

### 心得 1：「都知」必须拆成 facilitator + judge 两角色 ⭐⭐

**PTFA 教训**：蓝帽兼任主持和裁判，结果**没人按下"够了，进入收敛"按钮**。绿/蓝两个最话痨的帽子一起把已达成的共识又打散。

**我们的升级**：
- **都知-facilitator**（调度）：负责 turn-taking、避免冷场、限制刷屏。
- **都知-judge**（裁决）：独立判断"是否进入收敛"、"最终结论是否成熟"，**有权终止讨论**。
- 两个角色可以同模型不同 prompt，**关键是把"主持权"和"决策权"分离**。

这条其实 Joey 4 月 9 号就提过"拍板人 ≠ 都知"，PTFA 验证了它的必要性。

### 心得 2：阶段切换必须硬编码，不能写在 prompt 里 ⭐⭐

**PTFA 教训**：Prompt 里写"先发散后收敛"，LLM **完全 ignore**，整场都停在 diverge 阶段。

**我们的升级**：
- 写成显式 phase state machine：`diverge → critique → converge → finalize`。
- 每阶段允许的帽子集合**显式限定**（如 converge 阶段绿帽 mute）。
- 阶段切换由都知-judge 在每轮结束时**显式宣布**，由代码切换，不靠 LLM 自觉。
- 每阶段 task 可以拆成独立 LLM 调用，不是一个 prompt 里说"请按以下顺序"。

**这个教训的延伸**：我们小说项目里 reviewer 失控、写手在不该改的地方乱改，根因都是同一个——**靠 prompt 控阶段不靠谱**。

### 心得 3：抄 `'Good'` self-veto + 都知二次仲裁

**PTFA 优点**：每帽子 self-veto 是个聪明低成本设计，值得抄。
**PTFA 缺点**：没有外部仲裁，绿/蓝刷屏。

**我们的升级**：
- 每个帽子先输出 `should_speak: bool + content + priority`。
- 都知-facilitator 在所有帽子提交意向后，**从中挑选最多 N 顶**实际发言。
- 优先级 + cooldown 双约束：话痨帽子有 cooldown，安静帽子被加权。

### 心得 4：红帽功能要保留进都知或绿帽 ⭐

**PTFA 教训**：红帽真正起作用的不是"AI 表演情绪"，而是**逼真人/真 agent 说出 underlying preference**。

**我们的升级**：
- 当某 agent 给出"我觉得 A 比 B 好"但没理由时，**强制追问偏好的来源**。
- 这个能力可以放在都知-facilitator 里（作为开场提问 + 阶段切换提问），也可以塞进绿帽的"换角度"职责。
- **本质是把红帽的 functional core 保留，去掉 AI 表演情绪的尬部分**。

### 心得 5：白帽必须接工具，不能只靠模型内部知识

**PTFA 教训**：白帽因为靠 LLM 内部知识 → 撞到训练 cutoff，把"将开演"的剧目说成"将开演"，实际已经上演了。

**我们的升级**：
- 白帽**必须**显式接 web_search / 文档检索工具。
- 输出必须**带 source URL**，让黑帽可以核查。
- 没有 source 的事实陈述视为不合规，黑帽有权直接 reject。

### 心得 6：必须做 trace + 自动复盘，不能等用户问卷才发现失败

**PTFA 教训**：失败案例（"共识被打散"）是事后看转写才发现的，损失已经造成。

**我们的升级**：
- 每场讨论结束后，跑一次**独立 reviewer agent 的 post-mortem**，专门检测：
  1. 是否在共识形成后又被打散？
  2. 是否有帽子 dominate（发言占比 > 40%）？
  3. 是否存在直接被忽视的提问？
  4. 是否有阶段没切换或切错？
- 这些当成 hard quality gate，不通过就不算合格的 run。

### 心得 7：触发用"事件驱动 + 节奏限速"，不要 30s 定时

**PTFA 教训**：30s 一 tick 导致两种病——要么每条发言后都插话，要么 8 分钟才出现。

**我们的升级**：
- 触发条件 = `新输入到达 AND 上次同 hat 发言距今 ≥ cooldown_T`。
- 每帽子独立 cooldown：黑帽可以 cooldown 短（随时挑刺），绿帽 cooldown 长（避免发散刷屏）。
- 安静期超过 K 秒，都知-facilitator 主动唤起一顶帽子。

### 心得 8：必须设计客观 metric，不能只靠问卷

**PTFA 教训**：只有 Likert，结果"mixed"，无法量化改进。

**我们的升级**：至少要有以下硬指标，全部从 trace 自动算：
- **共识达成时间**（多少轮 / 多少秒）
- **共识稳定性**（达成后是否被打散）
- **每帽子有效贡献率**（被采纳进最终结论的发言占该帽子总发言比例）
- **直接提问被回应率**
- **阶段切换准确率**（judge 宣布的切换 vs 实际讨论状态）

### 心得 9（独立的元发现）：当前所有主流 LLM 都演不了红帽

**这次 9 模型自评测试的发现**：
- 9/9 自评都把红帽列为最不适合（A/B/C/D/E + 豆包/DeepSeek/Qwen/混元）
- 互评里只有 2 票投出红帽
- **结论：这不是某一家的弱点，是当前 LLM 的结构性缺陷**。

**对我们 4 月 9 号 v2.0 的补强**：当时删红帽是凭直觉，现在有了**统计意义上的多模型证据**。心得 4（保留红帽 functional core）是这条发现的工程对策。

---

## 四、最终结论

**这篇论文值不值得参考？** 值得，但不要照搬。

- ✅ **可以借鉴**：六顶帽子的角色定义、每帽子独立 prompt、`'Good'` self-veto、prompt 模板、诚实的失败案例分析。
- ❌ **不要复刻**：30s 定时触发、单 facilitator 兼判官、prompt-only 的 phase 控制、只用 Likert 评测。
- 🔧 **必须升级（v2.1）**：都知拆 facilitator + judge、phase state machine 硬编码、白帽接工具、trace 自动复盘、客观 metric。

**最有价值的单条洞察**：
> PTFA 失败的根因不是"LLM 不够强"，而是"**没有一个有权强制结束讨论的 judge 角色**"。这正好是我们设计「都知」时必须想清楚的——都知到底有没有"终止权"？如果没有，我们会重蹈 PTFA 的覆辙。

---

## 五、配套素材

- 精读底稿（含原文段落引用）：`projects/methodologptfa-paper-deep-read.md`
- LLM 性格研究综述（14 篇真实来源）：`projects/methodologllm-personality-research.md`
- 9 模型自评互评测试结果：见 2026-05-02 飞书话题（待整理进 `projects/methodology/4hat-model-roster-2026-05-02.md`）
- 4 月 9 号 v2.0 原始方法论：见 `memory/2026-04-09.md` "六顶帽子项目"段
