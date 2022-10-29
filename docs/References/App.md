# App（应用） {: .doctitle}

---

[TOC]

## App.argv

获取启动应用程序时的（过滤后的）命令行参数。

在 NW.js 中，有一些命令行参数是 NW.js 内部使用的，您的应用程序可能用不到，`App.argv` 将过滤掉这些参数，并返回剩下的参数。

您也可以从 `App.filteredArgv` 中获取被过滤掉的参数，或者通过 `App.fullArgv` 获取全部参数。

## App.fullArgv

获取启动应用程序时的全部命令行参数。

返回的值包括 NW.js 使用的参数，如 `--nwapp`、`--remote-debugging-port` 等。

## App.filteredArgv

获取在 `App.argv` 中被过滤掉的命令行参数。

默认情况下，通过以下规则来过滤参数：

```javascript
[
  /^--url=/,
  /^--remote-debugging-port=/,
  /^--renderer-cmd-prefix=/,
  /^--nwapp=/,
];
```

## App.startPath

获取应用程序的启动目录。应用程序启动后，会将当前目录更改为主程序文件所在的目录。

## App.dataPath

获取在用户目录中应用数据的路径。

* Windows：`%LOCALAPPDATA%/<name>`
* Linux：`~/.config/<name>`
* OS X：`~/Library/Application Support/<name>/Default` (0.12.3 及之前的版本路径为 `~/Library/Application Support/<name>`)

`<name>`为 `package.json` 配置文件中的 **name** 字段。

## App.manifest

获取配置文件的 JSON 格式对象。

## App.clearCache()

清除内存中的 HTTP 缓存和磁盘上的缓存。该方法为同步调用。

## App.clearAppCache(manifest_url)

将 manifest_url 指定的应用程序缓存组标记为废弃的。该方法为同步调用。

## App.closeAllWindows()

发送 `close`（关闭）事件给当前应用的所有窗口，如果没有窗口阻塞关闭行为，应用将在所有窗口关闭之后退出。通过该方法退出应用，所有窗口都可以保存数据。

## App.crashBrowser()
## App.crashRenderer()

这 2 个方法分别使浏览器进程和渲染器进程崩溃，用于测试 [崩溃机制](../For Developers/Understanding Crash Dump.md) 特性。

## App.enableComponent(component, callback)

!!! warning "实验性"
    实验性 API，预期行为可能会在未来发生变更。

* `component` `{String}` 组件ID，目前只支持 `WIDEVINE`。
* `callback` `function(version)` 组件启用后的回调，`version` 字符串参数为被启用的组件的版本，'0.0.0.0' 表示未安装。可通过 `App.updateComponent()` 进行安装。

## App.getProxyForURL(url)

* `url` `{String}` 要查询代理的 URL

