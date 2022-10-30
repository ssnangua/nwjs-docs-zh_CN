# 构建 NW.js {: .doctitle}
---

[TOC]

!!! important "重要"
    从 0.17 版本开始，我们切换到 Chromium upstream 里的新构建配置系统 GN 和 Ninja。 NW 里的 Node.js 仍然使用 GYP/Ninja 构建。
    本文档适用 **NW 0.13** 及之后的版本，关于早期版本的构建说明，可以参考 GitHub 上的 [wiki 页面](https://github.com/nwjs/nw.js/wiki/Building-nw.js)。
    查看我们的 buildbot 以了解官方构建配置和步骤：http://buildbot-master.nwjs.io:8010/waterfall。
    调试版本需要使用组件构建配置进行构建，参考 [upstream 里的讨论](https://groups.google.com/a/chromium.org/d/msg/chromium-dev/LWRR3fMlvQ4/qos4EaBtBgAJ)。

> **译者注：**
> Google 公司的一个员工在开发 Chromium 时，因为受不了 cmake 编译工程的耗时，于是自己开发了一个小型元构建系统 [**GN**](https://chromium.googlesource.com/chromium/src/tools/gn/+/48062805e19b4697c5fbd926dc649c78b6aaa138/README.md)，用来快速生成 Ninja 构建文件（GN 和 Ninja 的关系类似于 cmake 和 make）。据说，cmake 需要几天才能干完的事情，GN 搭配 Ninja 在几个小时内就能完成。

## 构建准备

NW.js 使用与 Chromium 相同的构建工具和构建流程。根据您的系统阅读相应的说明，以安装 `depot_tools` 和其他构建依赖：

* [Windows](http://www.chromium.org/developers/how-tos/build-instructions-windows)
* [Mac OS X](https://chromium.googlesource.com/chromium/src/+/master/docs/mac_build_instructions.md)
* [Linux](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_build_instructions.md)

!!! note "Windows"
    Chromium 文档建议，您需要运行 `set DEPOT_TOOLS_WIN_TOOLCHAIN=0` 或者将它设置为全局环境变量。

!!! note "Xcode 7"
    Xcode 7 不包含 SDK 10.11，如果您已经升级到 Xcode 7，可以像 [Chromium 文档](https://chromium.googlesource.com/chromium/src/+/master/docs/mac_build_instructions.md) 中建议的那样，降级到 Xcode 6，或者将其他电脑 ```xcode-select -p`/Platforms/MacOSX.platform/Developer/SDKs`` 目录下的 Mac SDK 10.10 复制过来。

## 获取代码

**步骤 1.** 创建一个文件夹用于存放 NW.js 源码，如 `$HOME/nwjs`，然后在该目录中运行下面的命令来生成 `.gclient` 文件：

```bash
mkdir -p $HOME/nwjs
cd $HOME/nwjs
gclient config --name=src https://github.com/nwjs/chromium.src.git@origin/nw17
```

如果您不需要运行 Chromium 测试，可以不用同步测试用例和相关构建，这样可以节省很多时间。打开刚刚创建的 `.gclient` 文件，将 `custom_deps` 部分替换为以下内容：

```python
"custom_deps" : {
    "src/third_party/WebKit/LayoutTests": None,
    "src/chrome_frame/tools/test/reference_build/chrome": None,
    "src/chrome_frame/tools/test/reference_build/chrome_win": None,
    "src/chrome/tools/test/reference_build/chrome": None,
    "src/chrome/tools/test/reference_build/chrome_linux": None,
    "src/chrome/tools/test/reference_build/chrome_mac": None,
    "src/chrome/tools/test/reference_build/chrome_win": None,
}
```

手动克隆下面的仓库，并切换到正确的分支：

| path | repo |
|:---- |:---- |
| src/content/nw | https://github.com/nwjs/nw.js |
| src/third_party/node | https://github.com/nwjs/node |
| src/v8 | https://github.com/nwjs/v8 |

**步骤 2.** 运行下面的命令：
```
gclient sync --with_branch_heads
```

该命令会从 GitHub 和 Google 的 Git 仓库下载 20G+ 的内容，请确保网络通畅，耐心等待。 

下载完成之后，您可以在 `.gclient` 文件所在的目录中看到一个新的 `src` 文件夹。

!!! note "Linux 系统中的首次构建"
    如果您是在 Linux 系统上构建，**第一次构建** 需要运行 `gclient sync --with_branch_heads --nohooks`，并运行 `./build/install-build-deps.sh` 安装 Ubuntu 的依赖。参考 [Chromium 文档](http://dev.chromium.org/developers/how-tos/get-the-code) 查看获取源代码的详细说明。

## 使用 GN 为 Chromium 生成 ninja 构建文件

```bash
cd src
gn gen out/nw
```
不支持修改路径中的 `nw` 部分，如果您对此不太熟悉，最好路径中的 `out` 部分也不要改动。

使用类似官方构建的参数：
````
is_debug=false
is_component_ffmpeg=true
target_cpu="x64"
````
我们支持组件构建以加快开发周期：`is_component_build = true`，如果您在尝试构建 debug 版本，应该启用这个。

关于 GN 和 GYP 的参数映射，参考 upstream 文档：https://chromium.googlesource.com/chromium/src/+/master/tools/gn/docs/cookbook.md#Variable-mappings

## 使用 GYP 为 Node 生成 ninja 构建文件

```bash
cd src
GYP_CHROMIUM_NO_ACTION=0 ./build/gyp_chromium -I \
third_party/node-nw/common.gypi -D building_nw=1 \
-D clang=1 third_party/node-nw/node.gyp
```

如果是组件构建，则是执行下面的命令：
```bash
./build/gyp_chromium -D component=shared_library -I \
third_party/node-nw/common.gypi -D building_nw=1 \
-D clang=1 third_party/node-nw/node.gyp
```

如果要更改 Node 的构建配置，则需要设置 GYP_DEFINES 环境变量：

### 32位/64位的构建方法

* Windows
    - 32位：默认的构建目标
    - 64位：`set GYP_DEFINES="target_arch=x64"`，然后在 `out/Debug_x64` 或 `out/Release_x64` 目录中重新构建
* Linux
    - 32位：**TODO: chroot**
    - 64位：默认的构建目标
* Mac
    - 32位：`export GYP_DEFINES="host_arch=ia32 target_arch=ia32"`，然后在 `out/Debug` 或 `out/Release` 目录中重新构建
    - 64位：默认的构建目标

## 构建 nwjs

运行 GN 后，会在 `out/nw` 目录中生成 ninja 构建文件。运行下面命令，会在 `out/nw` 目录中生成标准 NW.js 二进制文件的调试版本：

```bash
cd src
ninja -C out/nw nwjs
```

!!! tip "构建耗时"
    通常，一个完整的构建需要花费数个小时，具体取决于您机器的性能。建议用于构建的电脑配置为：多核 CPU（>=8 核）、SSD 固态硬盘、较大的内存（>= 8G）。参考下面的 [快速构建](#快速构建) 部分，查看一些提升构建速度的提示。

要构建 32位/64位 的二进制文件，或者非标准的构建风格，您需要编辑 `out/nw/args.gn` 文件，然后重新运行上面的命令来生成二进制文件。

## 构建 Node

```bash
cd src
ninja -C out/Release node
```

构建 Node 后，最后一步是将构建的 Node 库复制到 nwjs 二进制文件目录：

```bash
cd src
ninja -C out/nw copy_node
```

<span id="构建风格"></span>
## 构建风格

* 普通包：`nwjs_sdk=false`
* SDK包：`enable_nacl=true`

参考 [构建风格](../For Users/Advanced/Build Flavors.md) 查看支持的构建风格之间的差异。

## 集成私有编解码器

由于许可问题，NW.js 的预构建二进制文件不支持私有编解码器（如 H.264），所以在预构建版本中，您无法使用 `<audio>` 和 `<video>` 标签来播放 MP3/MP4。要构建支持这些媒体的 NW.js，可以参考 [使用私有编解码器](Enable Proprietary Codecs.md) 中的说明。

<span id="快速构建"></span>
## 快速构建

Google 的网站中，有一些可以提高构建速度的提示，参考下面的链接:

* [Mac 系统构建说明：快速构建](https://chromium.googlesource.com/chromium/src/+/master/docs/mac_build_instructions.md#Faster-builds)
* [Linux 系统上提高构建速度的提示](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_faster_builds.md)
