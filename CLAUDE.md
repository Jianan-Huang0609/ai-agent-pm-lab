# AI Agent PM Lab — Claude 上下文

## 项目定位
这是 Jianan 的个人精进项目，覆盖 AI Agent 产品从 PM 到 BD 的全角色思考。

## 目录导航
- `00-context/` — 🧠 项目记忆/唯一真相源（**每次会话先读**）
- `01-pm/` — 用户痛点 → AI Feature
- `02-architect/` — Feature → AI 架构设计（含 `prompts/` Prompt 原文、`decisions/` ADR）
- `03-po/` — 监控指标 → KPI 优化（含 CER 真实指标与 Badcase）
- `04-bd/` — BD → 创业拓展
- `05-interview-prep/` — Agent 面试题
- `06-templates/` — 可复用模板

## 真实项目（讨论/面试地基）
- **CER**：医疗器械法规文档（MEDDEV 2.7/1）差异定位自动化，核心项目。
- **R·Agent**：CER/PTR 自动化 pipeline 统称；tool 代号 Lefa/Refa（共享 domain，各自编排 app）。
- **架构一句话**：确定性骨架(script+config+skill) + Skill 叶子 + 窄 Agent（高判断/低确定性环节，带 HITL）。
- **分层**：L2 纯 LLM 调用 / L3 rule 后处理 / Stage 0…N（Stage 1=表格分流）。红线指标 = 漏检率 = 1−recall。
- 细节见 `00-context/memory-export-2026-05-31.md`。

## 每次会话
1. 先扫描上述目录，了解当前积累阶段
2. 根据 Jianan 的意图引导到对应角色视角
3. 讨论结束后，新产生的思考/决策写入对应目录文件
4. 文件名格式：`YYYY-MM-DD-主题.md`
5. 使用 `06-templates/reflection-template.md` 作为记录模板

## 核心原则
- 讨论落地：每次对话必须有产出物（文件）
- 角色明确：标注每条思考属于哪个角色视角
- 交叉引用：架构决策关联 PM 需求、PO 指标关联架构选择
