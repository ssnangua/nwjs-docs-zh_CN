# 自定义菜单栏
---

[TOC]

不同平台的窗口菜单有不同的展示形式，下面将讨论这些差异并给出最佳实践，使窗口菜单在所有平台上都能正常工作。

## 创建和设置菜单栏

要创建菜单栏，您只需要在创建 Menu 实例时在参数中指定
`type: 'menubar'` 即可：

```javascript
var your_menu = new nw.Menu({ type: 'menubar' });
```

确保添加到菜单栏的每个菜单项都有一个子菜单。在大多平台中，菜单栏的一级菜单只有一个显示文本（而没有子菜单）是没有什么意义的。

大多数系统中 , 文本菜单是没有意义的 . 

```javascript
var submenu = new nw.Menu();
submenu.append(new nw.MenuItem({ label: '菜单项 A' }));
submenu.append(new nw.MenuItem({ label: '菜单项 B' }));

// 添加到菜单栏的菜单项要有一个子菜单
your_menu.append(new nw.MenuItem({
  label: '第一个子菜单',
  submenu: submenu
}));
```

然后，您可以通过窗口对象的 `menu` 属性来设置窗口菜单：

```javascript
nw.Window.get().menu = your_menu;
```

参考 [Menu](../../References/Menu.md)（菜单）和[Window](../../References/Window.md)（窗口）查看 API 的详细说明。

## 平台差异

### Windows & Linux

在 Windows 和 Linux 系统上，菜单栏的行为完全相同，每个窗口的都会有一个菜单栏，位于标题栏下方。

!!! tip "全屏和 Kiosk 模式下的菜单栏"
    在全屏或 Kiosk 模式下，菜单栏会显示在窗口顶部，可以通过将 `win.menu` 设置为 `null` 来完全移除菜单栏。参考 [`win.menu`](../../References/Window.md#winmenu)。

### Mac OS X

!!! warning "特性变更"
    从 0.13.0 版本开始，该特性发生了变化，参考 [从 0.12 升级到 0.13](../Migration/From 0.12 to 0.13.md#menu（菜单）)。

在 Mac 系统中，不管一个应用有多少个窗口，都只会有一个菜单栏，称之为应用菜单。应用的快捷键都依赖这个应用菜单，如 *Quit（退出）*、*Close（关闭）*和 *Copy（复制）*。

NW.js 应用启动时，会有一个默认的应用菜单，包含 *应用名*、*Edit（编辑）*和 *Window（窗口）* 菜单项。您可以通过 [menu.createMacBuiltin](../../References/Menu.md#menucreatemacbuiltinappname-options-mac) 方法来获取这个默认菜单，有需要时也可以进行定制。

```javascript
var mb = new nw.Menu({ type:"menubar" });
mb.createMacBuiltin("应用名");
// 给 `mb` 添加、插入或删除菜单项来定制自己的应用菜单
nw.Window.get().menu = mb;
```

!!! note "修复应用菜单的标题"
    应用菜单的第一项显示的是 *nwjs* 而不是 *应用名*，要解决这个问题，您需要将 `nwjs.app/Contents/Resources/*.lproj/InfoPlist.strings` 目录下的所有文件中的 `CFBundleName` 的值都由 `nwjs` 改为 `应用名`。

## 最佳实践

综上所述，在 Windows 和 Linux 系统中，应用的每个窗口都可以有一个菜单栏，而在 Mac 系统中，一个应用只能有一个应用菜单，所以通常应该只给主窗口设置菜单栏。如果可能会有多个主窗口实例，尽量避免使用菜单栏。

如果您想给不同的平台设计不同的菜单栏，可以通过 [process.platform](http://nodejs.org/api/process.html#process_process_platform) 来获取当前运行的平台。