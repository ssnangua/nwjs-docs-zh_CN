# DOM 的改动 {: .doctitle}
---

[TOC]

## &lt;input type="file"&gt;

HTML5 的 `<input type="file" />` 元素给文件对话框提供了有限的支持，如 [`multiple`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#attr-multiple)、[`accept`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#attr-accept) 和 `webkitdirectory`。

NW.js 在此基础上进行了扩展，以更好的支持本地应用。

!!! note "注意"
    出于安全性考虑，NW.js 扩展的特性只在支持 Node 环境的框架中启用。参考 [安全](../For Users/Advanced/Security in NW.js.md)，查看支持Node的框架和普通框架的区别。

### fileinput.value

该属性变更为文件的本地路径。

例如，您可以使用 Node.js 的 API 读取用户所选的文件：

```javascript
// 获取用户所选文件的本地路径
var fileinput = document.querySelector('input[type=file]');
var path = fileinput.value;

// 使用 Node.js 的 API 读取文件
var fs = nw.require('fs');
fs.readFile(path, 'utf8', function(err, txt) {
  if (err) {
    console.error(err);
    return;
  }

  console.log(txt);
});
```

### fileitem.path

HTML5 的 `<input>` 标签提供了一个 `files` 属性，可以返回选择的所有文件。

NW.js 为 `files` 中每一个 file 项提供了一个额外的属性 `fileitem.path`，对应所选文件的本地路径。

```javascript
var fileinput = document.querySelector('input[type=file]');
var files = fileinput.files;

for (var i = 0; i < files.length; ++i) {
  console.log(files[i].path);
}
```

### 属性：`nwdirectory`

`nwdirectory` 类似于 `webkitdirectory`，但返回值为目录路径而不是文件列表。

例如：

```html
<input type="file" nwdirectory>
```

### 属性：`nwdirectorydesc`

设置 `nwdirectory` 文件对话框的标题描述，默认为 `选择文件夹`。

### 属性：`nwsaveas`

`nwsaveas` 会打开一个可以让用户输入文件名的 `另存为` 对话框。与 `打开` 对话框不同，它的值可以是一个不存在的文件。

例如：

```html
<input type="file" nwsaveas>
```

您也可以为要保存的文件设置一个默认的文件名：

```html
<input type="file" nwsaveas="filename.txt">
```

### 属性：`nwworkingdir`

`nwworkingdir` 用于设置文件对话框打开时的初始目录。

例如，通过下面的代码打开文件对话框，初始目录路径为 `/home/path/`：

```html
<input type="file" nwworkingdir="/home/path/">
```
### `oncancel` 事件

用户取消对话框时触发该事件。

## &lt;iframe&gt;

NW.js 扩展了 `<iframe>` 标签，可以绕过沙箱限制以及同源策略的跨域问题等，便于本地应用的开发。

更多详细信息，参考 [webview 标签](webview Tag.md)。

### 属性：nwdisable

使框架和子框架为普通框架。

!!! note "注意"
    该属性不能阻止普通框架中的页面访问上层框架，仍然有可能可以访问 Node.js 的 API，所以该属性通常会搭配 `nwfaketop` 属性一起使用。

### 属性：nwfaketop

阻止框架中的页面访问真实的 `window.parent` 和 `window.top`。设置该属性后，框架会有一个独立的 `window` 对象，访问到的上层框架也是假的。设置了该属性的框架，其内部的子框架也会受到影响。

该属性通常搭配 `nwdisable` 属性一起使用。

### 属性：nwUserAgent

框架和子框架加载页面时，重写默认的 `user-agent`。参考 [配置格式中的 `user-agent`](Manifest Format.md#user-agent)。

## webview 标签

新增了一些方法，参考 [webview 标签](webview Tag.md)
