# 保护 JavaScript 源代码 {: doctitle}
---

[TOC]

应用的 JavaScript 源代码能够编译为本地代码，然后在 NW.js 中进行加载。您只需要在发布的版本中打包编译后的代码。

### 编译

JS 源代码通过 SDK 版本中提供的 `nwjc` 工具来编译为本地代码。

使用方法：
```bash
nwjc source.js binary.bin
```

`*.bin` 文件需要打包到应用中，文件名可自定义。

如果您要编译一个模块，需要添加 `--nw-module` 参数。

> **译者注：**要编译的代码中带有 ES6 模块规范的语法（`export`）或 CommonJS 模块规范的语法（`exports`），要使用 `nwjc --nw-module` 来编译。

### 加载编译后的 JavaScript

```javascript
nw.Window.get().evalNWBin(frame, 'binary.bin');
```
[win.evalNWBin()](../../References/Window.md#winevalnwbinframe-path) 方法中的参数与 `Window.eval()` 方法类似，第一个参数为目标 iframe（主框架传入 `null`），第二个参数是编译后的 bin 文件。

如果要将 bin 文件作为模块加载，用 [win.evalNWBinModule()](../../References/Window.md#winevalnwbinmoduleframe-path-module_path) 替代。

> **译者注：**带 `--nw-module` 参数编译的模块，要用 `evalNWBinModule()` 加载，不带 `--nw-module` 参数编译的模块，要用 `evalNWBin()` 加载，严格对应，用错会导致应用崩溃。

### 从远程加载编译后的 JavaScript

可以从远程获取（如使用 AJAX）编译后的 JavaScript 并执行。

```javascript
var xhr = new XMLHttpRequest();
xhr.responseType = 'arraybuffer'; // 作为 ArrayBuffer 返回
xhr.open('GET', url, true);
xhr.send();
xhr.onload = () => {
  // xhr.response 包含编译后的 JavaScript，作为 ArrayBuffer 返回
  nw.Window.get().evalNWBin(null, xhr.response);
}
```

!!! note "注意"
    编译后的代码会在 [浏览器环境](JavaScript Contexts in NW.js.md#浏览器环境) 中执行，就如其他运行在浏览器环境中的脚本一样，您可以使用任何 Web API（如 DOM）和 [访问 Node.js 和 NW.js 的 API](JavaScript Contexts in NW.js.md#访问Node.js和NW.js的API)。

### 在 Web Workers 中使用

worker 环境引入了一个 `importNWBin(ArrayBuffer)` 函数，可以在主线程中将 bin 文件读取为 array buffer 传给 worker，然后在 worker 里通过这个函数来执行。

### 已知问题

0.22 版本之前，编译后的代码的运行速度会 **比普通的 JS 慢**：根据 v8bench，性能约为 30%。其他未编译的 JS 代码不会受到影响。在 0.22.0-beta1 版本中，该问题得到了修复，可以查看我们的博客：https://nwjs.io/blog/js-src-protect-perf/

编译后的代码 **不支持跨平台，也不兼容其他版本** 的 NW.js，打包应用时，每个平台都需要执行 `nwjc`。

### 译者注

下面是译者的一些实践，以供参考。

#### 编译普通的 ES5 代码

ES5.js：
```javascript
window.ES5 = function() {
    console.log("普通的 ES5 代码");
}
```

编译：
```bash
nwjc ES5.js ES5.bin
```

page.js：
```javascript
nw.Window.get().evalNWBin(null, 'ES5.bin');
window.ES5(); // 输出：普通的 ES5 代码
```

#### 编译 ES6 模块

ES6.js：
```javascript
export function ES6() {
    console.log("ES6 模块");
}
```

编译：
```bash
nwjc ES6.js ES6.bin --nw-module
```

> 使用了 ES6 模块语法，编译时需要带 `--nw-module` 参数。

page.js：
```javascript
nw.Window.get().evalNWBinModule(null, 'ES6.bin', "./ES6.js");
import { ES6 } from "./ES6.js";
ES6(); // 输出：ES6 模块
```

> 加载 `--nw-module` 编译的 bin 文件，要用 `evalNWBinModule()`，不能用 `evalNWBin()`。
> 通过 `import * from '模块路径'` 来引入 bin 文件中 `export` 出来的 API。

#### 编译 CommonJS 模块

CommonJS.js：
```javascript
exports.CommonJS = function () {
    console.log("CommonJS 模块");
}
```

编译：
```bash
nwjc CommonJS.js CommonJS.bin --nw-module
```

> 使用了 CommonJS 模块语法，编译时需要带 `--nw-module` 参数。

page.js：
```javascript
nw.Window.get().evalNWBinModule(null, 'CommonJS.bin', "./CommonJS.js");
const { CommonJS } = require("./CommonJS.js");
CommonJS(); // 输出：CommonJS 模块
```

> 加载 `--nw-module` 编译的 bin 文件，要用 `evalNWBinModule()`，不能用 `evalNWBin()`。
> 通过 `var * = require('模块路径')` 来引入 bin 文件中暴露出来的 API。

#### 总结

- 如果要编译的代码只使用了 **ES5**：
  - 编译时可以不带 `--nw-module` 参数
  - 加载时使用 `evalNWBin()`
- 如果要编译的代码为 **ES6 模块** 规范：
  - 编译时需要带 `--nw-module` 参数
  - 加载时使用 `evalNWBinModule()`
  - 通过 `import` 来引入 bin 文件中 `export` 出来的 API
- 如果要编译的代码为 **CommonJS 模块** 规范：
  - 编译时需要带 `--nw-module` 参数
  - 加载时使用 `evalNWBinModule()`
  - 通过 `require()` 来引入 bin 文件中暴露出来的 API

编译时如果带了 `--nw-module` 参数，加载时必须要用 `evalNWBinModule()`，不带则必须用 `evalNWBin()`，严格对应，用错加载方法会导致应用崩溃。

一个 bin 文件中不能同时使用两种模块规范，虽然编译能通过，但引入会报错：

- `import`：CommonJS 模块规范的 `exports` 对象未定义
- `require()`：识别不了 ES6 模块规范的 `export` 语法

> 如果通过 `module.exports = *` 导出，`import` 不会报错，是因为 NW.js 给浏览器环境注入了一个 `module` 全局对象，用来管理引入的模块，`module.exports = *` 其实是挂载到了这个对象上，本质上是赋值操作，并不是 CommonJS 模块的导出行为，因为您不是通过 `require()` 引入的。