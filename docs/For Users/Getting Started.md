# 开始使用 NW.js
---

[TOC]

## NW.js 能做什么？

NW.js 基于 [Chromium](http://www.chromium.org) 和 [Node.js](http://nodejs.org/)，让您可以在页面中调用 Node.js 的 API 和模块，而且可以很容易的将 Web 应用打包为桌面应用。

## 获取 NW.js

您可以在官网 http://nwjs.io 上获取预编译的最新版本，也可以参考 [构建 NW.js](../For Developers/Building NW.js.md) 构建自己的版本。

!!! tip "提示"
    开发时建议使用带有开发者工具的 SDK 版本，方便调试。参考 [构建风格](Advanced/Build Flavors.md) 查看不同构建风格之间的差异。

## 开发 NW.js 应用

### 示例 1 - Hello World

快速开发一个 NW.js 应用。

**步骤 1.** 创建 `package.json` 文件：

```json
{
  "name": "helloworld",
  "main": "index.html"
}
```

`package.json` 是应用的配置文件，使用 [JSON 格式](http://www.json.org/) 编写。
`main` 字段为入口文件，告诉 NW.js 启动时要打开哪个文件（本例中为 `"index.html"`）。
`name` 字段是唯一的 NW.js 应用名。
参考 [Manifest Format](../References/Manifest Format.md)。

!!! tip "使用 JS 文件作为入口文件"
    `"main"` 字段也可以设置为一个 JS 文件，如 `"main.js"`，应用启动时，不会打开任何窗口，而是在背景页面中加载这个 JS 文件，您可以在执行一些初始化操作后，再手动打开一个窗口，例如：

    ```javascript
    // 初始化应用
    // 然后 ...
    nw.Window.open('index.html', {}, function(win) {});
    ```

**步骤 2.** 创建 `index.html` 文件：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
  </body>
</html>
```

这是一个普通的 HTML 文件，您可以使用任何最新浏览器支持的 web 技术（**译者注：**因为 NW.js 打包了浏览器内核，且保持更新，所以您可以使用最新的 Web 特性而不用过多考虑兼容性问题）。

**步骤 3.** 运行应用

```bash
cd /path/to/your/app
/path/to/nw .
```

`/path/to/nw` 是 NW.js 执行文件：

- Windows：`nw.exe`
- Linux：`nw`
- Mac：`nwjs.app/Contents/MacOS/nwjs`

!!! tip "Windows 系统中的拖放操作"
    在 Windows 系统中，可以把包含 `package.json` 的文件夹拖放到 `nw.exe` 上来运行应用。

### 示例 2 - 使用 NW.js API

所有的 NW.js API 都被挂载到一个名为 `nw` 的全局对象中，可以直接使用。参考 [API](../index.md#api) 查看支持的所有 API 列表。

本例展示了如何在 NW.js 应用中创建一个原生的上下文菜单。用下面的内容创建 `index.html` 文件：

```html
<!DOCTYPE html>
<html>
<head>
  <title>上下文菜单</title>
</head>
<body style="width: 100%; height: 100%;">

<p>'右键点击' 来显示上下文菜单</p>

<script>
// 创建一个空的上下文菜单
var menu = new nw.Menu();

// 添加一些带有显示文本的菜单项
menu.append(new nw.MenuItem({
  label: '菜单项 A',
  click: function(){
    alert('您点击了 "菜单项 A"');
  }
}));
menu.append(new nw.MenuItem({ label: '菜单项 B' }));
menu.append(new nw.MenuItem({ type: 'separator' }));
menu.append(new nw.MenuItem({ label: '菜单项 C' }));

// 侦听 "contextmenu" 事件
document.body.addEventListener('contextmenu', function(ev) {
  // 阻止显示浏览器的默认上下文菜单
  ev.preventDefault();
  // 在点击处弹出定义的原生上下文菜单
  menu.popup(ev.x, ev.y);

  return false;
}, false);

</script>  
</body>
</html>
```

... 然后运行应用：
```bash
cd /path/to/your/app
/path/to/nw .
```

!!! tip "require('nw.gui')"
    早期版本引入 NW.js API 的方式 `require('nw.gui')` 同样兼容，会返回同一个 `nw` 对象。

### 示例 3 - 使用 Node.js API

您可以在 DOM 中直接调用 Node.js 的 API 和模块，用 NW.js 开发应用就是这么无拘无束。

本例展示了如何通过 Node.js 的 `os` 模块查询操作系统信息。用下面的内容创建 `index.html` 文件，然后在 NW.js 中加载它：

```html
<!DOCTYPE html>
<html>
<head>
  <title>我的操作系统</title>
</head>
<body>
<script>
  // 通过 Node.js 获取操作系统信息
  var os = require('os');
  document.write('您运行在 ', os.platform());
</script>
</body>
</html>
```

您还可以在 NW.js 中调用从 [`npm`](https://www.npmjs.com/) 上安装的模块。

!!! note "原生模块"
    执行 `npm install` 时需要构建的原生模块，NW.js ABI 不能兼容，需要在模块的源代码中通过 [`nw-gyp`](https://github.com/nwjs/nw-gyp) 重新构建一个支持 NW.js 的版本。参考 [原生模块](Advanced/Use Native Node Modules.md)。

## 下一步做什么

参考 [开发者工具与调试](Debugging with DevTools.md) 了解如何调试应用。

参考 [打包与发布](Package and Distribute.md) 了解如何打包应用和发布产品。

参考 [常见问题](FAQ.md) 查看您可能会遇到的问题。

如果您需要从 0.12 或更早的版本升级到新版本，请参考 [版本升级说明](Migration/From 0.12 to 0.13.md)。

## 获取帮助

[NW.js wiki](https://github.com/nwjs/nw.js/wiki) 上有许多有用的资料。wiki 向所有人开放，欢迎在 wiki 上发布您的使用经验。

您也可以在 [Google 网上论坛](https://groups.google.com/forum/#!forum/nwjs-general) 上提问，或者在 [Gitter](https://gitter.im/nwjs/nw.js) 上留言。

欢迎在 [GitHub](https://github.com/nwjs/nw.js/issues) 上反馈 bug 或特性需求，帮助 NW.js 更加完善。

