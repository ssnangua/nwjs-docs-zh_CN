# Screen（屏幕） {: .doctitle}
---

[TOC]

`Screen` 是一个 EventEmitter 对象的实例，可以使用 `Screen.on(...)` 来响应原生屏幕的事件。

`Screen` 是一个单例对象，需要调用一次 `nw.Screen.Init()` 来初始化。

## 示例

```javascript
// 需要在初始化期间执行一次 Init，之后才能使用 nw.Screen 的功能
nw.Screen.Init();

var screenCB = {
  onDisplayBoundsChanged: function(screen) {
    console.log('displayBoundsChanged', screen);
  },

  onDisplayAdded: function(screen) {
    console.log('displayAdded', screen);
  },

  onDisplayRemoved: function(screen) {
    console.log('displayRemoved', screen)
  }
};

// 侦听屏幕事件
nw.Screen.on('displayBoundsChanged', screenCB.onDisplayBoundsChanged);
nw.Screen.on('displayAdded', screenCB.onDisplayAdded);
nw.Screen.on('displayRemoved', screenCB.onDisplayRemoved);
```

## Screen.Init()

初始化 Screen 单例对象，该方法只需要调用一次。

## Screen.screens

获取屏幕列表（连接到电脑的所有显示器）

屏幕信息有以下结构：
```javascript
screen {
  // 屏幕的唯一 id
  id: int,

  // 物理屏幕分辨率，可以为负数，不一定从 0 开始，取决于屏幕排列
  bounds: {
    x: int,
    y: int,
    width: int,
    height: int
  },
 
  // 屏幕范围内的可用区域
  work_area: {
    x: int,
    y: int,
    width: int,
    height: int
  },

  scaleFactor: float,
  isBuiltIn: bool,
  rotation: int,
  touchSupport: int
}
```

## Screen.chooseDesktopMedia (sources, callback)

* `sources` `{String[]}` 源的类型数组。支持类型：`"screen"` 和 `"window"`。
* `callback` `{Function}` 带有所选 streamId 的回调函数。如果执行失败或现有会话处于活动状态时，streamId 将为 `false`。

!!! note "注意"
    会弹出一个列表供用户选择要共享的源。目前仅适用于 Windows 和 OSX 以及一些 linux 发行版。

例子：

```javascript
nw.Screen.Init(); // 只需要调用一次
nw.Screen.chooseDesktopMedia(["window","screen"], 
  function(streamId) {
    var vid_constraint = {
      mandatory: {
        chromeMediaSource: 'desktop', 
        chromeMediaSourceId: streamId, 
        maxWidth: 1920, 
        maxHeight: 1080
      }, 
      optional: []
    };
    navigator.webkitGetUserMedia(
      {
        audio: false,
        video: vid_constraint
      },
      success_func,
      fallback_func
    );
  }
);
```

## 事件：displayBoundsChanged(screen)

