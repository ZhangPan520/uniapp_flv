# uniapp_flv

一个基于 uni-app 的 Android 摄像头直播演示项目，用来验证在 App WebView 中通过 `flv.js` 播放 HTTP-FLV 实时流，以及 uni-app 页面与内嵌播放器之间的交互。

项目以“家庭监控”作为示例场景，包含设备列表、实时画面、播放/暂停、横竖屏切换、全屏控制和直播异常重试等功能。截图、录像、对讲、回放、云台和缩放目前主要用于界面与交互演示，尚未接入真实设备接口。

## 功能说明

- 展示摄像头设备列表和在线状态。
- 使用本地 `flv.js` 在 Android WebView 中播放 HTTP-FLV 直播流。
- 提供播放、暂停、加载、异常提示和最多 5 次自动重试。
- 支持手动全屏、设备旋转进入横屏，以及退出后恢复竖屏。
- 通过 `uni.webview`/`postMessage` 在 uni-app 页面和播放器 HTML 之间传递状态与控制命令。
- 提供两种直播页面实现，方便比较不同的 UI 承载方式：

| 方案 | 页面 | 说明 |
| --- | --- | --- |
| H5 自绘 UI | `src/pages/live-h5/live-h5.vue` | 整个直播界面由 App 内的 `web-view` 加载 `flv-player-h5.html` 渲染 |
| nvue 原生页面 | `src/pages/live-nvue/live-nvue.nvue` | 页面布局由 nvue 渲染，仅播放器区域使用 `web-view` 加载 `flv-player.html` |

> 当前播放器是 App/Android 方向的验证实现。普通浏览器中的 H5 构建可以用于查看部分页面，但直播页在非 `APP-PLUS` 平台会显示不支持提示。

## 技术栈

