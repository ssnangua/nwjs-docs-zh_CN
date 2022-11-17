# 使用 Node 模块 {: .doctitle}
---

[TOC]

> 原文链接：https://github.com/nwjs/nw.js/wiki/Using-Node-modules
> 译者根据新版本的 NW.js 对内容做了相应调整

Node.js 中的模块可分为三种类型：

* 内置模块（[Node API](http://nodejs.org/docs/latest/api/) 的一部分）
* 纯 JavaScript 编写的第三方模块
* 包含 C/C++ 插件的第三方模块

这三种类型全部都能在 NW.js 中使用。

## 内置模块

Node.js 的内置模块（参考 [Node API](http://nodejs.org/docs/latest/api/) 文档）可以像在 Node 环境中一样直接使用。

例如，只需要一行 `var fs = require('fs')` 就可以开始使用 [文件系统模块](http://nodejs.org/docs/latest/api/fs.html) 了。

再比如，您可以像在 Node 环境中一样直接使用 [`process` 模块](http://nodejs.org/docs/latest/api/process.html) 而无需 `require(…)`。

不过，NW.js 中的 Node API 也有一些小改动，详细信息请参阅 [Node 的改动](../References/Changes to Node.md)。

## 第三方 JavaScript 模块

如果一个第三方模块是完全用 JavaScript 编写的（即不包含任何 C/C++ [插件](http://nodejs.org/docs/latest/api/addons.html)），则在 NW.js 中可以通过和 Node 中一样的方式来使用：`require('模块名')`。

不过，相对路径的判断规则取决于父文件（执行 `require()` 的文件）本身在应用中的加载方式：

* 如果父文件也是通过 Node 的方式引入的（使用 `require()`），则子文件的相对路径将基于其父文件。
* 如果父文件是通过 WebKit 的方式引入的（可以是任何 Web 技术：传统的 DOM `window.open()`、NW.js 的 [`Window.open()`](../../References/Window.md#windowopenurl-options-callback)、传统的 DOM [`XMLHttpRequest`](https://developer.mozilla.org/en/docs/DOM/XMLHttpRequest)、jQuery 的 [`$.getScript()`](http://api.jquery.com/jQuery.getScript/)、HTML 的 `<script src="...">` 元素，等等），则子文件的相对路径将基于主页面。

前一条规则意味着任何模块的子模块都可以像在 Node 中一样引入并且正常工作。

但是，通常明智的做法是尽量不使用显式的相对路径（以 `../` 或 `./` 开头）。如果模块是在应用的 `/node_modules` 子目录中，只需要调用 `require('模块名')` 即可。（参考 Node API 的 [从 `node_modules` 文件夹中加载模块](http://nodejs.org/docs/latest/api/modules.html#modules_loading_from_node_modules_folders) 部分）。

例如，您可以在应用目录中（应用的 [配置文件](../References/Manifest Format.md) `package.json` 所在目录），通过运行 `npm install 模块名` 从 [`npm`](https://npmjs.org/) 上安装模块，`npm` 会自动将模块下载到 `/node_modules` 子目录中。

### 示例：async

下面是一个安装 `async` 模块的示例：
```bash
$ cd /path/to/your/app
$ npm install async
```

安装模块后的目录结构：
```bash
$ find .
.
./package.json
./index.html
./node_modules
./node_modules/async
./node_modules/async/.gitmodules
./node_modules/async/package.json
./node_modules/async/Makefile
./node_modules/async/LICENSE
./node_modules/async/README.md
./node_modules/async/.npmignore
./node_modules/async/lib
./node_modules/async/lib/async.js
./node_modules/async/index.js
```

应用配置（`./package.json`）如下：
```json
{
  "name": "nw-demo",
  "main": "index.html"
}
```

在 `index.html` 中引入 `async` 模块：
```html
<html>
<head>
<title>test</title>
<script>
var async = require('async');
</script>
</head>
<body>
test should be here.
</body>
</html>
```

## 包含 C/C++ 插件的第三方模块

对于包含 [C/C++ 插件](http://nodejs.org/docs/latest/api/addons.html) 的模块，情况与上述略有不同，会更加复杂，因为 NW.js 的 ABI（应用程序二进制接口）与 Node 的 ABI 有些不同，不能兼容。

当使用 `npm install` 命令为 Node 安装这类模块时，`npm` 会使用内置的 [`node-gyp`](https://github.com/TooTallNate/node-gyp) 工具从模块的源码构建适用于 Node.js 的插件。

而要为 NW.js 构建这类模块，则需要安装 [`nw-gyp`](https://github.com/rogerwang/nw-gyp)（`node-gyp` 的一个定制版本），并手动运行 `nw-gyp`。

运行 `npm install nw-gyp -g` 来全局安装 `nw-gyp`。

在正式使用 nw-gyp 之前，请查看它的 [要求](https://github.com/nwjs/nw-gyp#installation)（你需要一个合适的 Python 引擎和 C/C++ 编译器），这些要求与 node-gyp 没有什么不同。

要为 NW.js 构建一个模块，您首先要从 npm 上安装它，就和在 Node.js 中一样（`npm install 模块名`），然后进入模块目录，为 NW.js 重新构建它（`nw-gyp rebuild --target=0.5.0`）。

或者，您也可以从其他地方（如 GitHub）获取模块的源代码，然后在源码中运行 `nw-gyp rebuild --target=0.5.0`。

对于这两种方式，

* nw-gyp 需要在模块的目录中运行（模块的 `binding.gyp` 所在目录）
* 必须指定 NW.js 的目标版本（如 `nw-gyp rebuild --target=0.5.0` 里的 `0.5.0`），因为 nw-gyp 不会自动检测版本号
* 如果更换了 NW.js 版本，您需要重新构建该模块，因为 ABI 不是稳定的，并且会随着版本不断变化。例如：
    * 更换到 NW.js 0.6.0 版本，需要 `nw-gyp rebuild --target=0.6.0`
    * 更换到 NW.js 0.6.1 版本，需要 `nw-gyp rebuild --target=0.6.1`
    * 等等

ABI 的差异还意味着，为 Node 和 NW.js 构建的 C/C++ 插件（即 `.node` 文件）是 **互不兼容** 的：如果插件是为 NW.js 构建的，则不能在 Node 中使用它，反之亦然。

例如，如果插件是 **为 node-webkit 构建** 的，则您不能 **在 Node 中** 使用一些 `node test.js`（或 `npm test`）来测试这个包含插件的模块，因为测试将始终失败（或者出现一些奇怪的错误，甚至整个引擎的崩溃）。

关于 nw-gyp 的更多信息，请参阅 [使用 nw-gyp 构建原生模块](./Build native modules with nw-gyp.md)。

!!! note "译者注"
    1、一些模块有依赖的 dll（动态链接库），会放在构建目录中，`nw-gyp rebuild` 会清空构建目录，导致 dll 丢失，这时可以使用下面的命令来构建：

    ```bash
    # nw-gyp clean
    nw-gyp configure --target=0.5.0
    nw-gyp build --target=0.5.0
    ```

    `nw-gyp rebuild --target=0.5.0` 相当于上面三条命令，略过 `nw-gyp clean` 即可。

    2、在 Mac 上，如果 `nw-gyp` 构建失败，可以尝试用 `npm install 模块名 --build-from-source` 来安装模块再进行构建。