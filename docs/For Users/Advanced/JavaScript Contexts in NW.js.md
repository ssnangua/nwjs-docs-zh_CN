# JavaScript 环境 {: .doctitle}
---

[TOC]

## JavaScript 环境的概念

不同窗口中的脚本，运行在不同的 JavaScript 环境中，比如，每个窗口都有自己的全局对象和全局构造器（如 `Array`、`Object`）。

这是 web 浏览器的通用处理方式，这样做是有很多优点的，例如：

* 当一个对象的原型（prototype）被替换，或者被一些库（如 [Prototype](http://prototypejs.org/)）或类似的脚本扩展，其他窗口中的相同对象不会被影响。
* 当程序出现了一个错误（如 [创建类实例时构造器前面缺少 `new` 关键字](http://ejohn.org/blog/simple-class-instantiation/)），并且该 bug 也影响（污染）到全局作用域，但影响范围不会继续扩大（其他窗口）。
* 当前窗口中的恶意脚本无法访问其他窗口中的重要数据。

当脚本访问其他环境中定义的对象或函数时，JS 引擎会暂时进入目标环境，并在完成后立即离开。

## NW.js 环境

NW.js 基于 Chrome 应用构建，因此 NW.js 在启动时会自动加载一个不可见的后台页面。当一个窗口被创建时，也会同时被创建一个 JavaScript 环境。

在 NW.js 中，Node.js 模块会在后台页面所运行的上下文中加载。当以混合环境模式运行时，Node.js 模块也会在每个窗口或框架的上下文中加载。参考 [独立环境模式](#独立环境模式) 和 [混合环境模式](#混合环境模式)。

## 独立环境模式

除了浏览器创建的环境，NW.js 在后台页面中引入了一个额外的 Node 环境，用于运行 Node 模块。所以，NW.js 有两个 JavaScript 环境: **浏览器环境** 和 **Node 环境**。

!!! note "Web Worker"
    您可以在 Web Worker 中 [访问 Node.js APIs](https://nwjs.io/blog/v0.18.4/)，需要在配置文件中添加 `"chromium-args": "--enable-node-worker"`。

### 浏览器环境

#### 加载脚本

通过传统的 web 方式加载或内嵌的脚本（如使用 `<script>` 元素、jQuery 的 [`$.getScript()`](http://api.jquery.com/jQuery.getScript/) 或 [RequireJS](http://requirejs.org/)），运行在浏览器环境中。

#### 全局对象

在浏览器环境中，有许多的全局对象，包括 [JS 内建对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects)（如 `Date`、`Error`、`TypedArray`）和 [Web API](https://developer.mozilla.org/en-US/docs/Web/Reference/API)（如 DOM API）。

#### 创建新的浏览器环境

每个窗口或框架都有属于自己的环境，当您创建一个新的框架或者窗口时，将得到一个新的浏览器环境。

#### 访问 Node.js 和 NW.js 的 API

Node 环境的一些对象被拷贝到了浏览器环境中，所以运行在浏览器环境中的脚本可以访问这些 Node.js 对象：

* `nw` - **`API`** 部分中介绍的所有 NW.js 的 API 都包含在这个对象中
* `global` - Node 环境的全局对象，等同于 `nw.global`（译者注：同个引用）
* `require` - 用于加载 Node.js 模块的 `require()` 函数，类似于 `nw.require()`，但它还支持通过 `require('nw.gui')` 来加载 NW.js 的 API 模块。（译者注：**不同**引用）
* `process` - Node.js 的 [process 模块](https://nodejs.org/api/globals.html#globals_process)，等同于 `nw.process`（译者注：同个引用）
* `Buffer` - Node.js 的 [Buffer 类](https://nodejs.org/api/globals.html#globals_class_buffer)

#### `require()` 的相对路径解析

浏览器环境中的相对路径基于主页面的路径（与浏览器中的加载资源的方式相同）。

### Node 环境

#### 加载脚本
Node 环境中加载脚本有以下方式：

* 通过 Node.js 的 `require()` API 加载脚本
* 通过 [配置文件中的 `node-main`](../../References/Manifest Format.md#node-main) 字段加载脚本

#### 全局对象

Node 环境中运行的脚本，可以像浏览器环境中一样使用 [JS 内建对象]()，此外还可以使用 [Node.js 定义的全局对象](https://nodejs.org/api/globals.html)，如 `__dirname`、`process`、`Buffer` 等。

!!! note "注意"
    Node 环境不能使用 Web API。参考 [Node 环境访问浏览器和 NW.js API](#Node-环境访问浏览器和-NWjs-API) 查看使用的方法。

#### 创建新的 Node 环境
**在独立环境模式下，所有的 Node 模块共享同一个 Node 环境**，不过您也可以通过下面的方法来创建新的 Node 环境：

* 通过 [`Window.open()`](../../References/Window.md#windowopenurl-options-callback) 创建新窗口时，将参数 `new_instance` 设置为 `true`。
* 启动 NW.js 时，使用命令行参数 `--mixed-context` 来进入 [混合环境模式](#混合环境模式)

#### Node 环境访问浏览器和 NW.js API

Node 环境中没有浏览器端以及 NW.js 的 API，如 `alert()`、`document.*`、`nw.Clipboard` 等。要访问浏览器 API，您必须将相应的对象（如 `window` 对象）通过 Node 环境中的函数传进来。

下面的脚本运行在 Node 环境下（myscript.js）：
```javascript
// `el` 由浏览器环境传入
exports.setText = function(el) {
    el.innerHTML = 'hello';
};
```

在浏览器环境中（index.html）：
```html
<div id="el"></div>
<script>
var myscript = require('./myscript');
// 将 `el` 元素传给 Node 函数
myscript.setText(document.getElementbyId('el'));
// "hello" 将显示在 `el` 标签中
</script>
```

!!! note "Node 环境中的 `window`"
    Node 环境中 有一个 `window` 对象，指向后台页面 DOM `window` 对象。

#### `require()` 的相对路径解析

Node 模块中的相对路径基于当前模块的路径（与 Node.js 中引入模块的方式相同）。

## 混合环境模式

混合环境在 0.13 版本引入。如果带着 [`--mixed-context` 命令行参数](../../References/Command Line Options.md#mixedcontext) 启动 NW.js，当一个浏览器环境被创建的同时，也会创建一个 Node 环境，两个环境同时运行在同一个浏览器环境中，即混合环境。

### 加载脚本

要启用混合环境，可以在启动 NW.js 时添加 `--mixed-context` 命令行参数，或者将该参数添加到 [配置文件的 `chromium-args`](../../References/Manifest Format.md#chromium-args) 选项中。

无论是通过 web 的方式，还是在 Node.js 中使用`require()`，加载的脚本都运行在同一个环境中。

### 全局对象

在混合环境下，您可以在 Node 模块中使用所有浏览器和 NW.js 的 API。

`package.json`
```javascript
{
    "name": "test-context",
    "main": "index.html",
    "chromium-args": "--mixed-context"
}
```

`myscript.js`
```javascript
exports.createDate = function() {
    return new Date();
};

exports.showAlert = function() {
    alert("我运行在 Node 模块里！");
};
```

在混合环境下，下面的比较是成立的：
`index.html`
```html
<script>
var myscript = require('./myscript');

console.log(myscript.createDate() instanceof Date); // true
myscript.showAlert(); // 我运行在 Node 模块里！
</script>
```

### 混合环境模式和独立环境模式的对比

独立环境模式的优势是不会碰到下面那样的 [类型检测问题](#多环境的使用)。

> **译者注：**
> 如果我没翻译错的话，这里的描述貌似有问题。
> iframe 也是一个独立的浏览器环境，全局对象是新创建的，不可能是相同的引用。

混合环境模式的缺点是共享变量没那么容易了，要在不同环境之间共享变量，您需要将变量放入一个其他环境也能够访问的通用环境中，或者使用 [`window.postMessage()` API](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) 在环境之间发送和接收信息。

> **译者注：**
> 混合环境的浏览器环境和 Node 环境其实是一个环境，上面的“不同环境之间”指的应该是不同的浏览器环境或 Node 环境，可以简单理解为不同窗口。
> 如果是独立环境模式，在窗口 A 中赋值 `global.a = 1`，在窗口 B 中也能访问到 `global.a`，因为 `global` 是两个窗口共用的 Node 环境（后台页面）的全局对象。
> 但在混合环境模式下，这样就行不通了，因为每个浏览器环境都会生成一个独立的 Node 环境。

## 多环境的使用

环境之间区分开通常是有利的，但有时也可能引发一些问题。

例如，在不同的浏览器环境中，全局对象不是同一个，跨环境的类型检测会失败。

```html
<iframe id="myframe" src="myframe.html"></iframe>
<script>
// `window` 是当前浏览器环境的全局对象
// `myframe.contentWindow` 是 `<iframe>` 的浏览器环境的全局对象
var currentContext = window;
var iframeContext = document.getElementById('myframe').contentWindow;

// `myfunc` 是在当前环境中定义的
function myfunc() {

}

console.log(currentContext.Date === iframeContext.Date); // false
console.log(currentContext.Function === iframeContext.Function); // false
console.log(myfunc instanceof currentContext.Function); // true
console.log(myfunc instanceof iframeContext.Function); // false
console.log(myfunc.constructor === currentContext.Function); // true
console.log(myfunc.constructor === iframeContext.Function); // false
</script>
```

### `instanceOf` 问题

这类问题通常发生在使用 `instanceof` 操作符时，正如 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) 中所述，`someValue instanceof someConstructor` 可以测试一个对象（`someValue`）的 `prototype` 属性是否在指定的构造器（`someConstructor`）的原型链中。但是，如果 `someValue` 是从其他 JavaScript 环境传过来的，那么它会有自己的基类对象，所以 `instanceof` 测试自然也就失败了。

例如，如果一个变量是其他环境传过来的，简单的 `Value instanceof Array` 检测并不能确定该变量的值是否是数组类型。参考 [精准判断一个 JavaScript 对象是否是一个数组](http://web.mit.edu/jwalden/www/isArray.html)。

### `obj.constructor` 问题

`obj.constructor` 属性检测也会有相同的问题（例如用 `Value.constructor === Array` 代替 `Value instanceof Array`）。

### `obj.__proto__` 问题

传统的 `obj.__proto__` 可以直接访问对象的原型，原型判断也会有相同的问题。

### 第三方库的问题

第三方库也可能出现上面的类型检测问题，从而导致莫名其妙的错误，这是第三方库的 bug，建议给这个库提交一个 bug 报告，或者尝试自己修复。

### 可靠的跨环境类型检测

当一个值可能来自于其他 JavaScript 环境时，**避免使用 `instanceof`** 防止出现环境相关的问题。

可以使用 [`Array.isArray`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray) 方法来检测一个值是否是一个数组，该方式跨环境也是可靠的。

要检测 `someValue` 是否是一个依赖于其他环境全局变量的对象（如 `Function` 或 `Date`等），可以使用下面的方式来检测对象的真实类型：

```javascript
// 检测一个函数
Object.prototype.toString.apply(someValue) === "[object Function]"
// 检测一个日期
Object.prototype.toString.apply(someValue) === "[object Date]"
```

然而，如果这个方便的替代方案不起作用，或者如果您面对的问题是在别人的代码中，且修复起来有些困难，那么您需要其他替代方案。

您也可以使用 [`nwglobal`](https://github.com/Mithgol/nwglobal)，它返回 Node 环境里的全局对象，在某些场景下可以作为类型检测的替代方案。