- [uni-app](https://uniapp.dcloud.net.cn/) 3（Vue 3 版本）
- Vue 3.4
- Vite 5 + `@dcloudio/vite-plugin-uni`
- nvue / Weex 原生渲染
- HTML5 `<video>`、Media Source Extensions（MSE）
- `flv.js`，负责将 HTTP-FLV 流转封装并送入 `<video>` 播放
- HTML5 Plus API，负责 Android 系统栏、屏幕方向和 WebView 控制
- `uni.webview` 消息桥，负责页面与内嵌播放器通信

播放器依赖已存放在 `src/hybrid/html` 中，运行时不需要从 CDN 下载 `flv.js`。

## 项目结构

```text
src/
├── App.vue                         # App 生命周期、Android 沉浸式系统栏
├── manifest.json                   # uni-app 应用与 Android 权限配置
├── pages.json                      # 页面路由和导航栏配置
├── pages/
│   ├── index/index.vue             # 设备列表和播放方案选择
│   ├── live-h5/live-h5.vue         # H5 自绘 UI 方案
│   └── live-nvue/live-nvue.nvue    # nvue 原生页面方案
├── hybrid/html/
│   ├── flv-player-h5.html          # H5 方案的完整播放器页面
│   ├── flv-player.html             # nvue 方案使用的播放器
│   ├── flv.min.js                  # 本地 flv.js
│   └── uni.webview.1.5.8.js        # WebView 通信库
└── static/                         # 应用静态资源

public/monitor-live.html            # 设计稿和交互原型，不是生产入口
```

## 环境要求

- Node.js 18 或更高版本
- npm
- HBuilderX App 开发版（用于编译和运行 uni-app App）
- Android Studio（包含 Android SDK、Platform Tools 和 Emulator）
- Android 模拟器或已开启 USB 调试的 Android 真机
- 能访问直播地址的网络环境

首次运行前，建议在 HBuilderX 中打开 `src/manifest.json`，通过可视化界面获取并填写 DCloud AppID。正式离线打包还需要配置自己的 Android 包名、签名和对应的离线 SDK 信息。

## 安装依赖

```bash
npm install
```

可先运行 H5 构建检查项目能否正常编译：

```bash
npm run build:h5
```

## 使用 Android Studio 模拟器调试

这个仓库是 uni-app 前端工程，不是 Gradle Android 工程，不能直接在 Android Studio 中打开后点击 Run。日常调试的推荐方式是：Android Studio 管理模拟器，HBuilderX 编译并安装调试基座。

### 1. 创建并启动模拟器

1. 打开 Android Studio，进入 **Tools > Device Manager**。
2. 点击 **Create device**，选择一款带横竖屏传感器支持的 Phone 设备。
3. 安装并选择一个 Android 系统镜像，完成虚拟设备创建。
4. 点击启动按钮，等待模拟器完全进入桌面。
5. 在终端确认 ADB 已识别设备：

```bash
adb devices
```

正常情况下会看到类似下面的结果：

```text
List of devices attached
emulator-5554    device
```

如果提示找不到 `adb`，将 Android SDK 的 `platform-tools` 加入 `PATH`，或在 HBuilderX 的运行配置中指定 Android SDK/ADB 路径。

### 2. 从 HBuilderX 运行项目

1. 使用 HBuilderX 打开本仓库根目录。
2. 等待 npm 依赖识别完成；如未安装依赖，在项目目录执行 `npm install`。
3. 选择 **运行 > 运行到手机或模拟器 > 运行到 Android App 基座**。
4. 在设备列表中选择刚才启动的 Android Studio 模拟器。
5. 等待 HBuilderX 编译、安装调试基座并启动应用。

进入首页后点击“客厅摄像头”，再选择 H5 或 nvue 方案。点击播放器中央按钮开始拉流；旋转模拟器或点击全屏按钮可检查横竖屏逻辑。

模拟器旋转可使用模拟器工具栏的左转/右转按钮。若界面没有自动旋转，请同时确认模拟器的“自动旋转屏幕”已开启。

### 3. 查看 Android 日志

Android Studio 打开 **View > Tool Windows > Logcat**，选择当前模拟器和应用进程。也可以在终端查看日志：

```bash
adb logcat
```

建议按这些关键字过滤播放器相关信息：

```text
chromium
flv.js
web-view message
live-h5
live-nvue
system-ui
```

Vue/JavaScript 断点和 WebView DOM 调试仍建议使用 HBuilderX 调试器或 Chrome 的 `chrome://inspect/#devices`；Android Studio Logcat 更适合排查 WebView、网络、权限、Activity 和系统方向问题。

## 在 Android Studio 中直接运行原生工程

如果需要在 Android Studio 中点击 Run、修改原生代码或调试 Java/Kotlin，必须另行创建 DCloud Android 离线 SDK 工程。本仓库目前不包含 `settings.gradle`、Android `app` 模块或 DCloud 离线 SDK，因此不能直接进行这种调试。

基本流程如下：

1. 从 DCloud 官方文档下载与当前 HBuilderX/uni-app 编译器版本匹配的 Android 离线 SDK 和示例工程。
2. 在 HBuilderX 中生成 App 本地打包资源。
3. 本项目编译后的 App 资源可在 `dist/build/app` 找到；将它按离线 SDK 文档要求复制到原生工程的应用资源目录，并确保目录名与 AppID 一致。
4. 在原生工程中配置 AppID、包名、签名、所需权限和 DCloud SDK 依赖。
5. 使用 Android Studio 打开该 Gradle 工程，等待 Gradle Sync 完成后选择设备并点击 Run。

离线 SDK、HBuilderX 和编译产物的版本必须保持一致。具体目录名和 Gradle 配置会随 DCloud SDK 版本变化，应以所下载版本附带的官方示例与文档为准。

## 更换直播流

当前示例流地址定义在以下两个页面中：

```js
const FLV_URL = 'https://srs.unsj.edu.ar/live/xama/xama.flv'
```

- `src/pages/live-h5/live-h5.vue`
- `src/pages/live-nvue/live-nvue.nvue`

替换为自己的 HTTP-FLV 地址后重新运行项目即可。直播源需要满足：

- Android WebView 能直接访问该地址。
- 服务端允许跨域访问（正确配置 CORS 响应头）。
- 返回的是持续输出的 FLV 直播流，而不是下载页面或鉴权错误页面。
- WebView 内核支持 MSE；建议使用较新的 Android System WebView。
- HTTPS 页面优先使用 HTTPS 直播源。若必须使用 HTTP，还需要在 Android 原生配置中允许明文流量。

## 常见问题

### HBuilderX 找不到 Android Studio 模拟器

先保持模拟器在运行状态，并用 `adb devices` 确认其状态为 `device`。如果出现多个 ADB 版本互相冲突，统一 Android Studio 与 HBuilderX 使用的 SDK/ADB 路径，然后重启 ADB：

```bash
adb kill-server
adb start-server
adb devices
```

### 点击播放后一直加载或提示连接失败

用模拟器浏览器直接访问直播 URL，检查 DNS、证书、鉴权和网络是否正常；再查看 Logcat 中的 `chromium` 与 `flv.js` 日志。最常见原因是流已失效、CORS 未配置、证书异常或 WebView 无法访问局域网地址。

访问开发机本地服务时，Android 模拟器不能使用 `localhost` 指向宿主机。Android Studio 默认模拟器应改用 `10.0.2.2`，真机则使用开发机在同一局域网内的 IP 地址。

### 能看到界面，但无法播放 FLV

确认 Android System WebView 已更新，并检查 `flv.js.isSupported()` 是否返回 `true`。该播放器依赖 MSE，不支持 MSE 的 WebView 无法播放。

### 横屏或全屏行为异常

确认系统自动旋转已开启，并分别检查 `src/manifest.json` 的 `screenOrientation`、页面中的 `plus.screen` 调用以及 Logcat 中的方向传感器日志。真机传感器行为通常比模拟器更可靠，最终应至少在一台真机上验证。

## 构建 H5

开发模式：

```bash
npm run dev:h5
```

生产构建：

```bash
npm run build:h5
```

构建结果默认输出到 `dist/build/h5`。需要注意，H5 构建主要用于页面预览；本项目的完整播放器流程依赖 `APP-PLUS` 和 HTML5 Plus API，应以 Android App 端测试结果为准。
