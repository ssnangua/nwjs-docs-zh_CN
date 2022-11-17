# 故障排查 {: .doctitle}
---

[TOC]

> 原文链接：https://github.com/nwjs/nw.js/wiki/Troubleshooting
> 译者根据新版本的 NW.js 对内容做了相应调整

## 音频

如果音频不能正常工作，问题可能出在 mp3 或其他非免费媒体格式的编解码器。参考 [使用私有编解码器](../For Developers/Enable Proprietary Codecs.md)。

## WebGL

需要 2 个来自 DirectX 的额外 DLL 文件，或者您的 GFX 卡或驱动程序在 Chromium 的黑名单中。参考 [#185](https://github.com/nwjs/nw.js/issues/185)。

有时 css 不能正常显示，如 `-webkit-transform:translateZ(-1000px);`，也可能是 WebGL 的问题。

## 执行 JavaScript 阻塞 CSS 动画

默认情况下，它们运行在同一线程中。

您可以给 [配置文件的 `chromium-args`](../References/Manifest Format.md#chromium-args) 添加 `--enable-threaded-compositing` 标志来改变该行为。

## 缺少 libudev.so.0

可能 NW.js 会因为以下错误而无法在您的电脑上运行：

````
nw: error while loading shared libraries: libudev.so.0: cannot open shared object file: No such file or directory
````

参考 [#136](https://github.com/nwjs/nw.js/issues/136) 和 [#770](https://github.com/nwjs/nw.js/issues/770)。