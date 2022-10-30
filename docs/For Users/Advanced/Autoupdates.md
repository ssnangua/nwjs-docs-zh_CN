# 自更新 {: .doctitle}
---

[TOC]

NW.js 倾向于支持社区产出的自更新解决方案，而不是内置一个。以下是已有的值得参考的解决方案：

- [node-webkit-updater](https://github.com/edjafarov/node-webkit-updater)（by [@edjafarov](https://github.com/edjafarov)）
- [nwjs-autoupdater](https://github.com/oaleynik/nwjs-autoupdater)（by [@oaleynik](https://github.com/oaleynik)）
- [nw-autoupdater](https://github.com/dsheiko/nw-autoupdater)（by [@dsheiko](https://github.com/dsheiko)）

## node-webkit-updater

> **译者注：**
> 不建议使用。如果不在意增量更新（或自己实现），可以用下面的 [nw-autoupdater](#nw-autoupdater) 包，与 node-webkit-updater 一脉相承，不过做了一些扩展，功能更完善。
> 
> - 作者已不再维护（最后一次提交是 2016-6-3）
> - 不支持增量更新（需要从新包启动，所以只能整包下载）

提供了一些基础 API，可以实现：

- 检查远程配置文件和本地配置文件中的版本号
- 如果版本号不同，下载远程新包到临时目录
- 解压临时目录里的新包
- 从临时目录启动新版本应用（此时作为更新程序），并退出正在运行的旧版本
- 更新程序将新版本文件复制到旧版本的安装目录，覆盖旧版本
- 更新程序从安装目录启动新版本，然后退出自身

您需要参考 [简单示例](https://github.com/edjafarov/node-webkit-updater/blob/master/examples/basic.js) 自己实现这个逻辑。

## nwjs-autoupdater

> **译者注：**
> 需要搞 go 环境，自己在各个平台下编译相应版本的可执行文件（例如在 windows 系统下编译一个 updater.exe）。这个可执行文件其实只做 2 件事：解压更新包并覆盖到应用的安装目录，然后重新启动应用。它只是作为更新逻辑的依赖，完成更新流程最后的两个步骤，而检查更新、下载更新包、以及调用这个可执行文件等逻辑，都要自己实现（可以在 [examples](https://github.com/oaleynik/nwjs-autoupdater) 基础上修改）。
> 这其实算是一个通用的解决方案，不管是什么语言编写的应用，都可以通过这个逻辑实现间接的覆盖更新。
> 应该已不再维护（最后一次提交是 2019-5-7）。
> 优点：
>
> - 更新程序本身就是一个独立的应用，执行过程不依赖目标应用，不用考虑进程占用等情况
> - 支持增量更新
>
> 缺点：
>
> - 需要搞 go 环境，自己编译可执行文件，有一定门槛，出现问题也更难解决

小巧的 go 语言应用（构建后只有大约 2MB），可以打包进 NW.js 应用，用于实现自更新。
要更新目标应用，更新程序（updater）需要知道两件事：

- 新版本的 zip 包在哪里（用于实现覆盖更新）
- 应用的可执行文件在哪里（用于重启应用）

可以通过命令行参数 `--bundle` 和 `--inst-dir` 将它们传递给更新程序：

- `--bundle` 是新版本 zip 包的路径
- `--inst-dir` 是应用的可执行文件路径

相较于 `node-webkit-updater` 有几点优势：

- 更新程序自身可升级
- 不需要提升权限（除非要升级的应用本身需要更高的权限）
- 更新包更小，因为不需要打包整个新的 NW.js 应用

检查更新的逻辑也需要自行实现，这个 [例子](https://github.com/oaleynik/nwjs-autoupdater/blob/master/examples/index.js) 展示了如何使用 js 文件作为 NW.js 应用的入口并在后台检查更新。

## nw-autoupdater

> **译者注：**
> 
> - 作者已不再维护（最后一次提交是 2021-5-7）
> - 可以看成是上面的 [node-webkit-updater](#node-webkit-updater) 的升级版，原理一样，不过做了一些扩展，功能更完善
> - 不支持增量更新（需要从新包启动，所以只能整包下载）

提供了和 `node-webkit-updater` 类似的 API，但经过扩展，适配 Node 7.x 版本的 NW.js，并完全基于 async/await 语法。可以实现：

- 获取远程服务器上的配置文件
- 检查远程配置文件的版本号是否大于本地配置文件的版本号
- 根据远程配置文件中的 `packages` 映射表，下载匹配主机平台的最新有效版本到临时目录
    - 侦听下载进度事件
- 解压临时目录中的新包（zip 或 tar.gz）
    - 侦听安装进度事件
- 启动临时目录中的新版本，退出正在运行的旧版本
- 备份旧版本，然后替换为新版本
- 从安装目录重新启动更新后的应用

仓库里的 [example](https://github.com/dsheiko/nw-autoupdater) 包含了服务器和客户端的示例。