查询用于在 DOM 中加载 `url` 的代理。返回值与 [PAC](http://en.wikipedia.org/wiki/Proxy_auto-config) 中使用的格式相同（如："DIRECT"、"PROXY localhost:8080"）。

## App.setProxyConfig(config)

* `config` `{String}` 代理规则
* `pac_url` `{String}` PAC 链接

设置 web 引擎用于请求网络资源的代理配置，或指定 PAC 链接以自动检测代理。

规则（拷贝自 [`net/proxy/proxy_config.h`](https://github.com/nwjs/chromium.src/blob/nw13/net/proxy/proxy_config.h)）

```
    // 从字符串中解析规则，指示要使用的代理。
    //
    //   proxy-uri = [<proxy-scheme>"://"]<proxy-host>[":"<proxy-port>]
    //
    //   proxy-uri-list = <proxy-uri>[","<proxy-uri-list>]
    //
    //   url-scheme = "http" | "https" | "ftp" | "socks"
    //
    //   scheme-proxies = [<url-scheme>"="]<proxy-uri-list>
    //
    //   proxy-rules = scheme-proxies[";"<scheme-proxies>]
    //
    // 使用 proxy-rules，字符串为一个以分号分隔的有序代理列表，应用于特定的 URL 协议。
    // 如果 proxy-uri 未指定 proxy-scheme，默认为 http。
    //
    // 特殊情况：
    //  * 如果第一个代理列表中省略了协议，则该列表将应用于所有的 URL 协议，且后续列表将被忽略。
    //  * 在指定了协议的代理列表后面，任何省略了协议的列表都将被忽略。
    //  * 如果 url-scheme 设置为 'socks'，则代理列表中未指明协议的，都将默认为 socks4://。
    //
    // 示例：
    //   "http=foopy:80;ftp=foopy2"  -- http:// 的 URL 使用HTTP代理 "foopy:80"，ftp:// 的 URL 使用HTTP代理 "foopy2:80"。
    //   "foopy:80"                  -- 所有 URL 使用HTTP代理 "foopy:80"。
    //   "foopy:80,bar,direct://"    -- 所有 URL 使用HTTP代理 "foopy:80"，如果 "foopy:80" 不可用，则使用 "bar"，直至没有可用代理为止。
    //   "socks4://foopy"            -- 所有 URL 使用SOCKS4代理 "foopy:1080"。
    //   "http=foopy,socks5://bar.com" -- http:// 的 URL 使用HTTP代理 "foopy"，如果 "foopy" 不可用，则使用SOCKS5代理 "bar.com"。
    //   "http=foopy,direct://"      -- http:// 的 URL 使用HTTP代理 "foopy"，如果 "foopy" 不可用，则不使用代理
    //   "http=foopy;socks=foopy2"   -- http:// 的 URL 使用HTTP代理 "foopy"，其他 URL 使用SOCKS4代理 "foopy2"

```

## App.quit()

退出当前应用。该方法**不会**发送 `close` 事件给窗口，应用将静默退出。

## App.setCrashDumpDir(dir)

!!! warning "已弃用"
    从 0.11.0 版本开始，该 API 已被弃用。

- `dir` `{String}` 生成崩溃转储的路径

设置崩溃时保存 minidump 文件的目录。更多详细信息，参考 [崩溃机制](../For Developers/Understanding Crash Dump.md)。

## App.addOriginAccessWhitelistEntry(sourceOrigin, destinationProtocol, destinationHost, allowDestinationSubdomains)

- `sourceOrigin` `{String}` 源地址，如 `http://github.com/`
- `destinationProtocol` `{String}` 允许 `sourceOrigin` 访问的目标协议，如 `app`
- `destinationHost` `{String}` 允许 `sourceOrigin` 访问的目标主机，如 `myapp`
- `allowDestinationSubdomains` `{Boolean}` 是否允许 `sourceOrigin` 访问目标地址的子域

在跨域访问白名单中添加一个条目。

例如，想要允许 HTTP 从 github.com 重定向到您的应用程序页面，可以使用以下代码：

```javascript
App.addOriginAccessWhitelistEntry(
  "http://github.com/",
  "chrome-extension",
  location.host,
  true
);
```

如果想要移除添加的白名单，可使用 `App.removeOriginAccessWhitelistEntry` 方法，并传入完全相同的参数。

## App.removeOriginAccessWhitelistEntry(sourceOrigin, destinationProtocol, destinationHost, allowDestinationSubdomains)

- `sourceOrigin` `{String}` 源地址，如 `http://github.com/`
- `destinationProtocol` `{String}` 允许 `sourceOrigin` 访问的目标协议，如 `app`
- `destinationHost` `{String}` 允许 `sourceOrigin` 访问的目标主机，如 `myapp`
- `allowDestinationSubdomains` `{Boolean}` 是否允许 `sourceOrigin` 访问目标地址的子域

移除跨域访问白名单中指定的条目。参考上面的 `addOriginAccessWhitelistEntry` 方法。

## App.registerGlobalHotKey(shortcut)

- `shortcut` `{Shortcut}` 要注册快捷键的 `Shortcut` 对象。

注册一个全局快捷键（系统级别的热键）。

更多详细信息，参考 [Shortcut](Shortcut.md)（快捷键）。

## App.unregisterGlobalHotKey(shortcut)

- `shortcut` `{Shortcut}` 要注销快捷键的 `Shortcut` 对象。

注销一个全局快捷键。

更多详细信息，参考 [Shortcut](Shortcut.md)（快捷键）。

## App.updateComponent(component, callback)

!!! warning "实验性"
    实验性 API，预期行为可能会在未来发生变更。

* `component` `{String}` 组件ID，目前只支持 `WIDEVINE`。
* `callback` `function(success)` 组件更新后的回调，`success` 是一个布尔值，表示更新的结果。

## 事件：open(args)

- `args` `{String}` 程序的完整命令行

用户启动一个新的应用实例时触发。

## 事件：reopen

Mac 专有特性。当用户点击 dock 上运行中的应用程序图标时触发。
