# 考研倒计时 · Android 开发 Prompt

把本文「Prompt」一节整段交给你电脑上的 AI Agent（Claude Code、Cursor、Codex 等），它会从零做出一个装在 Redmi K80 至尊版（K80U）上的考研倒计时 App。

## 做出来是什么样

所有显示位置都是**每 1 秒跳动一次**，显示**小时数、分钟数、秒数**。小时不进位成天。

| 显示位置 | 样子 | 说明 |
|---|---|---|
| App 全屏页 | `1788 小时`<br>`22 分 05 秒` | 打开 App 就是大号倒计时 |
| 桌面小组件 · 方块（2×2） | `1788:22:05`<br>下方标注「小时 : 分 : 秒」 | 秒数由系统时钟控件自己走，不耗电、不怕被杀后台 |
| 桌面小组件 · 横条（4×1） | `考研初试   1788:22:05` | 同上 |
| 通知栏 / 锁屏通知（可选，默认关） | `距 考研初试 · 1788:22:05` | 系统通知自带的倒计时，同样不耗电 |

为什么小组件用冒号格式：安卓小组件里唯一能自己每秒走字的控件是系统 `Chronometer`，它只能输出 `时:分:秒` 这一种格式。如果改成 App 每秒推送一次「1788 小时 22 分 05 秒」，就得让 App 常驻后台，HyperOS 会把它杀掉，数字就停了。所以小组件用冒号格式，下方加一行单位标注。

默认目标：**2026 年 12 月 19 日 00:00（北京时间）**，名称「考研初试」。装好后可在 App 里随时修改。

## 开始前你要准备的

### 电脑

