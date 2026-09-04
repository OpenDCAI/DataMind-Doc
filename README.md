# DataMind-Doc

The official documentation site for [DataMind](https://github.com/OpenDCAI/DataMind) v1.0.0, built with [VuePress 2](https://vuepress.vuejs.org/) and [vuepress-theme-plume](https://theme-plume.vuejs.press/).

The stable v1.0.0 baseline is the native backend with local profile storage,
the StoreAgent / RetrieveAgent role boundary, and the Python / HTTP stable API.
SDK/CCR and remote database adapters are integration paths that need separate
validation in the target environment.

## Install

```sh
npm i
```

## Usage

```sh
# 启动开发服务（热更新，编辑 markdown 后自动刷新）
npm run docs:dev

# 构建生产包（上传 GitHub Pages 前测试有无 bug）
npm run docs:build

# 本地预览生产服务
npm run docs:preview
```

## Documentation structure

```
docs/
├── en/notes/guide/          # English documentation
│   ├── basicinfo/           #   intro, install, architecture, release
│   ├── modules/             #   RAG, GraphRAG, Database, Skills, Memory
│   ├── benchmark/           #   benchmark runner and evaluation
│   └── advanced/            #   configuration and data format
├── zh/notes/guide/          # Chinese mirror with the same structure
└── .vuepress/
    ├── config.ts            # VuePress site configuration
    ├── plume.config.ts      # theme configuration
    ├── navbars/             # navigation configuration
    └── notes/               # sidebar configuration
```

The English and Chinese pages are kept in sync.

## Markdown frontmatter

Each Markdown page starts with frontmatter:

```yaml
---
title: Page title         # title shown in the sidebar
icon: carbon:idea         # icon from https://icon-sets.iconify.design/
permalink: /en/guide/... # stable, unique URL
---
```

## Deploy to GitHub Pages

`.github/workflows/docs-deploy.yml` builds and deploys the site with GitHub
Actions on every push to `main`. The public site is
[opendcai.github.io/DataMind-Doc](https://opendcai.github.io/DataMind-Doc/).
You can also run the `Deploy Docs` workflow manually.

Key settings:

1. Keep `base` in `docs/.vuepress/config.ts` set to `'/DataMind-Doc/'`.
2. In repository Settings → Pages, choose `GitHub Actions` as the source.
3. Keep `hostname` set to the canonical site URL for sitemap / SEO.

The [v1.0.0 release guide](https://opendcai.github.io/DataMind-Doc/en/guide/basicinfo/release/) links to the stable API, support matrix, concepts, and public deployment boundaries.

## References

- [VuePress](https://vuepress.vuejs.org/)
- [vuepress-theme-plume](https://theme-plume.vuejs.press/)
