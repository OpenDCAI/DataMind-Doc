---
title: 架构设计
icon: material-symbols:auto-transmission-sharp
permalink: /zh/guide/basicinfo/architecture/
createTime: 2026/09/04 21:41:09
---

# v1.0.0 架构设计

DataMind 不是把所有能力塞进一个“万能 Agent”，而是由三个边界清晰的部分组成：共享的 data plane、两个 role-scoped Agent，以及可替换的模型和 provider 适配层。

## 一张图

```text
                                  ┌────────────────────────────────────┐
                                  │ Model gateway                      │
                                  │ Anthropic / OpenAI-compatible      │
                                  │ native loop  |  SDK + CCR (可选)   │
                                  └────────────────▲───────────────────┘
                                                   │ complete / stream
┌──────────────────────┐             ┌─────────────┴─────────────┐
│ CLI · HTTP · Python  │────────────▶│ DataMind facade           │
│ /api/chat (SSE)      │             │ shared services + profile │
└──────────────────────┘             └─────────────┬─────────────┘
                                                   │
                         ┌─────────────────────────┴─────────────────────────┐
                         │                                                   │
              ┌──────────▼──────────┐                             ┌──────────▼──────────┐
              │ StoreAgent          │                             │ RetrieveAgent       │
              │ ToolAccess.WRITE    │                             │ READ + UTILITY      │
              │ 写入 + receipt       │                             │ 读取 + evidence     │
              └──────────┬──────────┘                             └──────────┬──────────┘
                         │                                                   │
                         └────────────────┬──────────────────────────────────┘
                                          ▼
                         ┌────────────────────────────────────┐
                         │ HookChain · tool dispatch           │
                         │ Allow / Deny / AskUser / Rewrite   │
                         │ post-hook: audit log               │
                         └────────────────┬───────────────────┘
                                          ▼
                         ┌────────────────────────────────────┐
                         │ Shared capability services          │
                         │ KB · DB · Graph · Skills · Memory  │
                         └────────────────────────────────────┘
```

### 共享 data plane

`DataMind` facade 持有一组长生命周期服务和当前 profile。StoreAgent、RetrieveAgent 不各自创建数据库、向量库或图谱，而是共享同一组 service、registry、context 和日志，因此写入后可以在后续读取请求中被同一 profile 发现。

### 两个明确的角色

- **StoreAgent** 只拿到 `ToolAccess.WRITE` 工具。它负责把对话、文件、CSV 或结构化记录写进 KB、DB、Graph 或 Memory，并返回 `IngestReceipt`，而不是把写入结果藏在自然语言里。
- **RetrieveAgent** 只拿到 `READ + UTILITY` 工具。它负责查询、检索、遍历、技能调用和记忆召回，输出统一的 `Evidence`，让答案能够回溯到来源。

角色边界在 registry 和 dispatch 层由代码强制执行；模型即使提出了另一个角色没有的工具名，也不会获得该工具的执行权限。

### HookChain 是跨切面控制点

每次 tool 调用都会经过 HookChain。pre-hook 可以 `Allow`、`Deny`、`AskUser` 或 `Rewrite`，post-hook 可以写入结构化审计日志。内置规则覆盖 destructive SQL、路径白名单、工具访问控制和可校验的 audit log。Hooks 是安全与治理层，不是第六个 data surface。

## 两条请求路径

```text
写入请求（DataMind.ingest / POST /api/store）
  → StoreAgent loop
  → WRITE registry
  → HookChain.pre
  → surface handler
  → IngestLedger + receipt
  → StoreAgent result

读取请求（DataMind.query / POST /api/ask /api/chat）
  → RetrieveAgent loop
  → READ + UTILITY registry
  → HookChain.pre
  → surface handler
  → evidence normalizer
  → RetrieveAgent answer + evidence
```

`POST /api/store` 是显式写入入口；`POST /api/ask` 返回一次性 JSON；`POST /api/chat` 是 RetrieveAgent 的 SSE 流式入口。三者都在请求级别绑定 `RequestContext` 和 `trace_id`。

写入和读取共享数据服务，但不是共享权限：一次 ingest 产生的 receipt 记录“写入了什么、落在哪个 surface、何时完成”；一次 query 产生的 evidence 记录“答案引用了哪些 source ref”。这两个对象是跨请求传递事实的稳定边界。

## Agent 如何装配

`build_datamind(settings)` 是 v1.0.0 的 canonical builder，装配顺序如下：

1. 根据 `llm.protocol` 创建模型 client。内部 NL2SQL、query rewrite、memory fact extraction 和 graph extraction 复用同一 client contract。
2. 创建 embedding、KB、DB、Graph、Skills、Memory 和 ingest services，收集到 `AgentServices`。
3. 构建完整的 `ToolSpec` catalogue，再按 `ToolAccess` 生成 StoreAgent 与 RetrieveAgent 的 role-scoped registry。
4. 给写入工具加 receipt wrapper，创建 HookChain、system prompt 和两个 agent loop，最后交给 `DataMind` facade。

```python
from datamind.agent import build_datamind
from datamind.config import Settings

system = await build_datamind(Settings())
try:
    await system.ingest("记住：周报使用中文。")       # StoreAgent
    result = await system.query("周报应该使用什么语言？")  # RetrieveAgent
finally:
    await system.aclose()
```

