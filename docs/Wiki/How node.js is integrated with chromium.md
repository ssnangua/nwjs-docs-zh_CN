# Node 集成到 Chromium 的原理 {: .doctitle}
---

> 原文链接：https://github.com/nwjs/nw.js/wiki/How-node.js-is-integrated-with-chromium
> 参考源码：https://github.com/nwjs/chromium.src（WebKit 相关的可以尝试在 blink 下找）

## 概述

我们在 Node 和 Chromium 上保持最小的修改，只做了两件事：**主循环集成** 和 **上下文桥接**。

`Node` 和 `Chromium` 都有自己的主循环。所以要让 `Node` 在 `Chromium` 中运行需要一些努力。 NW.js 的目标特性之一是可以在 DOM 中 **直接** 调用 Node 函数，所以我们将它们集成到同一个线程中，这需要集成 Node 的主循环和 Chromium 的渲染进程循环。

为了使 Node 和 DOM 中的对象能互相访问，Node 的 V8 引擎被设置为 Chromium 的 V8 引擎，2 个环境中的对象处于不同的 `contexts`（上下文）中，以保证命名空间互不干扰。

## 主循环集成

`Chromium` 通过 `MessageLoop` 和 `MessagePump` 实现事件循环，而 `Node` 使用的是 `libuv`。所以我们需要以 `libuv` 作为底层事件库、为 `Chromium` 实现一个新的 `MessagePump`，这样它既是 `Chromium` 事件循环的一部分，又能通过 `libuv` 和 `Node` 通信，从而打通两个事件循环。

参考 `base/message_pump_uv.cc`、`base/message_pump_uv.h`、`base/message_loop.cc` 和 `base/message_loop.h` 了解我们的实现过程。

Mac 上的实现又有许多差异，参考 `base/message_pump_mac.h` 和 `base/message_pump_mac.mm`。

## 上下文桥接：将 Node 的 Symbol 插入到 Chromium 中

这是 NW.js 最重要也是最棘手的部分，首先我们为 `Node` 初始化一个上下文，并让 `Node` 在里面创建它的所有东西（参考 `content/renderer/renderer_main.cc`），然后在 `Chromium` 将 DOM 安装到它的上下文中时，我们将 Node 上下文中的所有东西移动到 `Chromium` 的上下文中（参考 `third_party/WebKit/Source/WebCore/bindings/v8/V8DOMWindowShell.cpp` 第 346 行）。

## 让 Node 从渲染进程 Start

`Node` 有一个 `Start` 函数用来创建所有的东西，并将它们重定向到自己的事件循环中，我们将 `Start` 函数拆分成了几个部分（参考 `third_party/node/src/node.cc`）。

`Node` 会执行一个脚本文件（`third_party/node/src/node.js`），其中有很多处理*脚本执行*的逻辑，我们对该部分进行了大量的修改，并通过一些技巧，让它的模块系统运行在 NW.js 下。

另一个修改位于 `third_party/WebKit/Source/WebCore/bindings/v8/V8DOMWindowShell.cpp` 第 363 行，我们在 `Chromium` 初始化后插入了一些脚本，这样 `Node` 就可以获得正确的 `pathname`（路径名）和 `filename`（文件名），这对 `require` 功能至关重要。
