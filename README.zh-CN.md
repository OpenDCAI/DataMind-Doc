# DataMind-Doc

[DataMind](https://github.com/OpenDCAI/DataMind) v1.0.0 的官方文档站点，使用 [VuePress 2](https://vuepress.vuejs.org/) 和 [vuepress-theme-plume](https://theme-plume.vuejs.press/) 构建。

当前稳定基线是 DataMind v1.0.0：native backend、本地 profile 存储、StoreAgent / RetrieveAgent 角色边界，以及 Python / HTTP 稳定 API。SDK/CCR 和远程数据库适配器属于需要单独验证的 integration 路径。

## 安装

```sh
npm i
```

## 使用

```sh
# 启动开发服务
npm run docs:dev
# 构建生产包
npm run docs:build
# 本地预览生产服务
npm run docs:preview
# 更新 vuepress 和主题
npm run vp-update
```

## 部署到 GitHub Pages

仓库已配置 `.github/workflows/docs-deploy.yml`：推送 `main` 后会自动构建并
通过 GitHub Actions 部署到 [opendcai.github.io/DataMind-Doc](https://opendcai.github.io/DataMind-Doc/)。
也可以在 Actions 页面手动运行 `Deploy Docs`。

关键配置：

1. `docs/.vuepress/config.ts` 中的 `base` 为 `"/DataMind-Doc/"`
2. `settings > Pages > Source` 选择 `GitHub Actions`
3. `hostname` 保持为站点规范 URL，用于生成 sitemap / SEO

## 参考

- [VuePress](https://vuepress.vuejs.org/)
- [vuepress-theme-plume](https://theme-plume.vuejs.press/)