`build_agent()` 保留为兼容别名，但现在返回只读的 RetrieveAgent。需要明确写入权限时使用 `build_store_agent()`，或直接使用 `build_datamind()`。

## 模型与 loop 层

模型和 Agent loop 通过 `datamind.core.protocols` 的 `TextModelClient` / `ToolCallingModelClient` 解耦：

| 实现 | 线路 | 默认用途 |
|---|---|---|
| `AnthropicModelClient` | Anthropic `/v1/messages` | 默认 native loop |
| `OpenAIChatCompletionsModelClient` | OpenAI `/v1/chat/completions` | OpenAI-compatible native loop |

| 文件 | 职责 |
|---|---|
| `agent/base.py` | `AgentEvent`、`AgentLoopConfig`、`AgentLoopProtocol` 等公共契约 |
| `agent/loop_native.py` | 协议中立的默认循环，包含 Hook、预算、evidence/receipt 收集 |
| `agent/loop_openai.py` | OpenAI wire path，复用 native 执行语义 |
| `agent/loop_sdk.py` | 可选的 Claude Agent SDK + CCR / local MCP 适配 |

不同 loop 都提供 `run_turn`、`stream_turn` 和统一事件：`text`、`tool_use`、`tool_result`、`error`、`done`。因此 HTTP SSE、CLI 和 Python 调用不需要感知底层 provider。

## Core 层

`datamind.core` 放置跨能力稳定契约，而不是具体后端：

| Protocol / contract | 作用 |
|---|---|
| `TextModelClient` / `ToolCallingModelClient` | 文本补全和工具调用 |
| `EmbeddingProvider` | 文本与查询向量化 |
| `VectorStore` / `Retriever` | 向量存储与检索 |
| `GraphStore` | 三元组写入、实体搜索、多跳遍历 |
| `DatabaseDialect` | 建表、描述、只读执行和 destructive 判断 |
| `MemoryStore` | 保存、召回、遗忘和 namespace 管理 |
| `DataSurface` / `ToolAccess` | surface 类型和角色权限 |
| `SourceRef` / `Evidence` | 来源和可回溯证据 |
| `IngestReceipt` / `InferenceResult` | 写入回执和推理结果 |

`ToolSpec(name, description, input_schema, handler)` 是 loop 的最小工具单元。完整 catalogue 只创建一次，role registry 通过 metadata 做筛选，dispatch 前再由 `assert_access` 做一次权限校验。`RequestContext`、错误层级、结构化日志和 HookChain 也位于 core，保证 CLI、HTTP、SDK 使用同一套语义。

## 五个 data surface

| Surface | 写入 | 读取 | 默认实现 |
|---|---|---|---|
| **KB / RAG** | 文件、文本、chunks | 混合检索、来源过滤 | Chroma + BM25 + RRF |
| **Database** | CSV / records / schema | 只读 SQL、schema、NL2SQL | SQLAlchemy + SQLite；MySQL 可选 |
| **Graph** | triples、文本抽取 | entity、neighbors、多跳 traverse | NetworkX + JSON |
| **Skills** | `SKILL.md`、manifest、代码 skill | SOP 语义搜索和安全 utility | profile skills + 内置计算器/单位换算 |
| **Memory** | facts、preferences、decisions、对话 | scope-aware recall / list | 短期滚动窗口 + SQLite 长期记忆 |

`capabilities/ingest` 把 KB、DB、Graph 的写入动作串到同一个 receipt ledger；`capabilities/hooks` 则是所有 surface 共用的治理层。每个 capability 继续遵循 `service.py`、`tools.py`、`providers/` 的边界，替换后端不需要改 Agent loop。

## 配置与 profile

```text
Settings
├── llm          # api_base / api_key / protocol / model / fallback_*
├── embedding    # provider / api_base / api_key / model / batch_size
├── retrieval    # strategy / top_k / chunk_size / chunk_overlap / rerank
├── graph        # backend / dsn / embed_entities
├── db           # dialect / dsn / read_only / row_limit / query_timeout_s
├── memory       # backend / dsn / short_term_turns / long_term_enabled
├── data         # profile / base_dir → data_dir + storage_dir
├── agent        # backend / max_turns / token and wall-clock budgets
└── hooks        # enabled / destructive_sql / path_allowlist / audit_log
```

环境变量使用双下划线分层：

```bash
DATAMIND__AGENT__BACKEND=native
DATAMIND__LLM__PROTOCOL=anthropic
DATAMIND__DATA__PROFILE=customer_a
```

切换 profile 会同时切换 `data/profiles/<profile>/` 和 `storage/<profile>/`。Profile 是数据隔离和命名空间，不是身份认证；公网部署仍需在反向代理或网关层补充认证、TLS、限流和网络隔离。

## 稳定性边界

v1.0.0 的稳定 core 是 native backend + 本地 profile 存储。SDK/CCR、远程数据库方言和自定义 provider 属于 integration 路径，需要在目标环境单独验证。稳定 API、支持矩阵和公网安全边界见：

- [稳定 API](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/STABLE_API.md)
- [native / SDK 支持矩阵](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SUPPORT_MATRIX.md)
- [公网部署安全边界](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SECURITY_BOUNDARIES.md)
- [术语与概念](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/CONCEPTS.md)

旧目录仍保留用于迁移和对照，但不属于 v1.0.0 的稳定 API。新增集成应优先使用 `build_datamind`、`DataMind.ingest/query`、`Evidence`、`IngestReceipt` 和 `core` protocols。
