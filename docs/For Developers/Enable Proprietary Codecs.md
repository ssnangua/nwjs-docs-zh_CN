# 使用私有编解码器
---

[TOC]

## 预构建的 NW.js 程序支持的编解码器

NW.js 基于 Chromium，媒体组件本质上是相同的。预构建的 NW.js 程序支持以下的编解码器:

    - theora
    - vorbis
    - vp8 & vp9
    - pcm_u8、pcm_s16le、pcm_s24le、pcm_f32le、pcm_s16be & pcm_s24be
    - mp3 (v0.22.1+)

... 同时支持以下 demuxer：

    - ogg
    - matroska
    - wav

## 使用私有编解码器

!!! warning "许可和专利费"
    使用 H.264 编解码器时，需要注意源代码的专利费和许可。如果您要在应用中使用专利媒体格式，可以咨询律师了解许可的限制。关于源代码许可的更多信息，参考 [这里](https://chromium.googlesource.com/chromium/third_party/ffmpeg.git/+/master/CREDITS.chromium)。

!!! warning "警告"
    如果您 **没有** 获得许可，下面的方法或其他解决方案并 **不能** 让您拥有分发私有编解码器的权利。

### 从社区获取 FFmpeg 二进制文件

Chromium 项目最近的版本将 FFmpeg 改为内置编解码器，所以您无法再从官方发行的 Chrome 中获取 FFmpeg DLL。不过，您可以从社区获取 [预编译文件](https://github.com/iteufel/nwjs-ffmpeg-prebuilt/releases)，也可以按照下面的说明自行构建 FFmpeg。

> **译者注：**本文档中的 **DLL** 是动态链接库（Dynamic Link Library）的统称，不是特指 Windows 系统的 `*.dll` 文件。

### 脱离 NW.js 构建 FFmpeg DLL

如果您使用的是官方预构建的 NW.js，只需要重新构建 FFmpeg DLL，然后替换预构建包中的 DLL 即可。相较于构建整个 NW.js（~20G），在下载大小（~1G）上可以节省很多时间。

**步骤 1.** 从 GitHub 下载定制的 Chromium zip 包。您可以在 https://github.com/nwjs/chromium.src/tags 里找到相应的版本，将 zip 包提取到一个本地文件夹中，如 `~/nw`，解压出来后会包含一个子目录，所以源码文件夹在 `~/nw/<sub-directory-name>` 里面。

**步骤 2.** 获取依赖

因为不是构建整个 NW.js，所以需要手动获取下面的依赖：

* 根据 `DEPS` 文件中相应的 commit，从 Git 上克隆以下文件夹：
    - `buildtools`
    - `tools/gyp`
    - `third_party/yasm/sources/patched-yasm`
    - `third_party/ffmpeg`
* 在 `buildtools/<os>` 中下载 `gn` 工具：
```bash
download_from_google_storage --no_resume \
                             --platform=<platform> \
                             --no_auth \
                             --bucket chromium-gn \
                             -s buildtools/<os>/<gn-exe>.sha1
```
    - `<platform>`：Windows 对应 `win32`，Mac 对应 `darwin`，Linux 对应 `linux*`
    - `<os>`：Windows 对应 `win`，Mac 对应 `mac`; Linux 对应 `linux64`
    - `<gn-exe>`：Windows 对应 `gn.exe`，Mac 和 Linux 对应 `gn`
* **[Mac & Linux]** 在 `third_party/llvm-build` 中下载 `clang` 工具：
```bash
python tools/clang/scripts/update.py --if-needed
```
* **[Linux]** 在 `third_party/llvm-build` 中下载库：
```bash
python build/linux/sysroot_scripts/install-sysroot.py --running-as-hook
```
* **[Mac]** 在 `third_party/libc++-static` 中下载 **libc++-static** 库：
```bash
download_from_google_storage --no_resume \
                             --platform=darwin \
                             --no_auth \
                             --bucket chromium-libcpp \
                             -s third_party/libc++-static/libc++.a.sha1
```

!!! tip "Linux 开发者"
    首次构建 FFmpeg DLL 或 NW.js，请在继续下面的步骤之前运行一次 `build/install-build-deps.sh`，它会自动安装构建的依赖，该操作您只需要执行一次。

**步骤 3.** 替换 BUILD.gn 

将源码根目录中的 `BUILD.gn` 替换为以下内容：

```
action("dummy") {
  deps = [
    "//third_party/ffmpeg"
  ]
  script = "dummy"
  outputs = ["$target_gen_dir/dummy.txt"]
}
```

**步骤 4.** 使用 GN 工具生成 Ninja 文件

```bash
cd path/to/nw/source/folder
gn gen //out/nw \
 --args='is_debug=false is_component_ffmpeg=true target_cpu="<target_cpu>" is_official_build=true ffmpeg_branding="Chrome"'
```

**注意**：`<target_cpu>` 必须设置为 `x86` 或 `x64`（根据要构建的是 32 位或 64 位）。

**步骤 4.** 构建 FFmpeg DDL

```bash
ninja -C out/nw ffmpeg
```

您可以在 `out/nw` 文件夹中找到构建出来的 DLL 文件，路径和文件名因平台而异：

* Windows：`ffmpeg.dll`
* Mac OS X：`libffmpeg.dylib`
* Linux：`lib/libffmpeg.so`

**步骤 5.** 将预构建的 NW.js 包中的 DLL 文件替换为您刚刚构建的那个，路径和文件名因平台而异：

* Windows：`ffmpeg.dll`
* Mac OS X：`nwjs.app/Contents/Versions/<chromium-version>/nwjs Framework.framework/libffmpeg.dylib`
* Linux `lib/libffmpeg.so`

### 完整构建集成了私有编解码器的 NW.js

如果您使用的不是官方预构建的 NW.js，可以根据下面的说明，构建一个集成了私有编解码器的 NW.js。参考 [构建 NW.js](Building NW.js.md) 查看每个步骤的详细说明。

**步骤 1.** 安装构建依赖并获取 NW.js 源代码。参考 [构建 NW.js](Building NW.js.md) 中的 *构建准备* 和 *获取代码* 部分。

**步骤 2.** 配置 GN 时，将 `ffmpeg_branding` 设置为 `Chrome`。

**步骤 3.** 再次生成 ninja 文件。

**步骤 4.** 重新构建 NW.js。
