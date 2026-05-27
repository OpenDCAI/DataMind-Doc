---
title: Introduction
icon: mdi:tooltip-text-outline
permalink: /en/guide/basicinfo/intro/
createTime: 2026/03/23 00:55:54
---

# Introduction

**DataMind** is a unified retrieval agent that wires six knowledge
capabilities into one Claude-powered tool registry:

| Capability | What it does | Default backend |
|---|---|---|
| **KB (RAG)** | Semantic + lexical document retrieval | Chroma + BM25 (Reciprocal Rank Fusion) |
| **Graph** | Entity lookup and multi-hop traversal | NetworkX (JSON-persisted) |
| **Database** | Natural language → SQL query | SQLAlchemy (SQLite / MySQL / Postgres) |
| **Skills** | Markdown SOPs + safe code skills (calculator, unit conversion, ...) | `.claude/skills/<name>/SKILL.md` |
| **Memory** *(v0.3 scope-typed)* | Short-term buffer + SQLite long-term with cosine recall; three scopes (global / profile / session) for multi-tenant isolation | SQLite + embeddings |
| **Hooks** *(v0.3 sandboxed)* | Intercept every tool call: Allow / Deny / AskUser / Rewrite; built-in destructive-SQL gate, path allow-list, tamper-evident audit log | `HookChain` + Merkle audit |

The **agent loop** picks tools on its own, recovers from errors, and
streams output throughout. You don't hard-code which question routes to
which capability.

## What changed since v0.1

v0.1 was a LlamaIndex `FunctionAgent` with everything wired into a global
`AppState`. v0.3 keeps the same six capabilities but reshapes the system
around three structural ideas:

- **Protocol + Registry kernel.** Each capability is defined by a small
  Protocol; concrete implementations register themselves under a name.
  Adding a new SQL dialect / embedding provider / retriever is a
  single-file change with zero impact on the core.
- **Pluggable agent loop.** Two interchangeable backends (`native` over
  the Anthropic SDK, `sdk` over `claude-agent-sdk` + CCR) share one tool
  registry and one SSE event format. Switch with one env variable.
- **Real SSE streaming**, not the v0.1 "character-sliced" simulation.
  Per-request `RequestContext` propagates `trace_id` through every tool
  call and audit record.

v0.3 adds two flagship features on top: **scope-typed Memory** (multi-
tenant isolation without a separate deployment per customer) and
**hook-interposed dispatch** (destructive-SQL confirmation, path
allow-list, tamper-evident audit log).

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
└── tests/              # 133 passing tests, no network
```

Every capability ships with a `hello_<cap>.py` that runs against a real
gateway. Start with [Install & run](./install.md).
