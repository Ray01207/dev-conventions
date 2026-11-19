# 组件开发与 UI 规范

> 适用时机：新增 / 修改任何组件、布局、样式、颜色、间距、主题时。
> 这是本套约定里体量最大的一册，覆盖「组件六原则 + 深色适配 + UI 设计规范」。

---

## 组件开发与改动通用原则（强制）

每开发一个组件、每改一处代码，都要**整体考虑适配和冲突**，不能只看眼前这一块：

- **主题适配**：新组件 / 改动必须跟随主题（浅色 / 深色 / 动态主题色）。颜色用主题属性或 ColorStateList，禁止硬编码色值；切主题后控件颜色要自动跟随。
- **UI 适配**：考虑不同屏幕尺寸、深色模式、挖孔屏 / 状态栏、横竖屏。尺寸用 `dp`/`sp`，间距走统一 dimens，布局不能写死像素或状态栏高度。
- **关联代码适配**：改一个组件要排查**所有引用它的地方**（布局、代码、样式、Adapter、其它 Fragment/Activity），同步更新，别漏改导致风格或行为不一致。
- **改动要考虑 UI 变化**：改逻辑 / 数据 / 配置前，先想清楚会带来什么界面变化（列表刷新、状态图标、按钮态、空态），确保 UI 正确反映新状态。
- **改动要考虑功能冲突**：改代码前排查会不会和已有功能冲突（同 ID 控件、同名资源、重复逻辑、并发 / 生命周期冲突），避免改 A 坏 B。
- **API / 配置 / 设置项必须核实，不能想当然（强制）**：调用系统 API、读写配置项（`Settings.*` / `device_config` / `SharedPreferences` / provider / 环境变量）前，**先核实它真实的存储位置、key 名和读取路径**，别凭记忆或"听起来合理"就用。同类设置可能分散在不同表（如 Android 的 `device_config` vs `Settings.Global`），用错表 API 会静默返回 null / 默认值，**不报错但恒错**（真实踩坑：用 `Settings.Global.getString` 读一个 device_config 的 key，返回 null 导致状态恒判错）。核实手段：官方文档确认 key 属于哪个 namespace、用 `adb shell settings/device_config get` 取真实值、跑起来打印实际读到的值。读不到 / 取不到时明确降级提示「无法确认，请手动检查」，**不要假装知道状态**。

> 一句话：**开发 = 组件 + 主题 + UI + 关联代码 + 潜在冲突 + API/配置核实**，六者一起考虑才算做完。

---

## 深色模式与屏幕适配

- **主题可扩展（核心）**：所有颜色、间距、圆角、字体统一收口到主题资源（`colors.xml` / `themes.xml` / `dimens.xml` / ColorStateList / Theme 属性），**代码和布局里禁止硬编码**。这样后续加深色模式、换配色、加新主题（如多套配色）时，只改资源文件即可全局生效，不用逐个控件改代码。
- **深色模式**：所有颜色必须定义 `res/values/colors.xml` 与 `res/values-night/colors.xml`（或依赖 Theme 属性），代码里严禁写死 `#FFFFFF` 等色值。
- **沉浸式与挖孔屏**：根布局合理使用 `android:fitsSystemWindows="true"` 或监听 `WindowInsets`，禁止写死状态栏高度，避免遮挡挖孔/摄像头区域。
- **尺寸单位**：字体统一用 `sp`，间距/尺寸统一用 `dp`，Kotlin 代码里设置宽高也必须做 `dp2px` 转换，严禁出现裸 `px` 赋值。

---

## UI 设计规范（强制）

### 核心原则

> **严禁使用任何默认样式 / 裸控件直接上屏。** 所有控件必须经过自定义样式包裹或 XML 属性显式定义。相同类型的控件间距必须一致，相邻控件之间必须留有间距，禁止紧贴。

### 一、禁止默认样式

- **所有控件必须显式指定样式**，禁止出现"系统长什么样就什么样"的裸控件。
- 全局基础控件样式统一定义在 `res/values/styles.xml` 中，至少覆盖 TextView / Button / EditText / CardView（字号、颜色、圆角、高度、padding、背景、elevation、stroke）。
- Material 组件必须继承 `Widget.Material3.*` 并二次定制，不允许直接用裸标签。
- **文本也要收口**：界面文字禁止直写布局（如 `android:text="设备状态"`），统一走 `res/values/strings.xml`。硬编码文字和硬编码色值是同一类问题——改文案要逐处找、将来做多语言无从下手。

### 二、间距系统（Spacing System）

