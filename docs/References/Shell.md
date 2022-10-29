# Shell（桌面操作） {: .doctitle}
---

[TOC]

`Shell` 是执行桌面相关操作的 API 集合。

## 示例

```javascript
// 在默认浏览器中打开指定的 URL
nw.Shell.openExternal('https://github.com/nwjs/nw.js');

// 在默认文本编辑器中打开指定的文本文件
nw.Shell.openItem('test.txt');

// 在文件管理器中打开指定文件所在目录，并选中该文件
nw.Shell.showItemInFolder('test.txt');
```

## Shell.openExternal(uri)

* `uri` `String` 要打开的 URL

以系统的默认方式打开指定链接。例如，`mailto:` 链接会以默认的邮件方式打开。

## Shell.openItem(file_path)

* `file_path` `{String}` 本地文件路径

以系统的默认方式打开指定的文件。

## Shell.showItemInFolder(file_path)

* `file_path` `{String}` 本地文件路径

在文件管理器打开指定文件所在目录，并选中该文件。