# Mac 应用商店支持 {: .doctitle}
---

[TOC]

## 概述

您可以通过 Mac 应用商店或其他地方来分发您的 macOS 应用，但在分发之前，必须对应用进行签名。没有签名的应用启动时会被 [Gatekeeper](https://support.apple.com/en-us/HT202491) 拦截。

本文档将介绍如何给一个基于 NW.js 的 macOS 应用签名。

<span id="签名准备"></span>
## 签名准备

* 通过 [iTunesConnect](https://itunesconnect.apple.com) 创建一个 macOS 应用
* 从 [Apple Developer](https://developer.apple.com) 获取应用的证书并安装
    - 如果是通过 **Mac 应用商店** 分发：
        + 3rd Party Mac Developer Application: Foo (XXXXXXXXXX)
        + 3rd Party Mac Developer Installer: Foo (XXXXXXXXXX)
    - 如果是在 **其他地方** 分发：
        + Developer ID Application: Foo (XXXXXXXXXX)
        + Developer ID Installer: Foo (XXXXXXXXXX)

## 构建应用

从 [nwjs.io](http://dl.nwjs.io/v0.19.5-mas-beta/) 下载 NW.js MAS 版本，按 [打包与发布](../Package and Distribute.md) 中的说明进行构建。

## 给应用签名

`build_mas.py` 用来给 macOS 应用签名，传入 `--pkg` 参数，该脚本将在签名后生成一个可以上传到 Mac 应用商店的 `.pkg` 文件。

**基本使用**

```bash
python build_mas.py -C build.cfg -I myapp-dev.app -O MyApp.app
```

## 配置文件格式

配置文件（`build.cfg`）是一个可读性很好的文本文件，它包含了用于签名和打包应用的重要设置。

`ApplicationIdentity` 和 `InstallerIdentity` 是用于签名和打包应用的证书名。参考 [签名准备](#签名准备) 确定您要用哪个证书。

`NWTeamID` 用于建立 IPC 通道以启动基于 NW.js 的应用，它可以从 `Apple Developer -> Membership -> Team ID` 获取。

`ParentEntitlements` 和 `ChildEntitlements` 必须是有效的 [entitlements 文件](https://developer.apple.com/library/content/documentation/Miscellaneous/Reference/EntitlementKeyReference/Chapters/AboutEntitlements.html)。默认使用最小的权限进行签名，如下所示。

**entitlements-parent.plist**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>com.apple.security.app-sandbox</key>
  <true/>
  <key>com.apple.security.application-groups</key>
  <string>NWTeamID.your.app.bundle.id</string>
</dict>
</plist>
```

**entitlements-child.plist**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>com.apple.security.app-sandbox</key>
  <true/>
  <key>com.apple.security.inherit</key>
  <true/>
</dict>
</plist>
```

查看示例 `build.cfg` 查看所有字段的详细含义.