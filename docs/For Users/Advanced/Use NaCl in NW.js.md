# NaCl
---

> **译者注：**
> 该技术已被 [弃用](https://developer.chrome.com/docs/native-client/migration/)，且于 2022-6 起 [不再支持](https://blog.chromium.org/2020/08/changes-to-chrome-app-support-timeline.html)。
> 事实上，该特性对于 NW.js 来说有些鸡肋，它能做的，Node 原生模块也都能做，实在没有太大的研究意义。
> 如果您想研究：
>
> - 教程中 NaCl SDK 的下载链接已失效，可以在 [这里](https://developer.chrome.com/docs/native-client/sdk/download/) 下载。
> - 参考 `nacl_sdk/sdk_tools/sdk_update_main.py` 的源码，拼接出 SDK 的 [远程配置链接](https://storage.googleapis.com/nativeclient-mirror/nacl/nacl_sdk/naclsdk_config.json)，貌似服务也已经挂了，可以在 [这里](https://developer.samsung.com/smarttv/develop/extension-libraries/nacl/download.html) 下载 Pepper 工具链，教程中的示例代码也在里面。

[TOC]

!!! note "SDK 版本特性"
    该特性只在 **SDK 版本** 上有效。参考 [构建风格](Build Flavors.md)。

NW.js 和 Chromium 一样支持 NaCl（Native Client）和 PNaCl（Portable Native Client）。您可以在应用中嵌入 NaCl 和 PNaCl。

!!! note "注意"
    以下教程引用自 [Chrome NaCl 文档](https://developer.chrome.com/native-client/devguide/tutorial/tutorial-part1)。

## 概述
本教程展示了如何使用 Portable Native Client（PNaCl）构建和运行一个 web 应用。这是一个客户端应用，使用了 HTML、JavaScript 和一个 C++ 编写的原生模块。PNaCl 工具链用于让页面中的 NaCl 模块可以直接运行。

在继续下面的教程之前，建议您先阅读 [Native Client 技术概述](https://developer.chrome.com/native-client/overview.html)。

### 本教程中的应用做了什么
本教程中的应用展示了如何在页面中加载一个 NaCl 模块，以及如何在 JavaScript 和 NaCl 模块之间发送消息。
在这个简单的应用中，JavaScript 发送了一条 `'hello'` 消息给 NaCl 模块。
当 NaCl 模块接收到这条消息时，它会检查是否是 `'hello'` 字符串，如果是，NaCl 模块就返回一条消息说 `'hello from NaCl'`。
JavaScript 接收到 NaCl 模块的消息后，会弹窗显示接收到的消息。

### JavaScript 和 NaCl 模块之间的通信
NaCl 编程模型支持 JavaScript 和 NaCl 模块之间的双向通信，两者都可以发起和响应消息。
所有的通信都是异步的：发送者（JavaScript 或 NaCl 模块）发送了一条消息，不会等待响应，类似于网页上客户端和服务器之间的通信，客户端向服务器发送了一个请求后就回来继续执行自己后面的逻辑。
NaCl 的通信系统是 Pepper API 的一部分，[开发者指南：通信系统](https://developer.chrome.com/native-client/devguide/coding/message-system.html) 中有更详细的说明。类似于 JavaScript 中 [web workers](http://en.wikipedia.org/wiki/Web_worker) 和主文档之间的交互方式。

## 步骤 1：下载并安装 NaCl SDK
根据 [下载](https://developer.chrome.com/native-client/sdk/download.html) 里的介绍，下载并安装 NaCl SDK。

## 步骤 2：启动一个本地服务器
为了模拟生产环境，SDK 提供了一个简单的 web 服务器，可以通过 `localhost` 访问。运行 Makefile 规则 `serve` 来调用它：

```bash
$ cd pepper_$(VERSION)/getting_started
$ make serve
```

!!! tip "提示"
    SDK中可能包含多个 Chrome/Pepper 版本（参考 [版本信息](https://developer.chrome.com/native-client/version.html)），可以通过上面示例中的 `pepper_$(VERSION)` 指定调用的版本，如 `pepper_37`。如果不知道要使用哪个版本，使用 `naclsdk list` 命令里标为 `stable` 的版本。参考 [下载 NaCl SDK](https://developer.chrome.com/native-client/sdk/download.html)。

如果没有指定端口号，服务器的默认端口为 5103，可以通过 http://localhost:5103 访问。

任何服务器都可以用于开发，SDK 只是提供了一个便捷的方式，不是必须的。

## 步骤 3：设置 Chrome 浏览器
Chrome 的 PNaCl 默认是开启的，建议使用与 SDK 包版本相同或更高的 Chrome 版本，低版本的 PNaCl 模块能在高版本的 Chrome 正常工作，但反过来却不一定。

!!! tip "提示"
    可以在地址栏中输入 `about:chrome` 来查看Chrome 的版本号。

为了有更好的开发体验，建议关闭 Chrome 的缓存。Chrome 会积极缓存资源，开发过程中，关闭缓存能确保加载的是最新的 NaCl 模块。

* 通过 ![菜单图标](https://developer.chrome.com/native-client/images/menu-icon.png) &gt; `更多工具` &gt; `开发者工具` 打开 Chorme 的开发者工具。
* 点击开发者工具窗口右上角的 ![齿轮图标](https://developer.chrome.com/native-client/images/gear-icon.png) 打开设置面板。
* 在 `偏好设置` 页签中，将 `网络`  &gt; `停用缓存（在开发者工具已打开时）` 勾选上。
* 在开发 NaCl 应用的过程中，始终打开开发者工具面板。

## 步骤 4：教程代码
本教程的代码在 SDK 里，路径为 `pepper_$(VERSION)/getting_started/part1`，包含有以下文件：

* `index.html`：包含页面布局、与 NaCl 模块交互的 JavaScript 代码。
NaCl 模块通过一个指向配置文件的 `<embed>` 标签引入页面里。
* `hello_tutorial.nmf`：一个配置文件，用于将 HTML 指向 NaCl 模块，并可以给 Chrome 的 PNaCl translator 提供附加的命令。
* `hello_tutorial.cc`: NaCl 模块的 C++ 代码。
* `MakeFile`: 编译命令，用于将 `hello_tutorial.cc` 中的 C++ 代码构建为 **pexe**（便携可执行文件）。

可以先看下这些文件，里面有很多解释其结构和内容的注释。有关 NaCl 应用结构的更多信息，请参考 [应用结构](https://developer.chrome.com/native-client/devguide/coding/application-structure.html)。

教程代码故意写得很简单，C++ 代码除了正确初始化，并没有做其他事情，JavaScript 代码等待 NaCl 模块的加载，并将页面中的状态文本修改为相应的信息。

## 步骤 5：编译 NaCl 模块并运行应用
执行 `make` 命令来编译 NaCl 模块：

```bash
$ cd pepper_$(VERSION)/getting_started/part1
$ make
```

因为示例包含在 SDK 目录结构中，所以 Makefile 知道如何找到 PNaCl 工具链并使用它来构建模块。如果您在 NaCl SDK 目录外部构建应用，需要设置 `$NACL_SDK_ROOT` 环境变量。参考 [构建 NaCl 模块](https://developer.chrome.com/native-client/devguide/devcycle/building.html)。

假设已按照 [步骤 2](#步骤-2启动一个本地服务器) 中的说明启动了本地服务器，您现在可以在 Chrome 中打开 http://localhost:5103/part1 来加载示例，如果 Native Client 模块成功加载，状态文本会从 “LOADING...” 变为 “SUCCESS”。如果遇到问题，请查看下面的 [排查故障](#排查故障) 部分。

## 步骤 6：修改 JavaScript 代码来给 NaCl 发送一条消息
该步骤中，您将修改页面文件（`index.html`），让页面在 NaCl 模块加载完成之后，向它发送一条消息。

找到 JavaScript 函数 `moduleDidLoad()`，添加发送 ‘hello’ 信息的代码：

```javascript
function moduleDidLoad() {
  HelloTutorialModule = document.getElementById('hello_tutorial');
  updateStatus('SUCCESS');
  // 向 NaCl 模块发送一条消息
  HelloTutorialModule.postMessage('hello');
}
```

## 步骤 7:  NaCl 实现信息处理
该步骤中，您将修改 NaCl 模块（`hello_tutorial.cc`），让它可以接收来自 JavaScript 发送的信息，并做出相应，您将：

* 实现模块实例的 `HandleMessage()` 成员函数。
* 使用 `PostMessage()` 成员函数向 JavaScript 发送一条信息。

首先，定义 NaCl 模块中需要用到的变量（期望从 JavaScript 接收到的 ‘hello’ 字符串，以及作为相应返回给 JavaScript 的字符串）。在 `hello_tutorial.cc` 文件中，在 #include 代码快后面加入以下代码：

```c++
namespace {
// 期望浏览器发送的字符串
const char* const kHelloString = "hello";
// 如果接收到的消息包含 "hello"，发回给浏览器的字符串
const char* const kReplyString = "hello from NaCl";
} // 命名空间
```

现在，实现 `HandleMessage()` 成员函数来检查`kHelloString` 和返回 `kReplyString`。找到下面这一行：

```c++
// TODO(sdk_user): 1. Make this function handle the incoming message.
给成员函数添加以下代码：

virtual void HandleMessage(const pp::Var& var_message) {
  if (!var_message.is_string())
    return;
  std::string message = var_message.AsString();
  pp::Var var_reply;
  if (message == kHelloString) {
    var_reply = pp::Var(kReplyString);
    PostMessage(var_reply);
  }
}
```

参考 Pepper API 文档，查看更多关于成员方法 [pp::Instance.HandleMessage](https://developer.chrome.com/native-client/pepper_stable/cpp/classpp_1_1_instance.html#a5dce8c8b36b1df7cfcc12e42397a35e8) 和 [pp::Instance.PostMessage](https://developer.chrome.com/native-client/pepper_stable/cpp/classpp_1_1_instance.html#a67e888a4e4e23effe7a09625e73ecae9) 的信息。

## 步骤 8：编译 NaCl 模块并再次运行应用

1. 再次运行 `make` 命令编译 NaCl 模块。
2. 运行 `make serve` 启动 SDK 服务器。
3. 在 Chrome 中重新加载 http://localhost:5103/part1 来重新运行应用。

在 Chrome 加载 NaCl 模块后，您将看到来自模块发送的消息。

## 问题排查

如果应用不能运行，查看上面的 [步骤 3](#步骤-3设置-Chrome-浏览器)，检查环境配置是否正确，包括浏览器和本地服务器。确保当前运行的 Chrome 版本大于或等于使用的 SDK 包版本。

另一个有用的调试辅助工具是 Chrome 的 JavaScript 控制台，查看它的输出来获取相关线索。例如，如果出现了“NaCl module crashed”（NaCl 模块崩溃）信息，则可能是 NaCl 模块存在错误，可能需要 [调试](https://developer.chrome.com/native-client/devguide/devcycle/debugging.html)。

文档中又更多关于问题排查的信息：

* [问题排查 FAQ](https://developer.chrome.com/native-client/faq.html#faq-troubleshooting)
* [Progress Events](https://developer.chrome.com/native-client/devguide/coding/progress-events.html) 文档包含了一些关于处理错误事件的有用信息。

## 下一步

* 查看开发者指南中 [应用结构](https://developer.chrome.com/native-client/devguide/coding/application-structure.html) 部分，获取关于如何构建 NaCl 模块的相关说明。
* 查看 [C++ 参考](https://developer.chrome.com/native-client/pepper_stable/cpp)，获取关于如何使用 Pepper API 的详细信息。
* 查看 SDK examples 目录中的示例代码，学习使用Pepper API 编写 NaCl 应用的其他技术。
* 查看 [构建](https://developer.chrome.com/native-client/devguide/devcycle/building.html)、[运行](https://developer.chrome.com/native-client/devguide/devcycle/running.html)和 [调试页面](https://developer.chrome.com/native-client/devguide/devcycle/debugging.html) 查看关于如何完成构建、运行以及调试 NaCl 应用的相关信息。
* 查看 [naclports](http://code.google.com/p/naclports/) 项目，获取 NaCl 可以使用哪些移植的库。如果您移植了一个开源库给自己使用，我们推荐您将它添加到 naclports（查看 [如何添加代码到 naclports](http://code.google.com/p/naclports/wiki/HowTo_Checkin)）。

_以上内容适用 [CC-By 3.0 license](http://creativecommons.org/licenses/by/3.0/)_
