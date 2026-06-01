---
title: AI Agent PM Lab — 记忆导出（Memory Export）
export_date: 2026-05-31
maintainer: Jianan
purpose: 跨会话记忆归档，供本地仓库维护 + OpenClaw(Sam) 调用
source: Claude.ai 过往会话（见末尾「会话索引」）
provenance_note: >
  本文件由 Claude 检索过往会话片段（snippet）整理。Prompt 与数字尽量逐字从相关原文复制；
  片段不完整或不确定处明确标注「（记不清/不确定/片段截断）」。原私人称呼/姓名已脱敏。
archived_by: Claude Code（2026-06-01 入库，逐字保存）
---

# AI Agent PM Lab — 记忆导出 · 2026-05-31

> 说明：这不是单次会话，而是一次**跨多会话的合并导出**。

---

## 1. SESSION 元信息

- **主题**：跨会话合并 —— 围绕 CER / PTR 法规文档自动化（R·Agent）的 PM / 架构师 / PO / BD 多角色思考；以及 OpenClaw 多 agent 体系、n8n/Skill/Agent 三层分工、Harness Engineering、个人知识库（Jianan Brain）等周边能力建设。
- **大概时间 / 阶段**：2026-01 ～ 2026-05（导出于 2026-05-31）。早期「脚本→agent 架构转型」、中后期「框架选型决策、跨团队/社区分享准备、知识库与 skill 体系」。
- **涉及的项目**：CER（核心）、PTR、R·Agent（CER/PTR 自动化 pipeline 的统称）、OpenClaw（Sam 等多 agent）、Jianan Brain（个人知识库）。

---

## 2. 真实项目细节（CER / PTR / R·Agent）

### 2.1 解决的痛点
- **脚本维护地狱**：现状是「执行脚本 + 业务逻辑 + 前端界面」，硬编码工作流维护成本高。
- **业务细节语义漂移**：医疗器械法规文档措辞的微妙变化可能导致安全/合规判断差异，反复打磨产出仍难稳定可控。
- **输入格式 hardcode**：不同 BU/产品/格式的输入处理不灵活；结论：这是「入口归一化」问题，不该用「整流程 Agent 化」来解。
- **LLM 输出层纠缠**：raw output / format normalization / business logic 三层在主脚本里搅在一起，导致 AB testing 失效、难 debug。

### 2.2 架构（单 / 多 Agent；分工）
当前对 CER 的明确结论（2026-05《CER自动化中的Agent框架选择与设计》会话）：
- **不是「全 Agent vs Script」二选一**，而是按 **结构确定性 × 语义判断复杂度 × 可追溯性** 三轴划分场景。
- **确定性 workflow 当骨架**（script + config + skill）：CER/PTR 主体（MEDDEV 2.7/1 结构固定）、产品描述/规格提取、标签/IFU 一致性、GSPR 对照、章节重组（2.1–2.6 → 3.1）抽成 config + 主脚本分流、跨文档一致性校验。
- **通用工具**：入口归一化层（anti-corruption layer，各引擎输入 → canonical IR）、术语统一、引用格式化、文档对齐/比对引擎、eval + 回测采集 harness。
- **Agent 只进窄环节**：高判断、低结构确定性（且需 HITL）：临床文献相关性筛选评价、A/B 结果集差的语义对齐裁决/歧义消解、利弊-风险论证综合、多法规交叉/gap 分析、无模板可套的新论证新写。
- **OpenClaw 侧多 agent 角色**（来自 MAE Protocol v3）：SAM（编排/协调）、SCOUT（解析/调研）、FORGE（实现）、INK（生成/数字）、LENS（eval/无幻觉检查）、AUX。企业法规项目（R·Agent/CER/PTR）中 **LENS 最重要**（关键质量门 = Q3 无幻觉 + 90–95% 覆盖率）。
- **表格 vs 文本分流**：表格走确定性逐单元格对比（`_compare_table_cells`），纯文本走 LLM（`cer_diff_localize`），提取时把带标记表格替换回文本的 `[表格数据_N]` 占位符。

