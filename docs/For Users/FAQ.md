# 常见问题 {: .doctitle}
---

[TOC]


## Linux 终端不打印 console.log 信息
需要在命令行中使用 `--enable-logging=stderr`，更多信息参考 https://www.chromium.org/for-testers/enable-logging

## `var crypto = require('crypto')` 获取到错误的对象

Chromium 也拥有一个名为 `crypto` 的全局对象，它无法被重写，所以不能使用 `crypto` 作为变量名，改为其他变量名，如 `nodeCrypto`，即可正常使用。


## AnugarJS 中图片破图，开发者工具中收到 `Failed to load resource XXX net::ERR_UNKNOWN_URL_SCHEME` 报错

AngularJS 为了防止 XSS 攻击，给未知协议增加了 `unsafe:` 前缀。NW.js 和 Chrome 应用中使用的是以 `chrome-extension:` 协议开头的 URL，不能被 AnuglarJS 所识别。
解决方法是，在 AnuglarJS 配置的已知协议白名单中添加以下行：

```javascript
myApp.config(['$compileProvider',
  function($compileProvider) {
    $compileProvider.imgSrcSanitizationWhitelist(/^\s*((https?|ftp|file|blob|chrome-extension):|data:image\/)/);
    $compileProvider.aHrefSanitizationWhitelist(/^\s*(https?|ftp|mailto|tel|file|chrome-extension):/);
  }]);
```

## AngularJS 2+ 中不能打印异常信息

AngularJS 2 会尝试创建一个名为 `global` 的全局对象，并在上面注册异常处理器，但 NW.js 环境中已经存在一个 `global`，导致不能正常打印异常信息。
解决方法是，在加载 AngularJS 库之前，重命名 `global`，例如：

```html
<script>
window.nw_global = window.global;
window.global = undefined;
</script>
<!-- 引入 Angular 2 -->
```

* 参考：
  * [教程：Angular-CLI 和 NW.js](https://dev.to/thejaredwilcurt/angular-cli-and-nwjs-for-development-49gl)
  * [NW.js + Angular 8 示例](https://github.com/nwutils/nw-angular-cli-example)


## 如何通过 ESC 键退出全屏模式？

用户通常会习惯使用 ESC 键来退出全屏模式，但 NW.js 默认是没有给退出全屏模式绑定 ESC 快捷键，不过提供了进入和退出全屏模式的 API，让开发者可以很方便地控制全屏模式。

要让 ESC 键能退出全屏模式，可以使用 [Shortcut](../References/Shortcut.md)（快捷键）API：

```javascript
nw.App.registerGlobalHotKey(new nw.Shortcut({
  key: "Escape",
  active: function () {
    // 决定是否退出全屏模式
    // 然后 ...
    nw.Window.get().leaveFullscreen();
  }
}));
```

## VSCode 中如何实现 `nw` API 智能联想（自动补全）？

![Screenshot of VSCode Intellisense for NW.js](https://user-images.githubusercontent.com/4629794/59388510-ea981b00-8d39-11e9-8e18-bf3425901410.png)

1. 安装 nwjs-types
   ```
   npm install --save-dev nwjs-types
   ```
1. 强制 VSCode 加载 TypeScript 引擎。
   * 在 JS 文件顶部添加 `// @ts-check`。
   * 不是 TypeScript 也会生效。
   * 当 VSCode 看到 `// @ts-check`，会加载 TypeScript 引擎，然后在 `node_modules` 中查找相应的类型定义，然后会找到 `nwjs-types` 并加载。
   * 然后智能联想功能就会知道如何给 `nw.` 进行自动补全。

> **译者注：**
> `nwjs-types` 许久未更新，推荐安装 `@types/nw.js`
> 如果安装的是 `@types/nw.js`，后面的步骤也不需要，VSCode 会自动去 `node_modules/@types` 里找对应的类型定义
