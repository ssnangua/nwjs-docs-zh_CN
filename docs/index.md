# NW.js 文档（适用于 0.13 及之后的版本）

> 本文档由 _ssnangua_ 汉化，当前版本：nw70。
> 
> - 中文文档源码：https://github.com/ssnangua/nwjs-docs-zh_CN
> - 官方文档站点：https://nwjs.readthedocs.io/
> - 官方项目仓库：https://github.com/nwjs/nw.js

---

> [NW.js](http://nwjs.io) 由 node-webkit 项目发展而来，通过使用 Web 技术来开发桌面应用，同时能够在 DOM 中直接调用所有的 [Node.js](https://nodejs.org/) 模块。

本文档由三个主要部分组成：

* **应用开发** - 面向想要开发 NW.js 应用的开发者
* **项目开发** - 面向想要扩展 NW.js 项目的开发者
* **API** - NW.js 的 API 文档

> * **WIKI** - 部分 [WIKI](https://github.com/nwjs/nw.js/wiki) 文章的翻译和相关资料，可作为补充参考

可在 [我们的 git](https://github.com/nwjs/nw.js/tree/nw13/docs) 上查阅文档的源码，也欢迎提交 PR。

## 目录

* [主页](index.md)
* 应用开发
    - [开始](For Users/Getting Started.md)
    - [开发者工具与调试](For Users/Debugging with DevTools.md)
    - [打包与发布](For Users/Package and Distribute.md)
    - [常见问题](For Users/FAQ.md)
    - [从 0.12 升级到 0.13](For Users/Migration/From 0.12 to 0.13.md)
    - 进阶
        + [构建风格](For Users/Advanced/Build Flavors.md)
        + [JavaScript 环境](For Users/Advanced/JavaScript Contexts in NW.js.md)
        + [保护 JavaScript 源代码](For Users/Advanced/Protect JavaScript Source Code.md)
        + [安全性](For Users/Advanced/Security in NW.js.md)（iframe）
        + [Mac 应用商店支持](For Users/Advanced/Support for Mac App Store.md)
        + [ChromeDriver 自动化测试](For Users/Advanced/Test with ChromeDriver.md)
        + [透明窗口](For Users/Advanced/Transparent Window.md)
        + [Flash 插件](For Users/Advanced/Use Flash Plugin.md)
        + [NaCl](For Users/Advanced/Use NaCl in NW.js.md)
        + [Node 原生模块](For Users/Advanced/Use Native Node Modules.md)
        + [内容验证](For Users/Advanced/Content Verification.md)
        + [自定义菜单栏](For Users/Advanced/Customize Menubar.md)
* 项目开发
    - [构建 NW.js](For Developers/Building NW.js.md)
    - [贡献代码](For Developers/Contributing to NW.js.md)
    - [私有编解码器](For Developers/Enable Proprietary Codecs.md)（FFmpeg）
    - [相关仓库](For Developers/Repositories.md)
    - [了解崩溃机制](For Developers/Understanding Crash Dump.md)
    - [编写文档](For Developers/Writing Documents for NW.js.md)
    - [编写测试用例](For Developers/Writing Test Cases for NW.js.md)
    - [文档贡献者](For Developers/Contributors of Documents.md)
* API
    - [App](References/App.md)（应用）
    - [DOM 的改动](References/Changes to DOM.md)
    - [Node 的改动](References/Changes to Node.md)
    - [Clipboard](References/Clipboard.md)（剪贴板）
    - [命令行参数](References/Command Line Options.md)
    - [Chrome 扩展 API](References/Chrome Extension APIs.md)
    - [配置格式](References/Manifest Format.md)
    - [Menu](References/Menu.md)（菜单）
    - [MenuItem](References/MenuItem.md)（菜单项）
    - [Screen](References/Screen.md)（屏幕）
    - [Shell](References/Shell.md)（桌面操作）
    - [Shortcut](References/Shortcut.md)（快捷键）
    - [Tray](References/Tray.md)（托盘图标）
    - [webview 标签](References/webview Tag.md)
    - [Window](References/Window.md)（窗口）
* WIKI
    - [特性列表](Wiki/Features list.md)
    - [入门指南](Wiki/Getting Started with nw.js.md)
    - [运行应用](Wiki/How to run apps.md)
    - [故障排查](Wiki/Troubleshooting.md)
    - [使用 Node 模块](Wiki/Using Node modules.md)
    - [使用 nw-gyp 构建原生模块](Wiki/Build native modules with nw-gyp.md)
    - [文件变更实时重新加载](Wiki/Livereload nw.js on changes.md)
    - [数据持久化](Wiki/Save persistent data in app.md)
    - [Node 集成到 Chromium 的原理](Wiki/How node.js is integrated with chromium.md)
    - [使用 NW.js 的应用和公司列表](Wiki/List of apps and companies using nw.js.md)
