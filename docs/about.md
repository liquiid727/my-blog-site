---
title: 关于这个站点
date: 2026-03-14
slug: about
tags:
  - blog
  - obsidian
status: published
visibility: public
summary: 说明这个站点如何从私有 Vault 中筛选并发布公开内容。
---

# 关于这个站点

这个站点采用 **私有 Vault 写作、白名单导出、MkDocs 发布** 的方式维护。

## 原则

- 私有目录不对外暴露
- 只有公开文章源会进入站点
- 文章发布前需要经过 frontmatter 门禁
- 公开素材和私有素材分离存放

## 公开边界

- 公开文章：`Blog/public/`
- 公开图片：`assets/images/blog/`
- 构建产物：`docs/`

## 工作流

1. 在 Vault 里完成写作与整理
2. 把准备发布的文章放入 `Blog/public/`
3. 执行 `python3 scripts/export_blog.py`
4. 本地 `mkdocs serve` 预览
5. 推送后交给 GitHub Actions 构建 / 发布

