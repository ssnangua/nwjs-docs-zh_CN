# 运行应用 {: .doctitle}
---

[TOC]

> 原文链接：https://github.com/nwjs/nw.js/wiki/How-to-run-apps
> 译者根据新版本的 NW.js 对内容做了相应调整

_因为我们的打包系统类似于 [LÖVE](https://love2d.org)，下面的指南基于它的 [Wiki](https://love2d.org/wiki/Getting_Started)修改。_

NW.js 可以通过两种方式来加载应用：

* 从一个文件夹启动：启动路径指定为该文件夹
* 从一个 `.nw` 文件（一个重命名的 .zip 文件）启动：启动路径指定为该文件

不管是哪种方式，都需要有一个名为 `package.json` 的配置文件（在应用文件夹或 `.nw` 压缩包中），NW.js 应用启动时会解析该文件，如果找不到，则该文件夹或 `.nw` 文件将不会被识别为应用，NW.js 会显示一个默认界面。

一个常见的错误操作是压缩 `.nw` 文件时，不是压缩应用文件，而是压缩了文件夹。这是习惯性的操作，因为不希望解压的时候在当前目录出现很多零散的文件。但作为 NW.js 的启动文件，您应该直接打包应用文件而不是外层目录，才能获得正确的 `.nw`（即 `package.json` 文件在 zip 包的最外层）。

## 全平台

将 NW.js 的文件与应用文件（包括 `package.json`）放在同一目录中，然后运行 `​​nw` 可执行文件。

## Windows

在 Windows 上，最简单的运行方式是将应用文件夹拖放到 `nw.exe` 或它的快捷方式上。注意是拖放包含 `package.json` 的文件夹而不是 `package.json` 文件自身。

也可以通过命令行来运行：

    C:\nw\nw.exe C:\apps\myapp
    C:\nw\nw.exe C:\apps\packagedapp.nw

还可以：

1. 复制命令：`C:\nw\nw.exe "C:\apps\myapp"`
2. 在文件夹中或桌面上右键
3. 新建 > 快捷方式
4. 粘贴 > 下一步
5. 输入应用名 > 完成

现在，您只需要双击这个快捷方式就能运行您的应用了。

> **译者注：**还有一种很实用的方式，将 `.nw` 文件的打开方式指定为 `nw.exe`，这样双击 `.nw` 文件就能直接启动应用了。

## Linux

在 Linux 上，可以使用下面的命令行：

    nw /home/path/to/appdir/
    nw /home/path/to/packagedapp.nw

如果您安装了 `.deb`，也可以在文件管理器中双击 .nw 文件。

如果出现以下错误：

    nw: error while loading shared libraries: libudev.so.0: cannot open shared object file: No such file or directory

请参考 [#136](https://github.com/nwjs/nw.js/issues/136) 和 [#770](https://github.com/nwjs/nw.js/issues/770)。

## Mac OS X

在 Mac OSX 上，可以将文件夹或 `.nw` 文件拖放到 nw.app 应用包上来运行应用。

> **译者注：**译者尝试拖放的方式始终不成功，会变成移动文件，不知是系统问题还是需要设置哪里

在 Mac OSX 命令行终端中，可以这样来使用 nw（假设它已经安装到应用程序目录）：

    open -n -a nwjs --args "/home/path/to/app"

在某些情况下，通过以下方式直接调用应用包内的 nw 二进制文件可能会更快：

    /Applications/nwjs.app/Contents/MacOS/nwjs myapp

您也可以给终端会话中的 nw 二进制文件设置一个别名，以便于调用，方法是将别名添加到 `~/.bash_profile`：

    open -a TextEdit ~/.bash_profile
    
    # 设置 nw 的别名
    alias nw="/Applications/nwjs.app/Contents/MacOS/nwjs"

别忘了刷新 bash shell 环境：

    source ~/.bash_profile

现在，您可以像 Linux 和 Windows 中一样，在命令中中执行 nw 了：

    nw "/home/path/to/game"

> **译者注：**一种不用命令行启动的方式，在 `nwjs.app/Contents/Resources/` 目录中创建一个名为 `app.nw` 的文件夹，将 `package.json` 等应用文件放进去，然后双击 `nwjs.app` 启动应用即可。

## 使用 npm `nw` 安装器

[npmjs.com 上的 nw 安装器](https://www.npmjs.com/package/nw) 和 [github 上的 nw 安装器](https://github.com/nwjs/npm-installer)

全局安装：
```
sudo npm install -g nw
nw ./my_nwjs_app
```

局部安装：
```
npm install nw
node_modules/.bin/nw ./my_nwjs_app
```