当屏幕分辨率、排列发生变化时触发。回调函数带有 1 个参数 `screen`，参考 [Screen.screens](#screenscreens) 查看数据格式。

## 事件：displayAdded (screen)

当一个新屏幕被添加时触发。回调函数带有 1 个参数 `screen`，参考 [Screen.screens](#screenscreens) 查看数据格式。

## 事件：displayRemoved (screen)

当一个现有的屏幕被移除时触发。回调函数带有 1 个参数 `screen`，参考 [Screen.screens](#screenscreens) 查看数据格式。

## Screen.DesktopCaptureMonitor

这个 API 的功能类似于 `Screen.chooseDesktopMedia`，但没有 GUI（不会自动弹出可共享的列表），可以使用该 API 来监视桌面上的屏幕或窗口，然后实现自己的 UI（自己实现一个可共享的列表）。

`Screen.DesktopCaptureMonitor` 是一个 `EventEmitter` 实例，可以使用 `Screen.DesktopCaptureMonitor.on()` 来侦听相关事件。

### 示例

```javascript
var dcm = nw.Screen.DesktopCaptureMonitor;
nw.Screen.Init();
dcm.on("added", function (id, name, order, type) {
   // 选择第一个流，然后停止监视
   var constraints = {
      audio: {
         mandatory: {
             chromeMediaSource: "system",
             chromeMediaSourceId: dcm.registerStream(id)
          }
      },
      video: {
         mandatory: {
             chromeMediaSource: 'desktop',
             chromeMediaSourceId: dcm.registerStream(id)
         }
      }
  };

  // TODO: 调用 getUserMedia 并传入 constraints

  dcm.stop();
});

dcm.on("removed", function (id) { });
dcm.on("orderchanged", function (id, new_order, old_order) { });
dcm.on("namechanged", function (id, name) { });
dcm.on("thumbnailchanged", function (id, thumbnail) { });
dcm.start(true, true);
```

### Screen.DesktopCaptureMonitor.started

一个布尔值，表示 DesktopCaptureMonitor 是否已启动。

### Screen.DesktopCaptureMonitor.start(should_include_screens, should_include_windows)

* `should_include_screens` `{Boolean}` 是否要包含屏幕
* `should_include_windows` `{Boolean}` 是否要包含窗口

`DesktopCaptureMonitor` 开始监视系统并触发相关事件。`DesktopCaptureMonitor` 在运行时，屏幕可能会闪烁。

### Screen.DesktopCaptureMonitor.stop()

`DesktopCaptureMonitor` 停止监视系统。选择一个流后，应该停止 `DesktopCaptureMonitor`。

### Screen.DesktopCaptureMonitor.registerStream(id)

注册并返回一个有效的流 id，可以传递给 `getUserMedia` 约束条件里的 `chromeMediaSourceId`。
参考 [示例](#示例-1)。

### 事件：added (id, name, order, type, primary)

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `id` `{String}` 媒体 id。使用 `registerStream(id)` 可获得一个有效的流 id，可用于 `getUserMedia()`。参考 [registerStream](#screendesktopcapturemonitorregisterstreamid)
* `name` `{String}` 窗口或屏幕的标题
* `order` `{Integer}` 窗口的 z 序号，被选择的屏幕将优先出现
* `type` `{String}` 流的类型："screen"、"window"、"other" 或 "unknown"
* `primary` `{Boolean}` _仅 Windows 系统有效_ 如果是主显示的源，该值将为 `true`

当一个新的源被添加时触发。

### 事件：removed (order)

* `order` `{Integer}` 已被移除（不再可捕获）的媒体源的序号

当一个源被移除时触发。

### 事件：orderchanged (id, new_order, old_order)

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `id` `{String}` z 序号发生了变更的屏幕或窗口的媒体 id
* `new_order` `{Integer}` 新的 z 序号
* `old_order` `{Integer}` 变更前的 z 序号

当一个源的 z 序号发生变更时触发（可以是其他窗口获得焦点）。

### 事件：namechanged (id, name)

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `id` `{String}` 名字发生了变更的屏幕或窗口的媒体 id
* `name` `{String}` 屏幕或窗口的新名字

当源的名字发生变更时触发（可以是窗口的名字发生变更）。

### 事件：thumbnailchanged (id, thumbnail)

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../For Users/Migration/From 0.12 to 0.13.md)。

* `id` `{String}` 更新了缩略图的屏幕或窗口的媒体 id
* `thumbnail` `{String}` 缩略图的 png base64 编码图片数据

当源的缩略图更新时触发。

## 完整示例

> 下面是译者的一些尝试，有兴趣的可以玩一下

### Screen.chooseDesktopMedia

```html
<html>
<head>
  <title>Screen.chooseDesktopMedia</title>
</head>
<body style="margin: 0; background: #000">
  <video id="video" autoplay="true" style="width: 100%; height: 100%"></video>

  <script>
    const video = document.getElementById("vid");
    nw.Screen.Init();
    nw.Screen.chooseDesktopMedia(["window","screen"], 
      (streamId) => {
        const vid_constraint = {
          mandatory: {
            chromeMediaSource: "desktop", 
            chromeMediaSourceId: streamId, 
            maxWidth: 1920, 
            maxHeight: 1080
          }, 
          optional: []
        };
        navigator.webkitGetUserMedia(
          {
            audio: false,
            video: vid_constraint
          },
          (stream) => {
            video.srcObject = stream;
          },
          (err) => {
            console.warn(err);
          }
        );
      }
    );
  </script>
</body>
</html>
```

### Screen.DesktopCaptureMonitor

```html
<html>
<head>
  <title>Screen.DesktopCaptureMonitor</title>
</head>
<body style="margin: 0; background: #000">
  <video id="video" autoplay="true" style="width: 100%; height: 100%"></video>

  <script>
    const video = document.getElementById("vid");
    nw.Screen.Init();
    const dcm = nw.Screen.DesktopCaptureMonitor;
    dcm.on("added", (id, name, order, type) => {
      const constraints = {
        audio: {
          mandatory: {
            chromeMediaSource: "system",
            chromeMediaSourceId: dcm.registerStream(id)
          }
        },
        video: {
          mandatory: {
            chromeMediaSource: "desktop",
            chromeMediaSourceId: dcm.registerStream(id)
          }
        }
      };

      navigator.webkitGetUserMedia(
        constraints,
        (stream) => {
          video.srcObject = stream;
        },
        (err) => {
          console.warn(err);
        }
      );

      dcm.stop();
    });
    dcm.start(true, true);
  </script>
</body>
</html>
```
