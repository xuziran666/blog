# 我的技术博客

基于 [Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 的静态博客，
通过 GitHub Actions 自动构建并发布到 GitHub Pages。

- 线上地址：<https://xuziran666.github.io/blog/>
- 仓库地址：<https://github.com/xuziran666/blog>

## 特性

- 支持深色 / 浅色主题，跟随系统并可手动切换
- 站内全文搜索（基于 Fuse.js，无需后端）
- 归档页、标签页、分类页
- 文章目录（TOC）、代码复制按钮、阅读时长与字数统计
- RSS 订阅、自动生成 sitemap 与 robots.txt
- 中文界面（i18n）
- 推送即部署，无需手动构建

## 环境要求

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Hugo | v0.166.0 **extended** | 必须为 extended 版本，PaperMod 依赖其处理 SCSS；且需 ≥ v0.146.0 |
| Git | 任意较新版本 | 需支持 submodule（主题以子模块方式引用） |

不需要 Node.js。检查本地环境：

```bash
hugo version   # 输出中应包含 extended
```

## 快速开始

### 1. 克隆项目

主题是 git submodule，**必须**连同子模块一起拉取：

```bash
git clone --recurse-submodules git@github.com:xuziran666/blog.git
cd blog
```

如果已经克隆但 `themes/PaperMod` 是空目录：

```bash
git submodule update --init --recursive
```

### 2. 本地预览

```bash
hugo server -D
```

访问 <http://localhost:1313/blog/>。

> 注意：因为 `baseURL` 带有 `/blog/` 子路径，本地预览也挂在该子路径下。
> 若希望以根路径 `http://localhost:1313/` 访问，可执行
> `hugo server -D --baseURL http://localhost:1313/`。

`-D` 表示同时预览草稿（`draft: true`）的文章。

### 3. 本地构建

```bash
hugo --minify --gc
```

产物输出到 `public/` 目录（已加入 `.gitignore`，不会被提交）。

## 目录结构

```text
blog/
├── .github/workflows/hugo.yaml   # CI：构建并部署到 gh-pages 分支
├── archetypes/default.md         # hugo new 生成新文章时使用的模板
├── content/
│   ├── archives.md               # 归档页（layout: archives）
│   ├── search.md                 # 搜索页（layout: search）
│   └── posts/                    # 所有文章
├── docs/problems/                # 问题排查与技术决策记录
├── i18n/zh-cn.yaml               # 中文翻译（文件名必须与语言键一致）
├── themes/PaperMod/              # 主题（git submodule，请勿直接修改）
└── hugo.toml                     # 站点配置
```

## 写文章

```bash
hugo new content posts/my-article.md
```

会按 `archetypes/default.md` 生成 front matter，默认 `draft = true`（草稿，不会发布），
写完后记得改为 `false`。

### front matter 常用字段

| 字段 | 说明 |
| --- | --- |
| `title` | 文章标题 |
| `date` | 发布时间 |
| `draft` | `true` 为草稿，本地 `-D` 可见但不会发布 |
| `summary` | 列表页摘要，不填则自动截取正文开头 |
| `tags` / `categories` | 标签与分类，会自动生成对应页面 |
| `cover.image` | 封面图 |
| `ShowToc` | 是否显示目录 |
| `hiddenInHomeList` | 是否从首页列表隐藏 |
| `searchHidden` | 是否从搜索结果中排除 |

### 页面快捷键

| 按键 | 功能 |
| --- | --- |
| `c` | 展开 / 收起目录 |
| `g` | 回到顶部 |
| `h` | 返回主页 |
| `t` | 切换明暗主题 |
| `/` | 跳转到搜索页 |

## 配置说明

站点配置全部在 `hugo.toml`，其中这几处的行为不直观，修改时需留意：

| 配置项 | 说明 |
| --- | --- |
| `baseURL` | 项目页站点必须包含 `/blog/` 子路径，写错会导致线上 CSS/JS 全部 404 |
| `defaultContentLanguage` | 决定 i18n 查找所用的语言键，需与 `i18n/` 下的文件名一致 |
| `params.mainSections` | 主题通过 `site.Params.mainSections` 过滤首页与归档文章，**必须写在 `[params]` 下** |
| `outputs.home` | 需包含 `JSON`，搜索页依赖它生成的索引 `index.json` |
| `menu.main[].identifier` | **必填**，主题用它反查页面 Layout，缺失会导致渲染报错 |
| `params.fuseOpts` | 搜索引擎参数，键名必须全小写，其中 `limit` 控制搜索结果条数上限 |
| `params.DateFormat` | 日期显示格式，如 `2006年1月2日` |

### 中文显示的两个约定

1. **`i18n/zh-cn.yaml` 不能删。** 主题只提供 `i18n/zh.yaml`，而 Hugo 的翻译文件按语言键
   **精确匹配文件名、不会回退**，因此语言键为 `zh-cn` 时必须存在同名文件，
   否则阅读时长、字数、翻页等文案会退回英文（如 `1 min`）。
2. **主题目录不要直接修改。** 它是 git submodule，改动会在升级时丢失；
   需要覆盖模板或翻译时，请在项目根目录建立同名文件，Hugo 会优先使用项目层文件。

## 部署

### 工作流程

```text
push 到 main
   ↓
GitHub Actions（.github/workflows/hugo.yaml）
   ↓  hugo --minify --gc
   ↓  peaceiris/actions-gh-pages 推送产物
gh-pages 分支
   ↓  GitHub Pages 从该分支发布
https://xuziran666.github.io/blog/
```

### 首次部署需手动完成的两件事

1. 仓库 **Settings → Pages** → Source 选 `Deploy from a branch`，
   分支选 `gh-pages`、目录选 `/ (root)`。
   （`gh-pages` 分支由 Actions 首次成功运行后自动创建，所以必须先跑通 Actions）
2. 确认 **Settings → Actions → General → Workflow permissions** 为
   `Read and write permissions`。workflow 中已声明 `permissions: contents: write`，
   通常无需手动修改。

### 日常发布

```bash
git add .
git commit -m "post: 新文章标题"
git push
```

推送后 Actions 会自动构建并发布，无需任何手动操作。

也可以在 GitHub 的 **Actions → Build and Deploy Hugo → Run workflow** 手动触发。

> `gh-pages` 分支在站点产物没有变化时不会新增提交，这是
> `peaceiris/actions-gh-pages` 的正常行为，不代表部署失败。

## 常见问题

| 现象 | 原因与处理 |
| --- | --- |
| 线上页面样式全丢 / 图片 404 | `baseURL` 与站点实际路径不匹配，项目页需带 `/blog/` 子路径 |
| 文案显示英文（`1 min`、`1 words`） | `i18n/zh-cn.yaml` 缺失，或 `defaultContentLanguage` 与文件名不一致 |
| 文章没出现在线上 | front matter 中 `draft: true`，改为 `false` 后重新推送 |
| `themes/PaperMod` 为空目录 | 未拉取子模块，执行 `git submodule update --init --recursive` |
| Actions 报 Invalid workflow file | workflow YAML 格式有误，注意文件不要包含 BOM 或其他语言的语法包装 |
| Actions 部署步骤 403 | workflow 缺少 `permissions: contents: write` |
| 首页简介卡片不显示 | `[params.homeInfoParams]` 配置缺失或拼写错误 |
| 搜索无结果 | `outputs.home` 缺少 `JSON`，或 `baseURL` 错误导致 `index.json` 请求失败 |
| 菜单点击后 404 | `/archives/`、`/search/` 需要有 `content/archives.md`、`content/search.md` 对应 |

## 相关文档

`docs/problems/` 目录记录了本项目中值得复用的排查过程与技术决策：

- [github-pages-hugo-deployment.md](docs/problems/github-pages-hugo-deployment.md)
  —— GitHub Pages 自动部署的配置方式与关键决策
- [hugo-papermod-zh-cn-i18n-fallback.md](docs/problems/hugo-papermod-zh-cn-i18n-fallback.md)
  —— 中文 i18n 失效的原因与解决方案

## License

文章内容版权归作者所有，转载请注明出处。
`themes/PaperMod` 遵循其自身许可证（MIT）。