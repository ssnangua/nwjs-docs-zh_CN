# Window（窗口） {: .doctitle}
---

[TOC]

`Window` 是 DOM 顶层 `window` 对象的一个包装器，扩展了一些操作，并且能接收到各种窗口事件。

每个 `Window` 实例都是一个 EventEmitter 类，可以使用 `Window.on(...)` 来响应原生窗口事件。

!!! warning "特性变更"
    从 0.13.0 版本开始，`Window` 的特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

## 示例

```javascript
// 获取当前窗口对象
var win = nw.Window.get();

// 监听最小化事件
win.on('minimize', function() {
  console.log('窗口最小化啦');
});

// 窗口最小化
win.minimize();

// 取消监听最小化事件
win.removeAllListeners('minimize');

// 创建一个新窗口，并获得它的引用
nw.Window.open('https://github.com', {}, function(new_win) {
  // 监听新窗口获得焦点事件
  new_win.on('focus', function() {
    console.log('新窗口获得焦点啦');
  });
});
```

## Window.get([window_object])

* `window_object` `{DOM Window}` _可选_ DOM 的 window 对象
* Returns `{Window}` 原生 `Window` 对象

如果没有指定 `window_object`，将返回当前窗口的`Window` 对象，否则返回 `window_object` 所在窗口的`Window` 对象。

!!! note "注意"
    如果 `window_object` 是 `iframe` 的 window 对象，该方法还是会返回顶层 window 的 `Window` 对象。

```javascript
// 获取当前窗口对象
var win = nw.Window.get();

// 获取 iframe 的窗口对象
var iframeWin = nw.Window.get(iframe.contentWindow);

// 会返回 true（两个窗口对象是同一个）
console.log(iframeWin === win);

// 创建一个新窗口，并获得它的引用
nw.Window.open('https://github.com/nwjs/nw.js', {}, function(new_win) {
  // 对新窗口进行一些操作
});
```

## Window.getAll(callback)

获取所有的窗口对象，回调函数的参数是一个 nw.Window 对象的数组。0.42.6 及以上版本支持该方法。

## Window.open(url, [options], [callback])

!!! warning "特性变更"
    从 0.13.0 版本开始，该方法发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `url` `{String}` 打开的窗口要加载的 URL 
