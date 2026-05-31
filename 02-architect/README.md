# 🏗️ 架构师视角

> 如何把 Feature 转化为真实的 AI 架构

## 核心命题

- 选型决策：Multi-Agent vs Prompt-Level vs Workflow Automation
- Agent 框架选型：LangGraph / AutoGen / CrewAI / 自研
- Tool 设计：提取、评估、修正、读写的接口边界
- Harness 设计：工具白名单 + 操作校验 + 人工 Gate
- 上下文工程：Prompt 模板、To-Do List 优化、记忆管理
- 查询改写与路由：首次生成 vs 多轮补充的链路区分
- 并行化：意图识别、多 Tool 并发调用
- 推理优化：batching、KV Cache、vLLM
- Skills 体系：执行轨迹采集 → 优化 → 自进化

## 决策记录

每条架构决策应包含：
- 背景（为什么要做）
- 方案对比（至少2个选项）
- 决策与理由
- 代价/取舍
- 验证方式

## 记录清单
