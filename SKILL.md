---
name: dev-conventions
version: 1.0.0
display_name: "Android通用开发&UI"
display_name_en: "Dev Conventions & Guardrails"
description: Android / 软件项目的通用开发约定与硬约束（详细版），覆盖组件与 UI 设计规范、主题与深色适配、构建验证与构建前排查、原生库 16KB 页对齐、工具链架构匹配、安装脚本自检、Kotlin 协程、内存泄漏、列表性能、外部 IO 容错、签名与混淆发版、敏感信息管控、Git 规范、开发记录与架构图、崩溃日志采集、Skill/MCP 管理。本技能应在以下场景使用：编写或修改 Android 项目代码、调整布局/样式/主题/颜色/间距、新增或改动组件、配置签名或混淆规则、打 release 包、安装 NDK/build-tools 等工具链、排查构建失败或运行闪退、以及任何 Android / 软件项目开工前需要对齐开发规范时。
description_zh: Android / 软件项目的通用开发约定与硬约束（详细版），覆盖组件与 UI 设计规范、主题与深色适配、构建验证与构建前排查、原生库 16KB 页对齐、工具链架构匹配、安装脚本自检、Kotlin 协程、内存泄漏、列表性能、外部 IO 容错、签名与混淆发版、敏感信息管控、Git 规范、开发记录与架构图、崩溃日志采集、Skill/MCP 管理。本技能应在以下场景使用：编写或修改 Android 项目代码、调整布局/样式/主题/颜色/间距、新增或改动组件、配置签名或混淆规则、打 release 包、安装 NDK/build-tools 等工具链、排查构建失败或运行闪退、以及任何 Android / 软件项目开工前需要对齐开发规范时。
description_en: "Universal development conventions and hard constraints for Android and general software projects, covering component and UI design specs, theming and dark mode, build verification and pre-build review, 16KB page alignment for native libraries, toolchain architecture matching, install-script self-checks, Kotlin coroutines, memory leaks, list performance, external IO fault tolerance, signing and obfuscated release, secret management, Git conventions, dev logs and architecture maps, crash log collection, and Skill/MCP management. Use this skill when: writing or modifying Android project code, adjusting layouts/styles/themes/colors/spacing, adding or changing UI components, configuring signing or ProGuard rules, building release packages, installing NDK/build-tools or other toolchains, troubleshooting build failures or runtime crashes, or aligning on development standards before starting any Android or software project."
agent_created: true
---

# 通用开发约定与约束

跨项目通用的开发习惯和硬约束。核心目标一句话：**代码对、UI 对、换环境也对**——不靠事后返工，靠动手前先对齐。

## 一、加载策略（先看这张表，别一次性全读）

细则按场景拆分到 `references/`，每次只读当前场景需要的 1–2 份。

| 当前要做什么 | 读哪份 |
|---|---|
| 改布局 / 样式 / 颜色 / 间距 / 主题 / 深色模式、新增或改动控件 | `references/component-and-ui.md` |
| 跑构建、构建失败、构建前自查、装 NDK / build-tools / 工具链、写安装脚本、打 .so、改 gradle 依赖 | `references/build-and-env.md` |
| 写业务逻辑、异步 / 协程、文件读写、列表与图片加载、Context 引用 | `references/code-and-runtime.md` |
| 配签名、开混淆、打 release 包、处理 API Key / 密码、提交代码 | `references/release-and-security.md` |
| 改完代码要留痕、开新项目要写架构图、接崩溃采集、管 Skill / MCP 配置 | `references/workflow-and-docs.md` |

跨场景就都读。例如「改完 UI 要发版」= `component-and-ui.md` + `release-and-security.md`。

## 二、十条最高频硬约束（不读细则也必须守）

1. **改动要六面一起想**：组件 + 主题 + UI + 关联代码 + 潜在冲突 + API/配置核实，六者都过一遍才算做完。
2. **颜色与尺寸禁止硬编码**：颜色走主题资源或 ColorStateList，间距走 `@dimen/spacing_*`（4dp 基准倍数），字体 `sp`、尺寸 `dp`，禁止裸写 `#xxxxxx` 和数字。
3. **控件不许裸上屏**：所有控件必须显式指定样式（统一定义在 `styles.xml`，布局里用 `style=` 引用），禁止系统默认样式直接上屏；相邻控件必须留间距，同类控件间距必须一致。
4. **编译通过 ≠ 完成**：必须装到实机 / 实际环境跑一遍，用日志和命令取证，不能只看 BUILD SUCCESSFUL。
5. **API 与配置项必须核实**：调用系统 API、读 `Settings.*` / `device_config` / `SharedPreferences` / provider 前，先确认 key 属于哪个 namespace、真实存储位置。用错表会静默返回 null，不报错但恒错。
6. **外部 IO 必须容错**：`exists()` 不代表可读可写，所有 `readText()` / `writeText()` / `delete()` / `listFiles()` 用 `runCatching` 包裹并降级，绝不裸抛。
7. **工具链必须同架构**：arm64 机器只下 aarch64 版本，装完校验 ELF `e_machine=0xB7`；给 arm64 下 x86_64 会浪费大量时间修「为什么跑不起来」。
8. **原生库必须 16KB 页对齐**：arm64-v8a 的 .so 链接加 `-Wl,-z,max-page-size=16384`，打包前用 `llvm-readelf -l | grep LOAD` 确认 `Align ≥ 0x4000`。
9. **密钥零硬编码、签名统一**：API Key / Token / 密码走不入库的 `gradle.properties` + `buildConfigField`；项目自带专用 debug keystore，签名文件进 `.gitignore`。
10. **改完立即留痕**：当天 devlog 立刻追加（背景、改动文件、关键实现、验证结果、待办），不攒到最后补。

## 三、使用方式

- **开工前**：扫一遍上表定位分册，读对应那份；动手写第一行代码前完成对齐。
- **改完前**：对照分册里的检查清单自查（UI 有 7 项清单，发版有 release 自查），清单项逐条过，不跳。
- **验证时**：优先日志和命令取证，不用「看起来没问题」代替实测。
- **遇到本技能没覆盖的坑**：先取证再下结论，然后把新踩的坑按场景补进对应分册，别只留在脑子里。

## 四、优先级

本技能内容与临时想法冲突时，以本技能为准。若某条在实际项目里确实不适用（如非 Android 项目遇到 Android 专属条目），按「场景不适用」跳过，并在 devlog 里记一句原因，不要默默忽略。
