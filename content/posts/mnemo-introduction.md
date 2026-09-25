---
title: "Mnemo：我给自己写了一个键盘优先的本地命令管理器"
date: 2026-09-25T8:00:00+08:00
draft: false
tags: ["Tauri", "Rust", "工具分享", "效率"]
---

# Mnemo：告别记不住命令的烦恼

> 一个键盘优先的本地命令管理器。收藏常用 shell 命令，秒搜秒复制，一键回到终端。

每次写代码或者折腾环境时，总会遇到那种“前两天刚用过，今天就死活想不起来”的复杂命令。翻历史记录太痛苦，存笔记里又太重。

于是我动手写了个小工具——**Mnemo**。它的核心诉求很简单：**键盘优先、本地优先、复制即走。** 不打断你原本的终端工作流，用完即焚。

## ✨ 它能做什么？

作为一个键盘党，我把所有高频操作都浓缩在了键盘上：

- **本地优先，拒绝云端** — 所有数据保存在本地的 SQLite 数据库中，没有账号，没有云端同步。你的命令只属于你自己。
- **秒级全文搜索** — 支持对标题、命令、备注、标签进行模糊搜索，敲个 `s` 就能瞬间定位。
- **极致的键盘流** — 按 `s` 搜索，`↑`/`↓` 选择，`Enter` 直接复制代码片段（或者打开知识笔记查看）。按 `r` 编辑，`Ctrl+N` / `Cmd+N` 新建。
- **复制即关闭** — 这是我最喜欢的设计！按下 `Enter` 复制命令后，窗口自动关闭，命令已经在剪贴板里了。你的终端工作流完全不会被打断。
- **不仅仅是代码片段** — 每条内容分为**代码片段**（用于复制）与**知识笔记**（用于查看）两种类型。它还支持富 Markdown 渲染：GFM 表格、KaTeX 数学公式、代码高亮全都有。
- **沉浸式查看模式** — 按 `Enter` 可以像 Quick Look 一样预览知识笔记（只读、无光标、无工具栏）。查看器内按 `Enter` 可以直接原位进入编辑，`Ctrl+S` 保存，`Esc` 返回，行云流水。

## 📦 怎么安装？

### 直接下载安装包

前往 [GitHub Releases](https://github.com/xuziran666/Mnemo/releases) 下载对应平台的包，开箱即用：

- **Linux**：AppImage / deb / rpm（支持 x86_64 和 aarch64）
- **Windows**：exe / msi
- **macOS**：dmg / app

### Arch Linux（AUR）用户

即将上架：`mnemo-cm`（源码编译版）和 `mnemo-cm-bin`（预编译二进制版），AUR 党可以蹲一下。

### 自己动手编译

如果你喜欢自己动手，或者想改点功能，需要准备好：[Node.js](https://nodejs.org) 18+、[pnpm](https://pnpm.io) 以及 [Rust](https://rustup.rs)（stable）。

*踩坑提示：Linux 用户记得先装好 [Tauri 的系统依赖](https://tauri.app/start/prerequisites/)（比如 `libwebkit2gtk-4.1-dev`、`librsvg2-dev` 等）。*

```bash
pnpm install
pnpm run tauri build
```

### ⌨️ 快捷键速查表
为了让你最快上手，我整理了一份快捷键表：

| 按键 | 功能 |
| --- | --- |
| `s` | 聚焦搜索框 |
| `↑` / `↓` | 在列表中移动选择 |
| `Enter` | 复制选中代码片段并关闭窗口，或打开知识笔记查看 |
| `c` | 复制选中代码片段并关闭窗口（仅代码片段） |
| `v` | 查看选中命令 |
| `d` | 删除选中命令（需二次确认） |
| `Esc` | 关闭窗口（列表中） |
| `r` | 编辑选中命令 |
| `Ctrl+N` / `Cmd+N` | 新建命令 |
| `+` | 新建命令（鼠标） |

**进入查看器/编辑模式后：**

|按键 |	功能|
|--- |---|
|`Enter`|	进入编辑（同一窗口）|
|`Esc`|	返回搜索列表|
|`Ctrl+S` / `Cmd+S`	|保存并返回查看|
|`Esc`|	取消修改并返回查看|
### 🛠️ 开发与折腾
如果你想在这个基础上二次开发，非常方便：

``` bash
pnpm install          # 安装依赖
pnpm run tauri dev    # 热重载开发
pnpm run tauri build  # 构建发布包
```
### 💾 你的数据在哪里？
因为主打本地优先，所有数据都存在系统 app data 目录下的单个 SQLite 文件（commands.db）里。备份这个文件，就等于备份了你所有的命令，软件也自带数据导入导出功能。

|系统|	路径|
|---|---|
|Linux |`~/.local/share/com.longanl.mnemo/commands.db`|
|macOS|	`~/Library/Application Support/com.longanl.mnemo/commands.db`|
|Windows|	`%APPDATA%\com.longanl.mnemo\commands.db`|

### 🧰 技术栈与感悟
作为一个 Tauri 爱好者，这个项目依旧采用了我最喜欢的组合：

[Tauri 2](https://tauri.app/) + [Rust](https://rust-lang.org/)（极致轻量，内存占用极低）

[React 19](https://react.dev/) + [TypeScript](https://www.typescriptlang.org/) + [Vite](https://vite.dev/)（丝滑的界面体验）

[SQLite](https://www.sqlite.org/)（通过 [rusqlite](https://github.com/rusqlite/rusqlite) 集成）（本地数据的安全感）

没有 Electron 那种动辄几百兆的笨重，Tauri 配合系统原生的 WebView，让这个小工具在后台静静待命时几乎不占什么资源。

### 📄 许可证
MIT - 随便玩，随意改。

如果这个工具帮到了你，或者你也受够了记不住终端命令的苦恼，欢迎去 [GitHub](https://github.com/xuziran666/Mnemo) 点个 Star 支持一下！有任何建议也欢迎随时提 Issue 交流。