* `options` `{Object}` _可选_ 参考配置格式中的 [Window 子字段](Manifest Format.md#Window子字段)。此外，还支持以下额外字段：
    - `new_instance` `{Boolean}` _可选_ 是否在独立的渲染进程中打开新窗口。
    - `mixed_context` `{Boolean}` _Optional_ 如果为 true，Node 环境和 DOM 上下文会被合并到新的窗口进程。只在 `new_instance` 为 true 时有效。
    - `inject_js_start` `{String}` _可选_ 在构建 DOM 和执行脚本之前，注入脚本。参考 [配置格式](Manifest Format.md#inject_js_start)。
    - `inject_js_end` `{String}` _可选_ 在 document 对象加载完成后注入脚本。参考 [配置格式](Manifest Format.md#inject_js_end)。
    - `id` `{String}` _可选_ 用于识别窗口，以纪录窗口的大小和位置，当拥有相同 `id` 的窗口被重新打开时会恢复纪录的状态。参考 [Chrome 应用文档](https://developer.chrome.com/apps/app_window#type-CreateWindowOptions)
* `callback(win)` `{Function}` _可选_ 回调函数，参数为打开的本地窗口的 `Window` 对象。

打开一个新窗口，并在里面加载 `url`。

!!! note "注意"
    需要在窗口的 `loaded` 事件触发后，才能和窗口内的组件交互。

!!! note "获得焦点"
    打开的窗口不会自动获得焦点，如果需要自动获得焦点，可将 `options` 里的 `focus` 设置为 `true`。

## win.window

获取本地窗口对应的 DOM 的 window 对象。

## win.x
## win.y

获取或设置窗口在屏幕中距离左侧和顶部的偏移值。

## win.width
## win.height

获取或设置窗口的尺寸（包含窗体）。

## win.title

获取或设置窗口的标题信息。

## win.menu

获取或设置窗口的菜单栏。值是一个类型为 `menubar` 的 Menu 对象。当将 `win.menu` 设为 `null` 时，在 Windows 和 Linux 系统中菜单栏会被移除，在 Mac 系统中菜单栏将被清空。

参考 [自定义菜单栏](../For Users/Advanced/Customize Menubar.md) 查看使用示例，参考 [菜单](Menu.md) 和 [菜单项](MenuItem.md) 查看相关 API 的详细信息。

## win.isAlwaysOnTop

获取窗口是否置顶（永远在其他窗口上面）。

## win.isFullscreen

获取窗口是否处于全屏模式。

## win.isTransparent 

获取窗口是否启用透明。

## win.isKioskMode

获取窗口是否处于 `Kiosk` 模式。

## win.zoomLevel

获取或设置页面的缩放等级。`0` 为正常大小，正数为放大，负数为缩小。

## win.cookies.*

包含多个用于操作 cookie 的函数，API 的定义方式和 [Chrome 扩展](http://developer.chrome.com/extensions/cookies.html) 相同。NW.js 支持 `get`、`getAll`、`remove` 和 `set` 方法，以及 `onChanged` 事件（该事件支持 `addListener` 和 `removeListener` 方法）。

不支持 Chrome 扩展中和 `CookieStore` 相关的 API,因为 NW.js 应用只存储了一个全局的 cookie。

## win.moveTo(x, y)

* `x` `{Integer}` 距离屏幕左侧的偏移值
* `y` `{Integer}` 距离屏幕顶部的偏移值

移动窗口的左上角到指定坐标。

## win.moveBy(x, y)

* `x` `{Integer}` 横向偏移值
* `y` `{Integer}` 纵向偏移值

移动窗口到指定偏移量的位置（相对于窗口当前位置）。

## win.resizeTo(width, height)

* `width` `{Integer}` 窗口的内部宽度
* `height` `{Integer}` 窗口的内部高度

调整窗口的尺寸为指定的宽度和高度。

## win.setInnerWidth(width)

* `width` `{Integer}` 窗口的内部宽度

## win.setInnerHeight(height)

* `height` `{Integer}` 窗口的内部高度

## win.resizeBy(width, height)

* `width` `{Integer}` 宽度偏移值
* `height` `{Integer}` 高度偏移值

根据偏移量调整窗口的尺寸。 

## win.focus()

窗口获得焦点。

## win.blur()

窗口失去焦点。通常失去的焦点会移到应用的其他窗口上，部分系统没有失去焦点的概念。

## win.show([is_show])

* `is_show` `{Boolean}` _可选_ 指定窗口是显示还是隐藏。默认值为 `true`。

显示隐藏的窗口。

`show(false)` 的效果与 `hide()` 相同。

!!! note "获得焦点"
    在部分系统中，`show` 不会使窗口自动获得焦点，如果有需要，应该手动调用 `focus`。

## win.hide()

隐藏窗口。窗口隐藏后，用户无法找到（不同于最小化）。

## win.close([force])

* `force` `{Boolean}` 是否强制关闭窗口并跳过 `close` 事件。

关闭当前窗口。可以通过监听 `close` 事件来阻止窗口关闭。如果设置了 `force` 且值为 `true`，则 `close` 事件会被忽略。

通常，我们会通过监听 `close` 事件来处理一些关闭前的操作，再使用 `close(true)` 真正地关闭窗口。

```javascript
win.on('close', function() {
  this.hide(); // 隐藏窗口，假装窗口已经关闭了
  console.log("正在关闭中...");
  this.close(true); // 然后强制关闭窗口
});

win.close(); // 会被上面的 close 事件拦截，要跳过上面的 close 事件，需要使用 win.close(true);
```

## win.reload()

重新加载当前窗口。

## win.reloadDev()

通过一个新的渲染进程重新加载当前页面。

## win.reloadIgnoringCache()

类似于 `reload()`，但不使用缓存（也叫做 "shift-reload"）。

## win.maximize()

Window 和 GTK 系统上，最大化窗口；
Mac OS X 系统上，放大窗口。

## win.unmaximize()

!!! warning "已弃用"
    从 0.13.0 版本开始，该属性已被弃用。现在使用 [`restore` 事件](#restore) 代替。参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

取消窗口最大化，`maximize()` 的逆向操作。

## win.minimize()

Windows 系统上，将窗口最小化到任务栏；
GTK 系统上，将窗口图标化；
Mac OS X 系统上，将窗口最小化。

## win.restore()

将窗口恢复到最小化或最大化前的状态。`minimize()` 和 `maximize()` 的逆向操作。

现在统一使用 `restore()`，不再使用 `unminimize()`。

## win.enterFullscreen()

使窗口全屏。

!!! note "注意"
    该方法不同于 HTML5 的全屏接口，HTML5 的全屏接口可以使页面的一部分全屏，而 `Window.enterFullscreen` 只能把整个窗口全屏。

## win.leaveFullscreen()

退出全屏模式。

## win.toggleFullscreen()

切换全屏模式。

## win.enterKioskMode()

进入 `Kiosk` 模式。该模式下，窗口会全屏，并阻止用户离开应用，所以记得要给用户提供一个可以退出 `Kiosk` 模式的途径。该模式主要用于公共演示。

## win.leaveKioskMode()

退出 `Kiosk` 模式。

## win.toggleKioskMode()

切换 `Kiosk` 模式。

## win.setTransparent(transparent)

!!! warning "已弃用"
    从 0.13.0 版本开始，该特性已被弃用。参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `transparent` `{Boolean}` 是否将窗口设为透明

开启或关闭窗口的透明支持。参考 [透明窗口](../For Users/Advanced/Transparent Window.md)。

## win.setShadow(shadow) (Mac)

* `shadow` `{Boolean}` 窗口是否有阴影

开启或关闭窗口的本地阴影。对无边框、透明的窗口很有用。

## win.showDevTools([iframe], [callback])

!!! note "注意"
    该方法只在 SDK 版本中有效。

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `iframe` `{String} or {HTMLIFrameElement}` _可选_ 要检视的 `<iframe>` 的 id 或元素。默认情况下，将作为整个窗口的开发者工具显示。
* `callback(dev_win)` `{Function}` 回到函数，参数为开发者工具的本地窗口对象。

打开开发者工具窗口。

可选参数 `iframe` 为 `String` 时，必须是窗口内任意 `<iframe>` 元素的 `id` 属性的值。如果是一个有效的值，开发者工具将只检视该 `<iframe>` 的内容。如果为空字符串则没有效果。

可选参数 `iframe` 为 `HTMLIFrameElement` 时，必须是一个 iframe 对象。作用与传入 `id` 参数相同。

可选参数 `callback` 的参数是开发者工具窗口的 `Window` 对象，该对象可以使用除事件以外的所有属性和方法。

参考 [webview 标签](webview Tag.md)，查看如何为 webview 打开开发者工具，或在 webview 中打开开发者工具。

## win.closeDevTools()

!!! note "注意"
    该方法只在 SDK 版本中有效。 

关闭开发者工具窗口。

## win.getPrinters(callback)

列出系统中的打印机。回调函数会接收到一个 JSON 对象格式的打印机信息数组。JSON 对象中的设备名可作为 `nw.Window.print()` 的参数。

## win.isDevToolsOpen()

!!! note "注意"
    该方法只在 SDK 版本中有效。

查询开发者工具窗口的状态。

参考 [`win.showDevTools()`](#winshowdevtoolsiframe-callback)。

## win.print(options)

打印窗口中网页内容（允许用户交互或者不交互）。`options` 是一个 JSON 对象，包含以下字段：

* `autoprint` `{Boolean}` 是否在无需用户交互的情况下执行打印操作。可选，默认为 `true`。
* `silent` `{Boolean}` 隐藏一闪而过的打印预览对话框。可选，默认为 `false`。
* `printer` `{String}` 打印机设备名。可通过 `nw.Window.getPrinters()` 获得。打印为 PDF 时不需要设置该属性。
* `pdf_path` `{String}` 打印为 PDF 时输出的路径。
* `headerFooterEnabled` `{Boolean}` 是否启用页眉和页脚。
* `landscape` `{Boolean}` 使用横向还是纵向进行打印。
* `mediaSize` `{JSON Object}` 页面尺寸
* `shouldPrintBackgrounds` `{Boolean}` 是否打印 CSS 背景
* `marginsType` `{Integer}` 0 - 默认；1 - 无留边；2 - 最小值；3 - 自定义，参考 `marginsCustom`。
* `marginsCustom` `{JSON Object}` 自定义留边设置，单位为 point（点）。
* `copies` `{Integer}` 要打印的份数。
* `scaleFactor` `{Integer}` 比例系数，默认值为 `100`。
* `headerString` `{String}` 一个字符串，用于替换页眉里的 URL
* `footerString` `{String}` 一个字符串，用于替换页脚里的 URL

`marginsCustom` 示例：
```json
"marginsCustom": {
  "marginBottom": 54,
  "marginLeft": 70,
  "marginRight": 28,
  "marginTop": 32
}
```

`mediaSize` 示例：
```json
"mediaSize": {
  "name": "CUSTOM",
  "width_microns": 279400,
  "height_microns": 215900,
  "custom_display_name": "Letter",
  "is_default": true
}
```

!!! note "注意"
    如果不需要传入任何选项，应该调用 `win.print({})`。

## win.setMaximumSize(width, height)

* `width` `{Integer}` 窗口的最大宽度
* `height` `{Integer}` 窗口的最大高度

设置窗口的最大尺寸。

## win.setMinimumSize(width, height)

* `width` `{Integer}` 窗口的最小宽度
* `height` `{Integer}` 窗口的最小高度

设置窗口的最小尺寸。

## win.setResizable(resizable)

* `resizable` `{Boolean}` 窗口是否可以调整大小。

设置窗口是否可以调整大小。

## win.setAlwaysOnTop(top)

* `top` `{Boolean}` 窗口是否置顶。

设置窗口是否置顶（永远在其他窗口上面）。

## win.setVisibleOnAllWorkspaces(visible) (Mac & Linux)

* `visible` `{Boolean}` 窗口是否在所有工作区中可见

只在支持多工作区的系统中有效（目前有 Mac OS X 和Linux），让 NW.js 窗口同时在所有的工作区中可见。

## win.canSetVisibleOnAllWorkspaces() (Mac & Linux)

返回一个布尔值，表示当前系统是否支持 `win.setVisibleOnAllWorkspaces(Boolean)` API。

## win.setPosition(position)

* `position` `{String}` 窗口的位置。有三个有效的值：`null`、`center`、`mouse`。

移动窗口到指定位置。目前只有 `center` 是全平台有效的，它会将窗口放置在屏幕正中央。

## win.setShowInTaskbar(show)

* `show` `{Boolean}` 是否在任务栏中显示。

控制窗口是否在任务栏或 dock 中显示。参考配置格式中的 [`show_in_tackbar`](Manifest Format.md#show_in_taskbar) 字段。

## win.requestAttention(attension)

* `attension` `{Boolean} | {Integer}` 值为布尔值时，表示开始或取消吸引用户注意。值为数值时，代表窗口闪烁的次数。

通过使任务栏中的窗口闪烁来吸引用户注意。

!!! note "Mac"
    在 Mac 系统上，value < 0 将触发 `NSInformationalRequest`，value > 0 将触发 `NSCriticalRequest`。

## win.capturePage(callback [, config ])

* `callback` `{Function}` 截取窗口完成后的回调
* `config` `{String} or {Object}` _可选_ 类型为字符串时，同 `config.format`。
    - `format` `{String}` _可选_ 生成图片的格式，支持两种格式：`"png"` 和 `"jpeg"`，默认为 `"jpeg"`。
    - `datatype` `{String}` 支持三种数据类型：`"raw"`、`"buffer"` 和 `"datauri"`，默认为 `"datauri"`。

截取窗口的可视区域。要截取整个页面，使用 `win.captureScreenshot`。

!!! note "`raw` 还是 `datauri`"
    `"raw"` 只包含 Base64 编码的图片数据，而 `"datauri"` 还包含 mine type 头，可以直接用于 `Image` 标签的 `src` 属性以加载图片。

示例：
```javascript
// png 格式的 base64 字符串
win.capturePage(function(base64string){
 // 对 base64 字符串执行一些操作
}, { format : 'png', datatype : 'raw'} );

// png 格式的 node buffer 数据
win.capturePage(function(buffer){
 // 对 buffer 执行一些操作
}, { format : 'png', datatype : 'buffer'} );
```

## win.captureScreenshot(options [, callback])

如果省略了 `callback`，将返回一个 Promise，参数与回调函数的 `data` 相同。

* `callback` `{Function}` _optional_ 截取窗口完成后的回调，参数有：
    - `err` 截图成功时为 `null`，截图失败则是一个 `{String}` 类型的错误信息。
    - `data` `{String}` base64 编码的图片数据。
* `options` `{Object}`
    - `fullSize` `{Boolean}` _optional_ 是否截取整个页面（包含超出可视区域的部分）。目前 Chromium 截取图像的最大高度为 16384 像素。
    - `format` `{String}` _optional_ 生成图片的格式，支持两种格式：`"png"` 和 `"jpeg"`，默认为 `"png"`。
    - `quality` `{Integer}` _optional_ 图片压缩质量，取值为 [0..100]，仅对 jpeg 格式有效。
    - `clip` `{Object}` _optional_ 只截取指定的区域。区域为一个对象，包含以下属性：
      - `x`: `{Number}` X 偏移（与设备无关的像素值（dip））。
      - `y`: `{Number}` Y 偏移（与设备无关的像素值（dip））。
      - `width`: `{Number}` 截取宽度（与设备无关的像素值（dip））。
      - `height`: `{Number}` 截取高度（与设备无关的像素值（dip））。
      - `scale`: `{Number}` 页面比例系数。

截取窗口。可截取整个页面（包含超出可视区域的部分）。

!!! note "实验性"
    实验性 API，预期行为可能会在未来发生变更。

示例：
```javascript
win.captureScreenshot({ fullSize: true }, (err, data) => {
  if (err) {
    alert(err.message);
    return;
  }
  var image = new Image();
  image.src = 'data:image/jpg;base64,' + data;
  document.body.appendChild(image);
});
```

## win.setProgressBar(progress)

* `progress` `{Float}` 有效值为 [0, 1]，设置为负数（<0）则移除进度条。

!!! note "Linux"
    Linux 系统只支持 Ubuntu，并且您需要通过 `NW_DESKTOP` 环境变量指定应用程序的 `.desktop` 文件。如果没有找到 `NW_DESKTOP` 环境变量，将默认使用 `nw.desktop`。

## win.setBadgeLabel(label)

设置任务栏或 dock 中的窗口图标的标记（俗称小红点，如未读消息数）。

!!! note "Linux"
    Linux 系统只支持 Ubuntu，且 label 只能是字符串数值，且同样需要指定应用程序的 `.desktop` 文件（参考 [`setProgressBar`](#winsetprogressbarprogress)）。

## win.eval(frame, script)

* `frame` `{HTMLIFrameElement}` 要执行的框架。如果 `iframe` 为 `null`，将在当前窗口或框架中执行。
* `script` `{String}` 要执行的代码

在框架中执行一段 JavaScript 代码。

## win.evalNWBin(frame, path)

* `frame` `{HTMLIFrameElement}` 要执行的框架。如果 `iframe` 为 `null`，将在当前窗口或框架中执行。
* `path` `{String|ArrayBuffer|Buffer}` 通过 `nwjc` 生成的二进制文件的路径或 `Buffer` 或 `ArrayBuffer`。

在框架中加载和执行编译好的二进制文件。参考 [保护 JavaScript 源代码](../For Users/Advanced/Protect JavaScript Source Code.md)。

## win.evalNWBinModule(frame, path, module_path)

!!! warning "实验性"
    实验性 API，预期行为可能会在未来发生变更。我们正在探索可以更好地支持该功能的方法。[在这里讨论](https://github.com/nwjs/nw.js/issues/6303)。

* `frame` `{HTMLIFrameElement}` 要执行的框架。如果 `iframe` 为 `null`，将在当前窗口或框架中执行。
* `path` `{String|ArrayBuffer|Buffer}` 通过 `nwjc` 生成的二进制文件的路径或 `Buffer` 或 `ArrayBuffer`。
* `module_path` `{String}` 相对于当前文档的模块 URL。用于 [resolve the module specifier](https://html.spec.whatwg.org/multipage/webappapis.html#resolve-a-module-specifier)。

将编译好的二进制文件作为模块在框架中的进行加载并执行。二进制文件可通过 `nwjc --nw-module` 编译获得。下面的代码会将 lib.bin 作为模块进行加载，然后其他模块可以通过类似 `import * from './lib.js'` 的方式进行引用：
```javascript
nw.Window.get().evalNWBinModule(null, 'lib.bin', 'lib.js');
```

## win.removeAllListeners([eventName])

移除所有的侦听器，或移除 `eventName` 指定的侦听器。

## 事件：close

`close` 事件是一个特殊的事件，它会影响 `Window.close()` 方法的执行结果，如果监听了窗口的 `close` 事件，`Window.close()` 只会触发一个 `close` 事件，而不会关闭窗口。

通常，我们会通过监听 `close` 事件来处理一些关闭前的操作，再使用 `close(true)` 真正地关闭窗口。如果在 `close` 事件中调用 `this.close()` 时忘记传入`true`，`close` 事件将陷入无限循环中。

如果关闭前的操作需要耗费一些事件，会导致用户感觉我们的应该很慢，带来不好的体验，这时您只需要在 `close` 事件中先把窗口隐藏了，让用户觉得窗口已经关闭了，体验上会好很多。

参考 [`win.close(true)`](#wincloseforce) 的示例中关于 `close` 事件的用法。

!!! note "Mac"
    在 Mac 上，回调函数会有一个参数表示是否是通过 <kbd>&#8984;</kbd>+<kbd>Q</kbd> 的方式关闭窗口，如果是，该参数为字符串 `quit`，否则为 `undefined`。

## 事件：closed

事件在窗口关闭后触发。通常情况下，您无法处理该事件，因为窗口关闭后所有的 js 对象都已经被释放了。不过，如果是在其他窗口中侦听该事件，则可正常捕获。

```javascript
// 打开一个新窗口
nw.Window.open('popup.html', {}, function (win) {
  // 新窗口关闭后，释放 'win' 对象
  win.on('closed', function () {
    win = null;
  });

  // 侦听主窗口的 `close` 事件
  nw.Window.get().on('close', function() {
    // 隐藏窗口，让用户感觉窗口是立即关闭的
    this.hide();

    // 如果新窗口未关闭，则关闭它
    if (win != null) {
      win.close(true);
    }

    // 关闭新窗口后，关闭主窗口
    this.close(true);
  });
});
```

## 事件：loading

窗口开始加载时触发。通常情况下，您无法处理该事件，因为它触发的时候，页面中的代码还没有初始化。

唯一可以捕获该事件方法是，在刷新窗口时，在其他窗口中侦听该事件。

## 事件：loaded

`loaded` 事件在窗口全部加载完成时触发。该事件的行为与 `window.onload` 相同，只是不需要依赖 DOM。

## 事件：document-start(frame)

* `frame` `{HTMLIFrameElement}` iframe 触发时为 iframe 对象，窗口触发时为 `null`。

窗口中的 document 对象或子 iframe 可用时触发，所有的文件都已加载完成，但 DOM 还未初始化，脚本也还没开始运行。
在使用 nw.Window.open() 创建的新窗口上不会触发该事件：那个函数的回调和该事件会在同一时间触发。 

参考配置格式中的 [`inject_js_start`](Manifest Format.md#inject_js_start) 字段。

## 事件：document-end(frame)

* `frame` `{HTMLIFrameElement}` iframe 触发时为 iframe 对象，窗口触发时为 `null`。

窗口中的 document 对象或子 iframe 卸载时触发，在 `onunload` 事件触发之前。

参考配置格式中的 [`inject_js_end`](Manifest Format.md#inject_js_end) 字段。

## 事件：focus

窗口获得焦点时触发。

## 事件：blur

窗口失去焦点时触发。

## 事件：minimize

窗口最小化时触发。

## 事件：restore

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

窗口从最小化、最大化、全屏状态下恢复之前的状态时触发。

## 事件：maximize

窗口最大化时触发。

## 事件：move(x, y)

窗口移动后触发。回调函数有两个参数：`(x, y)` 表示窗口移动后左上角的位置。

## 事件：resize(width, height)

窗口尺寸变更后触发，回调函数有两个参数：`(width, height)` 表示窗口调整后的新尺寸。

## 事件：enter-fullscreen

窗口进入全屏模式时触发。

## 事件：leave-fullscreen

!!! warning "已弃用"
    从 0.13.0 版本开始，该特性已被弃用。现在使用 [`restore` 事件](#restore) 代替。参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

窗口退出全屏模式时触发。

## 事件：zoom

窗口发生缩放时触发。回调函数有一个参数，表示新的缩放等级。参考 [`win.zoomLevel`](#winzoomlevel) 查看参数值的定义。

## 事件：capturepagedone

!!! warning "已弃用"
    从 0.13.0 版本开始，该特性已被弃用。使用 `win.capturePage()` 的回调函数代替。参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

调用 capturePage 方法后，图片数据准备完成后触发。参考 `win.capturePage()` 的回调函数，查看参数值的定义。

## 事件：devtools-opened(url)

!!! warning "已弃用"
    从 0.13.0 版本开始，该特性已被弃用。使用 [`win.showDevtools()` 方法](#winshowdevtoolsiframe-callback) 的 `callback` 参数代替。参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

## 事件：devtools-closed

开发者工具关闭后触发。

参考 [`win.closeDevTools()` 方法](#winclosedevtools)。

## 事件：new-win-policy (frame, url, policy)

* `frame` `{HTMLIFrameElement}` iframe 触发时为 iframe 对象，窗口触发时为 `null`。
* `url` `{String}` 要打开的链接地址
* `policy` `{Object}` 一个包含以下方法的对象：
    * `ignore()`：忽略请求，不会发生默认行为。
    * `forceCurrent()`：强制在同框架中打开链接
    * `forceDownload()`：强行将链接作为可下载内容，或使用外部程序打开
    * `forceNewWindow()`：强制在新窗口中打开链接
    * `forceNewPopup()`：强制在新弹出窗口中打开链接
    * `setNewWindowManifest(m)`：控制新弹出窗口中的设置。`m` 的格式与配置格式中的 [Window 子字段](Manifest Format.md#Window子字段) 相同。

当前窗口或子 iframe 要打开一个新窗口时触发。可以在回调函数中调用 `policy.*` 方法来改变打开新窗口的默认行为。

例如，当用户尝试打开一个新窗口时，可以阻止该行为，并在系统默认浏览器中打开指定的 URL：
```javascript
nw.Window.get().on('new-win-policy', function(frame, url, policy) {
  // 不要打开窗口
  policy.ignore();
  // 然后在系统默认浏览器中打开指定的 URL
  nw.Shell.openExternal(url);
});
```

> **译者注：**
> 尝试了下，对 `<a target="_blank">` 标签有效，对 `nw.Window.open()` 无效。
> 想要更细致的处理，可以用 `chrome.webRequest.onBeforeRequest` API 做请求拦截。

## 事件：navigation (frame, url, policy)

* `frame` `{HTMLIFrameElement}` iframe 触发时为 iframe 对象，窗口触发时为 `null`。
* `url` `{String}` 请求的链接地址
* `policy` `{Object}` 一个包含以下方法的对象：
    * `ignore()`：忽略请求，不会发生默认的导航行为。

要导航到其他页面时触发。类似于 [`new-win-policy`](#new-win-policy-frame-url-policy)，可以在回调函数中调用 `policy.ignore()` 来忽略默认的导航行为。

