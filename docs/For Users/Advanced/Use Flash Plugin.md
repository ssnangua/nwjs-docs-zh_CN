# 使用 Flash 插件 {: .doctitle}
---

[TOC]

> **译者注：**
> 该特性已失效，Chrome 88（2021-1-19）开始便 [**不再支持 Flash**](https://support.google.com/chrome/a/answer/10314655#87)，事实上 `chrome://plugins` 页面早在 2016-5-30 就已 [**移除**](https://bugs.chromium.org/p/chromium/issues/detail?id=615738)。
> 译者尝试将 [nw.js 仓库](https://github.com/nwjs/nw.js) 下的 `/test/data/PepperFlash/` 复制到新版本的 NW.js 里，然后打开记忆中的 4399，确定已经不支持 Flash。

NW.js 支持 Pepper Flash 插件，只需要将插件文件（包括它的 `manifest.json`）放入 NW.js 可执行文件同目录下的名为 `PepperFlash` 的子目录中。
Mac 系统中，`PepperFlash` 子目录需要放入 framework 包内的 `Internet Plug-Ins` 目录中。

加载的 flash 插件可以通过 `/path/to/nwjs --url=chrome://plugins` 来验证。
