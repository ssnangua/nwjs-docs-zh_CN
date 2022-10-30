# 配置格式 {: .doctitle}
---

[TOC]

每个应用包中都会有一个名为 `package.json` 的配置文件，文件内容为 [JSON](http://www.json.org/) 格式。本文档将详细说明配置文件中各个字段的含义。

## 快速开始

以下为最简单的配置：
```json
{
  "main": "index.html",
  "name": "nw-demo",
}
```

启动 NW.js 应用最少只需要这两个字段：

* `main`：告诉 NW.js 启动时打开 `package.json` 同目录下的 `index.html`
* `name`：给应用起一个独一无二的名字叫 `nw-demo`

## 必要字段

每个应用的配置文件中都必须包含以下字段。

### main

* `{String}` 启动 NW.js 时要打开的 HTML 页面或JavaScript 文件。

该字段可以设置为一个 URL，也可以设置为一个文件名（如 `index.html` 或 `script.js`）或路径（相对于 `package.json` 所在的目录）。

### name

* `{String}` 应用名。必须是一个唯一的、由小写字母或数字组成、且不包含空格的字符串。可以包含"."、"_"、"-"。不符合规范的应用名将不被 NW.js 识别。

`name` 字段需要全局唯一是因为 NW.js 会将应用的数据存储在名为 `name` 的目录中。

## 特性控制字段

以下字段控制 NW.js 要提供哪些特性以及如何打开主窗口。

### product_string

* `{String}` 用于 [重命名 macOS 下的帮助应用](../For Users/Package and Distribute/#mac-os-x)。

### nodejs

* `{Boolean}` 设置为 false 将禁用 Node 支持。

### node-main

* `{String}` 指定 node.js 脚本文件路径，启动时会在加载第一个窗口的 DOM 之前，在 Node 环境中执行。

### domain

* `{String}` 指定用于应用程序的 chrome-extension:// 协议 URL 中的主机。web 引擎将将在同域名的应用和网站之间共享相同的 cookie。

### single-instance

!!! warning "已弃用"
    从 0.13.0 版本开始，该属性已被弃用。参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `{Boolean}` 是否以单例的方式运行应用。默认为`true`。

默认情况下，NW.js 只允许应用运行一个实例，如果需要同时运行多个实例，把该属性设置为 `false`。

### bg-script

* `{String}` 后台脚本，会在应用启动时在后台页面中执行。

### window

* `{Object}` 控制窗口外观，参考 [Window 子字段](#Window子字段)。

### webkit

* `{Object}` 控制 WebKit 特性，参考 [WebKit 子字段](#WebKit子字段)。

### user-agent

* `{String}` 重写从应用发出的 HTTP 请求的 `User-Agent`。

以下占位符可用于动态组合 `User-Agent`：

* `%name`：替换为配置文件中的 `name` 字段。
* `%ver`：替换为配置文件中的 `version` 字段。
* `%nwver`：替换为 NW.js 版本。
* `%webkit_ver`：替换为 Blink 引擎版本。
* `%chromium_ver`：替换为 Chromium 版本。
* `%osinfo`：替换为浏览器 user agent 字符串中的 OS 和 CUP 信息。

### node-remote

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `{Array}` 或 `{String}` 控制哪些远程站点启用 Node。数组中各项遵循在 Chrome 扩展中使用的 [匹配模式](https://developer.chrome.com/extensions/match_patterns)。

匹配模式本质上是一个符合规定协议（`http`、`https`、`file` 或 `ftp`，可包含 `'*'` 字符）的 URL。特殊模式 `<all_urls>` 匹配所有符合规定协议的 URL。匹配模式包含三个部分：

* `scheme` — 如 `http` 或 `file` 或 `*`
* `host` — 如 `www.google.com` 或 `*.google.com`或 `*`。模式为文件时没有 host 部分
* `path` — 如 `/*`、`/foo*` 或 `/foo/bar`

基础语法：

```none
<url-pattern> := <scheme>://<host><path>
<scheme> := '*' | 'http' | 'https' | 'file' | 'ftp'
<host> := '*' | '*.' <除了'/'和'*'的任意字符>+
<path> := '/' <任意字符>
```

### chromium-args

* `{String}` chromium 命令行参数。

想要让分发的应用带上自定义的 chromium 参数时很有用。例如，想要禁止 GPU 加速视频显示，只需要添加`"chromium-args" : "--disable-accelerated-video"`。如果需要添加多个参数，两个参数之间用空格分开。该字段也可以在一个参数中使用多个 flag，通过单引号将它们括起来。

环境变量 `NW_PRE_ARGS` 的值会被添加到该字段的值前面。

参考 [命令行参数](Command Line Options.md) 查看更多信息。

### crash_report_url

* `{String}` 崩溃上报服务器的 URL

应用发生崩溃时，会把运行时环境的崩溃转储文件和信息发送给崩溃服务器。发送的方式和 Chromium 浏览器相同：一个 content type 为 `multipart/form-data` 的 HTTP POST 请求。理论上，任何 breakpad/crashpad 服务器都可以处理这些请求，因为 breakpad/crashpad 在 NW 中的工作方式与在 Chromium 中相同。可以参考我们在测试用例中使用的 [一个非常简单的服务器](https://github.com/nwjs/nw.js/blob/nw21/test/sanity/crash-dump-report/crash_server.py) 或 [simple-breakpad-server](https://github.com/acrisci/simple-breakpad-server)。

请求至少包含以下字段：

* `prod` - 应用配置文件中的 `name` 字段
* `ver` - 应用配置文件中的 `version` 字段
* `upload_file_minidump` - minidump 文件的二进制内容
* `switch-n` - 崩溃进程的命令行开关。每个开关有多个字段，其中 `n` 是从 1 开始的数字。

### js-flags

* `{String}` 要传递给 JS 引擎（V8）的 flag。例如打开 Harmony Proxies 和 Collections 特性。

```json
{
  "name": "nw-demo",
  "main": "index.html",
  "js-flags": "--harmony_proxies --harmony_collections"
}
```

可以在这里查看 V8 支持的 flag 列表：https://chromium.googlesource.com/v8/v8/+/master/src/flag-definitions.h

### inject_js_start
### inject_js_end

* `{String}` 相对于应用路径的本地文件名，指定一个 JavaScript 文件用于注入窗口。

`inject_js_start`：在 CSS 执行之后、其他 DOM 或脚本执行之前，执行注入的 JavaScript 代码。

`inject_js_end`：在 document 对象加载完成之后、触发 `onload` 事件之前，执行注入的 JavaScript 代码。主要用于作为 `Window.open()` 的参数在新窗口中注入 JavaScript 代码。

### additional_trust_anchors

* `{Array}` 包含一个 PEM 编码的证书列表，例如 `"-----BEGIN CERTIFICATE-----\n...certificate data...\n-----END CERTIFICATE-----\n"`。

这些证书将作为附加根证书用于验证，以允许使用自签名证书或自定义 CA 颁发的证书连接到服务。

### dom_storage_quota

* `{Integer}` DOM 存储配额的兆字节（MB）数。建议设置为期望值的两倍。

### no-edit-menu (Mac)

!!! warning "已弃用"
    从 0.13.0 版本开始，该属性已被弃用。参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `{Boolean}` Mac OS X 系统上是否显示默认的 `Edit`（`编辑`）菜单。默认值为 `false`。该字段只在 Mac OS X 系统上有效。

<span id="Window子字段"></span>
## Window 子字段

默认情况下，窗口的子字段大部分会被继承到使用 `window.open()` 或 `<a target="_blank">` 打开的子窗口。以下子字段不会被继承，打开窗口时会使用默认值：

* `fullscreen` -> `false`
* `kiosk` -> `false`
* `position` -> `null`
* `resizable` -> `true`
* `show` -> `true`

所有的窗口子字段都可以通过 [`new-win-policy` 事件](Window.md#new-win-policy-frame-url-policy) 进行重写。

### id

* `{String}` 用于识别窗口，以纪录窗口的大小和位置，当拥有相同 `id` 的窗口被重新打开时会恢复纪录的状态。参考 [Chrome 应用文档](https://developer.chrome.com/apps/app_window#type-CreateWindowOptions)

### title

* `{String}` 窗口的默认标题。使用该字段可以在在启动应用时显示自定义的标题。页面加载后会显示为页面的 `<title>`（`document.title`）。

### width
### height

* `{Integer}` 窗口的初始内部宽度和内部高度。

### toolbar

!!! warning "已弃用"
    从 0.13.0 版本开始，该属性已被弃用。参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `{Boolean}` 是否显示导航工具栏。

### icon

* `{String}` 窗口图标的路径。

### position

* `{String}` 控制窗口位置，可设置为 `null`、`center`、`mouse`。

### min_width
### min_height

* `{Integer}` 窗口的最小内部宽度和内部高度。

### max_width
### max_height

* `{Integer}` 窗口的最大内部宽度和内部高度。

### as_desktop (Linux)

* `{Boolean}` X11 环境下，作为桌面背景窗口显示。

### resizable

* `{Boolean}` 窗口是否可以调整大小。

注意：在 Mac OS X 系统中，如果 `resizable` 设置为 false：`frame` 设置为 true，窗口仍然是可以全屏的，需要将 `fullscreen` 设置为 false 以禁用全屏功能。

### always_on_top

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `{Boolean}` 窗口是否置顶（永远在其他窗口上面）。

### visible_on_all_workspaces (Mac & Linux)

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `{Boolean}` 窗口是否同时在所有的工作区中可见（只在支持多工作区的平台上有效，目前有 Mac OS X 和 Linux）。

### fullscreen

* `{Boolean}` 窗口是否全屏。

注意：如果全屏时将 `frame` 设置为 false，会导致鼠标在屏幕的边缘无法被捕获，要避免这一点。

### show_in_taskbar

* `{Boolean}` 窗口是否显示在任务栏或 dock 中，默认为 `true`。

### frame

* `{Boolean}` 设置为 `false` 时为无框架窗口。

注意：如果全屏时将 `frame` 设置为 false，会导致鼠标在屏幕的边缘无法被捕获，要避免这一点。

无框架窗口没有标题栏，用户无法拖动窗口，也无法通过双击标题栏最大化窗口。可以使用 CSS 将 DOM 元素指定为可拖动区域。

```css
.drag-enable {
  -webkit-app-region: drag;
}
.drag-disable {
  -webkit-app-region: no-drag;
}
```

### show

* `{Boolean}` 窗口是否显示。如要想要应用在启动时隐藏窗口，可以将其设置为 `false`。

### kiosk

* `{Boolean}` 是否使用 `Kiosk` 模式。该模式下，窗口会全屏，并阻止用户离开应用，所以记得要给用户提供一个可以退出 `Kiosk` 模式的途径。该模式主要用于公共演示。

### transparent

* `{Boolean}` 窗体是否透明。默认为 `false`。

在 CSS 中通过 rgba 背景色控制透明度。使用命令行参数 `--disable-transparency` 可以完全禁用该特性。

"click-through"（穿透点击）是一个实验性特性，支持鼠标点击时穿透窗体的透明区域：在命令行参数中添加 `--disable-gpu`。

<span id="WebKit子字段"></span>
## WebKit 子字段

### double_tap_to_zoom_enabled

* `{Boolean}` 是否启用 mac 上的两指缩放功能。默认为 `false`。

### plugin

* `{Boolean}` 是否加载外部浏览器插件，如 Flash。默认为 `true`。

## 其他字段

[Packages/1.0](http://wiki.commonjs.org/wiki/Packages/1.0) 标准中有许多其他 `package.json` 支持的字段，目前我们没有使用到，不过您仍然可以设置它们。
