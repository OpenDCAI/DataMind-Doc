---
title: Architecture
icon: material-symbols:auto-transmission-sharp
permalink: /en/guide/basicinfo/architecture/
createTime: 2026/09/04 21:42:06
---

# v1.0.0 architecture

DataMind is not one do-everything agent. It has three clear boundaries: a shared data plane, two role-scoped agents, and replaceable model/provider adapters.

## One picture

```text
                                  ┌────────────────────────────────────┐
                                  │ Model gateway                      │
                                  │ Anthropic / OpenAI-compatible      │
                                  │ native loop  |  SDK + CCR (opt.)   │
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
              │ writes + receipts   │                             │ reads + evidence    │
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

### The shared data plane

The `DataMind` facade owns one long-lived set of services and the active profile. StoreAgent and RetrieveAgent do not create separate databases, vector stores, or graphs; they share services, registries, request context, and logging. A write can therefore be discovered by a later read in the same profile.

### Two explicit roles

- **StoreAgent** receives only `ToolAccess.WRITE` tools. It writes conversations, files, CSVs, or records into KB, DB, Graph, or Memory and returns an `IngestReceipt` instead of hiding the write outcome in prose.
- **RetrieveAgent** receives only `READ + UTILITY` tools. It queries, retrieves, traverses, invokes skills, and recalls memory, returning normalized `Evidence` so an answer can be traced to its sources.

The role boundary is enforced in registries and dispatch code. If a model asks for a tool owned by the other role, that tool is not executable.

### HookChain is the cross-cutting control point

Every tool call passes through HookChain. Pre-hooks can `Allow`, `Deny`, `AskUser`, or `Rewrite`; post-hooks can emit structured audit records. Built-in rules cover destructive SQL, path allow-listing, tool access, and verifiable audit logs. Hooks are the governance layer, not a sixth data surface.

## Two request paths

```text
Write request (DataMind.ingest / POST /api/store)
  → StoreAgent loop
  → WRITE registry
  → HookChain.pre
  → surface handler
  → IngestLedger + receipt
  → StoreAgent result

Read request (DataMind.query / POST /api/ask /api/chat)
  → RetrieveAgent loop
  → READ + UTILITY registry
  → HookChain.pre
  → surface handler
  → evidence normalizer
  → RetrieveAgent answer + evidence
```

`POST /api/store` is the explicit write endpoint. `POST /api/ask` returns one JSON response, while `POST /api/chat` is the RetrieveAgent SSE streaming endpoint. All three bind a request-scoped `RequestContext` and `trace_id`.

Writes and reads share data services but not authority. An ingest produces a receipt describing what was written, to which surface, and when it completed. A query produces evidence describing which source refs support the answer. These objects are the stable cross-request fact boundary.

## How the agents are assembled

`build_datamind(settings)` is the canonical v1.0.0 builder:

1. Create the model client from `llm.protocol`. NL2SQL, query rewriting, memory fact extraction, and graph extraction reuse the same client contract.
2. Build embedding, KB, DB, Graph, Skills, Memory, and ingest services and collect them in `AgentServices`.
3. Build one `ToolSpec` catalogue, then derive role-scoped registries from `ToolAccess`.
4. Wrap write tools with receipt handling, create HookChain, prompts, and both agent loops, and return the `DataMind` facade.

```python
from datamind.agent import build_datamind
from datamind.config import Settings

system = await build_datamind(Settings())
try:
    await system.ingest("Remember: weekly reports use Chinese.")  # StoreAgent
    result = await system.query("What language should weekly reports use?")  # RetrieveAgent
finally:
    await system.aclose()
