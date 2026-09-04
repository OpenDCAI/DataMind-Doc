---
title: Introduction
icon: mdi:tooltip-text-outline
permalink: /en/guide/basicinfo/intro/
createTime: 2026/03/23 00:55:54
---

# Introduction

**DataMind v1.0.0** is a local-first inference-time data plane: agents can
write, organize, and read data while they are answering a request. It connects
five typed data surfaces to one agent system, with separate StoreAgent and
RetrieveAgent roles for write and read authority:

| Capability | What it does | Default backend |
|---|---|---|
| **KB (RAG)** | Semantic + lexical document retrieval | Chroma + BM25 (Reciprocal Rank Fusion) |
| **Graph** | Entity lookup and multi-hop traversal | NetworkX (JSON-persisted) |
| **Database** | Natural language → SQL query | SQLAlchemy (SQLite / MySQL / Postgres) |
| **Skills** | Markdown SOPs + safe code skills (calculator, unit conversion, ...) | `.claude/skills/<name>/SKILL.md` |
| **Memory** *(scope-typed)* | Short-term buffer + SQLite long-term with cosine recall; three scopes (global / profile / session) for multi-tenant isolation | SQLite + embeddings |

Every surface passes through shared **Hooks**: Allow / Deny / AskUser / Rewrite,
destructive-SQL confirmation, path allow-listing, and tamper-evident audit logs.

The **agent loop** picks tools on its own, recovers from errors, and
streams output throughout. You don't hard-code which question routes to
which capability.

## The v1.0.0 stable baseline

v0.1 was a LlamaIndex `FunctionAgent` with everything wired into a global
`AppState`. v1.0.0 reshapes the system into an auditable data plane:

- **Protocol + Registry kernel.** Each capability is defined by a small
  Protocol; concrete implementations register themselves under a name.
  Adding a new SQL dialect / embedding provider / retriever is a
  single-file change with zero impact on the core.
- **Two explicit roles.** StoreAgent exposes write tools and returns receipts;
  RetrieveAgent exposes read/utility tools and returns evidence. The boundary
  is enforced in code before dispatch.
- **Pluggable agent loop.** Two interchangeable backends (`native` over
  Anthropic or OpenAI-compatible protocols, `sdk` over `claude-agent-sdk` +
  CCR) share one tool registry and one SSE event format. Switch with one
  environment variable.
- **Real SSE streaming**, not the v0.1 "character-sliced" simulation.
  Per-request `RequestContext` propagates `trace_id` through every tool
  call and audit record.

The stable core is the `native` backend with local profile storage. SDK/CCR,
remote database dialects, and custom providers are integration paths that must
be validated in the target environment. See the release references:

- [Stable API](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/STABLE_API.md)
- [Native / SDK support matrix](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SUPPORT_MATRIX.md)
- [Public deployment security boundaries](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SECURITY_BOUNDARIES.md)
- [Concepts and terminology](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/CONCEPTS.md)

The legacy v0.1 entry points (`main.py` / `server.py` / `modules/`)
remain in the repo for side-by-side comparison and are still runnable.

## Repository layout

```
datamind/
├── agent/              # agent loop + system prompt + capability assembly
├── capabilities/
│   ├── embedding/      # OpenAI-compatible + HuggingFace
│   ├── kb/             # Chroma + simple/multi_query/hybrid retrievers
│   ├── graph/          # NetworkX graph store
│   ├── db/             # SQLAlchemy + SQLite/MySQL dialects + NL2SQL
│   ├── memory/         # short-term rolling + SQLite long-term (scope-typed)
│   ├── skills/         # SKILL.md loader + code skills
│   ├── ingest/         # conversational ingest into KB / DB / Graph
│   └── hooks/          # PathAllowlist / DestructiveSql / AuditLog
├── core/               # Protocol / Registry / Config / Logging / Tools / Hooks
├── scripts/            # one hello_*.py per capability — real smoke tests
├── cli.py              # `python -m datamind ...`
├── server.py           # FastAPI + real SSE
└── tests/              # 161 passing, 5 optional SDK tests skipped
```

Every capability ships with a `hello_<cap>.py` that runs against a real
gateway. Start with [Install & run](./install.md).
