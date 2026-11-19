# 发版、签名与安全

> 适用时机：打 release 包、配签名、加混淆规则、处理密钥、提交代码时。

---

## 可复用签名（换环境一致）

- **调试签名也要统一**：默认 `~/.android/debug.keystore` 是每台机器各自的，换环境构建签名就变，导致**无法覆盖安装**（签名不一致要先卸载，丢数据）。项目应自带一个专用 debug keystore，所有开发环境用同一个。
- **keystore 放仓库外的固定目录**（如项目上层 `keystore/`），`build.gradle` 用 `rootProject.file("../keystore/xxx.jks")` 相对引用，避免硬编码绝对路径。
- **签名文件不入库**：`.gitignore` 必须排除 `*.jks` / `*.keystore` / `keystore/`（见 Git 规范）。keystore 单独备份/同步，换环境时复制过去。
- **debug 和 release 可共用一个 keystore**（保证覆盖安装一致）；release 若需独立正式签名，走环境变量/不入库的 `gradle.properties` 注入，别把正式密码写进 build.gradle。
- **验证签名生效**：构建后 `apksigner verify --print-certs app-debug.apk` 看 `Subject DN` 是不是你的 keystore，而不是 `Android Debug`。

---

## 混淆与发版验证

- **混淆规则同步**：引入新库、使用反射、序列化 `Parcelable` 时，必须同步在 `proguard-rules.pro` 中添加 `-keep` 规则。
- **Release 实机验证**：发版前必须开启 `minifyEnabled true` 打 release 包，并装到实机完整走一遍核心流程（debug 包不混淆容易掩盖问题）。

---

## 敏感信息与日志管控

- **密钥零硬编码**：API Key / Token / 密码严禁写在源码或 XML 中。统一放 `gradle.properties`（不入库）或 `local.properties`，通过 `buildConfigField` 注入。
- **日志封装**：使用统一的日志工具类，Release 包自动关闭 `VERBOSE` / `DEBUG` 级别，严禁打印密码、私钥、用户隐私数据。
- **adb 调试属性**：`adb shell setprop log.tag.xxx` 仅限本地临时调试，不得写入正式业务逻辑。

---

## Git 与提交规范

- **分支策略**：功能开发走 `feat/xxx`，修 Bug 走 `fix/xxx`，禁止直接提交到 `main` / `master`。
- **提交信息**：采用 `类型(范围): 简述` 格式（如 `fix(ui): 修复列表错位`），方便自动生成变更日志。
- **禁止入库的文件**：`.gitignore` 必须覆盖 `local.properties`、签名文件（`*.jks`/`*.keystore`）、`build/` 目录及 IDE 配置；严禁把敏感信息提交到仓库。