```

`build_agent()` remains as a compatibility alias, but it now returns a read-only RetrieveAgent. Use `build_store_agent()` when explicit write authority is needed, or use `build_datamind()` for the complete system.

## Model and loop layers

Models and agent loops are decoupled through `TextModelClient` and `ToolCallingModelClient` in `datamind.core.protocols`:

| Implementation | Wire path | Default use |
|---|---|---|
| `AnthropicModelClient` | Anthropic `/v1/messages` | Default native loop |
| `OpenAIChatCompletionsModelClient` | OpenAI `/v1/chat/completions` | OpenAI-compatible native loop |

| File | Responsibility |
|---|---|
| `agent/base.py` | Shared `AgentEvent`, `AgentLoopConfig`, and `AgentLoopProtocol` contracts |
| `agent/loop_native.py` | Protocol-neutral default loop with hooks, budgets, evidence, and receipts |
| `agent/loop_openai.py` | OpenAI wire path; reuses native execution semantics |
| `agent/loop_sdk.py` | Optional Claude Agent SDK + CCR / local MCP adapter |

Each loop exposes `run_turn`, `stream_turn`, and the same events: `text`, `tool_use`, `tool_result`, `error`, and `done`. HTTP SSE, CLI, and Python callers therefore do not depend on a provider-specific event model.

## Core layer

`datamind.core` contains stable cross-capability contracts rather than concrete backends:

| Protocol / contract | Responsibility |
|---|---|
| `TextModelClient` / `ToolCallingModelClient` | Text completion and tool calling |
| `EmbeddingProvider` | Text and query embeddings |
| `VectorStore` / `Retriever` | Vector storage and retrieval |
| `GraphStore` | Triple writes, entity search, and traversal |
| `DatabaseDialect` | Schema, read-only execution, and destructive-query detection |
| `MemoryStore` | Save, recall, forget, and namespace management |
| `DataSurface` / `ToolAccess` | Surface types and role authority |
| `SourceRef` / `Evidence` | Sources and traceable support |
| `IngestReceipt` / `InferenceResult` | Write receipts and inference results |

`ToolSpec(name, description, input_schema, handler)` is the smallest unit understood by a loop. The complete catalogue is built once; role registries filter it by metadata, and `assert_access` performs a final check before dispatch. `RequestContext`, the error hierarchy, structured logging, and HookChain also live in core so CLI, HTTP, and SDK callers share the same semantics.

## Five data surfaces

| Surface | Writes | Reads | Default implementation |
|---|---|---|---|
| **KB / RAG** | Files, text, and chunks | Hybrid retrieval and source filters | Chroma + BM25 + RRF |
| **Database** | CSV, records, and schema | Read-only SQL, schema, and NL2SQL | SQLAlchemy + SQLite; MySQL optional |
| **Graph** | Triples and extracted text | Entities, neighbors, and multi-hop traversal | NetworkX + JSON |
| **Skills** | `SKILL.md`, manifests, and code skills | SOP search and safe utilities | Profile skills + calculator/unit conversion |
| **Memory** | Facts, preferences, decisions, and turns | Scope-aware recall / list | Rolling short-term + SQLite long-term |

`capabilities/ingest` connects KB, DB, and Graph writes to one receipt ledger. `capabilities/hooks` is the governance layer shared by every surface. Each capability follows the `service.py`, `tools.py`, and `providers/` boundary, so replacing a backend does not require changing the agent loop.

## Configuration and profiles

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

Environment variables use double underscores for nesting:

```bash
DATAMIND__AGENT__BACKEND=native
DATAMIND__LLM__PROTOCOL=anthropic
DATAMIND__DATA__PROFILE=customer_a
```

Switching profiles moves both `data/profiles/<profile>/` and `storage/<profile>/` in lockstep. A profile is a data namespace, not authentication; public deployments still need authentication, TLS, rate limiting, and network isolation at the gateway or reverse proxy.

## Stability boundary

The v1.0.0 stable core is the native backend with local profile storage. SDK/CCR, remote database dialects, and custom providers are integration paths that require validation in the target environment. See the [Stable API](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/STABLE_API.md), [native / SDK support matrix](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SUPPORT_MATRIX.md), [public deployment security boundaries](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SECURITY_BOUNDARIES.md), and [concepts and terminology](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/CONCEPTS.md).

The legacy layout remains available for migration and comparison, but it is not part of the v1.0.0 stable API. New integrations should prefer `build_datamind`, `DataMind.ingest/query`, `Evidence`, `IngestReceipt`, and the `core` protocols.
