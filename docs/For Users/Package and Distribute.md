# 打包与发布 {: .doctitle}
---

[TOC]

本文档介绍如果打包和发布基于 NW.js 开发的应用。

## 快速开始

您可以使用下面的工具，将基于 NW.js 的应用自动打包为可发布的版本。

* [nwjs-builder-phoenix](https://github.com/evshiron/nwjs-builder-phoenix)（推荐）
* [nw-builder](https://github.com/nwjs-community/nw-builder)

或者也可以按照下面的说明，手动构建您的应用。

## 准备应用

打包前，您需要准备好所有必要的文件，检查下面的清单，确定没有任何遗漏：

* [ ] 源码和资源文件
* [ ] [配置文件](../References/Manifest Format.md#icon) 中使用到的图标文件
* [ ] 如果使用了第三方 NPM 模块，执行 `npm install` 来安装所有的依赖模块
* [ ] 如果使用了第三方原生模块，[重新构建使用到的原生模块](Advanced/Use Native Node Modules.md)
* [ ] 如果使用了 NaCl，[构建 NaCl](Advanced/Use NaCl in NW.js.md)
* [ ] 如果使用了源码加密技术，[编译源码](Advanced/Protect JavaScript Source Code.md)，然后移除源码文件

!!! warning "注意"
    `node_modules` 在一个平台运行正常，不代表在其他平台上也没有问题。例如一些模块在 Windows 和 Mac OS X 下会有不同的 `npm install` 命令。此外，有些模块还需要 python 环境才能完成安装，而 Windows 系统默认是没有安装 python 的，需要手动安装。
    经验告诉我们，**在每一个想要发布的平台上都执行一遍 `npm install`** 是非常必要的，可以确保所有的东西都能如预期的正常工作。 

!!! note "文件名和路径"
    在大多数 Linux 系统以及部分 Mac OS X 系统中，文件系统是 **大小写敏感** 的，既 `test.js` 和 `Test.js` 是不同的文件。确保应用中的路径和文件名大小写正确，否则应用在这些系统上可能会显示异常或直接崩溃。

!!! note "Windows 系统中的长路径"
    在 Windows 系统上，路径的最大长度为 260 个字符，如果超过这个长度，将导致各种构建错误。该问题通常是因为使用了低版本（<3.0）的 NPM 来安装依赖包。建议在硬盘根目录（如 `C:\build\`）中构建应用，以尽可能避免出现该问题。

## 准备 NW.js

您的应用需要包含构建好的 NW.js 才能运行起来。NW.js 提供了不同 [构建风格](Advanced/Build Flavors.md) 的版本，以满足不同的需求和包大小，选择适当的版本或者 [从源码构建自己的版本](../For Developers/Building NW.js.md)。

发布的应用包中必须包含下载的 NW.js 包中的所有文件，除了 SDK 版本的 `nwjc`、`payload` 和 `chromedriver`。

## 打包引用

有两种打包方式：原始应用文件，或应用文件压缩包

### 打包方式 1. 原始应用文件 (推荐)

在 Windows 和 Linux 系统下，可以将应用文件放到 NW.js 执行文件同目录下，然后将整个目录（或打包成压缩包）发送给您的用户。确保 `nw`（或 `nw.exe`）与 `package.json` 在同级目录下。或者将应用文件放入一个名为 `package.nw` 的文件夹中，然后将该文件夹放在 `nw`（或 `nw.exe`）的同级目录下。

在 Mac 系统下，将应用文件放入一个名为 `app.nw` 的文件夹中，然后将该文件夹放到 `nwjs.app/Contents/Resources/` 目录中即可。

推荐使用原始应用文件的方式来打包应用。

### 打包方式 2. 应用文件压缩包

您可以将所有的应用文件打包成一个 zip 文件：
在 Windows 和 Linux 系统中，重命名为 `package.nw`，然后放入 NW.js 执行文件的同级目录下。
在 Mac 系统中，重命名为 `app.nw`，然后放入 `nwjs.app/Contents/Resources/` 目录中。

!!! warning "包过大或文件太多会导致应用启动变慢"
    应用启动时，NW.js 会将应用包解压到一个临时目录中进行加载，所以如果包过大或者包里的文件太多，会导致应用启动变慢

在 Windows 和 Linux 系统中，您甚至可以通过将应用文件压缩包追加到 `nw`（或 `nw.exe`）上来隐藏它（合并成一个文件）。

在 Windows 系统上运行下面的命令来合并：
```batch
copy /b nw.exe+package.nw app.exe
```

在 Linux 系统上运行下面的命令：
```bash
cat nw app.nw > app && chmod +x app 
```

## 平台特定步骤

### Windows

您可以使用一些工具来替换 `nw.exe` 的图标，如 [Resource Hacker](http://www.angusj.com/resourcehacker/)、[nw-builder](https://github.com/mllrsohn/node-webkit-builder)、[node-winresourcer](https://github.com/felicienfrancois/node-winresourcer)。

您还可以创建一个安装程序来将所有必要的文件部署到用户的系统上，如 [Windows Installer](https://msdn.microsoft.com/en-us/library/cc185688(VS.85).aspx)、[NSIS](http://nsis.sourceforge.net/Main_Page)、[Inno Setup](http://www.jrsoftware.org/isinfo.php)。

### Linux

在 Linux 上，您需要创建一个正确的 [`.desktop` 文件](https://wiki.archlinux.org/index.php/Desktop_Entries)。

您可以创建一个自解压安装脚本，如 [`shar`](https://en.wikipedia.org/wiki/Shar)、[`makeself`](http://stephanepeter.com/makeself/)。

也可以通过包管理系统来分发应用，如 `apt`、`yum`、`pacman` 等，请参考它们的官方文档进行打包。

### Mac OS X

在 Mac OS X 系统中，您需要修改以下文件，让应用拥有自己的图标和 bundle id。

* `Contents/Resources/nw.icns`：应用图标。`nw.icns` 遵循 [苹果图标图片格式](https://en.wikipedia.org/wiki/Apple_Icon_Image_format)，可以使用 [Image2Icon](http://www.img2icnsapp.com/) 将 PNG/JPEG 图片转换为 ICNS 格式。
* `Contents/Info.plist`：包描述文件。参考 [实现 Cocoa 的标准关于面板](http://cocoadevcentral.com/articles/000071.php) 查看此文件如何影响应用，以及您应该修改哪些字段。

要重命名应用，需要修改以下字段：
* `Contents/Info.plist` - CFBundleDisplayName
* `Contents/Resources/en.lproj/InfoPlist.strings` - CFBundleDisplayName
* `Contents/MacOS/nwjs` - 将文件重命名为 `CFBundleExecutable` 的值，如果你改了它的话
* `package.json` - 添加一个新的字符串字段 `product_string` （如 foobar），帮助应用将显示为 'foobar Helper'。（自 v0.24.4 起）
  * `Contents/Frameworks/nwjs Framework.framework/Versions/n.n.n.n/nwjs Helper.app/Contents/MacOS/nwjs Helper.app` - 目录重命名为 'foobar Helper.app'，其他三个帮助应用也是
  * `Contents/Frameworks/nwjs Framework.framework/Versions/n.n.n.n/nwjs Helper.app/Contents/MacOS/nwjs Helper.app/Contents/Info.plist` - 修改 CFBundleDisplayName，其他三个帮助应用也是

如果想要用户可以在启用 Gatekeeper 的情况下正常启动应用，您还需要对 Mac 应用进行签名。参考 [Mac 应用商店支持](Advanced/Support for Mac App Store.md)。

## 参考

查看 [NW.js wiki](https://github.com/nwjs/nw.js/wiki/How-to-package-and-distribute-your-apps) 获取更多应用打包工具。
