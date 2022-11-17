# 使用 Node 原生模块 {: .doctitle}
---

[TOC]

## 使用 NPM 安装

### 对于 LTS 版本

!!! warning "使用相同版本和架构的 Node.js 和 NW.js"
    以下说明仅适用于使用的是相同版本和架构的 Node.js 和 NW.js。

如果您使用的是 LTS 版本，在经过下面的操作后，可以支持通过 `npm` 安装的原生模块：

* Linux 和 Mac 上，不需要处理。
* Windows 上，在使用 `node-gyp` 或 `npm` 安装模块之前，您需要将 
`<npm-path>\node_modules\node-gyp\src\win_delay_load_hook.cc` 替换为 https://github.com/nwjs/nw.js/blob/nw18/tools/win_delay_load_hook.cc。

<span id="对于非LTS版本"></span>
### 对于非 LTS 版本

如果您使用的是非 LTS 版本，由于 [V8 中的 ABI 差异](https://github.com/nwjs/nw.js/issues/5025)，您需要全局安装 `nw-gyp` 以构建模块。

下面的说明同样可用于 LTS 版本。

要给 NW.js 安装原生模块，运行下面的命令：

```bash
# 全局安装 nw-gyp
npm install -g nw-gyp
# 配置目标 NW.js 版本
export npm_config_target=0.18.5
# 配置构建架构 ia32 或 x64
export npm_config_arch=x64
export npm_config_target_arch=x64
# 配置使用 node-pre-gyp 构建模块的环境
export npm_config_runtime=node-webkit
export npm_config_build_from_source=true
# 配置 nw-gyp 作为 node-gyp
export npm_config_node_gyp=$(which nw-gyp)
# 运行 npm install
npm install
```

!!! warning "Windows 系统"
    您需要有一个编译器，可以通过安装 [Visual C++ Build Tools](http://landinghub.visualstudio.com/visual-cpp-build-tools) 来获得一个。
    您还需要有 Python 2.7 环境（不支持 3.x 版本）。
    需要将环境变量 `npm_config_node_gyp` 设置为 `path\to\global\node_modules\nw-gyp\bin\nw-gyp.js`，而不是 `nw-gyp.cmd` 处理脚本，这是 [npm 的 bug](https://github.com/npm/npm/issues/14543)。
    在 Windows 上，您需要使用 `set` 来设置环境变量，而不是 `export`。

这是一个完整的示例（Windows 10，Visual C++ Build Tool v2015）：

```Batchfile
set PYTHON=C:\Python27\python.exe
set npm_config_target=0.21.6
set npm_config_arch=x64
set npm_config_runtime=node-webkit
set npm_config_build_from_source=true
set npm_config_node_gyp=C:\Users\xxxxxxxxx\AppData\Roaming\npm\node_modules\nw-gyp\bin\nw-gyp.js
npm install --msvs_version=2015
```

## 手动重新构建

!!! tip "推荐完整安装依赖项"
    在切换 NW.js 版本后，推荐的方式是删除 `node_modules` 文件夹并重新 `npm install`。

如果切换了 NW.js 版本，您可以使用下面的工具重新构建 **每一个** 原生模块，而 **无需重新安装所有模块**。因为构建原生模块需要 `binding.gyp`，您可以通过查找 `binding.gyp` 文件轻松找到所有的原生模块。

!!! warning "重新构建所有的原生模块"
    确保您 **重新构建了所有的原生模块**，否则您会看到各种各样的报错甚至崩溃。如果手动重新构建没有效果，尝试删除 `node_modules` 文件夹并重新 `npm install`。

## nw-gyp

[`nw-gyp`](https://github.com/nwjs/nw-gyp) 是 `node-gyp` 的一个定制版，用于支持 NW.js 的特定头文件和库。

它的用法与 `node-gyp` 相同，除了您需要手动指定 NW.js 的版本和架构（`x64` 或 `ia32`）。

````bash
npm install -g nw-gyp
cd myaddon
nw-gyp rebuild --target=0.13.0 --arch=x64
````

参考 https://github.com/nwjs/nw-gyp。

## node-pre-gyp

一些包使用了 [node-pre-gyp](https://github.com/mapbox/node-pre-gyp)，它即支持通过 `node-gyp` 构建 Node.js 的原生模块，也支持通过 `nw-gyp` 构建 NW.js 的原生模块。

`node-pre-gyp` 的用法如下：

````bash
npm install -g node-pre-gyp
cd myaddon
node-pre-gyp build --runtime=node-webkit --target=0.13.0 --target_arch=x64
````

参考 https://github.com/mapbox/node-pre-gyp。