### 2.3 链路路由（首次生成 vs 多轮补充是否区分）
- **（记不清/不确定）**：过往会话里**没有**找到一条明确的「首次生成 vs 多轮补充」路由规则的逐字定义。相关但不同的内容是：① 「确定性骨架 / 窄 Agent 节点」的路由判别（结构确定+判断低→script+skill，规则可枚举→script+config+skill，结构不定+判断高→Agent）；② 两阶段调用（先做差异分析→再生成带标记文本）。若问的是别的「路由」请补充，才好精确还原。

### 2.4 关键指标
- **ROI / executive presentation 用过的数字**：70% 自动化覆盖率、90%+ 准确率、约 8 天/项目时间节省（多次会话一致出现）。
- **线上红线指标**：**漏检率 = 1 − recall**（在「差异识别」这一步，真实存在的差异里被系统漏掉的比例）。这是监管场景最被罚的指标。
- **质量门**：企业法规项目 Q3 无幻觉 + 90–95% 覆盖率（MAE 角色矩阵）。
- 其余精确数值（如各 BU 实测漏检率、AB 参数）：**（记不清/不确定，过往片段未给出具体数）**。

### 2.5 已知 Badcase / 没解决的问题
CER 差异定位的五类典型 badcase（已沉淀进 Prompt 自检负例）：
- A. 表格整合成段（把含表格的整段当「新增」整体标记）
- B. 定位符为空（`<<- ->>` / `<<~ ~>>` 内无内容）
- C. 原文被改写（去掉定位符后与原文逐字不一致）
- D. 定位符嵌套
- E. 两侧完全一致仍加标记

其他：
- **幻觉/夸大**：捏造「内部测试验证」「早期临床试验数据」这类不存在的结论。
- **OpenClaw 侧**：SAM「说但不做（says but doesn't do）」幻觉输出 —— 正是「窄 Agent + HITL，不在信任未建立前放手」的理由。
- **数值空格误判**：`0.327% ±15%` vs `0.327%±15%` 加空格被误判为变化 → 需 `normalize_cell` 去空格。

### 2.6 用到的技术点
- **本地化定位符（核心约定）**：`<<- 内容 ->>`（对比有/申报无）、`<<+ 内容 +>>`（申报有/对比无）、`<<~ 内容 ~>>`（双方都有但值不同，文本二/文本三同时标记）。
- **A-anchored 对齐**：以产品 A 的结构为锚做语义对齐。
- **两阶段调用**：第一次只做对比分析输出差异列表 → 第二次把差异列表重新拼装带定位符的文本。
- **CoT 字段**：在输出 JSON 第一个字段 `CoT推理过程` 强制先写推理链。
- **Few-shot 正例 / 自检负例**：正例放 prompt 前锁定粒度；负例放 Step 4 自检。
- **占位符**：`[表格数据_N]` 带编号防多表格顺序错乱。
- **Harness Engineering / 可观测性**：run_id 贯穿；分层后处理（raw → normalized → business 各在一个 LLM Skill 调用内部，带 run_id + skill 名）；JSONL append-only 账本 + `runs/{run_id}/` 文件夹 + manifest（记时间/模型版本/prompt 版本/各 stage 状态/gate 裁决/eval 分）；resume（按 run_id + 输入哈希 复用 checkpoint）。
- **回测/记忆**：用户每次确认/修改 = 带标签数据 → 进 exemplar 库（few-shot/RAG）按 section type/产品/BU 索引 + gold set + falsification 集；反向分类修正 → SME gate → schema 长上层。**关键纪律：含有修正信号别「新 prompt 重来一遍」（会污染 context、不 scale）。**
- **微调（SFT）此时不做**：先建采反馈闭环积累干练样本，few-shot 顶着。没微调，模型本身永远不会跑自嗨。

---

## 3. Prompt 原文「最重要」