- **间距只使用 4dp 基准的倍数**：4 / 8 / 12 / 16 / 24 / 32 dp。
- **统一定义在 `res/values/dimens.xml`**：`spacing_xs=4dp` / `sm=8dp` / `md=12dp` / `lg=16dp` / `xl=24dp` / `xxl=32dp`。
- **页面级容器**：根布局 padding 统一 `@dimen/spacing_lg`（16dp）。
- **同组控件垂直间距**：统一 `@dimen/spacing_md`（12dp），同组内的相关控件用 `@dimen/spacing_sm`（8dp）。
- **卡片之间 / 列表项间距**：统一 `@dimen/spacing_md`（12dp），列表容器顶部/底部留 `@dimen/spacing_lg`（16dp）。

### 三、相同控件间距相同

- **同类型控件间距必须完全一致**，不允许 A 处按钮间距 8dp、B 处按钮间距 16dp。
- 所有水平并排按钮间距统一 `@dimen/spacing_md`（12dp）：`layout_marginStart="@dimen/spacing_md"`，禁止左右各 6dp 拆分的写法。
- 所有垂直相邻输入框间距统一 `@dimen/spacing_md`（12dp）。
- 所有标签-内容键值行间距统一 `@dimen/spacing_sm`（8dp）。
- **动态 addView 的控件同样遵守**：代码中 `LayoutParams` 的 margin 必须从 `R.dimen.spacing_*` 读取，禁止写死 `12`。

### 四、相邻控件必须留间距

- **任意两个相邻控件之间必须有明确的 margin 或 padding 间隔，禁止紧贴。**
- **容器边缘也要留间距**：内容距屏幕边缘（CardView 边缘）必须有 padding，不允许内容贴边。
- **特殊例外**：仅以下场景允许 0 间距——卡片内部内容区与 CardView 边缘之间（已由 contentPadding 处理）；Divider 分割线。

### 五、控件尺寸规范

| 控件类型 | 最小高度 | 圆角 | 说明 |
|---|---|---|---|
| 主按钮 | 48dp | 12dp | 全宽或自适应，padding 水平 24dp |
| 次按钮（描边） | 48dp | 12dp | stroke 1dp，padding 水平 24dp |
| 输入框 | 56dp | 12dp | 背景带边框或填充色 |
| MaterialCardView | — | 16dp | stroke 1dp、elevation 0 |
| 图标 | 20dp / 24dp | — | 统一 20dp，强调区 24dp |
| 列表项 | 64dp+ | 16dp | CardView 包裹 |

### 六、水平并排按钮间距

- 水平并排按钮 ≥ 12dp 间距，禁止两个 `weight=1` 按钮紧贴。
- 动态 addView 的按钮行同样设 `@dimen/spacing_md` 间距。
- 破坏性按钮（删除/恢复/格式化）与相邻按钮 ≥ 16dp，用**描边次要样式**区分 + **必须二次确认**；禁止 `colorError` 文字色。

### 七、列表项规范

- 动态列表项必须用 `item_*.xml` + `LayoutInflater.inflate()`，禁止 Kotlin 层层 addView 拼。
- 每项一个 MaterialCardView（圆角 16dp、stroke 1dp、elevation 0），项间距 `@dimen/spacing_md`（12dp）。
- 信息用「label 固定宽 72dp + value weight 1」键值行分行展示，value `singleLine` + `ellipsize="end"`，别把所有字段拼一坨。
- 目录扫描类列表必须过滤备份/临时/隐藏文件（`.bak` `.old` `.tmp` `~`、`.` 开头）。

### 八、状态图标与颜色

- 状态图标统一 20dp + fitCenter，图形贴合语义，颜色跟随主题（ColorStateList），禁止硬编码色值，tint 由代码 `setImageTintList` 设。
- 已安装组件状态行**必须显示真实版本号**，多源回退读取，拿不到写「版本未知」。

### 九、表单与交互

- 表单填 API Key/地址/连接参数**必须带「测试连接」按钮**就地验证；可选项用下拉选 + 留「自定义」兜底。
- 首页只做状态展示，低频/破坏性操作（备份/恢复/删除/清空）放独立工具页或设置页。
- 长耗时任务走**前台服务 + WakeLock + 进度条**，不放易被冻结的协程。

### 十、代码检查清单（提交前自查）

- [ ] 所有控件是否都带 style 或显式定义了外观属性？
- [ ] 是否存在任何两个相邻控件紧贴（间距为 0）的情况？
- [ ] 相同类型的控件间距是否一致（全局统一）？
- [ ] 所有间距是否引用 `R.dimen.spacing_*` 而非裸写数字？
- [ ] 是否存在硬编码色值（`#xxxxxx`）？
- [ ] 是否存在硬编码 `dp`/`px` 数值而非引用 dimens？
- [ ] 动态 addView 的控件是否设置了统一间距？

---

## 与之相关的两条通用习惯

- 颜色跟随主题，禁止硬编码色值。
- UI 改动先询问再动手，别自行调整视觉。
