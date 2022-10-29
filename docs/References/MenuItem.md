# MenuItem（菜单项） {: .doctitle}
---

[TOC]

`MenuItem` 是菜单中的菜单项。`MenuItem` 可以是带有文字和图标的普通菜单项或复选框菜单项，也可以是一个分隔线菜单项。它可以响应鼠标点击或键盘快捷键。

## 示例

```javascript
var item;

// 创建一个分隔线菜单项
item = new nw.MenuItem({ type: 'separator' });

// 创建一个带有文字和图标的普通菜单项
item = new nw.MenuItem({
  type: "normal", 
  label: "I'm a menu item",
  icon: "img/icon.png"
});

// 普通菜单项也可以省略 'type' 字段
item = new nw.MenuItem({ label: 'Simple item' });

// 给菜单项绑定一个回调函数
item = new nw.MenuItem({
  label: "Click me",
  click: function() {
    console.log("I'm clicked");
  },
  key: "s",
  modifiers: "ctrl+alt",
});

// 甚至可以带有子菜单！
var submenu = new nw.Menu();
submenu.append(new nw.MenuItem({ label: 'Item 1' }));
submenu.append(new nw.MenuItem({ label: 'Item 2' }));
submenu.append(new nw.MenuItem({ label: 'Item 3' }));
item.submenu = submenu;

// 并且全部可以实时修改
item.label = 'New label';
item.click = function() { console.log('New click callback'); };
```

## new MenuItem(option)

* `option` `{Object}` 一个包含 `MenuItem` 初始设置的对象
    - `label` `{String}` _Optional_ 显示文本
    - `icon` `{String}` _Optional_ 图标
    - `tooltip` `{String}` _Optional_ （Mac）提示文本
    - `type` `{String}` _Optional_ 类型，有三种类型：`normal`、`checkbox`、`separator`
    - `click` `{Function}` _Optional_ 回调函数，通过鼠标点击或键盘快捷键触发
    - `enabled` `{Boolean}` _Optional_ 启用或禁用，默认为 `true`（启用）
    - `checked` `{Boolean}` _Optional_ 复选框是否勾选，默认为 `false`
    - `submenu` `{Menu}` _Optional_ 子菜单
    - `key` `{String}` _Optional_ 快捷键
    - `modifiers` `{String}` _Optional_ 修饰快捷键（Ctrl、Shift 等）

每个字段在 `MenuItem` 实例上都有对应的属性，查看下面的文档了解各个属性的详细信息。

`MenuItem` 继承自 `EventEmitter`，可以通过 `on` 来侦听相关事件。

## item.type

获取菜单项的类型，可能的值有 `normal`（普通）、`checkbox`（复选框）和 `separator`（分隔线）。

!!! note "注意"
    `type` 属性为 *只读*。菜单项的类型只能在创建时设置，无法动态修改。

## item.label

获取或设置菜单项的显示文本。只能为无格式文本。

## item.icon

获取或设置菜单项的图标，值为图标文件的路径，可以是应用中的相对路径，也可以是系统中的绝对路径。

`icon` 属性对 `separator`（分隔线）菜单项无效。

## item.iconIsTemplate (Mac)

获取或设置图标图片是否作为模板图片（默认为 `true`）。当该属性设置为 `true` 时，图片将被作为模板图片，系统会根据状态项的各种状态（如深色菜单、浅色菜单等）自动显示合适的样式。模板图片必须只包含黑色和透明通道，可以使用图片中的透明通道来调整黑色内容的不透明度。该属性在 Linux 和 Windows 下无效。

## item.tooltip (Mac)

获取或设置菜单项的提示文本，只能为无格式文本。提示文本是用于描述菜单项的短字符串，会在鼠标移动到菜单项上时显示。

## item.checked

获取或设置菜单项的复选框是否勾选。勾选状态的菜单项左侧会有一个 "√" 标记，一般用于表示某个选项是否开启。

## item.enabled

获取或设置菜单项是启用还是禁用。禁用的菜单项会显示为灰色，且用户无法点击。

## item.submenu

获取或设置菜单项的子菜单。子菜单必须是一个 `Menu` 对象。动态修改子菜单在一些平台下会比较慢，所以建议创建菜单项时在 `option` 中传入。

## item.click

获取或设置菜单项的回调函数。回调函数会在用户激活菜单项（鼠标点击或键盘快捷键）时调用。

## item.key

一个代表菜单项快捷键的字符串。

### 全平台有效的键

* 字母：`a`-`z`
* 数字：`0`-`9`
* 盘区上的其他键：`[`、`]`、`'`、`,`、`.`、`/`、`` ` ``、`-`、`=`、`\`、`'`、`;`、`Tab`
* `Esc`
* `Down`、`Up`、`Left`、`Right`
* [W3C DOM Level 3 KeyboardEvent Key Values](http://www.w3.org/TR/DOM-Level-3-Events-key/)：`KeyA`（同 `A`）、`Escape`（同 `Esc`）、`F1`、`ArrowDown`（同 `Down`）等。

### Mac 专属的特殊键
在 Mac 上，您还可以通过 `String.fromCharCode(specialKey)` 使用特殊的键作为快捷键。以下是一些有用的键：

* **28**：Left（<kbd>&larr;</kbd>）
* **29**：Right（<kbd>&rarr;</kbd>）
* **30**：Up（<kbd>&uarr;</kbd>）
* **31**：Down（<kbd>&darr;</kbd>）
* **27**：Escape（<kbd>&#9099;</kbd>）
* **11**：PageUp（<kbd>&#8670;</kbd>）
* **12**：PageDown（<kbd>&#8671;</kbd>）

Mac 上支持的所有特殊键参考 [NSMenuItem.keyEquivalent](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ApplicationKit/Classes/NSMenuItem_Class/#//apple_ref/occ/instp/NSMenuItem/keyEquivalent) 和 [NSEvent: Function-Key Unicodes](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ApplicationKit/Classes/NSEvent_Class/index.html#//apple_ref/doc/constant_group/Function_Key_Unicodes)。

## item.modifiers

一个代表菜单项修饰快捷键的字符串。必须是以下字符串的串联：`cmd` / `command` / `super`、`shift`、`ctrl`、`alt`。例如 `"cmd+shift+alt"`。

`cmd` 在不同的平台上代表不同的键：在 Windows 和 Linux 上为 Windows 键（<kbd>Windows</kbd>），在 Mac 上为 Apple 键（<kbd>&#8984;</kbd>）。`super` 和 `command` 是 `cmd` 的别名。

## 事件：click

用户激活（鼠标点击或键盘快捷键）菜单项时触发。