> 已抽出单独存档于 [[02-architect/prompts/cer-diff-prompt]]（含 cer_diff_prompt 主体、fallback、硬性约束、Jianan Brain AGENTS.md）。
> ⚠️ 第 6 条严格约束片段截断，待本地原稿补全。

---

## 4. 关键决策与结论

- **决策｜CER 用「确定性骨架 + Skill 叶子 + 窄 Agent」，不上全 Agent**：监管要求可复现/可追溯/可审计，runtime 动态编排会让「为什么走这入这条」答不上；不会放大 raw/normalize/business 三层纠缠。
- **决策｜框架不掩盖控制流**：LangGraph/Mastra 等会把 state/orchestration 抽走，而你需要亲手把控分层、run_id、quality gate（即 Harness Engineering）；要用就用 LangGraph（放弃状态图贴合 MAE lifecycle 思路），但「业务核心逻辑 + 薄编排」对 CER 更有效，把 Agent 表面积收到最小，用稳定内部契约协作让底层框架可替换。
- **决策｜input hardcode 用归一化层解，不用 Agent 拆解**：前置 anti-corruption layer（各引擎输入 → canonical IR），下游只认 IR。
- **决策｜现在不微调**：建采反馈闭环积累干练样本，few-shot/RAG 优化，修正信号进 exemplar 库 / gold set / falsification 集，**不动 prompt**。
- **决策｜前端不用自由 chatbox**：用结构化审阅/编辑 gate（OpenClaw auto/confirm/approve），结构化修改=干练训练信号，自由对话=噪声。
- **决策｜Agent 不「定」「审核」，只「写」「初审」**：最终决策、相关法规裁决都给人（人来裁）；管线分歧处，agent 裁后必有人复审/审批；agent 拽出来供人复核，人做出最终裁决。
- **判断｜对「另一个团队 system-level 微调 + 文章聚合 + 仿 example 输出」的评价**：在 CER 上过早（可追溯性更差、法规上微调训成本高一个量级、幻觉不可接受、干练样本都还没有）；但**承认边界**——在「多法规综合」「群体讨论」「结果即时现 schema 模板」这类场景，model-driven 的泛化性不错，那才值得他们方向去；不只 few-shot 优化了微调。
- **判断｜n8n / Skill / Agent 三分法**：n8n =「图已经好」的多步自动化；Agent =「图一边跑一边给」的动态多步；Skill =「一次调用搞定」的能力封装。判别一问：路径是否预先可绘（是→n8n）；是否单次高质量输出（是→skill）；是否需要运行时决策（是→agent）。三者可嵌套（agent 调 skill，n8n 调 agent）。
- **判断｜开发期 vs 运行期承载体分工**：用 Claude Code/Codex 来「建」；用确定性状态机来「管」；Agent SDK 只在窄节点里「想」。别给一个 agent-loop 的自治性；开发期是资产，运行期是负债。
- **经验｜辩论背景的用法**（规划一场 meet-up 分享）：主动找对方立场最强、最有威胁性的版本在它真正能立的领域让给对方；找到 senior 卖买到合作；价值围栏阐释；前 30 分钟以听和提问为主。

---

## 5. 我的目标与改进方向

- **目标**：
  - 用 R·Agent / CER 真实项目应付 **多家 Agent 开发面试**。
  - 把方法论沉淀成**内容**（小红书 / 对外 PR / AI community 技术分享，如 Harness Engineering、三层框架、开发层 vs 运行层编排）。
  - 建成 **Jianan Brain** 个人知识库（GitHub 单一事实源 + 飞书人类可读阅读 + Airtable 半数据 + OpenClaw SCOUT/INK/AUX 做 ingestion/compile/lint）。
  - 把 CER 单点经验**平台化**为「组织级注册和自动化的操作系统」（争资源与 ownership，同时结合创业团队的架构口子）。
  - 长个月**跨轨学习**：数字 agent 编排 + 物理机器人（Maestro × OpenClaw 物理-数字协作）。
