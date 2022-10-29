# NW.js 里的安全性
---

[TOC]

## Node 框架与普通框架

> **译者注：**这里的框架指窗口页面或内部的 iframe

NW.js 里有两种框架：Node 框架和普通框架。

**Node 框架** 比 **普通框架** 多了以下功能：

* 访问 Node.js / NW.js 的 API
* 访问被 NW.js 扩展的 DOM 特性，如 [另存为对话框](../../References/Changes to DOM.md#属性nwsaveas)、[nwUserAgent 属性](../../References/Changes to DOM.md#属性nwuseragent) 等。
* 绕过安全限制，比如沙箱、同源策略等。例如，您可以向任何远程站点发送跨域的 XHR 请求，或者访问 `src` 指向远程站点的 `<iframe>` 元素。

在 NW.js 中，符合以下 **所有** 标准框架将是一个 **Node 框架**：

* [配置文件](../../References/Manifest Format.md#nodejs) 中的 `nodejs` 设置为 `true`
* 窗口或框架的 URL 符合 [配置文件](../../References/Manifest Format.md#node-remote) 中的 `node-remote` 的匹配规则，或者是 `chrome-extension://` 协议。
* 框架和父框架 **没有** [`nwdisable` 属性](../../References/Changes to DOM.md#属性nwdisable)
* 框架和父框架 **不在** [`<webview>` 标签](../../References/webview Tag.md) 内
