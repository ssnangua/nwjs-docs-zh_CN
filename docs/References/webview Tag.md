# &lt;webview&gt; 节点
---

[TOC]

在应用中，可以通过 `<webview>` 节点来嵌入第三方内容（如网页）。与 `<iframe>` 不同，`<webview>` 运行在一个单独的进程中，没有主应用的权限，且和主应用之间的所有交互都是异步的，可以避免主应用受到嵌入内容的影响，保证主应用的安全性。

## 示例

要在应用中嵌入一个网页，只需要给页面添加一个 webview 节点，再加上指向目标网页的 src 和用于控制容器外观的 css 样式：

```html
<webview id="foo" src="http://www.google.com/" style="width:640px; height:480px"></webview>
```

## 参考

参考 [Chrome 文档的 `<webview>` 标签](https://developer.chrome.com/docs/extensions/reference/webviewTag/) 查看详细的 API 参考列表。

除在上述的 API 外，NW.js 新增了以下方法：

### webview.showDevTools(show, [container])

* `show` `{boolean}` 打开或关闭开发者工具窗口
* `container` `{webview Element}` _Optional_ 用于显示开发者工具的 `<webview>` 元素。默认情况下，开发者工具会显示在一个新窗口中。

可以在嵌入的 webview 中使用开发者工具扩展，只需要通过 `--load-extensions` 来加载。容器 webview 需要可信任，在扩展的 `manifest.json` 中添加一个额外的 `webview` 配置，并给 webview 节点添加 `partition='trusted'` 属性。参考 [issue #6004](https://github.com/nwjs/nw.js/issues/6004) 里的例子。
```json
"webview": {
  "partitions": [
     {
       "name": "trusted",
       "accessible_resources": [
          "<all_urls>"
       ]
     }
  ]
}
```

### webview.inspectElementAt(x, y)

在通过 `webview.showDevTools()` 打开开发者工具后，该方法可以检查指定的 (x, y) 位置的元素。

### 在 webview 里加载本地文件

在配置里添加以下许可：
```json
  "webview": {
     "partitions": [
        {
          "name": "trusted",
          "accessible_resources": [ "<all_urls>" ]
        }
     ]
  }
```

然后给 webview 节点添加 `partition="trusted"` 属性。

### webview 里的 Node.js 支持

给 webview 节点添加 `allownw` 属性，就可以启用 webview 里的 Node.js 支持，无论它加载的是本地文件还是远程站点。谨慎使用此功能，因为不受信任的内容 webview 也会照常加载。

### 在真实环境中 executeScript

通过 [Chrome 的 executeScript 方法](https://developer.chrome.com/docs/extensions/reference/scripting/#method-executeScript) 可以在 webview 中注入 JS 代码，但却是在一个独立环境中。要访问目标 DOM 上下文中的 JS 对象，可以将代码注入到真实环境的上下文中。只需将 `{ mainWorld: true }` 添加到函数的 `InjectDetails` 类型参数里即可。

### webview 里的 Cookie 支持

webview 的 `getCookieStoreId()` 方法会返回 storeId，可用于 [chrome.cookies](https://developer.chrome.com/extensions/cookies) API。

##### 在控制台里的例子：
假设您有一个带有 webview 的 NW.js 应用。

在控制台里显示 http://docs.nwjs.io 的所有 cookie（您需要先访问这个页面，才会有一些 cookie）：
```js
chrome.cookies.getAll(
  {
    url: "http://docs.nwjs.io",
    storeId: webview.getCookieStoreId()
  },
  console.log.bind(console)
);
```
