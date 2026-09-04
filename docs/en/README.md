---
pageLayout: home
externalLinkIcon: false
config:
  -
    type: hero
    full: true
    background: tint-plate
    hero:
      name: DataMind
      tagline: Composable inference-time data framework (v1.0.0 stable · pip install datamind==1.0.0)
      text: "StoreAgent writes. RetrieveAgent reads. Evidence comes back."
      actions:
        -
          theme: brand
          text: Introduction
          link: /en/notes/guide/basicinfo/intro.md
        -
          theme: brand
          text: Quick Start
          link: /en/notes/guide/basicinfo/install.md
        -
          theme: alt
          text: v1.0.0 release ↗
          link: https://github.com/OpenDCAI/DataMind/releases/tag/v1.0.0
  -
    type: features
    features:
      -
        title: KB (Hybrid RAG)
        icon: carbon:search-locate
        details: Chroma + BM25 fused with Reciprocal Rank Fusion. Three pluggable strategies (simple / multi_query / hybrid). Multi-profile isolation.
      -
        title: Graph
        icon: carbon:chart-relationship
        details: NetworkX knowledge graph with JSON persistence. Multi-hop traversal with optional relation filters; Neo4j plug-in ready.
      -
        title: Database (NL2SQL)
        icon: carbon:data-table
        details: SQLite and MySQL out of the box (Postgres via one provider file). Three-layer safeguard keeps read-only truly read-only.
      -
        title: Skills
        icon: carbon:tools
        details: "SDK-style .claude/skills/<name>/SKILL.md manifests for knowledge skills; safe Python code skills (calculator, unit conversion, …)."
      -
        title: Memory (scope-typed)
        icon: carbon:ai-status-in-progress
        details: Three scopes (global / profile / session) plus typed kinds. Short-term rolling window + SQLite long-term with cosine recall + live LLM fact extraction.
      -
        title: Hooks (sandboxed)
        icon: carbon:security
        details: HookChain intercepts every tool call — Allow / Deny / AskUser / Rewrite. Built-in destructive-SQL gate, path allow-list, tamper-evident audit log.
---
