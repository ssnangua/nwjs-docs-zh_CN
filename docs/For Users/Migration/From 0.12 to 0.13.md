# 从 0.12 升级到 0.13 {: doctitle}
---

[TOC]

## 架构的变化
+ NW.js 内部是作为一个 Chrome 应用运行的，所有的 chrome.* 平台的 API 和特性都可以在 NW 应用中使用。URL 的默认协议由 `file://` 改为 `chrome-extension://`，URL 的主机部分是生成的 id。0.12 版本中的 `app://` 协议也改成了 `chrome-extension://` 协议。
+ 所有 NW 特定的 API，包括 `require()`，都从 `nw.gui` 库移到一个 `nw` 对象中。不过，我们提供了一个内置的包装库来向上兼容 0.12 的应用。我们可能会在 0.14 或之后的版本中废弃它，在此之前，您依然可以使用 `nw.gui` 库一段时间。（**译者注：**到目前 0.69.1 为止，依然支持 `nw.gui` 的写法，不过不建议使用）
+ 和 0.12 及之前的版本一样，Node.js 环境被嵌入到后台页面的 DOM 上下文中，便于在不同的窗口之间共享。不同的是，在 0.13 版本中，您可以在 Node 环境中访问所有的 DOM 特性和 chrome.* API。
+ 要调试 Node.js 模块，您需要以独立环境的模式打开后台页面的开发者工具。参考 [开发者工具与调试](../Debugging with DevTools.md)。
+ 和 0.12 版本一样，应用的入口可以是 JS 或 HTML，但由于应用内部是一个 Chrome 应用，所以第一个窗口应该是由后台页面执行 JS 来打开的。如果您给 package.json 的 "main" 字段指定了一个 HTML 页面，NW 将使用一个默认的 JS 来打开第一个窗口，然后加载这个页面。
+ 如果 NW.js 运行在 [混合环境模式](../Advanced/JavaScript Contexts in NW.js.md#混合环境模式) 下（使用 `--mixed-context` 参数启动 NW.js），`nw.*` 类似于 `window.*` 的镜像。在该模式下，您 **无法** 通过将变量保存在 Node 环境中来实现不同框架或窗口之间的共享，所以如果您的应用比较依赖变量共享特性，**不要** 开启混合环境模式。

## Node.js 的变化

+ Node.js 在最新版本中升级到 6.x，检查您的 NPM 模块，确保它们支持 Node.js 6.x 版本，特别是原生模块。参考 [原生模块列表](https://github.com/nodejs/node/issues/2798) 查看哪些需要升级到最新的 [NaN 2](https://github.com/nodejs/nan) 版本。（**译者注：**到目前 0.69.1 为止，Node.js 版本已经升级到 18.10.0，可以通过 `process.versions.node` 来查看当前的 Node.js 版本）
+ 新增 process.versions[`nw`] 用于获取 NW 版本信息，process.versions[`node-webkit`] 将在后续废弃。（**译者注：**到目前 0.69.1 为止，`node-webkit` 还是可以用的）

## API 的变化

### 构建风格

+ 不同的构建风格会有不同的 API 和功能。参考 [构建风格](../Advanced/Build Flavors.md) 以选择适当的版本，或 [构建您自己的版本](../../For Developers/Building NW.js.md)。

### Shortcut（快捷键）

+ 在 Mac OS X 系统上，`Shortcut` API **不会** 将<kbd>Ctrl</kbd> 修饰键映射到 <kbd>&#8984;</kbd>。不过在 0.13.0 版本中，支持跨平台的 `Command` 修饰键。您应该根据不同的平台使用正确的修饰键，参考 [Shortcut.key](../../References/Shortcut.md#shortcutkey)。

### Menu（菜单）

+ 在 Mac 系统中，不再是 0.12 版本的最小化菜单栏，而是默认的菜单栏，包含 `app-name`（应用名）、`Edit`（编辑）和 `Window`（窗口）。
+ 在 Mac 系统中，想要修复应用菜单的名字，您需要修改 `nwjs.app/Contents/Resources/en.lproj/InfoPlist.strings` 而不是 `nwjs.app/Contents/Info.plist`。参考 [自定义菜单栏](../Advanced/Customize Menubar.md#mac-os-x)。

### 配置格式

+ [`single-instance`](../../References/Manifest Format.md#single-instance) **已弃用**，并且一直为`true`，应用 **不能** 拥有多个实例，除非通过 `--user-data-dir` 来使用不同的用户数据目录。您可能会用到 [open 事件](../../References/App.md#openargs)：当用户尝试启动第二个实例时，第一个实例可以通过该事件接收到通知。
+ [`toolbar`](../../References/Manifest Format.md#toolbar) **已弃用**，并且一直为 `false`。传统工具栏（包含重新加载按钮、地址栏和开发者工具按钮）已不再支持。您可以通过 <kbd>F12</kbd>（Windows & Linux）或 <kbd>&#8984;</kbd>+<kbd>&#8997;</kbd>+<kbd>i</kbd>（Mac）来打开和关闭开发者工具，通过 [`win.reload()`](../../References/Window.md#winreload) 和 [`win.reloadDev()`](../../References/Window.md#winreloaddev) 来重新加载页面。
+ [`no-edit-menu`](../../References/Manifest Format.md#no-edit-menu-mac) **已弃用**。
+ `snapshot` **已弃用**，使用[`win.evalNWBin()`](../../References/Window.md#winevalnwbinframe-path) 代替。
+ [`node-remote`](../../References/Manifest Format.md#node-remote) 的格式改为 Chrome 扩展中使用的 [匹配模式](https://developer.chrome.com/extensions/match_patterns) 数组。
+ `package.json` 和 `Window.open()` 的参数中，部分窗口相关的参数命名发生了变更：`always-on-top` ➩ [`always_on_top`](../../References/Manifest Format.md#always_on_top)、`visible-on-all-workspaces` ➩ [`visible_on_all_workspaces`](../../References/Manifest Format.md#visible_on_all_workspaces-mac-linux)、`new-instance` ➩ `new_instance`、`inject-js-start` ➩ `inject_js_start`、`inject-js-end` ➩ `inject_js_end`。
+ 命令行参数 `--data-path` 改名为 `--user-data-dir`。

### Window（窗口）

+ 每个窗口都有一个用于识别身份的 `id`，以记录窗口的大小和位置，拥有相同 `id` 的窗口被重新打开时会恢复纪录的状态。`id` 可以在 [Window.open](../../References/Window.md#windowopenurl-options-callback) 或 [配置文件的 window 子字段](../../References/Manifest Format.md#id) 里进行指定。
+ `Window` API 的 [`capturepagedone`](../../References/Window.md#capturepagedone) 事件 **已弃用**，使用 [`win.capturePage(callback [, config ])`](../../References/Window.md#wincapturepagecallback-config) 的回调函数代替。
+ [Window.open](../../References/Window.md#windowopenurl-options-callback) 创建的窗口改为作为回调函数的参数传递。
+ [Window.showDevtools](../../References/Window.md#winshowdevtoolsiframe-callback) 创建的窗口改为作为回调函数的参数传递。
+ [win.setTransparent](../../References/Window.md#winsettransparenttransparent) **已弃用**，窗口创建之后不能修改透明特性。
+ 窗口的 `unmaximize` 和 [leave-fullscreen](../../References/Window.md#leave-fullscreen) 事件 **已弃用**，使用 [restore](../../References/Window.md#restore) 事件代替，当窗口从最小化、最大化或全屏的状态下恢复时，将触发 `restore` 事件。
+ `package.json` 的 window 子字段和 `Window.open()` 参数中的 `always-on-top` 重命名为 [`always_on_top`](../../References/Manifest Format.md#always_on_top)，`visible-on-all-workspaces` 重命名为 [`visible_on_all_workspaces`](../../References/Manifest Format.md#visible_on_all_workspaces-mac-linux)。
+ 窗口不再继承自 `EventEmitter`，不过依然支持 `on()`、`once()`、`removeListener()` 和 `removeAllListeners()` 方法。

### Screen（屏幕）

+ `added`、`orderchanged`、`namechanged`、`thumbnailchanged` 监视到的 `id`，应该通过 [`registerStream(id)`](../../References/Screen.md#screendesktopcapturemonitorregisterstreamid) 进行注册，用返回 stream id 传给 `getUserMedia`。参考 [示例](../../References/Screen.md#示例Screen.DesktopCaptureMonitor)。

### 已知问题

+ 在 Linux 系统中 , `nw.Window.open()` 参数中的 `resizable` 没有效果，可以尝试在回调函数中设置。
+ `nw.Window` 的 `reloadDev()` 和 `isDevToolsOpen()` 目前不支持。
+ `App.quit()` 不会触发 `nw.Window` 的 `closed` 事件。
+ `nw.Window` 的 `devtools-closed` 事件目前不支持。
+ `as_desktop` 参数目前不支持。
+ `package.json` 中的 `webkit.{plugin|java|page-cache}` 参数目前不支持，插件选项默认启用。
+ `<iframe>` 的 `nwUserAgent` 属性目前不支持。
+ `MenuItem` 的 `tooltip` 属性目前不支持。
+ `nw.App.setCrashDumpDir()` 目前不支持，崩溃转储被保存在 `app-data-path/Crash Reports`。