- **想精进的方向**：多 agent 编排下可观测/可信；解决 SAM「说但不做」「幻觉」；Harness Engineering（LLM pipeline 工程化：分层后处理 + run_id + eval/回测）；以及把这些沉成可复用 skill 与可分享内容。

---

## 6. 提到过的文件 / 链接 / 外部资源

- 代码/产物：`cer_diff_prompt_optimized.py`、`collect_notes.py`（多平台抓取统一入口）、`CheckpointManager`（回测检查点类，早期 LangGraph 方案）。
- Skill 名：`jianan-presentation-system`、`claw-vibe-project`、`ai-morphing-video`、`skill-creator`（meta-skill）、`codebase-to-course`。
- 规范/架构：MAE Protocol v3（14 节多 agent 工程化规范）、OpenClaw（EventBridge + Lambda + Airtable + Discord；agents = SAM/SCOUT/FORGE/INK/LENS/AUX）、Dream Cycle（夜间记忆整合）、Jianan Brain（GitHub + 飞书 + Airtable）。
- 视觉设计系统：暖米白 `#FBF6EC` 底色 + 克饱和橙 + 锐阳锐（PPT design token：H1 40px / H2 28px / 正文 16px / 画布 1280×720 / 外边距 64px）。
- 外部参考：Karpathy《LLM Wiki》概念、Karpathy《Neural Networks: Zero to Hero》、Raschka《LLMs from Scratch》、Anthropic《Building Effective AI Agents》/ Agent Harness 系列、Anthropic《Harness Engineering》；工具栏 super.dev、Jina Reader、wemp.app 采集、yt-dlp + Whisper、Nano Banana 生图 API、Kling/Runway 视频 API。
- 数据集 / gold set：**（记不清/不确定，过往片段未给出具体名称或大小）**。
- 你写过的「书」：**《AI PO 工程化手册》（AI PO Engineering Handbook · 作者 Jianan · 2026）**（文件 `AI_PO_Handbook_full.html`，已确认，2026-06-01 更新补入）。详见下方 §8。

---

## 7. 悬而未决 / 当时没说清但重要的点

- **「首次生成 vs 多轮补充」的路由**：模板里要求填，但过往会话无明确逐字定义（见 §2.3）。需 Jianan 补一句话定义和触发边界。
- **Agent 边界与框架稳定性**：Agent 该进哪些 CER 子环节的边界仍在演进；底层 Agent 框架仍在迭代；策略是「窄 + 契约协作 + 可替换」，但具体契约 schema 尚未定稿。
- **回测闭环落地度**：exemplar 库 / gold set / manifest / resume 的「方案」已定，但是否已在 R·Agent 真正跑起来、积累了多少干练样本 **（记不清/不确定）**。
- **跨团队张力**：在不对外展现成熟度/争资源 + 在于对 AI community 技术分享，那次准备里讲给对方的口子是否真能撬动架构师认同，结果未在后续会话中追到。
- **企业 Wiki 的人工编译文本**：approval = 审 diff 而非全键，但质量后顾的真实接入/SLA 未定。
- **PTR 细节**：PTR 多数时候与 CER 并发出现，同走「模板 + 各章节 skill」，但 PTR 独有的处理逻辑/badcase 还没着墨 **（记不清/不确定）**。

---

## 8.《AI PO 工程化手册》数字（6 章 · 痛点驱动）— 2026-06-01 补入

> 来源：上传文件 `AI_PO_Handbook_full.html`（约 636 KB，作者 Jianan · 2026）。这是「让 Claude 写的 6 章书」，**内容都来自真实痛点**，是「下一次开项目可复用/复盘」的资产。

