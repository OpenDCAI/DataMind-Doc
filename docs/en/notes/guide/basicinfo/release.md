---
title: v1.0.0 release
icon: carbon:launch
permalink: /en/guide/basicinfo/release/
createTime: 2026/09/04 21:21:31
---

# DataMind v1.0.0

DataMind v1.0.0 is the first stable release. Its core idea is an
**inference-time data plane**: an agent can write new information into shared
surfaces while answering a request, then retrieve it on the next question with
explicit authority boundaries, write receipts, and evidence provenance.

## Release status

| Item | v1.0.0 status |
| --- | --- |
| PyPI | [`datamind==1.0.0`](https://pypi.org/project/datamind/1.0.0/) |
| GitHub Release | [OpenDCAI/DataMind v1.0.0](https://github.com/OpenDCAI/DataMind/releases/tag/v1.0.0) |
| Stable core | `native` backend + local profile storage |
| Verification | `161 passed, 5 skipped`; SQLite demo, HTTP API, and CI verified |
| Python | 3.11+ |

## Two roles

```text
message / file / CSV / relationship
                 │
                 ▼
           StoreAgent ── write receipt ──▶ KB · DB · Graph · Skills · Memory
                 │
              next question
                 ▼
        RetrieveAgent ◀─ evidence + answer ── shared data plane
```

- **StoreAgent** exposes write tools for documents, tables, graph relations,
  skills, and durable memory, then returns an auditable `receipt`.
- **RetrieveAgent** exposes read and utility tools across KB, DB, Graph,
  Skills, and Memory, then returns normalized `evidence`.

This is not a prompt convention: the role boundary is enforced in code before
tool dispatch. Both paths also share the HookChain for path allow-listing,
destructive-SQL protection, and audit logging.

## Install

```bash
pip install datamind==1.0.0
```

Minimum configuration:

```bash
export DATAMIND__LLM__API_BASE=https://your-gateway.example.com
export DATAMIND__LLM__API_KEY=sk-...
export DATAMIND__LLM__PROTOCOL=anthropic  # or openai_chat_completions
export DATAMIND__LLM__MODEL=claude-sonnet-4-6
```

See [Install & run](./install.md) for the complete walkthrough.

## Stability boundary

- `native` with Anthropic `/v1/messages` and OpenAI-compatible
  `/v1/chat/completions` is the stable core.
- `sdk` / CCR, remote MySQL/PostgreSQL, and custom providers are integration
  paths that depend on external processes or deployment-specific setup.
- The Python facade methods `build_datamind`, `ingest`, `query`, `warmup`, and
  `aclose`, plus `/api/ask`, `/api/store`, `/api/chat`, and `/api/upload`, are
  stable API surfaces.

Read the detailed contracts:

- [Stable API](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/STABLE_API.md)
- [Native / SDK support matrix](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SUPPORT_MATRIX.md)
- [Public deployment security boundaries](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SECURITY_BOUNDARIES.md)
- [Concepts and terminology: beyond RAG, ETL, and Agent Memory](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/CONCEPTS.md)

## Public deployment warning

The bundled FastAPI server is a local/private-network interface, not an
authenticated public gateway. Before exposing it publicly, add authentication,
authorization, TLS, an explicit CORS allow-list, rate limits, upload scanning,
profile isolation, and controlled network egress at the edge. A profile name,
`X-Session-Id`, or evidence object is not an identity or authorization claim.

