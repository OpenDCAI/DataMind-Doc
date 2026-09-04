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
      tagline: 面向持续演化 Agent 的数据层
      text: 在推理过程中沉淀可复用、可验证的上下文。
      actions:
        -
          theme: brand
          text: 简介
          link: /zh/notes/guide/basicinfo/intro.md
        -
          theme: brand
          text: 快速开始
          link: /zh/notes/guide/basicinfo/install.md
        -
          theme: alt
          text: v1.0.0 发布 ↗
          link: https://github.com/OpenDCAI/DataMind/releases/tag/v1.0.0
  -
    type: features
    title: 一个数据面，两个角色，五个 surface
    description: StoreAgent 写入并返回 receipt，RetrieveAgent 读取并返回 provenance；边界在工具到达模型前由代码强制执行。
    features:
      -
        title: KB（混合检索）
        icon: carbon:search-locate
        details: Chroma + BM25 用 RRF 融合，三种可插拔策略（simple / multi_query / hybrid），多 profile 隔离
        link: /zh/notes/guide/modules/rag.md
        linkText: 查看 KB
      -
        title: Graph 图谱
        icon: carbon:chart-relationship
        details: NetworkX + JSON 持久化，支持多跳遍历和关系过滤；Neo4j 可直接作为 provider 插入
        link: /zh/notes/guide/modules/graphrag.md
        linkText: 查看 Graph
      -
        title: Database（NL2SQL）
        icon: carbon:data-table
        details: 内置 SQLite / MySQL（Postgres 一个文件即可接入），三层安全闸保障只读
        link: /zh/notes/guide/modules/database.md
        linkText: 查看 Database
      -
        title: Skills
        icon: carbon:tools
        details: SDK 风格 .claude/skills/<name>/SKILL.md 知识型 skill + 代码型 skill（计算器、单位换算等）
        link: /zh/notes/guide/modules/skills.md
        linkText: 查看 Skills
      -
        title: Memory（scope-typed）
        icon: carbon:ai-status-in-progress
        details: 三层 scope（global / profile / session）+ 类型化 kind；短期滚动窗口 + SQLite 长期语义记忆 + LLM 事实抽取
        link: /zh/notes/guide/modules/memory.md
        linkText: 查看 Memory
      -
        title: 安全 Hooks
        icon: carbon:security
        details: HookChain 拦截每次 tool 调用 —— Allow / Deny / AskUser / Rewrite；内置 destructive SQL 拦截、路径白名单、可校验 audit log
        link: /zh/notes/guide/modules/hooks.md
        linkText: 查看安全边界
---
