---
title: 从 Private Vault 到 Public Blog 的最小发布流水线
date: 2026-03-14
slug: private-vault-public-blog-pipeline
tags:
  - obsidian
  - mkdocs
  - github
  - architecture
status: published
visibility: public
summary: 用 Blog/public、白名单资源和导出脚本，把私有 Obsidian 仓库改造成可发布的公开博客。
---

# 从 Private Vault 到 Public Blog 的最小发布流水线

这套仓库改造的目标很简单：继续把 Obsidian 当作主写作环境，同时只公开明确允许发布的内容。

## 目录边界

- 私有内容继续保留在原有 Vault 目录中
- `Blog/public/` 作为唯一公开文章源
- `assets/images/blog/` 作为唯一公开图片源
- `docs/` 作为导出结果，交给 MkDocs 构建

## 导出原则

所有公开文章都必须带上 frontmatter，并满足下面两个条件：

- `status: published`
- `visibility: public`

这样草稿可以继续留在仓库里，但不会进入公开站点。

## 构建链路

```text
Blog/public/ + assets/images/blog/
            ↓
   scripts/export_blog.py
            ↓
          docs/
            ↓
         mkdocs build
            ↓
      GitHub Pages
```

## 为什么这样做

这套结构兼顾了三个目标：

1. Obsidian 写作体验不变
2. 私有与公开边界足够清晰
3. 后续可以继续加 RSS、标签页、归档页和自动化发布

## 发布前检查

- frontmatter 是否完整
- 是否含有私人信息
- 图片是否来自公开目录
- 链接是否可访问
- 标题与摘要是否适合公开阅读

这就是当前仓库从“纯 Vault”走向“可发布博客”的最小可用版本。