### 8.1 定位与读法
- **副标题**：给夹在业务和工程之间的 AI PO —— 能看懂代码但不是工程师，要带团队落 AI 产品，要**定位不被 AI 污染的时刻**。这本只讲「实时才有必要的那层工程能力」（能跟工程师平等对话、定义契约、识别风险）。
- **结构**：9 章「打地基 → 解耦 → 守解耦 → 长期」递进，本册含前 6 章；第 7–9 章（SemVer / Docker / SQL）按需补齐，未含。
- **每章统一模板**：① 痛点导入 → ② 费曼式核心（底层 3–4 原理）→ ③ 要学什么 → ④ 要会什么（核心动作）→ ⑤ 日常操作手册 → ⑥ 行业是怎么说的 → ⑦ 可视化图解 → ⑧ 学会检验（你能讲出来吗）→ ⑨ 精选资源 → ⑩ 本章总结一页纸 → 动手作业。
- **核心自检（读完应能答）**：用什么 Git 工作流、为什么不是另一种；Lefa 内部 Stage 之间怎么解耦、谁打标在哪个 Stage；R·Agent 整个 tool 分几层、domain 能否 import 外部库；测试分几层、LLM 怎么测、evals 与 test 区别；出问题靠什么定位到哪个 Stage；能否用架构师语言讨论自己的项目。

### 8.2 每章 = 一个痛点 + 学会「定」什么

| 章 | 主题（定什么） | 痛点导入（真实场景） | 关键结论 |
|---|---|---|---|
| 1 | Git 工作流（团队协作地基） | 跑完 anchor verification 兴冲冲 push；同事改了 normalize 把校验弄崩；git log 全是 `update/fix/WIP/save/works now`，六个问题一个都答不出 | GitHub Flow（main + 短 feature + PR）；分支 `<type>/<短描述>`；Conventional Commits；私有分支 rebase、PR 用 squash and merge；永远 `--force-with-lease` |
| 2 | LLM Pipeline 解耦（单脚本内部） | 你原话：「输出和 Prompt/前后置/Stage0-1 提取对齐/表格分流都相关，要追全链，**只能在 LLM 里不断加逆向方法补足**」= 三个致命信号 = 「污染链」 | L2 只做**纯 LLM 调用**；每个逆向方法抽成 L3 rule 函数；**中间态物化**（每 Stage 输出落 artifact）；Raw/Final 两段式 schema；指标降了先看 attribution 报告 → L3 触发频率 → 最后才看 prompt；换 LLM 只改 L2 |
| 3 | Hexagonal 架构（整个 tool） | 「单脚本解决了，但 tool 还没有」 | 三层 domain(纯)/adapter(外部·脏)/app(编排)；依赖只能指向 domain；Port vs Adapter；Driver vs Driven；业务规则集中 `domain/rules.py`；**Lefa 与 Refa 共享 domain、各自编排 app**；换 LLM 只改 adapter |
| 4 | 测试分层（守正确性）*（书内章节号误标为「第三章」/3.x）* | 「跑通了 ≠ 测过了」；smoke test 只证「没崩」；边界条件/fallback/Q-Gate 都没答案 | LLM 项目用**测试奖杯非金字塔**；集成测试是主战场；Static/Unit/Integration/E2E + 独立 Evals 目录；**测试=代码 pass/fail（CI 每次）vs 评估=LLM 质量趋势（定期）**；CI 里必 mock LLM，evals 真调；覆盖率核心 80%/整体 70% |
| 5 | 可观测性三件套（守可观测） | 「看不见才是被偷了更可怕」；线上延迟从 2.3s→6.8s、token +40% 无人可见；artifacts 被覆盖找不到是哪一次 | Log/Metric/Trace；structlog + contextvars（trace_id 贯穿）；INFO/WARN/ERROR 严格纪律；指标分业务/系统/基础设施；**现阶段不上 Prometheus**，jq + cron 够用 |
| 6 | CI 基础（守边界·闭环章） | 「规范都写好了，但没人遵守」 | 用 CI 把规范从**文档变强制**（机器可机械检测），形成闭环 = Harness Engineering；约束编码进工具 = 反脆弱增强器 |

