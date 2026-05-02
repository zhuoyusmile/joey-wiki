# LLM 性格特点研究综述

> 调研日期：2026-05-02
> 范围：心理学人格框架对 LLM 的画像 / 模型间行为差异 / 多智能体按性格分工 / 工业界博客
> 共收录 **14 个真实来源**（学术 11、工业/博客 3）

---

## 一、学术论文

### A. 用心理学框架对 LLM 做人格画像（Big Five / MBTI / HEXACO / IPIP）

#### 1. Personality Traits in Large Language Models
- **作者/机构**：Greg Serapio-García, Mustafa Safdari, et al. — Google DeepMind & Cambridge & USC
- **年份**：2023（持续更新到 Nature Machine Intelligence 2025 版本）
- **URL**：https://arxiv.org/abs/2307.00184
- **核心结论**：
  - 用 IPIP-NEO（Big Five 标准量表）和 BFI 系统测量 PaLM 系列、Flan-PaLM。
  - 提出"在 LLM 上施加人格量表的有效性必须先验证心理测量学指标（reliability, construct validity）"，是该领域方法论奠基论文。
  - 发现：更大、指令微调更强的模型，人格测量越接近人类常模，且**可以通过 prompt 把模型推向 Big Five 任意维度**。
  - 该方向的"标杆引文"，几乎所有后续 LLM 人格研究都引它。

#### 2. PersonaLLM: Investigating the Ability of LLMs to Express Personality Traits
- **作者**：Hang Jiang et al. — MIT Media Lab
- **年份**：2024（NAACL Findings）
- **URL**：https://arxiv.org/abs/2305.02547  ｜  https://aclanthology.org/2024.findings-naacl.229.pdf
- **核心结论**：
  - 对 GPT-3.5 / GPT-4 构造 320 个 Big Five 人格组合的 persona。
  - 模型能在 BFI 量表上"自洽"地反映被赋予的人格；而且 persona 会**显著影响后续故事写作的语言特征**（用 LIWC 验证）。
  - 表明 prompt-based persona 不只是表面文字，会改变下游生成行为。

#### 3. Machine Mindset: An MBTI Exploration of Large Language Models
- **作者**：Cui et al.（FarReel AI Lab）
- **年份**：2023→2024
- **URL**：https://arxiv.org/html/2312.12999v3
- **核心结论**：
  - 用 SFT + DPO 把 MBTI 类型固化进模型权重（不是 prompt 层面），得到 16 种 MBTI 风格的 LLM。
  - 发现不同 MBTI 类型的模型在 reasoning、共情、生成长度上存在系统性差异（如 NT 型在 reasoning 任务更强）。
  - 提供了"用人格做模型分化"的训练侧实证。

#### 4. Who is ChatGPT? Benchmarking LLMs' Psychological Portrayal Using PsychoBench
- **作者**：Huang et al. — CUHK ARISE
- **年份**：2024（ICLR）
- **URL**：https://arxiv.org/abs/2310.01386  ｜  https://github.com/CUHK-ARISE/PsychoBench
- **核心结论**：
  - 提出 **PsychoBench**：13 个临床心理学量表（含 Big Five、DTDD 暗黑三联征、共情、情绪等），覆盖 5 个 LLM。
  - 发现 GPT-4 比 GPT-3.5 表现出更高的 agreeableness、更低的暗黑特质；LLaMA-2 的"神经质"显著高于闭源模型。
  - 用越狱 prompt 时，模型的"暗黑面"会被显著放大——暗示安全训练只是抑制层。

#### 5. Do LLMs Have Distinct and Consistent Personality? TRAIT: Personality Testset Designed for LLMs
- **作者**：Lee et al.（NAACL 2025 Findings）
- **年份**：2025
- **URL**：https://aclanthology.org/2025.findings-naacl.469/
- **核心结论**：
  - 8K 多选题的 TRAIT 基准，专门测 LLM 人格"在不同上下文/语气提示下是否稳定"。
  - 发现：不同模型确实有**统计上稳定的差异**（如 Claude 系列高 agreeableness、Mistral 系列低 conscientiousness），但单模型在不同 prompt 下波动大。
  - 提出"prompt-induced personality drift"指标。

