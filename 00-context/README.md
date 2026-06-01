# 🧠 00-context — 项目记忆与上下文

> 这是整个 Lab 的「唯一真相源」。Claude Code 每次会话先读这里，承接历史而非从零开始。

## 文件
- [memory-export-2026-05-31.md](memory-export-2026-05-31.md) — 跨会话记忆归档（从 Claude.ai 过往会话捞回）。涵盖真实项目 **CER / PTR / R·Agent** 的痛点、架构决策、指标、Badcase、目标，以及《AI PO 工程化手册》6 章摘要。
- [AI_PO_Handbook_full.html](AI_PO_Handbook_full.html) — 《AI PO 工程化手册》全文（6 章，Jianan 著 · 2026）。正文为干净 UTF-8；注：源文件 `<title>` 标签有 pandoc 生成的乱码 `���`（仅标题，不影响正文），待修。摘要见 memory-export 的 §8。

## 真实项目速查（面试/讨论地基）
- **CER**：医疗器械法规文档（MEDDEV 2.7/1）差异定位自动化，核心项目。
- **R·Agent**：CER/PTR 自动化 pipeline 统称；内部 tool 代号 **Lefa / Refa**（共享 domain，各自编排 app）。
- **架构一句话**：确定性骨架（script+config+skill）+ Skill 叶子 + 窄 Agent（只进高判断/低确定性环节，带 HITL）。
- **分层**：L2 纯 LLM 调用 / L3 rule 后处理 / Stage 0…N（Stage 1=表格分流）。
- **红线指标**：漏检率 = 1 − recall。
- **Prompt 原文**：见 [../02-architect/prompts/cer-diff-prompt.md](../02-architect/prompts/cer-diff-prompt.md)
- **Badcase + 指标**：见 [../03-po/cer-badcases-and-metrics.md](../03-po/cer-badcases-and-metrics.md)

## 维护约定
- 新一轮跨会话导出 → 追加 `memory-export-YYYY-MM-DD.md`，不覆盖旧的。
- 标 `（记不清/不确定）` 的点是记忆缺口，后续会话补全后回填。
