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
      tagline: Agents don't just retrieve. They write new data during inference.
      text: "StoreAgent × RetrieveAgent — one data plane for every source."
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
    title: One data plane. Two roles. Five surfaces.
    description: StoreAgent writes with receipts. RetrieveAgent reads with provenance. The boundary is enforced before a tool reaches the model.
    features:
      -
        title: KB (Hybrid RAG)
        icon: carbon:search-locate
        details: Chroma + BM25 fused with Reciprocal Rank Fusion. Three pluggable strategies (simple / multi_query / hybrid). Multi-profile isolation.
        link: /en/notes/guide/modules/rag.md
        linkText: Explore KB
      -
        title: Graph
        icon: carbon:chart-relationship
        details: NetworkX knowledge graph with JSON persistence. Multi-hop traversal with optional relation filters; Neo4j plug-in ready.
        link: /en/notes/guide/modules/graphrag.md
        linkText: Explore Graph
      -
        title: Database (NL2SQL)
        icon: carbon:data-table
        details: SQLite and MySQL out of the box (Postgres via one provider file). Three-layer safeguard keeps read-only truly read-only.
        link: /en/notes/guide/modules/database.md
        linkText: Explore Database
      -
        title: Skills
        icon: carbon:tools
        details: "SDK-style .claude/skills/<name>/SKILL.md manifests for knowledge skills; safe Python code skills (calculator, unit conversion, …)."
        link: /en/notes/guide/modules/skills.md
        linkText: Explore Skills
      -
        title: Memory (scope-typed)
        icon: carbon:ai-status-in-progress
        details: Three scopes (global / profile / session) plus typed kinds. Short-term rolling window + SQLite long-term with cosine recall + live LLM fact extraction.
        link: /en/notes/guide/modules/memory.md
        linkText: Explore Memory
      -
        title: Safety hooks
        icon: carbon:security
        details: HookChain intercepts every tool call — Allow / Deny / AskUser / Rewrite. Built-in destructive-SQL gate, path allow-list, tamper-evident audit log.
        link: /en/notes/guide/modules/hooks.md
        linkText: Review safeguards
---
