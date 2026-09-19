# Hugo 博客托管到 GitHub Pages（Actions 自动部署）

## 问题现象

项目本地可正常构建，但无法自动发布到线上，且存在三处会导致部署失败的隐患：

1. `.github/workflows/hugo.yaml` 内容不是 YAML，而是 PowerShell here-string 的原文：
   首行是 `@'`，末行是 `'@ | Out-File .github\workflows\hugo.yaml -Encoding utf8`。
   推送后 GitHub Actions 会直接报 Invalid workflow file，流程根本不执行。
2. 仓库没有 `.gitignore`，`public/` 构建产物与 `.hugo_build.lock` 处于 untracked 状态，易被误提交。
3. `hugo.toml` 的 `baseURL = "https://xxx.github.io/"` 仍是占位符，部署后样式与资源路径会 404。

另外，`themes/PaperMod` 是以 git submodule 形式引用的。

## 问题原因

- 生成 workflow 文件时用了 PowerShell here-string，但 `Out-File` 那一步的原文被一并写入了文件，
  即文件内容的产生方式有误（属于工具链误用，而非 YAML 语法本身出错）。
- 缺少忽略规则与真实 `baseURL`，属于首次部署的配置缺失。
- GitHub 新版仓库的 `GITHUB_TOKEN` 默认权限为只读，而 `peaceiris/actions-gh-pages` 需要向
  `gh-pages` 分支推送，若 workflow 不声明 `contents: write`，部署步骤会 403。

## 解决方案

### 1. 重写 workflow（`.github/workflows/hugo.yaml`）

```yaml
name: Build and Deploy Hugo

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  contents: write        # 必需：默认只读会导致推送 gh-pages 失败

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true          # 拉取 PaperMod 主题
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: '0.166.0'   # 与本地版本一致，不用 latest
          extended: true            # PaperMod 依赖 extended 处理 SCSS

      - name: Build
        run: hugo --minify --gc

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages
```

### 2. 新增 `.gitignore`

```
public/
resources/
.hugo_build.lock
.DS_Store
Thumbs.db
```

### 3. 修正 `baseURL`

仓库为项目页（`xuziran666/blog`），必须带子路径：

```toml
baseURL = "https://xuziran666.github.io/blog/"
```

### 4. 分支与 Pages 设置

- 主分支命名为 `main`（workflow 只监听 `main`）。
- 仓库 Settings → Pages → Source 选 `Deploy from a branch` → `gh-pages` / `/ (root)`。
  `gh-pages` 分支由 Actions 首次成功运行后自动创建，因此必须**先跑通 Actions 再设置 Pages**。

## 为什么采用这个方案

| 方案 | 说明 | 结论 |
| --- | --- | --- |
| GitHub Pages + Actions 构建 `gh-pages` 分支 | 与代码仓库同源，无需第三方授权，免费版公开仓库即可用 | ✅ 采用 |
| Actions 产物直接走 Pages 官方 `actions/deploy-pages` | 需在 Pages 里把 Source 改为 GitHub Actions，配置更绕，且对 `gh-pages` 分支可见性不如前者直观 | ❌ 未采用 |
| Cloudflare Pages 关联仓库构建 | 免费版支持私有仓库、自带 CDN 与预览部署，但需额外授权第三方平台，且必须显式指定 `HUGO_VERSION`（其默认 Hugo 版本较旧，PaperMod 会构建失败） | ❌ 本次未采用（可后续并行接入） |
| 本地 `hugo` 构建 + `wrangler` 直传 | 不依赖 Git 托管，但每次发文需手动执行，无 CI 记录 | ❌ 未采用 |

关键决策点：

- **`hugo-version` 钉死为 `0.166.0`**：与本地版本一致，避免上游发新版导致主题不兼容而构建失败。
- **删除 `pull_request` 触发器**：原配置会在每个 PR 上完整构建并部署，浪费额度且无意义，
  改为 `workflow_dispatch` 支持手动触发。
- **主题保留 submodule 形式**：`checkout` 加 `submodules: true` 即可，`.gitmodules` 中主题 URL 为
  https 形式，CI 无需 SSH 密钥，不会卡在授权环节。
- **`baseURL` 必须含子路径**：项目页站点挂在 `/<repo>/` 下，写错会导致 CSS/JS 全部 404。

## 解决了什么问题

- 部署流程打通：`push` 到 `main` 后自动构建并发布，线上地址
  `https://xuziran666.github.io/blog/` 返回 200。
- 明确了几条易踩的约束：`contents: write` 权限、`baseURL` 子路径、`gh-pages` 由 CI 创建
  故需先跑通 Actions、Hugo 版本需钉死并开启 extended。

遗留说明（不影响发布，未处理）：

1. `post_meta.html` 里的日期 `title` 属性在 CI 环境输出为 `2026-09-19 10:00:00 +0800 +0800`，
   本地为 `+0800 CST`。原因是该处直接打印 `.Date`，Go 格式化时 CI 环境无时区名可用，
   属主题实现与环境差异，仅在鼠标悬停提示中可见。
2. 构建时有 Hugo 0.158.0 起对 `languageCode` / `.Language.LanguageCode` /
   `.Language.LanguageDirection` 的 deprecation WARN，来自主题模板，待上游修复。

## 相关文件

- `.github/workflows/hugo.yaml` — CI 构建与部署
- `.gitignore` — 忽略构建产物
- `hugo.toml` — `baseURL`、`enableRobotsTXT`
- `.gitmodules` — PaperMod 子模块定义
- `content/posts/first-article.md` — 文章 `draft` 状态影响是否发布