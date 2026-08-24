# my-blog-site

这个仓库是从私有 Obsidian Vault 中导出的 **公开博客仓库**。

- 私有源仓库：`personal_journey`
- 公开文章源：`Blog/public/`
- 公开图片源：`assets/images/blog/`
- 当前仓库只接收导出后的公开内容

## 本地预览

```bash
python3 -m pip install -r requirements.txt
mkdocs serve
```

## 评论区

文章页使用 giscus 对接 GitHub Discussions。首次启用时，先在仓库安装 [giscus GitHub App](https://github.com/apps/giscus)，再打开 [giscus 配置页](https://giscus.app/zh-CN) 获取 `category_id`，填入 `mkdocs.yml` 的 `extra.giscus.category_id`。

## 内容来源

所有内容由私有仓库中的导出脚本同步而来，请不要直接在这里维护正文。
