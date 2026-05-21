# jmux

一个为「人 × AI Agent」并排工作而设计的原生 macOS 终端，基于 [Ghostty](https://ghostty.org) GPU 渲染内核，垂直标签页 + 通知面板 + 工作区管理 + 全套 Agent 自动化接口。

<p align="center">
  <img src="docs/screenshot.png" alt="jmux 截图" width="800">
</p>

---

## 为什么用 jmux

写代码越来越是「自己写一半 + Agent 在另一个 pane 跑一半」。传统终端不知道哪些 pane 是 Claude / Codex 在跑，没法被外部脚本可靠驱动，命令完成你都不知道；jmux 把这些缝补上：

- 终端本身：Ghostty 内核，GPU 渲染，配置 100% 兼容。
- Agent 视角：每个 Claude / Codex 会话有独立状态、独立通知规则、独立侧边栏徽章。
- 自动化视角：每个 pane 都有稳定句柄，CLI 可以远程发命令、抓输出、聚焦窗口、做拓扑探测——Agent 自己就能驱动 jmux。

---

## Agent 能力（亮点）

> 这一节是 jmux 与普通终端最大的差异。下列功能默认开启，无需任何配置。

### 1. `jmux` CLI —— 给 Agent 用的脚本接口

整个 jmux 提供一套 Unix socket RPC，任何 Agent（Claude Code、Codex、自定义脚本）都可以用 `jmux` CLI 远程驱动：

```bash
# 探测当前布局（哪些 window / workspace / pane / surface，谁在 focus）
jmux tree
jmux tree --all                            # 跨所有窗口

# 在指定 surface 里输入命令（不抢用户焦点）
jmux send --surface 7 -- "pnpm test\n"
jmux send-key --surface 7 C-c              # 发送控制键

# 把指定 pane 切到前台
jmux focus-pane --pane pane:2

# 抓取 pane 当前显示的内容或滚动历史
jmux capture-pane --surface 7

# 让 jmux 弹一条系统通知（脚本结束时通知用户）
jmux notify --title "Build done" --body "All tests passed"
```

设计原则：socket 命令默认 **不抢焦点**，只有显式 `*.focus / *.select` 才会动用户的视线。Agent 可以在后台默默驱动 jmux 而不打断你正在编辑的窗口。

### 2. 内置 Agent skills（Claude Code + Codex）

`jmux skills install` 会把内置的 skill 写到本地：

- Claude Code：`~/.claude/skills/<name>/`
- Codex：`$CODEX_HOME/skills/<name>/`（默认 `~/.codex/skills/`）

每个目录会带 `.jmux-managed` marker，`jmux skills uninstall` 只会移除 jmux 自己写入的目录，不会动你手写的 skill。

当前内置（持续扩充中）：

- **jmux-tree** —— 教会 Agent 在自动化之前先用 `jmux tree` 探清拓扑，避免"对错误的 pane 发命令"。
- **jmux-send-keys** —— 教会 Agent 用 `jmux send` / `jmux send-key` 在指定 pane 输入文本、发控制序列、驱动 TUI（如 vim、tmux、less）。

意义：Agent 不需要你解释 jmux 怎么用，安装一次以后，它自己知道。

### 3. Claude Code 集成（Hooks + 会话追踪）

「设置 → 集成 → Claude Code Integration」开启后，jmux 会自动 wrap `claude` 命令注入 hook：

- 每个 Claude 会话的状态（思考中 / 等输入 / 已结束）会显示在 jmux 侧边栏徽章上，跨多个工作区也能一眼看全。
- 命令完成时按你设的「通知规则」触发系统通知 + 音效。
- 关闭一个 Claude 会话的 tab 后用「文件 → 重新打开关闭的工作区」（默认快捷键 `⌘⇧T`，可在设置里改），jmux 会自动 `claude --resume` 续上原会话，不丢上下文。
- 同样支持 Codex（`codex resume`）。

### 4. 每会话独立通知规则

设置 → 通知里有完整的「按 Agent 分通知」规则引擎：

- Claude 思考完一段、需要你输入时——音效 / Dock 徽章 / pane 闪烁 / pane 边框闪环，任选组合。
- Codex 写完一个文件时——只闪烁不响铃。
- 规则按 Agent（Claude / Codex）分别配置，可在「设置 → 通知」里图形化编辑。

### 5. 命令完成 / 进度条 / 渲染器健康

每个 pane 底部有状态行：

- **命令完成**：你跑 `pnpm build && pnpm test`，jmux 知道它什么时候结束（Ghostty OSC + jmux hook 双源），无需 Agent 自己 echo 完成标记。
- **OSC 9;4 进度**：长任务（`npm install`、`gh pr checks --watch`）有进度条会在 pane 底部画出来。
- **Renderer Health**：渲染卡顿 / GPU 异常会用颜色提示，避免你以为 Agent 死了实际只是 60→0 fps。

### 6. 远程开发会话（`jmux ssh`）

```bash
jmux ssh <host>
```

jmux 会自动：

1. 用 ssh 连过去
2. 部署 `jmuxd-remote` daemon
3. 把远端端口映射到本地（浏览器开 localhost 直接打开远端的 webserver）
4. 在 jmux 里开一个工作区——和本地工作区使用方式完全一致

适合：Agent 在远端 dev 机器跑、你本地用 jmux 看输出 / 发命令。

---

## 终端 / UI 基础能力

- **垂直标签页** —— 适合长标题、几十个会话，不用横向滚动。
- **多工作区 + Bonsplit 分屏** —— 任意拖拽分割，每个工作区独立标签栏。
- **Ghostty 内核** —— 同样的 GPU 加速渲染，`~/.config/ghostty/config` 直接复用。
- **撤销关闭** —— `⌘⇧T` 重新打开刚关掉的工作区 / surface，会话 / Agent 上下文一并恢复（快捷键可改）。
- **快捷键全可改** —— 设置里覆盖任意 jmux 自有快捷键，也可通过 `~/.config/cmux/settings.json` 配置。
- **多语言界面** —— 已本地化 19 种语言（中英日韩、阿拉伯、德法意西俄等）。
- **Sparkle 自动更新** —— 详见下方[自动更新](#自动更新)。

---

## 下载

最新版本：<https://github.com/Sean529/jmux-releases/releases/latest>

从 Assets 里下载 `jmux-v<版本号>.zip`，解压后把 `jmux.app` 拖到 `/Applications`。

### 首次启动（Gatekeeper）

构建采用 ad-hoc 签名，首次启动会被 macOS Gatekeeper 拦下。任选其一：

- 右键 `jmux.app` → **打开** → 在弹窗里确认；或者
- 用命令去掉一次隔离属性：
  ```bash
  xattr -dr com.apple.quarantine /Applications/jmux.app
  ```

发布构建只包含 arm64（Apple Silicon）。Intel Mac 用户需要自己从源码构建，加上 `ARCHS='arm64 x86_64' ONLY_ACTIVE_ARCH=NO`。

---

## 自动更新

jmux 内置了 [Sparkle](https://sparkle-project.org)，会自动查询本仓库的更新：

- Appcast 地址：`https://github.com/Sean529/jmux-releases/releases/latest/download/appcast.xml`
- 应用会在安装每个更新前校验 `ed25519` 签名，被篡改的 appcast 会直接被拒。

装好之后你就不用再回到这个页面看了——jmux 每次启动会自己检查更新，有新版本会弹提示。

---

## 许可证

详见 [LICENSE](./LICENSE)。

## 源代码

源代码仓库是私有的。本仓库只托管发布产物（`.zip` + `appcast.xml`），供 Sparkle 拉取使用。
