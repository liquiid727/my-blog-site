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

## 主题

- 夜间（默认）：纯黑画布、玻璃面板、pill 控件、mono 数据，视觉语言来自 design-lab 的 `brands/noir`。
- 日间：同一套 token 的对照色阶，Material palette 切到 `default`。
- 主题源文件在私有仓库的 `Blog/public-site-template/assets/stylesheets/brand.css`，导出时复制到这里；不要直接改 `docs/assets/stylesheets/brand.css`。
- 字体使用系统 SF Pro / PingFang SC / SF Mono，不请求外部字体 CDN。

## 评论区

文章页使用 giscus 对接 GitHub Discussions。首次启用时，先在仓库安装 [giscus GitHub App](https://github.com/apps/giscus)，再打开 [giscus 配置页](https://giscus.app/zh-CN) 获取 `category_id`，填入 `mkdocs.yml` 的 `extra.giscus.category_id`。

## 内容来源

所有内容由私有仓库中的导出脚本同步而来，请不要直接在这里维护正文。
