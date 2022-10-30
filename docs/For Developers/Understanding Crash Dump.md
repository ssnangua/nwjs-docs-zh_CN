# 了解崩溃机制 {: .doctitle}
---

[TOC]

当 NW.js 崩溃时，会在硬盘上生成一个 `minidump` 文件（`.dmp`）。用户可以将它包含在 bug 报告里，您可以解码该文件来获取崩溃时的堆栈跟踪信息，在某些情况下，这对于定位问题是很有帮助的。

要提取 `minidump` 文件中的堆栈跟踪信息，需要有三个条件：

- 崩溃时生成的 minidump 文件（`.dmp`，）（**译者注：**[小型转储文件](https://learn.microsoft.com/zh-cn/windows/win32/debug/minidump-files)）
- NW.js 的 Symbol 文件（`.sym`）（**译者注：**[符号文件](https://learn.microsoft.com/zh-cn/windows-hardware/drivers/debugger/using-symbols)）
- `minidump_stackwalk` 工具

## 获取 minidump 文件

NW.js 崩溃时会在下面的默认目录中生成 minidump 文件：

* Linux：`~/.config/<name-in-manifest>/Crash\ Reports/`
* Windows：`%LOCALAPPDATA%\<name-in-manifest>\User Data\CrashPad`
*（因为 bug [#5248](https://github.com/nwjs/nw.js/issues/5248)，0.21.5 之前的版本在 `%LOCALAPPDATA%\Chromium\User Data\CrashPad` 目录中）*
* Mac：`~/Library/Application\ Support/<name-in-manifest>/CrashPad/`

`<name-in=manifest>` 为 [配置文件](../References/Manifest Format.md#name) 中的 `name` 字段。

!!! note "删除 Linux Minidump 文件中的头信息"
    Linux 系统中生成的 minidump 文件会附加文本格式的头信息，解码之前需要删除。minidump 文件的真实内容以 `MDMP` 开头，后面的内容是不可读的二进制数据，所以只需要删除 `MDMP` 前面的文本即可。

## 整理 Symbol 文件

发布版本的 NW.js 的 Symbol 文件包可以在 [https://dl.nwjs.io/](https://dl.nwjs.io/) 上下载。

例如，Mac 上，`0.57.1` 版本：
[https://dl.nwjs.io/v0.57.1/nwjs-symbol-v0.57.1-osx-x64.zip](https://dl.nwjs.io/v0.57.1/nwjs-symbol-v0.57.1-osx-x64.zip)
[https://dl.nwjs.io/v0.57.1/nwjs-sdk-symbol-v0.57.1-osx-x64.zip](https://dl.nwjs.io/v0.57.1/nwjs-sdk-symbol-v0.57.1-osx-x64.zip)

然后您需要将 Symbol 文件改为正确的文件名并整理到正确的路径，以便于 `minidump_stackwalk` 工具使用。`minidump_stackwalk` 使用[simple symbol supplier](https://code.google.com/p/chromium/codesearch#chromium/src/breakpad/src/processor/simple_symbol_supplier.cc&l=142) 来查找 Symbol 文件，下面是它查找的方式。

工具会尝试以下面的规则来查找 `.sym` 文件（对于 Mac，将 `.breakpad` 改为 `.sym`）：
`{SYMBOLS_ROOT}/{DEBUG_FILE_NAME}/{DEBUG_IDENTIFIER}/{DEBUG_FILE_NAME_WITHOUT_PDB}.sym`

* `{SYMBOLS_ROOT}` 是所有 Symbol 文件的根目录。您可以将所有版本和平台的 NW `.sym` 文件都放在同一个目录中。
* `{DEBUG_FILE_NAME}`、`{DEBUG_IDENTIFIER}` 和 `{DEBUG_FILE_NAME_WITHOUT_PDB}` 可以从 `.sym` 文件的首行获取。
如：`MODULE Linux x86_64 265BDB6BE043D5C70D3A1E279A8F0B1A0 nw`
（Mac 上为：`MODULE mac x86_64 4E7C70708AFD3C889F02B149AB5007080 nwjs`）
    - `265BDB6BE043D5C70D3A1E279A8F0B1A0` 对应 `{DEBUG_IDENTIFIER}`
    - `nw` 对应 `{DEBUG_FILE_NAME}`.
    - `{DEBUG_FILE_NAME_WITHOUT_PDB}` 可以通过删除 `{DEBUG_FILE_NAME}` 的 `.pdb` 扩展名转换而来，只有 Windows 系统需要该操作。


例如，Mac 上，`0.57.1` 版本：

```bash
-symbols_root/
 -nwjs/
    -4E7C70708AFD3C889F02B149AB5007080/
        -nwjs.sym
 -nwjs Framework/
    -87A9EA49BC473F4C8B7817631E820BEB0/
        -nwjs Framework.sym
 -nwjs Helper/
    -5598EA295F4F36FDA21CB9A5B11B11AA0/
        -nwjs Helper.sym
```

## 使用 `minidump_stackwalk` 解码 Minidump

在 Mac 和 Linux 系统中，`minidump_stackwalk` 可以通过 NW.js 构建，或者通过 Breakpad 的源码直接构建。在 Windows 系统中，可以通过 Cygwin 安装预构建。

要从 `minidump` 文件中获取堆栈跟踪，运行以下命令：

```bash
minidump_stackwalk minidump_file.dmp /path/to/symbols_root 2>&1
```

如果 Symbol 文件没有整理正确，仍然可以从该工具获取到调用堆栈，但无法找到 Symbol，并且输出的最后一部分会是一个警告 - "Loaded modules"，类似于：

```none
0x00240000 - 0x02b29fff nw.exe ??? (main) (WARNING: No symbols, nw.exe.pdb, 669008F7B6EE44058CBD5F21BEB5B5CFe)
```

## 主动触发崩溃用于测试

为了测试崩溃转储特性，您可以通过 NW.js 提供的 API `App.crashBrowser()` 和 `App.crashRenderer()` 来主动触发崩溃，这两个方法分别让浏览器进程和渲染进程崩溃。

## 参考

* http://www.chromium.org/developers/decoding-crash-dumps  
* http://code.google.com/p/google-breakpad/wiki/GettingStartedWithBreakpad

