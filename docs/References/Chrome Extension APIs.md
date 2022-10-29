# Chrome 扩展 API {: .doctitle}
---

NW.js 支持所有 [Chrome 应用平台](https://developer.chrome.com/apps/api_index) 的 chrome.* API。

NW.js 也支持部分 [扩展平台](https://developer.chrome.com/extensions/api_index) 的 chrome.* API，参考下面的文档列表：

* [chrome.contentSettings](https://developer.chrome.com/extensions/contentSettings)：用于控制通知的设置
* [chrome.tabs](https://developer.chrome.com/extensions/tabs)：用于完全支持开发工具扩展
* [chrome.proxy](https://developer.chrome.com/extensions/proxy)：管理代理设置
* [chrome.cookies](https://developer.chrome.com/extensions/cookies)：方法 `get`、`getAll`、`set`、`remove` 支持带有 webview storeId 的 webview （`webview.getCookieStoreId()`）