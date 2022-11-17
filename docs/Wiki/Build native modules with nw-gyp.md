# 使用 nw-gyp 构建原生模块 {: .doctitle}
---

[TOC]

> 原文链接：https://github.com/nwjs/nw-gyp/blob/node-gyp-3.6.2/README.md
> 合并了 https://github.com/nwjs/nw.js/wiki/Build-native-modules-with-nw-gyp 的部分内容

## NW.js 的本地插件构建工具

`nw-gyp` 是 `node-gyp` 的一个定制版本，以支持 NW.js 特定的头文件和库，用于给 NW.js 构建原生模块。我们尝试为开发者提供更为流畅的构建方式，而不需要指定大量的命令行参数。

从 NW.js v0.3.2 开始支持，**目前需要手动指定 NW.js 的版本**。

**关于 Node 原生模块的使用，请参考 [使用 Node 原生模块](../For Users/Advanced/Use Native Node Modules.md#对于非LTS版本)。**

## 特性

* 易于使用，统一的接口
* 所有平台使用相同的命令来构建模块
* 支持构建 NW.js 的不同版本


## 安装

通过 `npm` 安装：

``` bash
$ npm install -g nw-gyp
```

您还需要安装：

* 在 Unix 上：
    * `python`（需要 `v2.7`，**不支持** `v3.x.x`）
    * `make`
    * 一个合适的 C/C++ 编译器工具链，如 [GCC](https://gcc.gnu.org)
* 在 Mac OS X 上：
    * `python`（需要 `v2.7`，**不支持** `v3.x.x`）（在 Mac OS X 上默认已安装）
    * [Xcode](https://developer.apple.com/xcode/download/)
        * 您还需要通过 Xcode 安装 `Command Line Tools`（命令行工具），可以通过这个方式找到它：`Xcode -> Preferences -> Locations`
        * 该步骤会安装 `gcc` 和包含 `make`相关的工具链
* 在 Windows 上：
    * 方式 1：通过 Microsoft 的 [windows-build-tools](https://github.com/felixrieseberg/windows-build-tools) 来安装所有必要的工具和配置。以管理员身份运行 PowerShell 或 CMD.exe，执行：
    `npm install --global --production windows-build-tools`
    * 方式 2：手动安装工具和配置：
        * Visual C++ 构建环境：
            * 方式 1：安装 [Visual C++ Build Tools](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools)（使用 “Visual C++ 生成工具” 工作负荷）
            * 方式 2：安装 [Visual Studio Community](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=Community)（使用 “使用 C++ 的桌面开发” 工作负荷）

        > 注意：Windows Vista/7 还需要安装 [.NET Framework 4.5.1](http://www.microsoft.com/en-us/download/details.aspx?id=40773)

        * 安装 [Python 2.7](https://www.python.org/downloads/)（**不支持** `v3.x.x`），然后运行 `npm config set python python2.7`（或参阅下文以获取关于指定正确的 Python 版本和路径的更多说明。）
        * 运行 `npm config set msvs_version 2015`

    如果上述步骤没有作用，请访问 [Microsoft's Node.js Guidelines for Windows](https://github.com/Microsoft/nodejs-guidelines/blob/master/windows-environment.md#compiling-native-addon-modules) 以获取更多提示。

如果您安装了多个版本的 Python，可以通过设置 `--python` 变量来指定 `nw-gyp` 要使用哪个版本：

``` bash
$ nw-gyp --python /path/to/python2.7
```

如果通过 `npm` 的方式调用 `nw-gyp`，并且您安装了多个版本的 Python，可以将 `npm` 的 `python` 配置设置为相应的值：

``` bash
$ npm config set python /path/to/executable/python2.7
```

注意，OS X 只是 Unix 的一种定制版本，所以也需要 `python`、`make` 和 C/C++。
获得这些工具和配置的一个简单方法是安装 Xcode，然后通过 Xcode 安装 `Command Line Tools`（命令行工具）。

## 如何使用

要编译原生模块，先进入模块目录：

``` bash
$ cd my_node_addon
```

下一步是使用 `configure` 命令为当前平台生成相应的项目构建文件：

``` bash
$ nw-gyp configure --target=<0.3.2 或其他 nw 版本>
```

该指令会下载并缓存指定的 NW.js 版本所需的头文件（v8.h 和 node.h）。

如果自动检测 Visual C++ Build Tools 2015 失败，添加 `--msvs_version=2015`：

``` bash
$ nw-gyp configure --msvs_version=2015
```

!!! note "注意"
    `configure` 步骤会在当前目录中查找要处理的 `binding.gyp` 文件。有关创建 `binding.gyp` 文件的说明请参阅下文。

现在，您的 `build/` 目录中会有一个 `Makefile` 文件（在 Unix 平台上）或 `vcxproj` 文件（在 Windows 上）。接下来执行 `build` 命令：

``` bash
$ nw-gyp build
```

你有了自己编译的 `.node` 文件啦！编译后的文件在 `build/Debug/` 或 `build/Release/` 中，具体取决于构建模式。现在，您可以在 NW.js 中使用 Node API 请求 `.node` 文件并运行您的测试了！

!!! note "注意"
    要创建一个 _Debug_ 构建文件，在执行 `configure`、`build` 和 `rebuild` 命令时，
    传入 `--debug`（或 `-d`）开关。

!!! note "注意"
    NW.js 打包的 V8 版本可能与 Node.js 内置的不同，这可能会导致在构建原生模块时出现一些不一致的行为（参考 [rvagg/nan#285](https://github.com/nodejs/nan/pull/285)）。


## `binding.gyp` 文件

以前当 node 有 `node-waf` 时，你需要编写一个 `wscript` 文件。现在作为替代的是 `binding.gyp` 文件，它以类 JSON 的格式描述了构建模块的配置。该文件与 `package.json` 文件一起被放置在包的根目录中。

一个用于构建 node 插件的最简单的 `gyp` 文件如下：

``` python
{
  "targets": [
    {
      "target_name": "binding",
      "sources": [ "src/binding.cc" ]
    }
  ]
}
```

关于插件和编写 `gyp` 文件的一些额外资料：

 * ["Going Native" a nodeschool.io tutorial](http://nodeschool.io/#goingnative)
 * ["Hello World" node addon example](https://github.com/nodejs/node/tree/master/test/addons/hello-world)
 * [gyp user documentation](https://gyp.gsrc.io/docs/UserDocumentation.md)
 * [gyp input format reference](https://gyp.gsrc.io/docs/InputFormatReference.md)
 * [*"binding.gyp" files out in the wild* wiki page](https://github.com/nodejs/node-gyp/wiki/%22binding.gyp%22-files-out-in-the-wild)


## 命令

`nw-gyp` 有以下命令：

| **命令**      | **描述**
|:--------------|:---------------------------------------------------------------
| `help`        | 显示帮助信息
| `build`       | 调用 `make`/`msbuild.exe` 构建原生模块
| `clean`       | 如果存在 `build` 目录则移除
| `configure`   | 为当前平台生成项目构建文件
| `rebuild`     | 依次执行 `clean`、`configure` 和 `build` 三个命令
| `install`     | 安装指定版本的 NW.js 头文件
| `list`        | 列出当前安装的 NW.js 头文件版本
| `remove`      | 移除指定版本的 NW.js 头文件


## 命令行参数

`nw-gyp` 接受以下的命令行参数：

| **参数**                          | **描述**
|:----------------------------------|:------------------------------------------
| `-j n`, `--jobs n`                | 并行运行 make
| `--target=0.5.0`                  | 要构建的 NW.js 版本
| `--silly`, `--loglevel=silly`     | 将所有进度打印到控制台
| `--verbose`, `--loglevel=verbose` | 将大部分进度打印到控制台
| `--silent`, `--loglevel=silent`   | 控制台不要打印任何信息
| `debug`, `--debug`                | 构建 Debug 版本（默认为 Release 版本）
| `--release`, `--no-debug`         | 构建 Release 版本
| `-C $dir`, `--directory=$dir`     | 在不同的目录中运行命令
| `--make=$make`                    | 重写 make 命令（如 gmake）
| `--thin=yes`                      | 启用精简静态库
| `--arch=$arch`                    | 设置目标架构（如 ia32）
| `--tarball=$path`                 | 从本地 tarball 获取头文件
| `--devdir=$path`                  | SDK 版本下载目录（默认为 ~/.nw-gyp）
| `--ensure`                        | 如果头文件已存在，不要重新安装
| `--dist-url=$url`                 | 从自定义 URL 下载头文件 tarball
| `--proxy=$url`                    | 设置用于下载头文件 tarball 的 HTTP 代理
| `--cafile=$cafile`                | 重写默认的 CA 链（用于下载 tarball）
| `--nodedir=$path`                 | 设置 node 二进制文件的路径
| `--python=$path`                  | 设置 python2 二进制文件的路径
| `--msvs_version=$version`         | 设置 Visual Studio 版本（win）
| `--solution=$solution`            | 设置 Visual Studio Solution 版本（win）


## 配置

__`nw-gyp` 接受的环境变量或 `npm` 配置__

1. 上面列举的所有命令行参数，都可以通过 npm_config_OPTION_NAME 形式的环境变量被接受（参数名称中的 `-` 应替换为下 `_`）。
直接调用 `nw-gyp` 时，这些环境变量也起作用： 
`$ export npm_config_devdir=/tmp/.gyp`  
或 Windows 上
`> set npm_config_devdir=c:\temp\.gyp`  
2. 作为 `npm` 配置，变量采用 `OPTION_NAME` 形式，该方式只在通过 `npm` 调用 `nw-gyp` 时有效：
`$ npm config set [--global] devdir /tmp/.gyp`  
`$ npm i buffertools`

## node-pre-gyp

有些包可能需要使用 [node-pre-gyp](https://github.com/mapbox/node-pre-gyp)（比如，当出现该错误时："Undefined variable module_name in binding.gyp while trying to load binding.gyp"），该工具支持使用 node-gyp 构建适用于 node.js 的插件，也支持使用 nw-gyp 构建适用于 NW.js 的插件。

```bash
$ npm install -g node-pre-gyp
$ node-pre-gyp build --runtime=node-webkit --target=0.3.3
```