# 入门指南 {: .doctitle}
---

[TOC]

> 原文链接：https://github.com/nwjs/nw.js/wiki/Getting-Started-with-nw.js
> 译者根据新版本的 NW.js 对内容做了相应调整

本章节包含一些教程，可以帮助您开始编写 NW.js 应用。假设您已经有 NW.js 的二进制文件（可以从官网的 [Downloads](https://nwjs.io/downloads/) 页面下载。如果您想要构建自己的 NW.js 版本，请参考 [构建 NW.js](../For Developers/Building NW.js.md)）。

NW.js 基于 [Chromium](http://www.chromium.org) 和 [Node.js](http://nodejs.org/)，让您可以在页面中直接调用 Node.js 代码和模块，通过 Web 技术来开发应用。而且，您也可以很容易的将 Web 应用打包为本地应用。

## 基础

我们从一个最简单的应用开始。

**示例 1. Hello World**

![Hello World](https://github.com/ssnangua/nwjs-docs-zh_CN/blob/main/assets/wiki_getting_started_with_nwjs_hello_world.jpg)

创建 `index.html`：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
  </body>
</html>
```

创建 `package.json`：

```json
{
  "name": "nw-demo",
  "main": "index.html"
}
```

下载您的系统对应的预构建二进制文件，然后通过下面的命令来运行应用（注意：该命令需要在 package.json 文件的同目录下运行）：

```bash
$ /nw可执行文件路径 .
```

> 提示：在 Windows 系统上，您可以将 `package.json` 拖放到 `nw.exe` 上来运行应用。
> 译者注：最简单的方式是将 `index.html` 和 `package.json` 放在 `nw.exe` 同目录下，然后双击 `nw.exe` 运行。

或者，您也可以将 `index.html` 和 `package.json` 压缩进一个 zip 包中，然后运行这个压缩包，zip 包可命名为 `app.nw`：

```bash
app.nw
 ┣ 📜package.json
 ┗ 📜index.html
```

使用下面的命令来从 zip 包运行应用（注意：该命令需要在 app.nw 压缩包的同目录下运行）：

```bash
$ /nw可执行文件路径 ./app.nw
```

> 提示：在 Windows 系统上，您可以将 `app.nw` 拖放到 `nw.exe` 上来运行应用。


**示例 2. 原生 UI API**

![原生 UI API](https://github.com/ssnangua/nwjs-docs-zh_CN/blob/main/assets/wiki_getting_started_with_nwjs_native_ui_api.jpg)

NW.js 有一些可以控制原生 UI 的 API，您可以通过它们来控制窗口、菜单等等。

下面的例子展示了如何使用菜单。

```html
<html>
<head>
  <meta charset="UTF-8">
  <title> Menu </title>
</head>
<body>
<script>
// 创建一个空菜单
var menu = new nw.Menu();

// 添加一些带有文本标签的菜单项
menu.append(new nw.MenuItem({ label: 'Item A' }));
menu.append(new nw.MenuItem({ label: 'Item B' }));
menu.append(new nw.MenuItem({ type: 'separator' }));
menu.append(new nw.MenuItem({ label: 'Item C' }));

// 移除一个菜单项
menu.removeAt(1);

// 遍历菜单项
for (var i = 0; i < menu.items.length; ++i) {
  console.log(menu.items[i]);
}

// 添加一个菜单项并给它绑定一个回调函数
menu.append(
  new nw.MenuItem({
    label: 'Click Me',
    click: function() {
      // 在 html body 中创建一个元素
      var element = document.createElement('div');
      element.appendChild(document.createTextNode('Clicked OK'));
      document.body.appendChild(element);
    }
  })
);

// 作为上下文菜单弹出
document.body.addEventListener('contextmenu', function(ev) { 
  ev.preventDefault();
  // 在点击的位置弹出
  menu.popup(ev.x, ev.y);
  return false;
}, false);

// 创建一个菜单栏
var menubar = new nw.Menu({ type: 'menubar' });

// 创建一个子菜单
var sub1 = new nw.Menu();

sub1.append(
  new nw.MenuItem({
    label: 'Test1',
    click: function() {
      var element = document.createElement('div');
      element.appendChild(document.createTextNode('Test 1'));
      document.body.appendChild(element);
    }
  })
);

// 给菜单栏添加子菜单
menubar.append(new nw.MenuItem({ label: 'Sub1', submenu: sub1}));

// 获取当前窗口
var win = nw.Window.get();

// 给窗口指定菜单栏
win.menu = menubar;

// 给已存在的菜单项添加一个点击事件
menu.items[0].click = function() { 
  console.log("CLICK"); 
};

</script>  
</body>
</html>
```

**示例 3. 使用 Node.js**

您可以在页面中直接调用 Node.js 及其模块，这为使用 NW.js 编写应用带来了无限可能。

```html
<html>
<head>
  <meta charset="UTF-8">
</head>
<body>
<script>
// 通过 Node.js 获取系统信息
var os = require('os');
document.write('当前系统是：', os.platform());
</script>
</body>
</html>
```


## 运行和打包应用

现在，我们可以写简单的 NW.js 应用了。接下来是学习更多关于运行和打包应用的知识。

**运行应用**

全平台通用的两种方式：

* 从一个文件夹启动：启动路径指定为该文件夹
* 从一个 `.nw` 文件（一个重命名的 .zip 文件）启动：启动路径指定为该文件

例如：

````bash
nw 文件夹路径
nw app.nw文件路径
````

学习更多关于 [运行应用](./How to run apps.md)。

学习更多关于 [打包与发布](../For Users/Package and Distribute.md)。

如果您需要管理多个版本的 NW.js，可以尝试这个工具：[@licq/tnwjs](https://www.npmjs.com/package/@licq/tnwjs)。

## 故障排查

如果应用出现了一些故障，请参阅 [故障排查](./Troubleshooting.md)。

返回 [Wiki](https://github.com/nwjs/nw.js/wiki) 获取更多信息。