# 构建风格 {: .doctitle}
---

[TOC]

NW.js 支持不同的构建风格，以减小应用包大小。目前支持以下构建风格：

* SDK包：内置开发工具和 NaCl 插件支持。
* 普通包：最小的应用包，不支持开发工具 和 NaCl 插件。

代码中可以通过 `process.versions['nw-flavor']` 来查看当前应用是哪种构建风格。 

参考 [构建 NW.js](../../For Developers/Building NW.js.md#构建风格) 查看如何从源码构建自己的 NW.js 版本。

