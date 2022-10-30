# 开发者工具与调试 {: .doctitle}
---

!!! note "仅适用 SDK 版本"
    开发者工具只存在于 [SDK 版本](Advanced/Build Flavors.md) 中，推荐使用 SDK 版本来开发和测试应用，构建普通版本用于发布。

## 打开开发者工具

Windows 和 Linux 下使用快捷键 <kbd>F12</kbd> 来打开开发者工具，Mac 下使用 </kbd>+<kbd>&#8997;</kbd>+<kbd>i</kbd> 来打开。

此外，也可以在代码中使用 NW.js 的 API [`win.showDevTools()`](../References/Window.md##winshowdevtoolsiframe-callback) 来打开一个窗口的开发者工具。

## 调试 Node.js 模块

NW.js 默认运行在 [独立环境模式](Advanced/JavaScript Contexts in NW.js.md#独立环境模式) 下，要调试 Node.js 模块，可以在应用上右键，选择 "Inspect Background Page"（检查背景页），当运行到 Node.js 模块的 debugger 时，背景页的开发者工具会自动获得焦点并暂停在断点处。

如果您的应用运行在 [混合环境模式](Advanced/JavaScript Contexts in NW.js.md#混合环境模式) 下，Node.js 模块可以直接在窗口的开发者工具中调试。

参考 [JavaScript 环境](Advanced/JavaScript Contexts in NW.js.md) 查看它们的差异。

## 远程调试

通过命令行参数 `--remote-debugging-port=port` 指定开发者工具要监听哪个端口。例如，运行 `nw --remote-debugging-port=9222`，然后在浏览器中访问 http://localhost:9222/ 来进行远程调试。

## 使用开发者工具扩展

开发者工具扩展已完全支持，包括 ReactJS、Vue.js 等。需要在扩展的 manifest.json 中添加 "chrome-extension://*" 权限，然后在 nw 启动时带上 `--load-extension=path/to/extension` 命令行参数进行加载。

在 Chrome 应用商店中安装扩展后，在 Chrome 浏览器的扩展目录中，可以根据扩展 ID 找到对应的文件夹，直接复制出来用即可。

### React 示例

* https://s3-us-west-2.amazonaws.com/nwjs/sample/react-app.zip
* https://s3-us-west-2.amazonaws.com/nwjs/sample/react-devtools.zip

解压出来，下载 SDK 版本的 NW.js，然后带上命令行参数运行：`nw.exe --load-extension=path/to/devtools path/to/app/folder`

react-app 是一个简单的 react 应用。react-devtools 是在 Chrome 应用商店中安装 react 官方扩展后，从 Chrome 浏览器的扩展目录中找到的扩展文件夹，只是修改了 manifest 文件，添加了一个权限："chrome-extension://*"。

### Vue 示例

1. `npm install --save-dev nw-vue-devtools-prebuilt`
    会从 Chrome 应用商店下载最新版本的 Vue-DevTools，然后自动解压并修改以适合 NW.js 使用。
1. 在应用的 `package.json` 中添加：
    ```js
    "chromium-args": "--load-extension='./node_modules/nw-vue-devtools-prebuilt/extension'",
    ```
1. 在应用使用非压缩版本的 Vue.js（使用 `vue.js` 而不是 `vue.min.js`）。

如果使用了 `nwjs-builder-phoenix`，将 `"chromium-args"` 添加到 `package.json` 的 `build.strippedProperties` 数组中。

参考 [nw-vue-devtools-prebuilt](https://www.npmjs.com/package/nw-vue-devtools-prebuilt)。
