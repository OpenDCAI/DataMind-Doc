---
title: v1.0.0 发布说明
icon: carbon:launch
permalink: /zh/guide/basicinfo/release/
createTime: 2026/09/04 21:21:31
---

# DataMind v1.0.0

DataMind v1.0.0 是第一个稳定发布版本，核心定位是 **inference-time data
plane**：Agent 可以在推理过程中把新信息写入共享数据面，并在下一次问题中
读取它，同时保留明确的权限边界、写入回执和证据 provenance。

## 发布状态

| 项目 | v1.0.0 状态 |
| --- | --- |
| PyPI | [`datamind==1.0.0`](https://pypi.org/project/datamind/1.0.0/) |
| GitHub Release | [OpenDCAI/DataMind v1.0.0](https://github.com/OpenDCAI/DataMind/releases/tag/v1.0.0) |
| 稳定 core | `native` backend + 本地 profile 存储 |
| 测试 | `161 passed, 5 skipped`；SQLite demo、HTTP API 和 CI 已验证 |
| Python | 3.11+ |

## 两个角色

```text
消息 / 文件 / CSV / 关系
          │
          ▼
     StoreAgent ── write receipt ──▶ KB · DB · Graph · Skills · Memory
          │
       下一次问题
          ▼
   RetrieveAgent ◀─ evidence + answer ── 共享 data plane
```

- **StoreAgent**：只暴露写入工具，负责文档、表格、图关系、技能和长期
  记忆的入库，并返回可审计的 `receipt`。
- **RetrieveAgent**：只暴露读取和 utility 工具，负责跨 KB、DB、Graph、
  Skills、Memory 查询，并返回统一的 `evidence`。

这不是 prompt 约定：角色边界在工具 dispatch 前由代码强制执行，并且两条
路径共用 HookChain（路径白名单、destructive SQL 保护、audit log）。

## 安装

```bash
pip install datamind==1.0.0
```

最小配置：

```bash
export DATAMIND__LLM__API_BASE=https://your-gateway.example.com
export DATAMIND__LLM__API_KEY=sk-...
export DATAMIND__LLM__PROTOCOL=anthropic  # 或 openai_chat_completions
export DATAMIND__LLM__MODEL=claude-sonnet-4-6
```

完整安装步骤见[安装与运行](./install.md)。

## 稳定边界

- `native` + Anthropic `/v1/messages` 和 OpenAI-compatible
  `/v1/chat/completions` 是稳定 core。
- `sdk` / CCR、远程 MySQL/PostgreSQL 和自定义 provider 是 integration
  路径，依赖外部进程或部署环境，需要自行验证。
- Python facade 的 `build_datamind`、`ingest`、`query`、`warmup`、`aclose`
  以及 `/api/ask`、`/api/store`、`/api/chat`、`/api/upload` 是稳定 API。

详细合约请看：

- [Stable API](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/STABLE_API.md)
- [Native / SDK 支持矩阵](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SUPPORT_MATRIX.md)
- [公网部署安全边界](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/SECURITY_BOUNDARIES.md)
- [术语与概念：区别于 RAG、ETL、Agent Memory](https://github.com/OpenDCAI/DataMind/blob/v1.0.0/docs/CONCEPTS.md)

## 公网部署提醒

内置 FastAPI 服务是 local/private-network 接口，不是带认证的公网网关。
公网部署前必须在边缘层补上认证、授权、TLS、明确的 CORS、限流、上传扫描、
profile 隔离和网络出口控制。不要把 profile 名称、`X-Session-Id` 或 evidence
当成身份凭证或授权依据。

