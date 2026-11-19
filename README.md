# Dev Conventions & Guardrails

> Android 通用开发 & UI 规范 —— 一份让 AI 写代码前自动遵守的工程约定包。

**中文** | [English](#english)

[![Release](https://img.shields.io/github/v/release/Ray01207/dev-conventions?label=release)](https://github.com/Ray01207/dev-conventions/releases)
[![License](https://img.shields.io/github/license/Ray01207/dev-conventions)](LICENSE)

---

## 这是什么

一份跨项目通用的开发约定与硬约束，封装成 AI 工具（WorkBuddy / Claude Code 等支持 `SKILL.md` 标准的客户端）可以自动加载的 Skill 包。

它解决的不是"知识不够"，而是**AI 每次动手都要重新提醒一遍**的问题：

- 布局里直接写 `#FF5722`，切深色模式全崩
- 控件之间零间距，换台设备就露馅
- 外部存储 `readText()` 裸调用，一个权限问题崩掉整页
- 给 arm64 机器下 x86_64 的 NDK，然后花两小时查「为什么 aapt2 跑不起来」
- `.so` 没做 16KB 页对齐，Android 15 装不上
- 每台机器各自生成 debug keystore，换环境无法覆盖安装

这些坑写进规范，AI 动手前读一遍，比事后返工便宜得多。

## 结构

采用**渐进披露**：入口文件只保留 52 行，常驻上下文不占地方；细则按场景拆分，用到哪份读哪份。

```
dev-conventions/
├── SKILL.md                      入口：触发条件 + 场景路由表 + 十条最高频硬约束
├── references/
│   ├── component-and-ui.md       组件六原则 / 深色适配 / UI 设计规范十节
│   ├── build-and-env.md          构建验证 / 16KB 页对齐 / 工具链架构匹配 / 脚本自检
│   ├── code-and-runtime.md       外部 IO 容错 / Kotlin 协程 / 内存泄漏 / 列表性能
│   ├── release-and-security.md   签名 / 混淆发版 / 密钥管控 / Git 规范
│   └── workflow-and-docs.md      开发记录 / 架构图 / 崩溃日志 / Skill-MCP 管理
├── README.md
└── LICENSE
```

| 文件 | 行数 | 什么时候会读到 |
|---|---|---|
| `SKILL.md` | 52 | 总是（决定要不要触发、去哪找细则） |
| `component-and-ui.md` | 117 | 改布局 / 样式 / 颜色 / 间距 / 主题 |
| `build-and-env.md` | 68 | 跑构建 / 装工具链 / 打 .so / 改依赖 |
| `code-and-runtime.md` | 40 | 写业务逻辑 / 协程 / 文件 IO / 列表 |
| `release-and-security.md` | 36 | 配签名 / 开混淆 / 打 release / 提交代码 |
| `workflow-and-docs.md` | 47 | 改完留痕 / 开新项目 / 接崩溃采集 |

## 安装

三种方式任选一种。共同点是：最后放进技能目录的必须是 `SKILL.md` 与 `references/` 的**同级结构**，外层多套一层目录会导致加载不到。

### 方式一：下载 zip（不需要 git）

从 [Releases](https://github.com/Ray01207/dev-conventions/releases/latest) 下载 `dev-conventions-vX.Y.Z.zip`，解压后把 `dev-conventions` 整个目录放进技能目录：

```bash
# 文件名以实际下载的版本为准，下面以 v1.0.1 为例

# macOS / Linux
unzip dev-conventions-v1.0.1.zip -d ~/.workbuddy/skills/

# Windows PowerShell
Expand-Archive dev-conventions-v1.0.1.zip -DestinationPath "$env:USERPROFILE\.workbuddy\skills"
```

### 方式二：git clone（便于后续 `git pull` 更新）

**用户级**（所有项目都生效）：

```bash
git clone https://github.com/Ray01207/dev-conventions.git ~/.workbuddy/skills/dev-conventions
```

以后更新只需：

```bash
git -C ~/.workbuddy/skills/dev-conventions pull
```

**项目级**（只对当前仓库生效，可随代码提交给团队共享）：

```bash
git clone https://github.com/Ray01207/dev-conventions.git .workbuddy/skills/dev-conventions
```

### 方式三：手动复制（已有本地副本）

```bash
cp -r dev-conventions ~/.workbuddy/skills/
```

其他遵循 `SKILL.md` 开放标准的工具（如 `~/.agents/skills/`）同理，把整个目录复制过去即可。装完**新开一个会话**才会被扫描到。

## 适用与不适用

**适用**：Android 项目开发、UI 与主题调整、构建与发版、native 库打包、环境安装脚本。

**部分适用**：非 Android 项目可参考其中的通用条目 —— 外部 IO 容错、脚本真实自检、下载完整性校验、Git 规范、开发记录与架构图、崩溃日志采集。

**需要按实际调整**：工具链架构匹配、16KB 页对齐两条基于 arm64 (aarch64) 开发环境写成，x86_64 环境请自行替换对应架构。

## 设计取舍

几处刻意的选择，供参考：

1. **入口不写细则**。细则全塞进 `SKILL.md` 会让每次对话都吃满上下文；拆开后，改 UI 只读 117 行那份，不会把发版、签名的规矩一起拖进来。
2. **每条约束尽量带验证动作**。写「`.so` 必须 16KB 对齐」不如写「用 `llvm-readelf -l | grep LOAD` 确认 `Align ≥ 0x4000`」——后者可以被执行，前者只能被相信。
3. **标注真实踩坑**。「用 `Settings.Global.getString` 读 `device_config` 的 key 会静默返回 null」这种带出处的一句话，比十条抽象原则更容易被记住。

## 贡献

Issue / PR 都欢迎。

新增条目时建议带上**踩坑场景**（什么情况下会出问题、怎么发现的、怎么验证修好了），而不是只写结论。这份文档的价值就在这些具体证据上。

## License

[MIT](LICENSE)

---

## English

A cross-project development conventions pack, packaged as a `SKILL.md`-standard skill that AI coding assistants load automatically before touching Android or general software code.

**What it covers**: component and UI design specs, theming and dark mode, build verification and pre-build review, 16KB page alignment for native libraries, toolchain architecture matching, install-script self-checks, Kotlin coroutines, memory leaks, list performance, external IO fault tolerance, signing and obfuscated release, secret management, Git conventions, dev logs and architecture maps, crash log collection, and Skill/MCP management.

**Design**: progressive disclosure — a 52-line entry file plus five topic-specific references loaded on demand, so the skill never floods the context window.

**Install**: download `dev-conventions-vX.Y.Z.zip` from [Releases](https://github.com/Ray01207/dev-conventions/releases/latest) and unpack the `dev-conventions` folder into your skills directory, or clone it directly:

```bash
git clone https://github.com/Ray01207/dev-conventions.git ~/.workbuddy/skills/dev-conventions
```

**Scope**: primarily Android; the IO fault-tolerance, script self-check, Git, and dev-log sections apply to general software projects. The toolchain-architecture and 16KB-page-alignment rules assume an arm64 (aarch64) host — adjust for x86_64.

Licensed under [MIT](LICENSE).
