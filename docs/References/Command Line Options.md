# 命令行参数 {: .doctitle}
---

[TOC]

启动 NW.js 时，可以使用下面的命令行参数，以改变一些默认行为。

## 关于命令行参数

如果用户通过命令行启动应用，如 `your-app file.txt file2.txt`，`file.txt file2.txt` 会被记录在命令行参数中，您可以通过 [nw.App.argv](App.md) 来获取命令行参数数组。如果您的应用已经有实例正在运行中，启动新实例时，运行中的实例的 `App` 对象会触发 `open` 事件，可以获取到新实例的完整命令行。

!!! note "注意"
    现在有一个限制：如果命令行参数是硬盘上 .js 文件的文件名，NW 只会将其作为上层 Node.js 运行，这是为了支持 `child_process.fork()` API。

## `--url`

应用加载 URL：`--url=http://nwjs.io`

## `--mixed-context`

使用 [混合环境模式](../For Users/Advanced/JavaScript Contexts in NW.js.md#mixed-context-mode) 运行 NW.js，而不是独立环境模式。

## `--nwapp`

指定应用路径的可选方式。该参数在 [测试](../For Users/Advanced/Test with ChromeDriver.md) 时很有用。

## `--user-data-dir`

指定应用的数据目录，该目录用于存储数据、缓存以及崩溃转储等信息。各平台下默认的数据目录如下：

* Windows：`%LOCALAPPDATA%/<name-in-manifest>/`
* Mac：`~/Library/Application Support/<name-in-manifest>/`
* Linux：`~/.config/<name-in-manifest>`

`<name-in-manifest>` 为 [配置文件中的 `name`](Manifest Format.md#name) 字段。

## `--disable-devtools`

禁止用户访问开发工具，用于 SDK 版本。

## `--enable-node-worker`

Web Workers 启用 Node.js 整合。当使用高效 [结构化克隆算法](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) 与 DOM [交换大量数据](https://developer.mozilla.org/en-US/docs/Web/API/Worker/postMessage) 时，有助于新线程卸载占用 CPU 的任务。

注意：Node.js 的二进制模块需要线程安全才能使用该功能。我们已经对 Node.js 核心进行了修改，以确保核心 API 是线程安全的，但对于第三方二进制模块，我们无法保证这一点。对于纯 JS 模块，只要依赖的模块都是线程安全的，就是线程安全的。不启用该功能则不会有任何副作用。

## `--disable-raf-throttling`

使用该参数，requestAnimationFrame() 的回调在窗口最小化或隐藏时也会持续触发，这对于游戏开发者来说特别有用。不使用该参数时，行为与浏览器相同。

## `--disable-cookie-encryption`

默认情况下，存储在磁盘上的 cookie 在 Chromium 中是加密的。使用该参数可以禁用加密以方便测试（如在不同系统之间共享 cookie）。

## `--disable-crash-handler=true`

禁用单进程模式的崩溃处理进程。注意：需要明确地将其设置为 'true'。当它和 `--single-process` 一起使用时，您的应用程序将只有一个 NW 进程。使用该参数将禁用崩溃转储功能。该选项只在命令行中（而不是在 package.json 中）有效。

## `--enable-gcm`

启用 chrome.gcm API。

## `--enable-transparent-visuals`
## `--disable-transparency`
## `--disable-gpu`
## `--force-cpu-draw`

这些是与窗口透明度相关的参数。参考[透明窗体](../For Users/Advanced/Transparent Window.md) . 

## 其他 Chromium 参数

同时也支持 https://github.com/nwjs/chromium.src/blob/nw13/chrome/common/chrome_switches.cc 和 https://github.com/nwjs/chromium.src/blob/nw13/content/public/common/content_switches.cc 列举的 Chromium 参数。也可以参考 http://peter.sh/experiments/chromium-command-line-switches/。

这些参数可以添加到 [配置文件的 `chromium-args`](Manifest Format.md#chromium-args) 字段中，作为 NW.js 的启动参数。

## 环境变量

环境变量 `NW_PRE_ARGS` 的值会被添加到配置文件的 `chromium-args` 的值前面。

例如：

```
set NW_PRE_ARGS=--load-extension='./node_modules/nw-vue-devtools-prebuilt/extension' && nw .
```