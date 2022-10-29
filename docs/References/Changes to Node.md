# Node 的改动 {: .doctitle}
---

[TOC]

## console
因为 NW.js 支持 GUI 应用，而不是控制台应用，所以 `console.log()`、`console.warn()`、`console.error()` 等控制台输出的相关方法，会被重定向到 Chromium 的控制台，您可以在 [开发者工具](../For Users/Debugging with DevTools.md) 的 Console（控制台）面板中查看输出的信息。

## process
全局的 `process` 对象中，新增了两个属性：

* `process.versions['nw']` NW.js 的版本
* `process.versions['chromium']` Chromium 的版本
* `process.versions['nw-flavor']` SDK 版本为 'sdk'，普通版本为 'normal'
* `process.mainModule` 配置文件中 [`main`](Manifest Format.md#main) 字段设置的应用入口（如`index.html`）。如果配置文件中也设置了 [`node-main`](Manifest Format.md#node-main) 字段，则 `process.mainModule` 会指向 `node-main`。

## require

Node 的 `require()` 方法中的子文件（被 require 的文件）的相对路径，取决于父文件（调用 require 的文件）所运行的 [JavaScript 环境](../For Users/Advanced/JavaScript Contexts in NW.js.md)：

* 如果父文件运行在 Node 环境下，则子文件的相对路径将基于父文件
* 如果父文件运行在浏览器环境下，则子文件的相对路径将基于应用根目录，如配置文件所在的目录。
