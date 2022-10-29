# Tray（托盘图标） {: doctitle}
---

[TOC]

`Tray` 是不同平台上不同控件的抽象，通常是显示在操作系统通知区域的一个小图标。在 Mac OS X 上叫做 `Status Item`（状态栏项），在 GTK 上是 `Status Icon`（状态栏图标），而在 Windows 上则是 `System Tray Icon`（系统托盘图标）。

## 示例

```javascript
// 创建一个托盘图标
var tray = new nw.Tray({ title: 'Tray', icon: 'img/icon.png' });

// 给它设置一个菜单
var menu = new nw.Menu();
menu.append(new nw.MenuItem({ type: 'checkbox', label: 'box1' }));
tray.menu = menu;

// 移除托盘图标
tray.remove();
tray = null;
```

!!! warning "页面导航与托盘的使用"
    在页面中创建的托盘图标，在页面刷新或跳转后会失效，是因为托盘图标乃至网页在导航后都会被 JS 引擎垃圾回收，以防止内存泄漏。所以建议在 **后台页面** 中使用托盘图标，可以存在于应用的整个生命周期。参考 [`bg-script`](Manifest Format.md#bg-script) 和 [`main`](Manifest Format.md#main) 查看如何在后台页面中执行脚本。


## new Tray(option)

* `option` `{Object}`
    - `title` `{String}` （Mac）标题
    - `tooltip` `{String}` 提示文本
    - `icon` `{String}` 图标
    - `alticon` `{String}` （Mac）替换图标
    - `iconsAreTemplates` `{Boolean}` （Mac）图标是否为模板图片
    - `menu` `{Menu}` 弹出菜单

创建一个 `Tray` 实例，`option` 是一个包含 `Tray` 初始设置信息的对象，每个字段在 `Tray` 实例上都有对应的属性，查看下面的文档了解各个属性的详细信息。

## tray.title (Mac)

获取或设置托盘图标的标题。

在 Mac OS X 上，`title` 会显示在状态栏上 `icon` 的旁边，而在 GTK 和 Windows 下，该属性没有效果，因为这两个平台的托盘只支持显示一个图标。

## tray.tooltip

获取或设置托盘图标的提示文本。提示文本会在鼠标移动到托盘图标上时显示。

!!! note "注意"
    `tooltip` 在所有三个平台上都可以显示，通过 `Tray` 实例的属性进行设置比作为构造函数的 `option` 参数传入更好。

## tray.icon

获取或设置托盘图标的图片，值为图标文件的路径，可以是应用中的相对路径，也可以是系统中的绝对路径。

!!! note "Mac"
    Mac OS X 警告：在通知上下文中使用时，png 图标不会像 Windows 通知区域那样自动缩小适配，而是以原始比例进行显示。

## tray.alticon (Mac)

获取或设置替换（活动）托盘图标。

## tray.iconsAreTemplates (Mac)

获取或设置图标图片是否作为模板图片（默认为 `true`）。当该属性设置为 `true` 时，图片将被作为模板图片，系统会根据状态项的各种状态（如深色菜单、浅色菜单等）自动显示合适的样式。模板图片必须只包含黑色和透明通道，可以使用图片中的透明通道来调整黑色内容的不透明度。

## tray.menu

获取或设置托盘图标的菜单，菜单会在点击托盘图标时显示。

在 Mac OS X 上，菜单会在 **单击** 托盘图标时显示（这是 Mac OS X 上托盘图标唯一有效的操作）。
而在 Windows 和 Linux 上，菜单则是通过 **鼠标右键单击** 托盘图标来显示，左键单击不会显示菜单，而是会发送一个单击事件。

为了减少不同平台的差异，您只能给托盘图标绑定一个菜单，在 Linux 和 Windows 下无法通过鼠标左键单击来弹出托盘菜单。

## tray.remove()

移除托盘图标。

一旦移除，托盘图标将不在显示，并且您需要将保存托盘图标实例的变量设置为 `null`，让它可以被垃圾回收。没有临时隐藏托盘图标的方法。

## 事件：click

鼠标左键点击托盘图标时触发。

您无法侦听到鼠标右键点击，因为它用于弹出托盘菜单，即使没有给托盘图标绑定菜单，也无法侦听到。
双击事件也被忽略。

!!! note "Mac"
    NW.js 不支持 menulet（<kbd>&#8984;</kbd>+drag），因为它会导致 NW.js 应用无法在 App Store 中发布。