### 8.3 这本书补的新硬细节
- **Lefa / Refa**：项目里两个 tool/pipeline 的代号；**共享同一 domain 层、各自编排 app**；CER 与 PTR 的具体对应关系书中未逐字点明（**不臆断**）。
- **分层命名**：L1 / L2 / L3 —— L2=纯 LLM 调用层，L3=rule/后处理函数层（对应早期记的「raw/normalized/business」三层）。
- **Stage 0 / Stage 1 …**：pipeline 阶段；**Stage 1 = 表格分流**（书中举例「质量下降 70% 来自 Stage 1 表格分流、与 prompt 无关」）。
- **anchor verification**：A-anchored 对齐的校验逻辑（与 §2.6 本地化定位符是上下游）。
- **Q-Gate**：集成测试层的质量门。
- **ArtifactStore / `artifacts/`**：中间态落存储；现状以时间戳命名导致「找不到是哪一次运行」——与 §2.6 的 `run_id` 是同一痛点，书用 `run_id` 方案解。

---

## 会话索引（本次导出的主要来源）

| 主题 | 时间 | 链接 |
|---|---|---|
| CER 自动化中的 Agent 框架选择与设计 | 2026-05-23 | https://claude.ai/chat/9a1e6df9-d791-48f4-b1c5-ca01e60e5fdc |
| Agent 架构中 workflow 集成的实践小技（含漏检率/三层分离/resume） | 2026-05-26 | https://claude.ai/chat/dae3f3fa-8aff-477e-95c3-fa3a89c2d314 |
| CER 文档 AI 工具从脚本到 agent 框架的架构转型 | 2026-01-29 | https://claude.ai/chat/b6ddc33d-ad24-4439-b483-37348c7c94c9 |
| 问号回答模糊（CER diff prompt 打磨 + 定位符逐字原文） | 2026-03-06 | https://claude.ai/chat/00676892-65bf-4d4c-b40c-f2bfab19eca9 |
| 30 分钟 AI 工程师面试问题设计与优化 | 2026-01-29 | https://claude.ai/chat/a8675688-45c3-4f49-b965-eca26d7a0cb1 |
| MAE Protocol 多智能体工程化规范 | 2026-03-21 | https://claude.ai/chat/fa902e99-3f56-4d26-9ae1-5b33b4119077 |
| Harness Engineering（OpenAI 家）SDLC 落地 | 2026-03-19 | https://claude.ai/chat/a1b7e48f-dd17-4ffc-b0d7-573346ae086e |
| JN-N8N 自动化工作流分层（n8n/Skill/Agent 三分法） | 2026-04-20 | https://claude.ai/chat/9ff10f26-5c47-4363-8900-046bd30861d0 |
| 用 LLM 搭建个人知识库系统 | 2026-04-05 | https://claude.ai/chat/24df9d01-8725-45e7-ba6c-303315cc9e40 |
| 个人知识库的 Obsidian 实践方案 | 2026-04-14 | https://claude.ai/chat/1556af88-5c84-4f3b-a248-5f120f689a04 |
| GitHub 和飞书知识库的整合方案（AGENTS.md 原文） | 2026-04-06 | https://claude.ai/chat/8e401c65-25c6-4287-9ebb-0e954eefacd0 |
| 多平台内容抓取与摘编工作流（collect_notes.py） | 2026-03-15 | https://claude.ai/chat/b2dcec5b-fde2-4947-bcc3-f2628387c197 |
| 两小时准备分享与剪辑视频的提效框架 | 2026-04-20 | https://claude.ai/chat/0f165be9-ddeb-412b-bbe9-746dcefe605a |
| Markdown 转 HTML 的 PPT 自动化生成系统 | 2026-03-22 | https://claude.ai/chat/793d8e48-3127-48f0-a714-b9d15cc9f380 |

> 维护提示：下次导出新增内容建议追加为 `..._2026-06-xx.md`，并在仓库根 README/CLAUDE.md 留指针。
