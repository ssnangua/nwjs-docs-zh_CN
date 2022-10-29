# 透明窗口 {: .doctitle}
---

[TOC]

## 基本要求

透明特性需要和无框架窗口一起使用。

### Windows

透明度特性只支持 Vista 以上版本，且需要开启桌面窗口管理器（DWM：Desktop Window Manager）。
透明特性在经典主题、基础版本或远程桌面的系统中可能无效。

### Linux

启动 NW.js 需要带上这些参数，且窗口管理器需要支持 **compositing**:
```params
--enable-transparent-visuals --disable-gpu
```

## 创建一个透明窗口

将 HTML body 的背景色设置为透明：
```html
<body style="background-color:rgba(0,0,0,0);">
```

将配置文件中的 [`transparent` 字段](../../References/Manifest Format.md#transparent) 设置为 `true`：
```json
  "window": {
    "frame": false,
    "transparent": true
  }
```

## 点击穿透（Windows & Mac）

在 Windows 和 Mac 系统中，您可以启用透明区域的点击穿透，该特性可以让您在点击窗口 **透明度为 `0` 的区域** 时，可以透过去点到后面的东西。

要启用透明区域点击穿透特性，需要下面的命令行参数:
```params
--disable-gpu --force-cpu-draw
```

!!! note "注意"
    点击穿透只支持 **无框架** 且 **不可调整大小** 的框架，也可能在其他配置下有效，具体取决于系统。
    （**译者注：**意思就是，我也不确定，您自己试 =_=）