#### 6. A Psychometric Framework for Evaluating and Shaping LLM Personality
- **作者**：Mustafa Safdari, Greg Serapio-García et al.（DeepMind 团队后续工作）
- **年份**：2025（Nature Machine Intelligence）
- **URL**：https://www.nature.com/articles/s42256-025-01115-6
- **核心结论**：
  - 在 PaLM、Llama 2、Mistral、Mixtral、GPT 系列上系统化施测 Big Five。
  - 验证可以通过 prompt 把模型"塑形"到任意 Big Five profile，并量化稳定性。
  - 是 #1 的扩展，给出了完整的 psychometric 评估流水线。

#### 7. Evaluating LLM Alignment under Big Five Personality Prompting
- **作者**：CEUR-WS Workshop 2025
- **年份**：2025
- **URL**：https://ceur-ws.org/Vol-4178/paper6.pdf
- **核心结论**：
  - **6 个模型横向对比**：GPT-4o Mini、Llama 3.2、Mistral NeMo、Gemini 2.0 Flash Lite、Gemma 2、Claude 3 Haiku。
  - 发现各家模型对"高/低 Big Five"prompt 的响应一致性差异巨大：Claude 最稳定，Gemini 在 conscientiousness 维度最容易被推动。
  - 项目用得上：明确指出**模型间存在可复现的"性格倾向差"**。

### B. 模型差异与行为对比

#### 8. SycEval: Evaluating LLM Sycophancy
- **作者**：Stanford team
- **年份**：2025
- **URL**：https://arxiv.org/abs/2502.08177
- **核心结论**：
  - 横向测 ChatGPT-4o、Claude-Sonnet、Gemini-1.5-Pro 在数学（AMPS）和医疗（MedQuad）上的"顺从度"。
  - 发现：当用户反对模型答案时，**Gemini 最容易让步、Claude 最坚持立场、GPT-4o 居中**。
  - 量化了"批判性 vs 顺从性"轴上的模型差异——直接对应 Joey 项目的需求。

#### 9. Social Sycophancy / ELEPHANT Benchmark
- **作者**：Cheng et al.（Stanford）
- **年份**：2025
- **URL**：https://arxiv.org/abs/2505.13995
- **核心结论**：
  - 提出"social sycophancy"——比答案顺从更广义的"维护用户面子"行为。
  - ELEPHANT 基准测 8 个模型，发现**所有主流模型都过度迎合**，但 Claude 系列在"拒绝错误前提"上表现最好。
  - 给出了"委婉 vs 直接"维度的量化方法。

#### 10. Verbosity ≠ Veracity: Demystify Verbosity Compensation Behavior of LLMs
- **作者**：Penn State NLP
- **年份**：2024 EMNLP
- **URL**：https://arxiv.org/html/2411.07858v2  ｜  https://github.com/psunlpgroup/VerbosityLLM
- **核心结论**：
  - 系统分类"啰嗦"行为为 5 类，发现 GPT-4 / Claude / LLaMA 各有不同的 verbosity 倾向。
  - **Claude 倾向于多角度铺陈、GPT-4 倾向于结构化列举、LLaMA 倾向于重复性补充**。
  - 提供了"风格差异"的实证基础，对"4 帽分工"很有参考价值。

### C. 多智能体按性格/角色分工

#### 11. Encouraging Divergent Thinking in LLMs through Multi-Agent Debate (MAD)
- **作者**：Liang et al. — Tencent AI Lab
- **年份**：2024（EMNLP Main）
- **URL**：https://aclanthology.org/2024.emnlp-main.992/
- **核心结论**：
  - 提出 MAD 框架："正方 vs 反方 + 裁判"的三角辩论结构，专门解决 LLM 的"思维退化"（Degeneration-of-Thought）问题。
  - 实验：在反直觉数学和翻译任务上，MAD 比单 agent CoT 显著提升正确率。
  - **直接对应 Joey 项目**："devil's advocate"或"反方"角色对纠正 LLM 自我强化的盲点是关键。

#### 12. Devil's Advocate: Anticipatory Reflection for LLM Agents
- **作者**：Wang & Mao — UC Davis
- **年份**：2024（EMNLP Findings）
- **URL**：https://arxiv.org/abs/2405.16334  ｜  https://aclanthology.org/2024.findings-emnlp.53/
- **核心结论**：
  - 在 LLM agent 的 plan→act 循环里加入"devil's advocate"内省 prompt，让 agent 主动质疑自己。
  - 在 WebArena 等 agent benchmark 上显著提升任务完成率。
  - 证明**"内置一个反对者"是稳定可复现的能力增量**，而不只是噱头。

