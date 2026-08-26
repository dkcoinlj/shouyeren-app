# 守夜人 App · 编译与安装指南

这是一个 **Capacitor 原生封装**的个人作息工具。界面是 H5（在 `www/index.html`），被包进安卓 WebView，
因此能拿到**系统通知权限**，关屏 / 杀后台也能弹作息提醒。数据**只存在你手机本地**，不联网、不上传。

## 你电脑只需准备一次
- 安装 **Node.js**（https://nodejs.org ，LTS 版即可）
- 安装 **Android Studio**（https://developer.android.com/studio ，勾选 Android SDK）

## 方式一：本地编译 APK（推荐，全程可控）
```bash
cd 守夜人App
npm install              # 已装可跳过
npx cap add android      # 生成 android/ 原生工程（需联网）
npx cap sync             # 把 www/ 网页同步进安卓工程
npx cap open android     # 用 Android Studio 打开
```
Android Studio 里等 Gradle 同步完成 → 菜单 **Build → Build Bundle(s)/APK(s) → Build APK(s)**。
产物：`android/app/build/outputs/apk/debug/app-debug.apk`，用数据线 / 微信传到手机安装。

> 首次 Build 会联网下载 Gradle 依赖，稍慢，正常。

## 方式二：GitHub Actions 云端出包（零本地安装 Android Studio）
适合嫌装 Android Studio 麻烦。把本目录推到你的 GitHub 仓库，仓库里已配好 workflow（见下文「补充」），
push 后 Actions 自动编译出 APK，去仓库 **Actions → 最新任务 → Artifacts** 下载 `app-debug.apk` 安装即可。
以后每次改 `www/index.html`，push 一下就自动出新包。

## 装好后必做一步
打开 App → 「设置」→ 点 **🔔 启用系统通知** → 允许通知权限。
之后每天 12 个作息点 + 周二/四/六去角质 + 阶段切换日，都会由手机系统直接弹提醒。

## 任务卡 + 任务闹钟
「今日」页顶部就是任务看板（未完成 / 已完成）。右上角 **➕ 新增任务**：
- 写任务内容，选一个提醒时间（可选）。
- 保存后，到点手机会弹系统通知叫你做这件事。
- 任务拖到/点进「已完成」，对应提醒自动取消。
- 删除任务也会取消提醒。

## 各安卓版本注意
- **Android 12+**：若个别闹钟不响，去系统「设置 → 应用 → 守夜人 → 通知 / 闹钟与提醒」里，把*通知*和*精确闹钟*权限都打开（本项目已自动申请 `SCHEDULE_EXACT_ALARM`）。
- **小米/华为/OPPO 等**：这些厂商会杀后台通知，建议把「守夜人」加入*自启动/电池优化白名单*，并把「设置 → 通知 → 守夜人」设为允许。
- 万一某机型系统通知仍被限制，可用「今日」页的 **⬇️ 导出日历提醒(.ics)** 导入系统日历，作为原生闹钟兜底。

## 后续更新界面
直接改 `www/index.html`，然后：
```bash
npx cap sync
npx cap open android   # 重新 Build APK
```
代码里只改界面/逻辑，你的历史数据（手机本地）不受影响。

## 数据备份
「设置 → 导出备份(.json)」存一份到手机/电脑，换机或重装后「导入恢复」即可。

## App 启动图标（换成本项目 LOGO）
工程里已附矢量源 `logo.svg`（teal 渐变方块 + 弯月 + 守护星点，与 App 内顶栏一致）。
编译前用 Android Studio 把它变成桌面图标：
1. `npx cap open android` 打开工程
2. 左侧 `app` 右键 → **New → Image Asset**
3. **Foreground Layer** 选 "Image" → 导入 `logo.svg`；**Background Layer** 选纯色 `#2dd4bf`（teal）
4. 一路 Next → Finish，Android Studio 会生成所有尺寸 mipmap 图标，覆盖默认 Capacitor 图标
> 想换其它图标，只要改 `logo.svg` 重新走一遍 Image Asset 即可。

---
### 补充：GitHub Actions workflow（方式二用）
在仓库根建 `.github/workflows/build.yml`：
```yaml
name: Build APK
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm install
      - run: npx cap add android
      - run: npx cap sync
      - run: cd android && chmod +x gradlew && ./gradlew assembleDebug
      - uses: actions/upload-artifact@v4
        with: { name: app-debug.apk, path: android/app/build/outputs/apk/debug/app-debug.apk }
```
