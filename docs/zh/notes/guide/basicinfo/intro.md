---
title: 简介
icon: mdi:tooltip-text-outline
permalink: /zh/guide/basicinfo/intro/
createTime: 2026/03/23 00:55:54
---

# 简介

**DataMind v1.0.0** 是一个 local-first 的 inference-time data plane：让 Agent 在推理过程中写入、整理并读取数据。它把五种数据 surface 接到同一个 Agent 系统里，并用独立的 StoreAgent / RetrieveAgent 角色控制写入和读取：

| 能力 | 作用 | 默认后端 |
|---|---|---|
| **KB（RAG）** | 基于语义 + 词法的文档检索 | Chroma + BM25（RRF 融合） |
| **Graph** | 实体查找与多跳图谱遍历 | NetworkX（JSON 持久化） |
| **Database** | 自然语言 → SQL 查询 | SQLAlchemy（SQLite / MySQL / …） |
| **Skills** | Markdown 运维手册 + 安全代码技能（计算器、单位换算等） | `.claude/skills/<name>/SKILL.md` |
| **Memory**（scope-typed） | 三层 scope（global / profile / session）+ 类型化 kind；短期对话缓冲 + 长期语义记忆 | SQLite + 向量召回 |

所有 surface 都经过共享的 **Hooks**：拦截每次 tool 调用，支持 Allow / Deny / AskUser / Rewrite，并提供 destructive SQL 拦截、路径白名单和可校验 audit log。

**Agent 主循环** 自行选择工具、能从工具错误中恢复、全程流式输出。你**不用**硬编码哪类问题走哪个能力。

## v1.0.0 稳定基线

v0.1 是一个 LlamaIndex `FunctionAgent`，所有能力都硬绑在全局 `AppState` 里。v1.0.0 把系统重塑为一个可审计的数据平面：

- **Protocol + Registry 内核**——每个能力背后是一个 `Protocol`（接口）+ `Registry`（注册表）。新增一个 DB 方言 / Embedding 提供商 / 检索策略都是**单文件改动**，核心零改动。
- **双角色边界**——StoreAgent 只暴露写入工具并返回 receipt；RetrieveAgent 只暴露读取/utility 工具并返回 evidence，边界在代码 dispatch 前强制执行。
- **可插拔 Agent loop**——两个互换的后端（`native` 走 Anthropic 或 OpenAI-compatible 协议 / `sdk` 走 claude-agent-sdk + CCR）共用一份工具表 + 一套 SSE 协议，一个 ENV 切换。
- **真 SSE 流式**——不是旧版的"字符切片伪流式"。每个请求有自己的 `RequestContext`，`trace_id` 全链路注入到工具调用和 audit log。

v1.0.0 的稳定 core 是 `native` backend + 本地 profile 存储；SDK/CCR、远程数据库和自定义 provider 是 integration 路径，需要在目标环境单独验证。完整的稳定 API、支持矩阵、安全边界和术语说明见：

- [稳定 API（STABLE_API.md）](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/STABLE_API.md)
- [native / SDK 支持矩阵](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SUPPORT_MATRIX.md)
- [公网部署安全边界](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SECURITY_BOUNDARIES.md)
- [术语与概念](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/CONCEPTS.md)

旧版未删除——旧 `main.py` / `server.py` / `modules/` 仍可运行，可以随时对照两种实现。

## 新目录速查

```
datamind/
├── agent/              # agent loop + 系统提示 + 能力装配
├── capabilities/
│   ├── embedding/      # OpenAI 兼容 + HuggingFace
│   ├── kb/             # Chroma + simple/multi_query/hybrid 三个检索器
│   ├── graph/          # NetworkX graph store
│   ├── db/             # SQLAlchemy + SQLite/MySQL 方言 + NL2SQL
│   ├── memory/         # 短期滚动 + SQLite 长期记忆（scope-typed）
│   ├── skills/         # SKILL.md 加载器 + code skills
│   ├── ingest/         # 对话式 ingest（kb_add_file / db_import_csv / ...）
│   └── hooks/          # PathAllowlist / DestructiveSql / AuditLog
├── core/               # Protocol / Registry / Config / Logging / Tool / Hooks
├── scripts/            # 每个能力一个 hello_*.py 真实冒烟脚本
├── cli.py              # `python -m datamind ...`
├── server.py           # FastAPI + 真流式 SSE
└── tests/              # 161 个通过，5 个可选 SDK 测试跳过
```

每个能力都配了一个 `hello_<cap>.py`，会连真实网关跑端到端测试。详见[快速开始](./install.md)。