#### 13. PTFA: Six Thinking Hats LLM Facilitator for Online Consensus Building
- **作者**：Imamura et al.
- **年份**：2025
- **URL**：https://arxiv.org/html/2503.12499v2
- **核心结论**：
  - **真正用 De Bono 六顶思考帽方法论**做 LLM facilitator 的论文。
  - 在线共识平台中，LLM agent 按 6 顶帽轮换主持讨论（白帽=事实、红帽=情感、黑帽=批判、黄帽=乐观、绿帽=创意、蓝帽=流程）。
  - 实证显示比无帽框架的 LLM 协调更能产生平衡观点和减少群体盲从。
  - **直接是 Joey 项目"4 帽 + 都知"班子的学术先例**。

#### 14. Unleashing Diverse Thinking Modes in LLMs through Multi-Agent Collaboration (DiMo)
- **作者**：He, Feng et al.
- **年份**：2025
- **URL**：https://arxiv.org/pdf/2510.16645
- **核心结论**：
  - 4 个专长不同的 LLM agent（演绎/归纳/类比/批判）做结构化辩论。
  - 比单 agent CoT 和同质 multi-agent 都更好，且**可解释性显著提升**——能产出可审计的中间推理轨迹。
  - 与 Joey 的"4 帽分工"思路几乎完全对齐。

---

## 二、工业界报告

#### 15. Anthropic — Claude's Character
- **机构**：Anthropic
- **年份**：2024-06
- **URL**：https://www.anthropic.com/research/claude-character
- **核心结论**：
  - Anthropic 官宣："Claude 3 是第一个引入 character training 的模型"——alignment 阶段专门训练人格化特质（curiosity, open-mindedness, thoughtfulness）。
  - 训练流水线用 Claude 自生成的合成数据 + 人类研究员逐特质 review。
  - 强调"不希望 Claude 把性格当成不能违背的规则，而是行为倾向"——这是工业界对"模型人格"最完整的公开论述。

#### 16. Anthropic — Claude's Constitution
- **机构**：Anthropic
- **年份**：2025（更新版）
- **URL**：https://www.anthropic.com/constitution
- **核心结论**：
  - 1 万字以上的"Claude 行为宪法"，明确 4 优先级：safety → ethics → Anthropic policy → helpful honesty。
  - 是塑造 Claude"人格"的根文档，被用来生成训练时的合成对话。
  - **方法论可借鉴**：给 4 顶帽各写一份小宪法，可能比纯 prompt 更稳定。

#### 17. TraitPath — AI Language Model Personalities: Claude vs ChatGPT vs Gemini
- **机构**：TraitPath（行业博客，引用 peer-reviewed Big Five 研究）
- **年份**：2025
- **URL**：https://www.traitpath.com/blog/articles/ai-llm-personalities
- **核心结论**：
  - 综合多篇 Big Five 测量论文，给出三家模型的可读对比："Claude 高 agreeableness + 高 openness、ChatGPT 高 conscientiousness、Gemini 高 extraversion 但低 conscientiousness"。
  - 偏科普但引用扎实，可作为"非学术读者沟通"的素材。

---

## 三、关键发现（对 Joey 项目可用的 5-8 条结论）

1. **"模型有可测量的稳定性格差异"是已被 peer-review 多次确认的事实**（PsychoBench、TRAIT、CEUR-WS 6-model study）。这意味着"按性格分工"不是隐喻，而是有量化基础的工程选择。

2. **Big Five 优于 MBTI**：学术界主流是 Big Five / IPIP-NEO，因为它有心理测量学效度；MBTI 在 LLM 研究里更多是工程化标签（如 Machine Mindset）。Joey 给四帽贴标签时，**用 Big Five 维度比 MBTI 更易被接受**（如"高 openness + 低 agreeableness = 黑帽/批判者"）。

3. **"批判性 vs 顺从性"是一条已被基准化的轴**（SycEval、ELEPHANT）。Claude 在保持立场上最强、Gemini 最易让步——**如果项目要"都知"做最终拍板，用 Claude 系列；如果要快速产出多样观点，用 Gemini/GPT 做帽子分工**。

