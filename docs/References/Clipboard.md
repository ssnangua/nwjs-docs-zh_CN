# Clipboard（剪贴板） {: .doctitle}

---

[TOC]

`Clipboard` 是 Windows、Linux、Mac 剪贴板的抽象模块。

## 示例

```javascript
// 获取系统剪贴板
var clipboard = nw.Clipboard.get();

// 从剪贴板中读取数据
var text = clipboard.get('text');
console.log(text);

// 向剪贴板写入数据
clipboard.set('I love NW.js :)', 'text');

// 清空剪贴板
clipboard.clear();
```

## Clipboard.get()

* 返回 `{Clipboard}`，剪贴板对象

!!! note "注意"
    不支持 X11 中的选择剪贴板。

## clip.set(data, [type, [raw]])

* `data` `{String}` 要写入剪贴板的数据
* `type` `{String}` _可选_ 数据类型。支持 `text`、`png`、`jpeg`、`html` 和 `rtf`。默认为 `text`。
* `raw`  `{Boolean}` _可选_ 需要原始图片数据。该参数只在数据类型为 `png` 和 `jpeg` 时有效。默认值为 `false`。

将 `type` 类型的 `data` 写入剪贴板.

该方法将清空剪贴板并替换为传入的 `data`，每次调用都会覆盖掉原来的数据，如果想要同时写入多种类型的数据，需要使用 [clip.set(clipboardDataList)](#clipsetclipboarddatalist)。

!!! tip "图片格式"
    剪贴板支持读写 JPEG 或 PNG 类型的图片数据。如果不传入 `raw` 或传入 `false`，`data` 为有效的 Base64 编码的 [data URI](https://developer.mozilla.org/en-US/docs/Web/HTTP/data_URIs)。如果 `raw` 传入 `true`，则 `data` 为 Base64 编码的图片数据（不包含 `data:<mime-type>;base64,` 部分）。

## clip.set(clipboardData)

* `clipboardData` `{Object}` JSON 对象，包含要写入剪贴板的 `data`、`type` 和 `raw` 信息。参考 [clip.set(data, \[type, \[raw\]\])](#clipsetdata-type-raw) 的参数说明。

## clip.set(clipboardDataList)

* `clipboardDataList` `{Array}` 要写入剪贴板的 `clipboardData` 数组。参考 [clip.set(clipboardData)](#clipsetclipboarddata) 和 [clip.set(data, [type, [raw]])](#clipsetdata-type-raw) 的参数说明。

该方法可同时将多种类型的数据写入剪贴板。

例如，将一张图片和一个指向该图片的 `<img>` 节点写入剪贴板：

```javascript
var fs = require('fs');
var path = require('path');

// 使用绝对路径以便于其他应用使用
var pngPath = path.resolve('nw.png');
// 读取图片文件并通过 base64 进行编码
var data = fs.readFileSync(pngPath).toString('base64');
// 转换文件路径为 URL
var html = '<img src="file:///' + encodeURI(data.replace(/^\//, '')) + '">';

var clip = nw.Clipboard.get();
// 将 PNG 和 HTML 写入剪贴板
clip.set([
  {type: 'png', data: data, raw: true},
  {type: 'html', data: html}
]);
```

## clip.get([type, [raw]])

* `type` `{String}` _可选_ 数据类型。支持 `text`、`png`、`jpeg`、`html` 和 `rtf`。默认为 `text`。
* `raw`  `{Boolean}` _可选_ 需要原始图片数据。该参数只在数据类型为 `png` 和 `jpeg` 时有效。
* 返回 `{String}` 从剪贴板中获取到的数据

从剪贴板中获取 `type` 类型的数据。

## clip.get(clipboardData)

* `clipboardData` `{Object}` JSON 对象，包含要从剪贴板中读取的 `type` 和 `raw` 信息。参考 [clip.get([type, \[raw\]])](#clipgettype-raw) 的参数说明。
* 返回 `{String}` 从剪贴板中获取到的数据

## clip.get(clipboardDataList)

* `clipboardDataList` `{Array}` 要从剪贴板中读取的 `clipboardData` 数组。该方法支持从剪贴板中读取多种类型的数据。参考 [clip.get(clipboardData)](#clipgetclipboarddata) 和 [clip.get([type, [raw]])](#clipgettype-raw) 的参数说明。
* 返回 `{Array}` 从剪贴板中获取到的数据列表。`clipboardData` 数组，每个元素都包含 `type`、`data` 和 `raw` 信息。

## clip.readAvailableTypes()

* 返回 `{Array}` 当前剪贴板中有效的数据类型列表。可能的数据类型有：
  - `text`：无格式文本，可通过 `clip.get('text')` 读取。
  - `html`：HTML 文本，可通过 `clip.get('html')` 读取。 
  - `rtf`：富文本，可通过 `clip.get('rtf')` 读取。 
  - `png`：PNG 图片，可通过 `clip.get('png')` 读取。 
  - `jpeg`：JPEG 图片，可通过 `clip.get('jpeg')` 读取。 

可以参考返回的类型列表，从剪贴板中获取相应类型的数据。

## clip.clear()

清空剪贴板。