1. 安装 [Android Studio](https://developer.android.com/studio)（国内可用 https://developer.android.google.cn/studio ）。装完打开一次，按向导装好默认 SDK，然后可以关掉。它自带 JDK 和 Android SDK，Agent 会直接用。
2. 准备好你的本地 Agent。

### 手机（Redmi K80 至尊版）

1. **打开开发者选项**：设置 → 我的设备 → 全部参数与信息 → 连续点击「OS 版本」7 次，直到提示已进入开发者模式。
2. **进入开发者选项**：设置 → 更多设置 → 开发者选项。
3. 打开这三个开关：
   - 「USB 调试」
   - 「USB 安装」
   - 「USB 调试（安全设置）」（需要登录小米账号，部分系统版本还要求插着 SIM 卡）
4. 用数据线连电脑。手机弹出「允许 USB 调试吗？」时，勾选「一律允许」再点确定。USB 用途选「传输文件」。

### 开工

1. 在电脑上新建一个空文件夹，比如 `D:\kaoyan-countdown` 或 `~/kaoyan-countdown`。
2. 在这个文件夹里启动 Agent。
3. 把下面「Prompt」一节代码框里的**全部内容**复制粘贴给它。
4. 过程中 Agent 会请你做几件只能手动完成的事：在手机上点「继续安装」、把小组件拖到桌面、打开自启动权限。照做即可。

---

## Prompt

````markdown
# 0. 你的角色和总要求

你是资深 Android 工程师。请在**当前空目录**里从零开发、编译、安装并验证一个原生 Android App「考研倒计时」，目标设备是通过 USB 连着这台电脑的 **Redmi K80 至尊版（小米 HyperOS，Android 15 或 16）**。

我是考研学生，不懂 Android 开发。规则如下：

- 本文档里的需求、默认值、设计都是最终决定。**不要再问我设计问题**，直接按文档做。
- 只有必须由我亲手操作的事才停下来找我：在手机上点按钮、插拔数据线、安装软件。找我时用中文写清楚「去哪里 → 点什么」，一次只给一组操作。
- 每个阶段结束都要**真的编译、真的装到手机、真的截图检查**。不能只写代码不运行。
- 遇到报错先读完整输出，定位原因再改，不要盲目重试同一条命令。第 11 节有常见问题的解法。
- 第 10 节「禁止事项」必须遵守。
- 全程用中文跟我沟通，简洁直白。

---

# 1. 产品需求（全部必须实现）

## 1.1 核心概念

- **目标**：一个名称 + 一个时刻。默认名称「考研初试」，默认时刻 `2026-12-19 00:00`，时区固定 `Asia/Shanghai`（北京时间）。时区不提供修改入口：考试时间就是北京时间，手机时区变了也不影响。
- **剩余时间** = 目标时刻 − 当前时刻，以**整秒向下取整**。拆成：
  - 小时：不封顶，不进位成天。例如 74 天 12 小时显示为 1788 小时。
  - 分钟：0–59，两位补零。
  - 秒：0–59，两位补零。
- **刷新频率**：所有显示位置每 1 秒跳一次。
- **到点**：剩余时间 ≤ 0 时进入「已到达」状态。设置里有一个选项「到点后」：
  - `显示已到达`（默认）：显示「已到达」，秒数停止。
  - `继续正计时`：显示已过去的时间，前面加 `+`，例如 `+0:12:03`。

## 1.2 显示位置一：App 全屏页（主界面）

打开 App 直接进入此页。从上到下排列：

1. 右上角一个齿轮图标按钮，点击进入设置页。
2. 目标名称，例如「距 考研初试」：20sp，强调色。
3. 目标时刻，例如「2026年12月19日 星期六 00:00（北京时间）」：14sp，次要文字色。
4. 留出 48dp 空白。
5. 第一行大数字：`1788` + 小号单位 `小时`。
6. 第二行大数字：`22` + 小号单位 `分`，空一格，`05` + 小号单位 `秒`。
7. 底部居中一行小字「每一秒都算数」：12sp，次要文字色。

其他要求：

- 数字字号：第一行最大 96sp，第二行最大 72sp。单位字号 = 数字字号 × 0.28（用 `em` 相对单位实现，见第 4.9 节）。数字放不下一行时自动缩小，见第 4.9 节的 `AutoShrinkText`。
- 数字必须**等宽**（`fontFeatureSettings = "tnum"`），跳动时整行宽度不变、不左右抖。
- 剩余时间小于 24 小时：数字变成强调色。
- 已到达且选择「显示已到达」：两行数字换成一行「已到达」（72sp）和一行「加油！」（24sp）。
- 已到达且选择「继续正计时」：第一行是 `+1 小时`，第二行同格式，单位前加一行小字「已过去」。
- 秒数在**系统时钟的整秒边界**跳动，做法见第 4.9 节。
- App 退到后台时停止计时循环，回到前台立刻恢复并校正。
- 设置里开了「保持屏幕常亮」时，此页显示期间屏幕不灭。
- 全面屏适配：`enableEdgeToEdge()`，内容加 `safeDrawingPadding()`，状态栏图标为浅色。

## 1.3 显示位置二：桌面小组件（两种尺寸）

在 HyperOS 小部件列表里出现两个可选项：

| 名称（小部件列表中显示） | 默认尺寸 | 内容 |
|---|---|---|
| 考研倒计时 · 方块 | 2×2 | 第 1 行「距 考研初试」；第 2 行大号 `1788:22:05`；第 3 行「小时 : 分 : 秒」；第 4 行「12月19日 00:00」 |
| 考研倒计时 · 横条 | 4×1 | 左侧上「距 考研初试」、下「12月19日 00:00」；右侧上大号 `1788:22:05`、下「小时 : 分 : 秒」 |

要求：

- 秒数跳动**必须**用 `Chronometer` 控件实现：`RemoteViews.setChronometer` + `setChronometerCountDown(true)`。它在桌面进程里自己走秒，App 进程不用活着。
- 两种都能自由调整大小，数字随尺寸自动缩放（XML autosize）。
- 点击小组件任意位置打开 App 全屏页。
- 支持三种外观，在 App 设置里切换，所有小组件同时生效：
  - `深色`（默认）：近黑底、白字。
  - `浅色`：白底、黑字。
  - `透明`：半透明黑底、白字。
- 到点时（App 不在前台也要生效）自动切换为「已到达」或正计时。
- 以下情况发生后，小组件数值要自动校正：手机重启、手动改系统时间、改时区、App 升级重装。
- 小组件列表里有预览图（`previewLayout`），预览里显示静态的 `1788:00:00`。

## 1.4 显示位置三：常驻通知（可选功能，默认关闭）

- 设置里有开关「在通知栏和锁屏显示倒计时」。打开时，若没有通知权限，先申请（Android 13+）。
- 通知内容：标题「距 考研初试」，正文「2026年12月19日 00:00」，右上角时间位置显示系统倒计时（`setUsesChronometer(true)` + `setChronometerCountDown(true)` + `setWhen(目标时刻)`）。
- 静音、不振动、不弹横幅：通知渠道重要性用 `IMPORTANCE_LOW`。`ongoing = true`，锁屏可见（`VISIBILITY_PUBLIC`）。
- 点击通知打开 App。
- 到点后按「到点后」设置更新通知。关闭开关时立即移除通知。
- 不使用前台服务。

## 1.5 设置页

从上到下：

1. **目标名称**：输入框，最多 12 个字，不能为空。
2. **目标日期**：点击弹出 Material3 日期选择器。
3. **目标时间**：点击弹出 Material3 时间选择器，24 小时制，精确到分钟。
4. 一行只读说明「时区：北京时间（Asia/Shanghai）」。
5. **到点后**：单选，「显示已到达」/「继续正计时」。
6. **小组件外观**：单选，「深色」/「浅色」/「透明」。
7. **在通知栏和锁屏显示倒计时**：开关。
8. **全屏页保持屏幕常亮**：开关，默认关。
9. **添加小组件到桌面**：两个按钮「添加方块」「添加横条」，调用 `AppWidgetManager.requestPinAppWidget`。如果桌面不支持（`isRequestPinAppWidgetSupported() == false`），按钮改为显示文字说明：「长按桌面空白处 → 添加小部件 → 搜索『考研倒计时』」。
10. **后台权限检查**：见第 5 节。
11. 底部两个按钮：「恢复默认」「保存」。
    - 点「保存」：校验输入 → 写入存储 → 调用 `CountdownSync.refreshAll(context)` → Toast「已保存」→ 返回全屏页。
    - 选择的时刻早于当前时间时，先弹确认框「这个时间已经过去了，确定保存吗？」。
    - 点「恢复默认」：弹确认框，确认后恢复第 1.1 节的默认值并保存。

---

# 2. 技术选型（已定，不要更换）

| 项目 | 选择 |
|---|---|
| 语言 | Kotlin |
| App 界面 | Jetpack Compose + Material 3 |
| 小组件 | 传统 `AppWidgetProvider` + XML 布局 + `RemoteViews`。**不要用 Jetpack Glance**，它不支持 `Chronometer` |
| 设置存储 | `SharedPreferences`：广播接收器里需要同步读，比 DataStore 简单可靠 |
| 构建 | Gradle Kotlin DSL + `gradle/libs.versions.toml` + Gradle Wrapper |
| 包名 / applicationId | `com.kaoyan.countdown` |
| App 显示名 | 考研倒计时 |
| minSdk | 31 |
| compileSdk / targetSdk | 本机已安装的最高稳定版 SDK，至少 35 |
| 版本号 | versionCode 1，versionName "1.0" |
| 第三方依赖 | 只用 AndroidX 和 Kotlin 官方库，不加网络、统计、广告 SDK |
| 权限 | 只申请第 3.3 节清单里的权限，**不要申请 INTERNET** |

### 2.1 依赖版本怎么定

1. 优先用最新稳定版组合：
   - Android Gradle Plugin 和 Gradle Wrapper 的对应关系，查 https://developer.android.com/build/releases/gradle-plugin
   - Compose BOM 查 https://developer.android.com/develop/ui/compose/bom/bom-mapping
   - Kotlin 和 Compose 编译器插件版本必须一致（`org.jetbrains.kotlin.plugin.compose`）
2. 如果 AGP 主版本 ≥ 9：按官方迁移说明处理内置 Kotlin 支持。如果构建提示不要再单独应用 `org.jetbrains.kotlin.android`，就照提示去掉。
3. 查不到或新组合编译失败，就用这套**保底组合**，它们彼此兼容：
   - AGP 8.7.3
   - Gradle 8.9
   - Kotlin 2.0.21（含 compose 插件 2.0.21）
   - Compose BOM 2024.12.01
   - androidx.core:core-ktx 1.15.0
   - androidx.activity:activity-compose 1.9.3
   - androidx.lifecycle:lifecycle-runtime-ktx 2.8.7
   - androidx.lifecycle:lifecycle-runtime-compose 2.8.7
   - compileSdk / targetSdk 35
   - 单元测试用 junit:junit 4.13.2
4. 不要凭空编版本号。写进 `libs.versions.toml` 的每个版本都要确认存在。

### 2.2 国内网络

如果 Gradle 或依赖下载超时：

- 在 `settings.gradle.kts` 的 `pluginManagement.repositories` 和 `dependencyResolutionManagement.repositories` 中，把下面三个镜像放在 `google()`、`mavenCentral()`、`gradlePluginPortal()` 前面：
  - `https://maven.aliyun.com/repository/google`
  - `https://maven.aliyun.com/repository/public`
  - `https://maven.aliyun.com/repository/gradle-plugin`
- `gradle/wrapper/gradle-wrapper.properties` 的 `distributionUrl` 改用腾讯镜像，例如 `https\://mirrors.cloud.tencent.com/gradle/gradle-8.9-bin.zip`（版本号替换成实际使用的）。

---

# 3. 项目结构

## 3.1 目录和文件（全部要创建）

```
kaoyan-countdown/                      ← 当前目录
├── .gitignore
├── README.md                           ← 给我看的中文使用说明（第 6 节第 11 步）
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── gradle/
│   ├── libs.versions.toml
│   └── wrapper/ (gradle-wrapper.jar, gradle-wrapper.properties)
├── gradlew
├── gradlew.bat
├── local.properties                    ← sdk.dir，不提交
├── keystore.properties                 ← 签名密码，不提交
├── keystore/release.jks                ← 签名文件，不提交
├── dist/                               ← 最终 APK 输出处
└── app/
    ├── build.gradle.kts
    ├── proguard-rules.pro
    └── src/
        ├── main/
        │   ├── AndroidManifest.xml
        │   ├── java/com/kaoyan/countdown/
        │   │   ├── CountdownApp.kt                 ← Application，创建通知渠道
        │   │   ├── core/Countdown.kt               ← 纯计算与格式化（无 Android 依赖）
        │   │   ├── data/Settings.kt                ← 设置数据类与枚举
        │   │   ├── data/SettingsRepository.kt      ← SharedPreferences 读写
        │   │   ├── sync/CountdownSync.kt           ← 统一刷新：小组件 + 闹钟 + 通知
        │   │   ├── widget/CountdownWidgets.kt      ← 两个 AppWidgetProvider
        │   │   ├── widget/WidgetRenderer.kt        ← 构建 RemoteViews
        │   │   ├── receiver/SystemEventReceiver.kt ← 开机/改时间/改时区/升级/到点
        │   │   ├── notify/CountdownNotifier.kt     ← 常驻通知
        │   │   ├── system/HyperOsSettings.kt       ← 跳转小米权限页
        │   │   └── ui/
        │   │       ├── MainActivity.kt
        │   │       ├── CountdownScreen.kt
        │   │       ├── SettingsScreen.kt
        │   │       ├── AutoShrinkText.kt
        │   │       └── theme/Theme.kt
        │   └── res/
        │       ├── layout/widget_square.xml
        │       ├── layout/widget_bar.xml
        │       ├── layout/widget_square_preview.xml
        │       ├── layout/widget_bar_preview.xml
        │       ├── xml/widget_square_info.xml
        │       ├── xml/widget_bar_info.xml
        │       ├── drawable/widget_bg_dark.xml
        │       ├── drawable/widget_bg_light.xml
        │       ├── drawable/widget_bg_translucent.xml
        │       ├── drawable/ic_launcher_foreground.xml
        │       ├── drawable/ic_stat_countdown.xml
        │       ├── mipmap-anydpi-v26/ic_launcher.xml
        │       ├── mipmap-anydpi-v26/ic_launcher_round.xml
        │       ├── values/colors.xml
        │       ├── values/strings.xml
        │       └── values/themes.xml
        ├── debug/
        │   ├── AndroidManifest.xml                 ← 只在 debug 包里注册调试接收器
        │   └── java/com/kaoyan/countdown/debug/DebugCommandReceiver.kt
        └── test/java/com/kaoyan/countdown/core/CountdownTest.kt
```

## 3.2 `.gitignore`

```
.gradle/
build/
app/build/
local.properties
keystore.properties
keystore/
*.jks
*.keystore
.idea/
*.iml
captures/
.cxx/
.DS_Store
shots/
```

`dist/` 要提交，方便我直接拿 APK。

## 3.3 `AndroidManifest.xml`（main）

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    <!-- 到点时精确切换显示。侧载安装，USE_EXACT_ALARM 安装即授予 -->
    <uses-permission android:name="android.permission.USE_EXACT_ALARM" />
    <uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM"
        android:maxSdkVersion="32" />

    <application
        android:name=".CountdownApp"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.KaoyanCountdown">

        <activity
            android:name=".ui.MainActivity"
            android:exported="true"
            android:launchMode="singleTask"
            android:theme="@style/Theme.KaoyanCountdown">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <receiver
            android:name=".widget.SquareCountdownWidget"
            android:exported="true"
            android:label="@string/widget_square_label">
            <intent-filter>
                <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
            </intent-filter>
            <meta-data
                android:name="android.appwidget.provider"
                android:resource="@xml/widget_square_info" />
        </receiver>

        <receiver
            android:name=".widget.BarCountdownWidget"
            android:exported="true"
            android:label="@string/widget_bar_label">
            <intent-filter>
                <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
            </intent-filter>
            <meta-data
                android:name="android.appwidget.provider"
                android:resource="@xml/widget_bar_info" />
        </receiver>

        <receiver
            android:name=".receiver.SystemEventReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <action android:name="android.intent.action.TIME_SET" />
                <action android:name="android.intent.action.TIMEZONE_CHANGED" />
                <action android:name="android.intent.action.MY_PACKAGE_REPLACED" />
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

`SystemEventReceiver` 还要处理 App 自己发的到点广播 `com.kaoyan.countdown.action.TARGET_REACHED`。这个广播用显式 Intent 发送，不需要写进 intent-filter。

---

# 4. 关键实现（参考代码）

下面的代码是参考实现，照此编写。编译不过时按报错修正，但**不得改变行为**。

## 4.1 `core/Countdown.kt`：纯计算（必须有单元测试）

```kotlin
package com.kaoyan.countdown.core

import java.time.Instant
import java.time.LocalDateTime
import java.time.ZoneId
import java.time.format.DateTimeFormatter
import java.util.Locale

val BEIJING: ZoneId = ZoneId.of("Asia/Shanghai")

/** 时:分:秒，hours 不封顶 */
data class Hms(val hours: Long, val minutes: Int, val seconds: Int)

sealed interface CountdownState {
    /** 还没到。remaining 为剩余时间（向下取整到秒） */
    data class Running(val remaining: Hms) : CountdownState
    /** 已到达。elapsed 为已过去的时间（向下取整到秒），刚好到点时为 0 */
    data class Reached(val elapsed: Hms) : CountdownState
}

fun secondsToHms(totalSeconds: Long): Hms {
    require(totalSeconds >= 0)
    return Hms(
        hours = totalSeconds / 3600,
        minutes = ((totalSeconds % 3600) / 60).toInt(),
        seconds = (totalSeconds % 60).toInt(),
    )
}

/**
 * diff > 0 才算 Running。最后不足 1 秒时显示 0:00:00，
 * 和系统 Chronometer 的截断行为一致。
 */
fun computeState(targetEpochMillis: Long, nowEpochMillis: Long): CountdownState {
    val diff = targetEpochMillis - nowEpochMillis
    return if (diff > 0) {
        CountdownState.Running(secondsToHms(diff / 1000))
    } else {
        CountdownState.Reached(secondsToHms((-diff) / 1000))
    }
}

/** 1788:22:05，与小组件 Chronometer 的格式一致 */
fun formatColon(h: Hms): String =
    String.format(Locale.ROOT, "%d:%02d:%02d", h.hours, h.minutes, h.seconds)

fun twoDigits(n: Int): String = String.format(Locale.ROOT, "%02d", n)

/** 北京时间的年月日时分 → epoch 毫秒，秒和毫秒清零 */
fun beijingToEpochMillis(dateTime: LocalDateTime): Long =
    dateTime.withSecond(0).withNano(0).atZone(BEIJING).toInstant().toEpochMilli()

fun epochMillisToBeijing(epochMillis: Long): LocalDateTime =
    Instant.ofEpochMilli(epochMillis).atZone(BEIJING).toLocalDateTime()

/** 2026年12月19日 星期六 00:00 */
fun formatTargetLong(epochMillis: Long): String =
    epochMillisToBeijing(epochMillis)
        .format(DateTimeFormatter.ofPattern("yyyy年M月d日 EEEE HH:mm", Locale.CHINA))

/** 12月19日 00:00 */
fun formatTargetShort(epochMillis: Long): String =
    epochMillisToBeijing(epochMillis)
        .format(DateTimeFormatter.ofPattern("M月d日 HH:mm", Locale.CHINA))
```

## 4.2 `test/.../CountdownTest.kt`：单元测试（全部要通过）

下面每个期望值都已人工核对过，不要改期望值去迁就代码。

```kotlin
package com.kaoyan.countdown.core

import org.junit.Assert.assertEquals
import org.junit.Assert.assertTrue
import org.junit.Test
import java.time.LocalDateTime
import java.time.ZoneId

class CountdownTest {
    private val target = beijingToEpochMillis(LocalDateTime.of(2026, 12, 19, 0, 0))
    private fun bj(y: Int, mo: Int, d: Int, h: Int, mi: Int, s: Int, ms: Int = 0): Long =
        LocalDateTime.of(y, mo, d, h, mi, s, ms * 1_000_000).atZone(BEIJING).toInstant().toEpochMilli()

    @Test fun defaultTargetEpoch() = assertEquals(1_797_609_600_000L, target)

    @Test fun oct5Noon() = assertEquals(
        CountdownState.Running(Hms(1788, 0, 0)),
        computeState(target, bj(2026, 10, 5, 12, 0, 0)))

    @Test fun sep1Morning() = assertEquals(
        CountdownState.Running(Hms(2607, 29, 45)),
        computeState(target, bj(2026, 9, 1, 8, 30, 15)))

    @Test fun oneHourOneMinThreeSec() = assertEquals(
        CountdownState.Running(Hms(1, 1, 3)),
        computeState(target, bj(2026, 12, 18, 22, 58, 57)))

    @Test fun lessThanOneSecondShowsZero() = assertEquals(
        CountdownState.Running(Hms(0, 0, 0)),
        computeState(target, target - 999))

    @Test fun truncatesNotRounds() = assertEquals(
        CountdownState.Running(Hms(0, 0, 59)),
        computeState(target, target - 59_999))

    @Test fun exactlyReached() = assertEquals(
        CountdownState.Reached(Hms(0, 0, 0)),
        computeState(target, target))

    @Test fun afterTarget() = assertEquals(
        CountdownState.Reached(Hms(0, 1, 1)),
        computeState(target, target + 61_900))

    @Test fun colonFormat() {
        assertEquals("1788:22:05", formatColon(Hms(1788, 22, 5)))
        assertEquals("0:00:09", formatColon(Hms(0, 0, 9)))
    }

    @Test fun longFormat() =
        assertEquals("2026年12月19日 星期六 00:00", formatTargetLong(target))

    @Test fun shortFormat() =
        assertEquals("12月19日 00:00", formatTargetShort(target))

    @Test fun secondsAreDropped() =
        assertEquals(target, beijingToEpochMillis(LocalDateTime.of(2026, 12, 19, 0, 0, 37, 5)))

    /** 用绝对时刻相减，跨夏令时也正确：纽约 2026-11-01 结束夏令时，这一天有 25 小时 */
    @Test fun dstSafe() {
        val ny = ZoneId.of("America/New_York")
        val a = LocalDateTime.of(2026, 10, 31, 12, 0).atZone(ny).toInstant().toEpochMilli()
        val b = LocalDateTime.of(2026, 11, 1, 12, 0).atZone(ny).toInstant().toEpochMilli()
        assertEquals(CountdownState.Running(Hms(25, 0, 0)), computeState(b, a))
    }

    @Test fun hoursNotCapped() {
        val s = computeState(target, bj(2025, 1, 1, 0, 0, 0)) as CountdownState.Running
        assertTrue(s.remaining.hours > 9999)
    }
}
```

## 4.3 `data/Settings.kt` 和 `data/SettingsRepository.kt`

```kotlin
package com.kaoyan.countdown.data

enum class AfterReached { SHOW_REACHED, COUNT_UP }
enum class WidgetTheme { DARK, LIGHT, TRANSLUCENT }

data class Settings(
    val targetName: String,
    val targetEpochMillis: Long,
    val afterReached: AfterReached,
    val widgetTheme: WidgetTheme,
    val notificationEnabled: Boolean,
    val keepScreenOn: Boolean,
) {
    companion object {
        const val DEFAULT_NAME = "考研初试"
        const val DEFAULT_TARGET = 1_797_609_600_000L // 2026-12-19 00:00 北京时间
        val DEFAULT = Settings(
            DEFAULT_NAME, DEFAULT_TARGET, AfterReached.SHOW_REACHED,
            WidgetTheme.DARK, notificationEnabled = false, keepScreenOn = false,
        )
    }
}
```

`SettingsRepository` 规格：

- SharedPreferences 文件名 `countdown_settings`。
- 键：`target_name`、`target_epoch_ms`、`after_reached`（存枚举 name）、`widget_theme`、`notification_enabled`、`keep_screen_on`。
- `fun load(): Settings`：同步读取。缺失或非法值回退到 `Settings.DEFAULT` 的对应字段，枚举解析失败也回退。
- `fun save(s: Settings)`：用 `commit()` 同步写入，保证随后 `refreshAll` 读到新值。
- `fun flow(): Flow<Settings>`：用 `callbackFlow` + `OnSharedPreferenceChangeListener` 实现，先发射当前值。给 Compose 用。
- 用 `object SettingsRepository { fun get(context: Context) }` 或单例，通过 `context.applicationContext` 取 SharedPreferences。

## 4.4 `sync/CountdownSync.kt`：唯一的刷新入口

所有会影响显示的事件都只调用这一个函数，不允许在别处单独更新小组件或通知。

```kotlin
package com.kaoyan.countdown.sync

object CountdownSync {
    const val TAG = "KaoyanCountdown"
    const val ACTION_TARGET_REACHED = "com.kaoyan.countdown.action.TARGET_REACHED"
    private const val REQ_TARGET = 1001

    fun refreshAll(context: Context) {
        val app = context.applicationContext
        val settings = SettingsRepository.get(app).load()
        WidgetRenderer.updateAll(app, settings)
        scheduleTargetAlarm(app, settings)
        CountdownNotifier.update(app, settings)
        Log.i(TAG, "refreshAll target=${settings.targetEpochMillis} now=${System.currentTimeMillis()}")
    }

    private fun scheduleTargetAlarm(context: Context, settings: Settings) {
        val am = context.getSystemService(AlarmManager::class.java)
        val pi = PendingIntent.getBroadcast(
            context, REQ_TARGET,
            Intent(context, SystemEventReceiver::class.java).setAction(ACTION_TARGET_REACHED),
            PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_UPDATE_CURRENT,
        )
        am.cancel(pi)
        val target = settings.targetEpochMillis
        if (target <= System.currentTimeMillis()) return
        // RTC 闹钟跟随墙上时钟，用户改时间后依然在目标时刻触发
        if (am.canScheduleExactAlarms()) {
            am.setExactAndAllowWhileIdle(AlarmManager.RTC_WAKEUP, target, pi)
        } else {
            am.setAndAllowWhileIdle(AlarmManager.RTC_WAKEUP, target, pi)
        }
    }
}
```

## 4.5 `receiver/SystemEventReceiver.kt`

```kotlin
class SystemEventReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Log.i(CountdownSync.TAG, "event ${intent.action}")
        when (intent.action) {
            Intent.ACTION_BOOT_COMPLETED,
            Intent.ACTION_TIME_CHANGED,      // 值为 android.intent.action.TIME_SET
            Intent.ACTION_TIMEZONE_CHANGED,
            Intent.ACTION_MY_PACKAGE_REPLACED,
            CountdownSync.ACTION_TARGET_REACHED -> CountdownSync.refreshAll(context)
        }
    }
}
```

## 4.6 `widget/CountdownWidgets.kt`

```kotlin
abstract class BaseCountdownWidget : AppWidgetProvider() {
    override fun onUpdate(context: Context, mgr: AppWidgetManager, ids: IntArray) =
        CountdownSync.refreshAll(context)
    override fun onEnabled(context: Context) = CountdownSync.refreshAll(context)
    override fun onAppWidgetOptionsChanged(
        context: Context, mgr: AppWidgetManager, id: Int, options: Bundle,
    ) = CountdownSync.refreshAll(context)
}

class SquareCountdownWidget : BaseCountdownWidget()
class BarCountdownWidget : BaseCountdownWidget()
```

## 4.7 `widget/WidgetRenderer.kt`：核心

```kotlin
object WidgetRenderer {
    private val targets = listOf(
        SquareCountdownWidget::class.java to R.layout.widget_square,
        BarCountdownWidget::class.java to R.layout.widget_bar,
    )

    fun updateAll(context: Context, s: Settings) {
        val mgr = AppWidgetManager.getInstance(context)
        for ((cls, layout) in targets) {
            val ids = mgr.getAppWidgetIds(ComponentName(context, cls))
            if (ids.isEmpty()) continue
            mgr.updateAppWidget(ids, build(context, layout, s))
        }
    }

    private fun build(context: Context, layout: Int, s: Settings): RemoteViews {
        val v = RemoteViews(context.packageName, layout)
        // 两个时钟必须在同一时刻读取，否则会差 1 秒
        val nowWall = System.currentTimeMillis()
        val nowElapsed = SystemClock.elapsedRealtime()
        val diff = s.targetEpochMillis - nowWall

        v.setTextViewText(R.id.widget_title, "距 ${s.targetName}")
        v.setTextViewText(R.id.widget_date, formatTargetShort(s.targetEpochMillis))

        when {
            diff > 0 -> {
                v.setViewVisibility(R.id.widget_chrono, View.VISIBLE)
                v.setViewVisibility(R.id.widget_reached, View.GONE)
                v.setChronometerCountDown(R.id.widget_chrono, true)
                // base 是 elapsedRealtime 时间轴上的「目标时刻」
                v.setChronometer(R.id.widget_chrono, nowElapsed + diff, null, true)
                v.setTextViewText(R.id.widget_caption, "小时 : 分 : 秒")
            }
            s.afterReached == AfterReached.COUNT_UP -> {
                v.setViewVisibility(R.id.widget_chrono, View.VISIBLE)
                v.setViewVisibility(R.id.widget_reached, View.GONE)
                v.setChronometerCountDown(R.id.widget_chrono, false)
                v.setChronometer(R.id.widget_chrono, nowElapsed + diff, "+%s", true)
                v.setTextViewText(R.id.widget_caption, "已过去 小时 : 分 : 秒")
            }
            else -> {
                v.setChronometer(R.id.widget_chrono, nowElapsed, null, false)
                v.setViewVisibility(R.id.widget_chrono, View.GONE)
                v.setViewVisibility(R.id.widget_reached, View.VISIBLE)
                v.setTextViewText(R.id.widget_caption, "加油！")
            }
        }

        applyTheme(v, s.widgetTheme)

        val open = PendingIntent.getActivity(
            context, 0,
            Intent(context, MainActivity::class.java).addFlags(Intent.FLAG_ACTIVITY_NEW_TASK),
            PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_UPDATE_CURRENT,
        )
        v.setOnClickPendingIntent(android.R.id.background, open)
        return v
    }

    private fun applyTheme(v: RemoteViews, t: WidgetTheme) {
        val (bg, main, sub) = when (t) {
            WidgetTheme.DARK -> Triple(R.drawable.widget_bg_dark, 0xFFF5F5F7.toInt(), 0xFF9A9AA5.toInt())
            WidgetTheme.LIGHT -> Triple(R.drawable.widget_bg_light, 0xFF141416.toInt(), 0xFF6B6B75.toInt())
            WidgetTheme.TRANSLUCENT -> Triple(R.drawable.widget_bg_translucent, 0xFFFFFFFF.toInt(), 0xCCFFFFFF.toInt())
        }
        v.setInt(android.R.id.background, "setBackgroundResource", bg)
        v.setTextColor(R.id.widget_chrono, main)
        v.setTextColor(R.id.widget_reached, main)
        v.setTextColor(R.id.widget_caption, sub)
        v.setTextColor(R.id.widget_date, sub)
        // widget_title 固定用强调色 #FF5A4E，不随主题变
    }
}
```

## 4.8 小组件 XML

### `res/layout/widget_square.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/background"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/widget_bg_dark"
    android:clipToOutline="true"
    android:gravity="center"
    android:orientation="vertical"
    android:padding="12dp">

    <TextView
        android:id="@+id/widget_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:ellipsize="end"
        android:gravity="center"
        android:maxLines="1"
        android:text="距 考研初试"
        android:textColor="#FF5A4E"
        android:textSize="14sp" />

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <Chronometer
            android:id="@+id/widget_chrono"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:autoSizeMaxTextSize="44sp"
            android:autoSizeMinTextSize="14sp"
            android:autoSizeStepGranularity="1sp"
            android:autoSizeTextType="uniform"
            android:fontFamily="sans-serif-medium"
            android:fontFeatureSettings="tnum"
            android:gravity="center"
            android:maxLines="1"
            android:textColor="#F5F5F7" />

        <TextView
            android:id="@+id/widget_reached"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:autoSizeMaxTextSize="36sp"
            android:autoSizeMinTextSize="14sp"
            android:autoSizeTextType="uniform"
            android:gravity="center"
            android:maxLines="1"
            android:text="已到达"
            android:textColor="#F5F5F7"
            android:visibility="gone" />
    </FrameLayout>

    <TextView
        android:id="@+id/widget_caption"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:maxLines="1"
        android:text="小时 : 分 : 秒"
        android:textColor="#9A9AA5"
        android:textSize="11sp" />

    <TextView
        android:id="@+id/widget_date"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="2dp"
        android:gravity="center"
        android:maxLines="1"
        android:text="12月19日 00:00"
        android:textColor="#9A9AA5"
        android:textSize="11sp" />
</LinearLayout>
```

### `res/layout/widget_bar.xml`

ID 必须和方块版完全一致（`@android:id/background`、`widget_title`、`widget_date`、`widget_chrono`、`widget_reached`、`widget_caption`），这样 `WidgetRenderer` 可以共用。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/background"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/widget_bg_dark"
    android:clipToOutline="true"
    android:gravity="center_vertical"
    android:orientation="horizontal"
    android:paddingHorizontal="16dp"
    android:paddingVertical="8dp">

    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:orientation="vertical">

        <TextView
            android:id="@+id/widget_title"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:ellipsize="end"
            android:maxLines="1"
            android:text="距 考研初试"
            android:textColor="#FF5A4E"
            android:textSize="14sp" />

        <TextView
            android:id="@+id/widget_date"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:maxLines="1"
            android:text="12月19日 00:00"
            android:textColor="#9A9AA5"
            android:textSize="11sp" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1.6"
        android:gravity="end|center_vertical"
        android:orientation="vertical">

        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1">

            <Chronometer
                android:id="@+id/widget_chrono"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:autoSizeMaxTextSize="34sp"
                android:autoSizeMinTextSize="14sp"
                android:autoSizeStepGranularity="1sp"
                android:autoSizeTextType="uniform"
                android:fontFamily="sans-serif-medium"
                android:fontFeatureSettings="tnum"
                android:gravity="end|center_vertical"
                android:maxLines="1"
                android:textColor="#F5F5F7" />

            <TextView
                android:id="@+id/widget_reached"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:autoSizeMaxTextSize="28sp"
                android:autoSizeMinTextSize="14sp"
                android:autoSizeTextType="uniform"
                android:gravity="end|center_vertical"
                android:maxLines="1"
                android:text="已到达"
                android:textColor="#F5F5F7"
                android:visibility="gone" />
        </FrameLayout>

        <TextView
            android:id="@+id/widget_caption"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="end"
            android:maxLines="1"
            android:text="小时 : 分 : 秒"
            android:textColor="#9A9AA5"
            android:textSize="10sp" />
    </LinearLayout>
</LinearLayout>
```

### 预览布局

`widget_square_preview.xml` 和 `widget_bar_preview.xml` 复制对应布局，做两处改动：

- 把 `Chronometer` 换成普通 `TextView`，文字为 `1788:00:00`（Chronometer 在预览里会显示 00:00）。
- 删掉 `widget_reached`。

### `res/xml/widget_square_info.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:description="@string/widget_square_desc"
    android:initialLayout="@layout/widget_square"
    android:minWidth="110dp"
    android:minHeight="110dp"
    android:minResizeWidth="110dp"
    android:minResizeHeight="60dp"
    android:previewLayout="@layout/widget_square_preview"
    android:resizeMode="horizontal|vertical"
    android:targetCellWidth="2"
    android:targetCellHeight="2"
    android:updatePeriodMillis="1800000"
    android:widgetCategory="home_screen" />
```

### `res/xml/widget_bar_info.xml`

和上面相同，以下属性改掉：

- `minWidth="250dp"`、`minHeight="50dp"`
- `minResizeWidth="180dp"`、`minResizeHeight="40dp"`
- `targetCellWidth="4"`、`targetCellHeight="1"`
- 布局和预览改用 `widget_bar` / `widget_bar_preview`
- 描述改用 `widget_bar_desc`

`updatePeriodMillis="1800000"`（30 分钟）是兜底自愈：万一某次校正事件丢了，最多 30 分钟后数值会自动修正。

### 背景 drawable

三个文件都是 `<shape android:shape="rectangle">`，圆角 `24dp`，只有颜色不同：

| 文件 | 颜色 |
|---|---|
| `widget_bg_dark.xml` | `#F2121216` |
| `widget_bg_light.xml` | `#F2FFFFFF` |
| `widget_bg_translucent.xml` | `#66000000` |

### `res/values/strings.xml`

```xml
<resources>
    <string name="app_name">考研倒计时</string>
    <string name="widget_square_label">考研倒计时 · 方块</string>
    <string name="widget_bar_label">考研倒计时 · 横条</string>
    <string name="widget_square_desc">时:分:秒 每秒跳动</string>
    <string name="widget_bar_desc">时:分:秒 每秒跳动</string>
    <string name="channel_name">倒计时常驻通知</string>
</resources>
```

## 4.9 App 界面要点

### 配色（`ui/theme/Theme.kt`）

只用深色主题，不跟随系统：

| 用途 | 颜色 |
|---|---|
| 背景 | `#0E0F12` |
| 主文字 | `#F5F5F7` |
| 次要文字 | `#9A9AA5` |
| 强调色 | `#FF5A4E` |
| 卡片 / 输入框背景 | `#1A1B20` |

### 每秒刷新：对齐整秒边界，只在前台运行

```kotlin
@Composable
fun rememberNowMillis(): State<Long> {
    val lifecycle = LocalLifecycleOwner.current.lifecycle
    return produceState(System.currentTimeMillis(), lifecycle) {
        lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
            while (true) {
                val now = System.currentTimeMillis()
                value = now
                delay(1000 - now % 1000)   // 睡到下一个整秒
            }
        }
    }
}
```

目标时刻的秒和毫秒都是 0，所以显示会在整秒边界跳变，和系统时钟同步。

### `AutoShrinkText`：数字放不下时自动缩小

不依赖新版 Compose 的 autoSize API，任何版本都能用：

```kotlin
@Composable
fun AutoShrinkText(
    text: AnnotatedString,
    maxFontSize: TextUnit,
    color: Color,
    modifier: Modifier = Modifier,
) {
    // 只在字符数变化时（例如小时从 4 位变 3 位）重新适配
    var scale by remember(text.text.length) { mutableFloatStateOf(1f) }
    var ready by remember(text.text.length) { mutableStateOf(false) }
    Text(
        text = text,
        color = color,
        fontSize = maxFontSize * scale,
        fontFeatureSettings = "tnum",
        fontWeight = FontWeight.Medium,
        maxLines = 1,
        softWrap = false,
        modifier = modifier.drawWithContent { if (ready) drawContent() },
        onTextLayout = { r ->
            if (r.didOverflowWidth && scale > 0.3f) scale *= 0.92f else ready = true
        },
    )
}
```

### 数字 + 小号单位

单位用 `em` 相对单位，随数字一起缩放：

```kotlin
buildAnnotatedString {
    append(hours.toString())
    withStyle(SpanStyle(fontSize = 0.28.em, color = SubText)) { append(" 小时") }
}
```

第二行同理：`22` + ` 分  ` + `05` + ` 秒`，分钟和秒用 `twoDigits()` 补零。

### 日期 / 时间选择器的坑

- `DatePickerState.selectedDateMillis` 是 **UTC 零点**。必须这样转换，否则在东八区会差一天：

  ```kotlin
  Instant.ofEpochMilli(ms).atZone(ZoneOffset.UTC).toLocalDate()
  ```

- 反过来设置初始值时，用 `date.atStartOfDay(ZoneOffset.UTC).toInstant().toEpochMilli()`。
- 时间选择器用 `TimePicker(state = rememberTimePickerState(h, m, is24Hour = true))`，包在 `AlertDialog` 里。
- 保存时用 `beijingToEpochMillis(LocalDateTime.of(date, LocalTime.of(h, m)))`。

### 其他

- **保持常亮**：在 `CountdownScreen` 里

  ```kotlin
  val view = LocalView.current
  DisposableEffect(keepScreenOn) {
      view.keepScreenOn = keepScreenOn
      onDispose { view.keepScreenOn = false }
  }
  ```

- **导航**：两个页面用一个 `var showSettings by rememberSaveable { mutableStateOf(false) }` 切换。不引入 Navigation 库。系统返回键在设置页时回到全屏页（`BackHandler`）。
- **`MainActivity.onResume()`**：调用一次 `CountdownSync.refreshAll(this)`。每次打开 App 都会顺手校正小组件和通知，作为额外保险。
- **通知权限**：用 `rememberLauncherForActivityResult(ActivityResultContracts.RequestPermission())` 申请 `POST_NOTIFICATIONS`。被拒时开关自动回到关闭，并 Toast「需要通知权限」。

## 4.10 `notify/CountdownNotifier.kt`

```kotlin
object CountdownNotifier {
    const val CHANNEL_ID = "countdown"
    private const val NOTIFICATION_ID = 1

    fun createChannel(context: Context) {
        val ch = NotificationChannel(
            CHANNEL_ID, context.getString(R.string.channel_name),
            NotificationManager.IMPORTANCE_LOW,
        ).apply { setShowBadge(false); lockscreenVisibility = Notification.VISIBILITY_PUBLIC }
        context.getSystemService(NotificationManager::class.java).createNotificationChannel(ch)
    }

    fun update(context: Context, s: Settings) {
        val nm = NotificationManagerCompat.from(context)
        val granted = ContextCompat.checkSelfPermission(
            context, Manifest.permission.POST_NOTIFICATIONS,
        ) == PackageManager.PERMISSION_GRANTED
        if (!s.notificationEnabled || !granted) { nm.cancel(NOTIFICATION_ID); return }

        val open = PendingIntent.getActivity(
            context, 0, Intent(context, MainActivity::class.java),
            PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_UPDATE_CURRENT,
        )
        val b = NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_stat_countdown)
            .setContentText(formatTargetLong(s.targetEpochMillis))
            .setOngoing(true)
            .setSilent(true)
            .setOnlyAlertOnce(true)
            .setVisibility(NotificationCompat.VISIBILITY_PUBLIC)
            .setContentIntent(open)

        val reached = s.targetEpochMillis <= System.currentTimeMillis()
        when {
            !reached -> b.setContentTitle("距 ${s.targetName}")
                .setWhen(s.targetEpochMillis).setShowWhen(true)
                .setUsesChronometer(true).setChronometerCountDown(true)
            s.afterReached == AfterReached.COUNT_UP -> b.setContentTitle("${s.targetName} 已过去")
                .setWhen(s.targetEpochMillis).setShowWhen(true)
                .setUsesChronometer(true).setChronometerCountDown(false)
            else -> b.setContentTitle("${s.targetName} 已到达").setShowWhen(false)
        }
        nm.notify(NOTIFICATION_ID, b.build())
    }
}
```

`CountdownApp.onCreate()` 里调用 `CountdownNotifier.createChannel(this)`。

## 4.11 图标

### `drawable/ic_launcher_foreground.xml`

108dp 视口，白色沙漏：

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp" android:height="108dp"
    android:viewportWidth="108" android:viewportHeight="108">
    <path android:fillColor="#FFFFFFFF" android:pathData="M36,32h36v4h-36z" />
    <path android:fillColor="#FFFFFFFF" android:pathData="M36,72h36v4h-36z" />
    <path android:fillColor="#FFFF5A4E" android:pathData="M40,36L68,36L56,54L68,72L40,72L52,54Z" />
</vector>
```

### `mipmap-anydpi-v26/ic_launcher.xml` 和 `ic_launcher_round.xml`

两个文件内容相同：

```xml
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/icon_bg" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
    <monochrome android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>
```

`values/colors.xml` 里定义 `icon_bg` = `#0E0F12`。

### `drawable/ic_stat_countdown.xml`

24dp 纯白通知小图标：

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp" android:height="24dp"
    android:viewportWidth="24" android:viewportHeight="24">
    <path android:fillColor="#FFFFFFFF"
        android:pathData="M5,2h14v2h-14z M5,20h14v2h-14z M6,4L18,4L13,12L18,20L6,20L11,12Z" />
</vector>
```

## 4.12 调试接收器（只在 debug 包里，用于自动化验证）

### `src/debug/AndroidManifest.xml`

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application>
        <receiver android:name="com.kaoyan.countdown.debug.DebugCommandReceiver"
            android:exported="true" />
    </application>
</manifest>
```

### `DebugCommandReceiver` 规格

收到广播时读取 `cmd` 字符串参数：

| cmd | 参数 | 行为 |
|---|---|---|
| `set` | `in_seconds`（Long，目标 = 现在 + N 秒，**保留秒数，不清零**）或 `epoch_ms`（Long）；可选 `name`（String，默认 `TEST`）；可选 `after`（`SHOW_REACHED` / `COUNT_UP`） | 保存设置，然后 `refreshAll` |
| `theme` | `value`（`DARK` / `LIGHT` / `TRANSLUCENT`） | 改小组件外观，然后 `refreshAll` |
| `notify` | `value`（Boolean） | 开 / 关通知，然后 `refreshAll` |
| `reset` | 无 | 恢复默认，然后 `refreshAll` |
| `dump` | 无 | 用 `Log.i("KaoyanCountdown", ...)` 打印当前设置、`computeState` 结果和 `formatColon` 结果 |

调用示例：

```
adb shell am broadcast -n com.kaoyan.countdown/.debug.DebugCommandReceiver --es cmd set --el in_seconds 90
adb shell am broadcast -n com.kaoyan.countdown/.debug.DebugCommandReceiver --es cmd dump
```

`name` 参数只传英文，避免 Windows 终端中文编码问题。

## 4.13 构建配置要点（`app/build.gradle.kts`）

- **`buildFeatures`**：`compose = true`。
- **Java / Kotlin 目标**：
  - `compileOptions` 的 source / target 设为 `JavaVersion.VERSION_17`。
  - Kotlin 用 `jvmToolchain(17)`，或 `compilerOptions.jvmTarget = JVM_17`。
- **`release` 构建类型**：
  - `isMinifyEnabled = true`、`isShrinkResources = true`。
  - 使用 `proguard-android-optimize.txt` + `proguard-rules.pro`。
- **签名**：存在 `keystore.properties` 时，读取其中的 `storeFile`、`storePassword`、`keyAlias`、`keyPassword` 配置 `release` 签名。文件不存在时 release 不签名，但构建不能报错。
- **APK 文件名**：输出为 `KaoyanCountdown-<versionName>-<buildType>.apk`。

---

# 5. HyperOS 适配

秒数跳动由桌面进程和系统通知负责，不依赖 App 存活。但以下三件事需要 App 能被系统唤醒：开机后校正、改时间后校正、到点切换。HyperOS 默认会拦截没有「自启动」权限的 App 接收这些广播。所以设置页要有「后台权限检查」区域，每项显示状态和一个按钮。

| 项目 | 状态检测 | 按钮动作 |
|---|---|---|
| 自启动 | 无法检测，显示「请确认已开启」 | 依次尝试下面的 Intent，全部失败就打开本 App 的系统应用详情页 |
| 省电策略 | `PowerManager.isIgnoringBatteryOptimizations(packageName)`，显示「无限制 ✓」或「受限」 | 先试小米省电策略页，失败就用 `Settings.ACTION_IGNORE_BATTERY_OPTIMIZATION_SETTINGS`，再失败就打开应用详情页 |
| 通知权限 | 只在通知开关打开时显示此行 | 跳转 `Settings.ACTION_APP_NOTIFICATION_SETTINGS` |
| 精确闹钟 | `AlarmManager.canScheduleExactAlarms()`，正常应显示 ✓ | 若为 false，跳转 `Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM` |

### 自启动 Intent

按顺序尝试。每次 `startActivity` 都用 `try/catch (ActivityNotFoundException, SecurityException)` 包住：

```kotlin
Intent().setComponent(ComponentName(
    "com.miui.securitycenter",
    "com.miui.permcenter.autostart.AutoStartManagementActivity"))

Intent("miui.intent.action.OP_AUTO_START").addCategory(Intent.CATEGORY_DEFAULT)
```

### 省电策略 Intent（小米）

```kotlin
Intent().setComponent(ComponentName(
    "com.miui.powerkeeper", "com.miui.powerkeeper.ui.HiddenAppsConfigActivity"))
    .putExtra("package_name", packageName)
    .putExtra("package_label", "考研倒计时")
```

### 应用详情页兜底

```kotlin
Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS, Uri.fromParts("package", packageName, null))
```

### 区域底部说明文字（原样显示）

> 小组件和通知里的秒数由系统负责跳动，不耗电。开启「自启动」并把省电策略设为「无限制」后，重启手机、修改时间、到达目标时刻时，倒计时才能自动校正。

---

# 6. 执行步骤（严格按顺序）

每一步完成后，用一两句话告诉我结果，再进入下一步。

## 第 1 步：检查环境

1. 判断我的操作系统（Windows / macOS / Linux），后续命令都按这个系统写。
   - Windows 下用 PowerShell，Gradle 命令写 `.\gradlew.bat`。
2. 找 Android SDK，默认位置：
   - Windows：`%LOCALAPPDATA%\Android\Sdk`
   - macOS：`~/Library/Android/sdk`
   - Linux：`~/Android/Sdk`
   - 找不到就停下来，让我先装 Android Studio 并打开一次。
3. 找 JDK 17 或更高版本：
   - 优先用 Android Studio 自带的 JBR：
     - Windows：`C:\Program Files\Android\Android Studio\jbr`
     - macOS：`/Applications/Android Studio.app/Contents/jbr/Contents/Home`
     - Linux：Studio 安装目录下的 `jbr`
   - 本次会话里设置 `JAVA_HOME` 指向它。
4. `adb` 在 `<SDK>/platform-tools/` 下。后续命令用完整路径，或把它加进本会话的 PATH。
5. 列出 `<SDK>/platforms/` 下已安装的 `android-XX`，选最高稳定版作为 compileSdk。低于 35 时：
   - 有 `sdkmanager` 就安装 `platforms;android-35` 和 `build-tools;35.0.0`；
   - 否则停下来，让我在 Android Studio 的 SDK Manager 里勾选安装。
6. 执行 `adb devices`：
   - 显示 `unauthorized`：让我在手机上点「允许」。
   - 什么都没有：让我检查数据线（要用能传数据的线）、USB 用途选「传输文件」、USB 调试已开启。
7. 读取并告诉我设备信息：

   ```
   adb shell getprop ro.product.model
   adb shell getprop ro.build.version.sdk
   adb shell getprop ro.build.version.release
   adb shell getprop ro.mi.os.version.name
   ```

## 第 2 步：创建工程

1. 按第 3 节创建全部文件。
2. 生成 Gradle Wrapper：
   - 本机有 `gradle` 命令：执行 `gradle wrapper --gradle-version <选定版本>`。
   - 没有：手动写 `gradle/wrapper/gradle-wrapper.properties`，`gradle-wrapper.jar`、`gradlew`、`gradlew.bat` 从 Android Studio 安装目录里任一自带模板或已有工程复制；或者下载对应版本的 Gradle 发行包解压后执行它的 `bin/gradle wrapper`。
3. 写 `local.properties`：`sdk.dir=<SDK 路径>`。Windows 路径要转义，例如 `sdk.dir=C\:\\Users\\me\\AppData\\Local\\Android\\Sdk`。
4. 初始化仓库：`git init`。若 `git` 不存在，跳过 git 相关步骤并告诉我。

## 第 3 步：单元测试

1. 执行 `gradlew :app:testDebugUnitTest`，第 4.2 节的测试必须全部通过。
2. 有失败就修代码，不改期望值。
3. 通过后 git 提交：`feat: 倒计时核心逻辑与单元测试`。

## 第 4 步：编译并安装 debug 版

1. 执行 `gradlew :app:assembleDebug`。
2. 执行 `adb install -r app/build/outputs/apk/debug/<文件名>.apk`。
   - 手机上弹出安装确认时，让我点「继续安装」。
   - 报 `INSTALL_FAILED_USER_RESTRICTED`：让我打开「USB 安装」后重试。
3. 启动 App：`adb shell am start -n com.kaoyan.countdown/.ui.MainActivity`。
4. 截图检查。截图方式见第 7 节，**Windows 下不要用 `adb exec-out screencap > x.png`，PowerShell 会损坏二进制文件**。
5. 隔 2 秒再截一次，两张图秒数要不同。
6. 用你的图片查看能力检查截图：
   - 布局与第 1.2 节一致，数字没有被截断；
   - 剩余时间与当前时间推算一致，用 `adb shell date` 对照。

## 第 5 步：小组件验证

1. 请我把两种小组件都加到桌面，操作写清楚：长按桌面空白处 → 添加小部件 → 搜索「考研倒计时」→ 分别拖出「方块」和「横条」。也可以让我在 App 设置页点「添加方块」「添加横条」。
2. 我确认后：
   - 执行 `adb shell input keyevent KEYCODE_HOME`，然后截图；
   - 隔 2 秒再截一次，确认两个小组件的秒数都在跳，且两张图上的数值差 2 秒左右；
   - 检查数字宽度没有抖动，文字没有被截断。
3. 执行 `adb shell dumpsys appwidget | findstr kaoyan`（macOS / Linux 用 `grep`），确认两个小组件已绑定。
4. 用调试命令切换三种主题（`theme` DARK / LIGHT / TRANSLUCENT），每种截一张图，检查文字清晰可读。

## 第 6 步：到点切换验证

1. `cmd=set in_seconds=70 after=SHOW_REACHED`。
2. 截图，应显示约 `0:01:10`，并且在跳。
3. 执行 `adb shell dumpsys alarm`，确认能找到 `com.kaoyan.countdown` 的 RTC_WAKEUP 闹钟。
4. 按 HOME 键回桌面，等待约 80 秒（用你的运行环境支持的等待方式），然后截图，两个小组件都应显示「已到达」。
5. `cmd=set in_seconds=-125 after=COUNT_UP`（目标设在 125 秒前）。截图应显示约 `+0:02:05`，并且在增加。
6. `cmd=reset` 恢复默认。截图确认回到默认目标。

## 第 7 步：通知验证

1. 执行 `adb shell pm grant com.kaoyan.countdown android.permission.POST_NOTIFICATIONS`，然后 `cmd=notify value=true`。
2. 执行 `adb shell cmd statusbar expand-notifications`，截图，确认通知里有倒计时且在跳。
3. 执行 `adb shell cmd statusbar collapse`。
4. `cmd=notify value=false`，确认通知消失。
5. 默认状态保持关闭。

## 第 8 步：自启动和重启验证

1. 让我按这个路径打开自启动：设置 → 应用设置 → 应用管理 → 考研倒计时 → 自启动 打开；同页「省电策略」选「无限制」。
2. 我确认后执行 `adb reboot`，然后等待 `adb wait-for-device`。
3. 再循环检查 `adb shell getprop sys.boot_completed` 直到等于 `1`，然后让我解锁手机。
4. 截图检查小组件数值正确且在跳。
5. 用 `adb logcat -d -s KaoyanCountdown` 确认收到了 `BOOT_COMPLETED` 并执行了 `refreshAll`。

## 第 9 步：改时间验证（需要我配合）

1. 让我操作：设置 → 更多设置 → 日期和时间 → 关闭「自动设置时间」→ 把日期往后改 1 天。
2. 回桌面后截图：小组件小时数应比之前少约 24。
3. 让我重新打开「自动设置时间」。
4. 再截图确认数值恢复正确。

## 第 10 步：正式版

1. 生成签名文件：

   ```
   keytool -genkeypair -v -keystore keystore/release.jks -alias kaoyan -keyalg RSA -keysize 2048 -validity 36500 -storepass <随机16位> -keypass <同上> -dname "CN=Kaoyan Countdown"
   ```

   `keytool` 在 JDK 的 `bin` 目录。
2. 写 `keystore.properties`：`storeFile=../keystore/release.jks` 和三项密码 / 别名。
3. 执行 `gradlew :app:assembleRelease`。
4. 把产物复制到 `dist/KaoyanCountdown-1.0.apk`。
5. 签名不同不能覆盖安装，所以：
   - 先 `adb uninstall com.kaoyan.countdown`；
   - 再 `adb install dist/KaoyanCountdown-1.0.apk`。
   卸载会移除桌面上的小组件。
6. 告诉我：重新打开一次 App，再把小组件加回桌面，并重新打开自启动和省电策略「无限制」。卸载后这些设置会丢失。
7. 我确认后最后截图一次，确认正式版运行正常。
8. git 提交：`release: v1.0`。

## 第 11 步：写 README 并汇报

1. 在工程根目录写 `README.md`（中文），包含：
   - 功能简介
   - 安装方法：数据线 adb 安装；或把 `dist/` 里的 APK 发到手机直接安装，遇到「纯净模式」拦截时的处理
   - 怎么添加小组件
   - HyperOS 必开权限及路径
   - 怎么改目标时间
   - 已知限制：小组件只能用冒号格式
   - **签名文件 `keystore/` 和 `keystore.properties` 必须备份**，丢了以后升级只能先卸载
2. git 提交。
3. 给我一份最终汇报（中文，简洁），包含：
   - 做了什么
   - APK 位置
   - 我还需要在手机上手动做的事
   - 所有验证截图的文件路径
   - 尚未解决的问题（没有就写「无」）

---

# 7. 截图方法

截图统一存到工程下的 `shots/` 目录（已在 `.gitignore` 中），文件名带步骤号，例如 `shots/05-widget-a.png`。

**所有系统通用（推荐）：**

```
adb shell screencap -p /sdcard/kc.png
adb pull /sdcard/kc.png shots/05-widget-a.png
adb shell rm /sdcard/kc.png
```

**macOS / Linux 也可以用：**

```
adb exec-out screencap -p > shots/05-widget-a.png
```

看截图时要真的打开图片检查内容，不能只确认文件存在。

---

# 8. 验收清单（全部打勾才算完成）

最终汇报里逐条写明 ✓ / ✗，以及对应的截图或命令输出。

- [ ] 单元测试全部通过
- [ ] App 全屏页显示「X 小时 / X 分 X 秒」，每秒跳一次，数字不抖、不截断
- [ ] 设置页能改名称、日期、时间；保存后全屏页、小组件、通知同时更新
- [ ] 选择 12 月 19 日保存后显示的还是 12 月 19 日（没有差一天）
- [ ] 两种小组件都能添加，都每秒跳动，点击能打开 App
- [ ] 三种小组件外观都清晰
- [ ] 到点后小组件自动显示「已到达」；正计时模式显示 `+` 时间
- [ ] 通知开关有效，通知里倒计时在跳
- [ ] 重启手机后小组件数值正确
- [ ] 手动改系统时间后小组件数值自动校正
- [ ] 后台权限检查区域的按钮都能跳到对应页面，跳不了时有兜底，不闪退
- [ ] release APK 已签名，并已安装验证
- [ ] `dist/` 里有 APK，README 已写好

---

# 9. 已知坑（提前规避）

1. **`Chronometer` 的 base 用 `SystemClock.elapsedRealtime()` 时间轴**。不要把墙上时间直接传进去。换算公式：`base = elapsedRealtime + (目标墙上时间 − 当前墙上时间)`。
2. **重启后 `elapsedRealtime` 归零**，旧 base 失效，必须在开机后重新 `refreshAll`。这就是需要开机广播和自启动权限的原因。
3. **小组件布局只能用 RemoteViews 支持的控件**：`FrameLayout`、`LinearLayout`、`RelativeLayout`、`TextView`、`Chronometer`、`ImageView` 等。用了不支持的控件（如 `View`、`Space`、`ConstraintLayout`、自定义 View），小组件会显示「无法加载」。
4. **`DatePicker` 返回 UTC 零点**，见第 4.9 节。
5. **targetSdk ≥ 35 强制全面屏**，必须处理系统栏留白。
6. **新装的 App 处于「停止」状态**，打开过一次之后才能收到广播。安装后先打开一次 App。
7. **小米「纯净模式」或「安全守护」**可能拦截安装：设置 → 隐私与安全 → 安全守护（或纯净模式）→ 暂时关闭，装完再打开。
8. **debug 和 release 签名不同**，切换时要先卸载。
9. **Windows PowerShell 重定向会破坏二进制**，截图用第 7 节通用方法。
10. **中文参数**通过 `adb shell` 传递可能乱码，调试命令只用英文。

---

# 10. 禁止事项

- 不要用前台服务、`Handler` 循环、`WorkManager` 或 `AlarmManager` 每秒推送小组件更新。秒数跳动只能靠 `Chronometer` 和通知的 `setUsesChronometer`。
- 不要用 Jetpack Glance。
- 不要申请 `INTERNET` 或其他清单外权限。
- 不要接入任何第三方 SDK。
- 不要改单元测试的期望值来让测试通过。
- 不要把 `keystore/`、`keystore.properties`、`local.properties` 提交进 git。
- 不要在没截图检查的情况下声称某项「已完成」或「已验证」。
- 不要跳过第 6 节的任何验证步骤。确实无法执行时（例如我不在），在汇报里明确标出哪步没做、为什么。

---

# 11. 常见问题

| 现象 | 原因和处理 |
|---|---|
| `SDK location not found` | `local.properties` 缺失，或路径未转义 |
| `Unsupported class file major version` / JDK 版本不对 | `JAVA_HOME` 没指向 JDK 17+，改用 Android Studio 自带的 `jbr` |
| 依赖下载超时 | 按第 2.2 节加国内镜像 |
| `adb: device unauthorized` | 手机上点「允许 USB 调试」，必要时在开发者选项里「撤销 USB 调试授权」后重连 |
| `INSTALL_FAILED_USER_RESTRICTED` | 开发者选项打开「USB 安装」 |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | 签名不同，先 `adb uninstall com.kaoyan.countdown` |
| 小组件显示「无法加载小部件」 | 布局用了不支持的控件或属性。查 `adb logcat -d \| findstr /i "RemoteViews AppWidget"`（或 grep），修正后重装 |
| 小组件秒数不动 | 检查 `setChronometer` 第 4 个参数是否为 `true`、是否调用了 `setChronometerCountDown(true)`。用 `cmd=dump` 看 base 计算 |
| 重启后小组件数值不对 | 自启动没开，或开机广播没收到。看 `logcat -s KaoyanCountdown` |
| 小组件列表里找不到 | 安装后先打开一次 App；HyperOS 小部件列表要往下翻，或用搜索 |
| 通知不显示 | 通知权限没给，或渠道被关。看 `adb shell dumpsys notification` |
| 自启动跳转页面打不开 | 已兜底到应用详情页，在汇报里告诉我手动路径 |
````
