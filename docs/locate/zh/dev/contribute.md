# 📜 贡献指南 Contributing

我们欢迎各位 Contributors 参与贡献发展 awesome-nijigen-ai-exp 社区

这篇指南会指导你如何为 awesome-nijigen-ai-exp 贡献你的 `exp`，或是对某个 `exp` 寻求理解帮助  
可以在你要提出 Pull Request 之前花几分钟来阅读一遍这篇指南。

这篇文章包含什么？

- [Pull Request](#pull-request)
  - [文档](#文档)
  - [代码](#代码)

## Pull Request

- 一个 PR 应该只对应一个 `exp` 相关的事，而不应混合不相关的 `exp`

- 在提 PR 的标题与描述中，最好对 `exp` 做简短的说明

初始阶段会在有时间的最快阶段 Review 贡献者提的 PR 并讨论或批准合并(Approve Merge)。

往后将计划由 ai/bot 托管处理 PR

### 文档

文档皆存放在 docs 目录上，仅限 markdown
撰写文档请使用规范的书面化用语，遵照 Markdown 语法，以及 [中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines) 中的规范。

::: tip 除 markdown 通用语法外可使用 [vitepress-markdown-extensions](https://vitepress.dev/guide/markdown)
:::
::: warning 过大的静态资源或模型等请勿上传至此 repo, 正确做法是使用外链引用
:::

### 代码

如附带代码，需放在仓库根目录 code 下新建同名 `exp` 目录上  
这是由于 docs 目录会通过 workflow 自动构筑成 vitepress 的 github-pages，避免发生异常，统一遵守规范
