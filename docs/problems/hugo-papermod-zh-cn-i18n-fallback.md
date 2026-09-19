# PaperMod 主题 zh-cn 语言下 i18n 翻译失效

## 问题现象

站点 `hugo.toml` 中配置 `languageCode = "zh-cn"`，但部署后页面文案仍是英文：

- 文章阅读时长显示 `1 min`，而不是 `1 分钟`
- 字数显示 `3 words`，而不是 `3 字`
- 同时日期却能正确显示为 `2026年9月19日`

即「日期已中文化，但 i18n 文案没中文化」，两类文案表现不一致。

## 问题原因

`themes/PaperMod/i18n/` 只提供 `zh.yaml`，**没有 `zh-cn.yaml`**（已核对全部 46 个语言文件）。

Hugo 的 i18n 翻译文件按**语言键精确匹配文件名**，不会把 `zh-cn` 回退到 `zh`。因此语言键为
`zh-cn` 时找不到对应翻译文件，`i18n "read_time"` 返回空字符串。

主题 `layouts/_partials/post_meta.html` 中又给 i18n 结果加了英文兜底：

```go-html-template
{{- if (.Param "ShowReadingTime") -}}
    {{- $scratch.Add "meta" (slice (printf "<span>%s</span>" (i18n "read_time" .ReadingTime | default (printf "%d min" .ReadingTime)))) }}
{{- end -}}
```

所以最终渲染出 `1 min` / `3 words`，**表面看像「主题不支持中文」，实际是翻译文件未命中**。

日期能中文化的原因不同：它走 `DateFormat`（`hugo.toml` 中 `params.DateFormat`），与 i18n 无关，
所以出现了「日期中文 + 文案英文」的割裂现象。

## 解决方案

在**项目层**新增 `i18n/zh-cn.yaml`，补齐主题 `zh.yaml` 中的全部翻译条目：

```
i18n/zh-cn.yaml   # 新增，文件名与语言键 zh-cn 精确对应
```

项目级 i18n 会与主题 i18n 合并，因此该文件只是「补一个 Hugo 能命中的语言键」，无需改动主题。

同时配置：

```toml
languageCode = "zh-cn"
defaultContentLanguage = "zh-cn"   # 决定 i18n 查找所用的语言键
```

验证结果（线上实测）：

```html
<span title='...'>2026年9月19日</span>&nbsp;·&nbsp;<span>1 分钟</span>&nbsp;·&nbsp;<span>3 字</span>
```

## 为什么采用这个方案

| 方案 | 说明 | 是否采用 |
| --- | --- | --- |
| 项目层补 `i18n/zh-cn.yaml` | 保留 `lang="zh-cn"` 这一更精确的语言标签，符合 BCP-47 习惯，SEO/无障碍更好 | ✅ 采用 |
| 把语言键改成 `zh` | 一行配置即可复用主题 `zh.yaml`，但页面 `<html lang>` 会退化为 `zh`，且 RSS 语言信息也变粗 | ❌ 未采用 |
| 覆盖主题 `post_meta.html` 去掉英文兜底 | 需要 fork 主题模板，主题升级后要同步维护 | ❌ 未采用 |
| 修改主题内 `i18n/zh.yaml` 改名 | 会污染 git submodule，污染上游代码 | ❌ 未采用 |

关键权衡：`lang` 属性的取值来自主题 `baseof.html` 的 `{{ site.Language }}`（即**语言键**），
而非 `languageCode`。因此只要语言键是 `zh-cn`，就必须存在同名的 `zh-cn` 翻译文件；
反过来若想复用主题的 `zh.yaml`，则必须把语言键降级为 `zh`。二者只能取其一，本次选择前者。

## 解决了什么问题

- 修复了阅读时长、字数、翻页（上一页/下一页）、目录、复制按钮等全部 i18n 文案的中文化。
- 明确了「Hugo i18n 按语言键精确匹配文件名、不做 `zh-cn` → `zh` 回退」这一约束，
  后续新增语言或升级主题时可复用该结论，避免重复排查。

遗留说明：`languageCode` 与 `.Language.LanguageCode` / `.Language.LanguageDirection` 在
Hugo v0.158.0 起已被标记 deprecated（建议改用 `locale` / `.Language.Direction`），构建时会输出
WARN，但不影响功能。该 WARN 来自主题模板本身，需等主题上游升级，本次不擅自修改主题。

## 相关文件

- `hugo.toml` — `languageCode` / `defaultContentLanguage` / `params.DateFormat`
- `i18n/zh-cn.yaml` — 新增，中文翻译
- `themes/PaperMod/i18n/zh.yaml` — 主题提供的中文翻译（仅语言键为 `zh` 时生效）
- `themes/PaperMod/layouts/_partials/post_meta.html` — 英文兜底值来源
- `themes/PaperMod/layouts/baseof.html` — `<html lang>` 取自 `site.Language`