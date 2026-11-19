# 代码质量与运行时

> 适用时机：写业务逻辑、异步/协程、列表、文件 IO、Context 引用时。

---

## 通用工程习惯

- 优先用成熟方案不自造轮子；解析/渲染/编解码/协议/格式转换先找成熟库，别手写正则解析标准格式。
- **方法/函数复用**：开发新功能调用方法时先想「这个逻辑会不会在别处重复用到」。会复用的抽成**公共函数库/工具类**（如 `util/` 下的单例 object），避免重复造轮子、避免多处各写一份改一处漏一处。
- **卡住时参考成熟开源项目**：遇到解决不了的问题，先查成熟开源项目怎么实现（思路/架构/边界处理），借鉴实现思路而不是照抄代码。**同时保证不侵权**：只参考思路、自己实现；确需引入第三方库/代码时核对 License（Apache-2.0/MIT/BSD 可用，GPL 传染性慎用，专有协议禁用），并在注释/CREDITS 标注来源与协议。
- 全中文交流。
- 防崩溃：可能返回 null 的调用判 null；Fragment 网络回调 `runOnUiThread` 里加 `isAdded()`；`lateinit` 属性在首次使用前必须确保已赋值（漏调 `init()` 会抛 `UninitializedPropertyAccessException`，这种崩溃出现在启动路径上时最难定位）。
- **文件该存哪，比"要不要容错"更上游**：App 自有数据走 `context.filesDir` / `getExternalFilesDir()` / `MediaStore`，别用 `Environment.getExternalStorageDirectory()`——它在 Android 10+ 已废弃，且受 Scoped Storage 限制，沙盒外读写会直接失败或被迫申请 `MANAGE_EXTERNAL_STORAGE` 这类高风险权限。目录选错，后面写多少容错都救不回来。
- **外部 IO / 文件读取必须容错（强制）**：读写**外部存储 / 共享目录 / 用户路径**的文件时，`exists()` 只保证存在、**不保证可读可写**（权限未授予/路径受限时抛 `EACCES Permission denied`）。所有 `readText()` / `writeText()` / `delete()` / `listFiles()` 必须 `runCatching { ... }.getOrNull()/getOrDefault()` 包裹，**失败降级（当 null/跳过/提示）绝不裸抛**。否则一个文件读不到就会异常一路抛到 `onResume`/`onCreate` **崩掉整页**。判 null 只管返回值，管不了 IO 抛异常——两者都要防。
- **文件 IO 不许占主线程**：同步 `readText()` / `writeText()` / `listFiles()` 一律包进 `withContext(Dispatchers.IO)`，或对外提供 `suspend` 接口。主线程做文件 IO 会触发 StrictMode 告警，大文件直接 ANR——界面卡住比崩掉更难排查。
- 点击运行脚本/长耗时准备时**先立刻切到进度界面并提示**再后台准备，别等准备完才切页。

---

## Kotlin 协程与异步处理

- **禁用 GlobalScope**：UI 层强制使用 `lifecycleScope` / `viewModelScope`；底层库使用自定义 `CoroutineScope` + `SupervisorJob` 便于统一取消。
- **异常与取消**：耗时操作（`Dispatchers.IO`）切回主线程时严禁用 `try-catch` 吞掉 `CancellationException`；`withContext` 的异常必须向上传播。
- **回调改造**：跨模块通信优先改为 `suspend` 函数挂起，禁止新增多层嵌套的接口回调。

---

## 内存泄漏与 Context 管理

- **Context 防泄漏**：单例、后台任务、静态变量禁止持有 `Activity` Context，统一使用 `ApplicationContext`。
- **资源释放**：非静态 `Handler` / `Thread` 必须在 `onDestroy` 中移除回调或中断；动态注册的 `BroadcastReceiver` / `ContentObserver` 必须在对应生命周期反注册。
- **Lifecycle 感知**：使用 `LifecycleObserver` 时确保在 `onDestroy` 中 `removeObserver`，避免 Fragment/Activity 无法被 GC 回收。

---

## 列表性能与图片加载

- **强制 RecyclerView**：动态列表一律使用 `RecyclerView` + `ListAdapter` / `AsyncListDiffer` + `DiffUtil`，禁止在 UI 线程手动全量 `notifyDataSetChanged`。
- **图片加载库**：统一使用 Coil 或 Glide 加载网络/本地图片，禁止手写 `decodeStream` 直接塞 `Bitmap`（极易 OOM）；列表图片必须设置 `placeholder` 和 `error` 占位图。