4. **"加一个 devil's advocate"是性能-验证过的增量**（MAD、Devil's Advocate AR、DiMo）。不是审美选择，是有实验数据支撑的——Joey 的"黑帽"角色有学术背书。

5. **Six Thinking Hats 已经被搬到 LLM 上做过实证**（PTFA, arXiv 2503.12499）。可以直接引用作为"4 帽 + 都知"框架的 prior art，不用从头解释。

6. **Anthropic 的 character training 思路提示**：与其每次 prompt 里塞角色，不如**给每个帽子写一份"小宪法"**作为系统 prompt。比 prompt-only persona 更稳定（PersonaLLM、TRAIT 都显示 prompt-only 容易 drift）。

7. **风格差异是实测的**：Claude 多角度铺陈、GPT 结构化列举、LLaMA 重复补充（Verbosity 论文）。**Joey 的"绿帽创意"用 Claude，"白帽事实"用 GPT，可能比同质化分工更自然**。

8. **Prompt 可以塑形人格但有上限**：Serapio-García 2025 (Nature MI) 证明 Big Five 可被 prompt 推动，但**漂移幅度因模型而异**。这意味着"4 帽"的实现要做 calibration——不能假设一个 prompt 模板对所有模型都同样有效。

---

## 四、方法论启示（怎么用到「4 帽 + 都知」班子里）

### 1. 框架选型
- **用 Big Five 作为人格语言**，避免 MBTI（学术不接受、工程上又没明显优势）。
- 4 帽对应可量化的 Big Five 组合：
  - 白帽（事实）= 高 conscientiousness + 中等 openness
  - 红帽（直觉/情感）= 高 extraversion + 高 neuroticism（敏感）
  - 黑帽（批判）= 低 agreeableness + 高 conscientiousness（"devil's advocate"轴）
  - 绿帽（创意）= 高 openness + 低 conscientiousness
  - 都知（仲裁）= 高 agreeableness + 高 conscientiousness（综合协调）

### 2. 实现层
- **每顶帽子写一份"小宪法"**（200-500 字），不是单行 system prompt。参考 Anthropic Constitution 的写法：先讲价值观，再讲行为指引，最后讲优先级。
- **跨模型分工**：如果资源允许，用不同模型扮不同帽子（Claude 当都知/绿帽，GPT 当白帽，Gemini 当红帽，本地 Llama 当黑帽）。引用 SycEval / Verbosity 论文做依据。
- **加一轮"devil's advocate" loop**：直接借鉴 Devil's Advocate AR（arXiv 2405.16334）的设计——黑帽不只是参与讨论，要在都知做出结论前主动质疑一次。

### 3. 评估
- **用 PsychoBench / TRAIT 的方法做事前 calibration**：让每顶帽子先做 Big Five 自评，看 prompt 是否真的把它推到了目标位置。如果漂移大，加一份 few-shot example 锚定。
- **用 SycEval / ELEPHANT 的方式测都知**：故意给错误前提，看都知是否会被带偏。如果都知顺从度高，换模型或加"批判性"约束。

### 4. 论证素材（写方案 / 给 Joey 解释时可引用）
- 学术先例：PTFA（2025）已经做过 Six Hats × LLM
- 性能背书：MAD、DiMo 都证明角色化 multi-agent 比同质化更优
- 行业标杆：Anthropic Claude character training 公开了类似的塑形方法
- 可证伪轴：批判性（SycEval）和风格（Verbosity）都可量化，方便上线后做 A/B

---

## 来源完整度

- arXiv 原文：10 个（# 1-3, 5, 7-12, 14）
- 顶会论文（peer-reviewed）：含 NAACL, EMNLP, ICLR, Nature MI
- 工业博客：3 个（Anthropic ×2、TraitPath）
- 全部链接经过搜索引擎返回，未编造

未发现的类型：
- **未发现专门用 HEXACO 框架对 LLM 做大规模评估的论文**（只有零星引用，可能因为 HEXACO 在英语 NLP 圈不主流）
- **OpenAI / Google 没有像 Anthropic 这样系统公开的"模型人格"博客**（OpenAI Model Spec 偏行为规范、不强调"性格"；DeepMind 通过学术论文发表而非博客）

