# Menu（菜单） {: .doctitle}
---

[TOC]

`Menu` 显示为菜单栏或上下文菜单（通常作为右键菜单），`MenuItem` 则为菜单项。参考 [`MenuItem`](MenuItem.md)。

## 示例

一个创建上下文菜单的例子：
```javascript
// 创建一个空白的上下文菜单
var menu = new nw.Menu();

// 添加一些菜单项
menu.append(new nw.MenuItem({ label: 'Item A' }));
menu.append(new nw.MenuItem({ label: 'Item B' }));
menu.append(new nw.MenuItem({ type: 'separator' }));
menu.append(new nw.MenuItem({ label: 'Item C' }));

// 移除指定索引的菜单项
menu.removeAt(1);

// 弹出上下文菜单
menu.popup(10, 10);

// 遍历菜单项
for (var i = 0; i < menu.items.length; ++i) {
  console.log(menu.items[i]);
}
```

要创建一个菜单栏，通常需要创建一个 2 级的菜单，并将其赋值给 `win.menu`。这是一个创建菜单栏的例子：
```javascript
// 创建一个空白的菜单栏
var menu = new nw.Menu({type: 'menubar'});

// 创建一个子菜单作为二级菜单
var submenu = new nw.Menu();
submenu.append(new nw.MenuItem({ label: 'Item A' }));
submenu.append(new nw.MenuItem({ label: 'Item B' }));

// 将上面的二级菜单作为子菜单，创建一个一级菜单项，并添加到菜单栏
menu.append(new nw.MenuItem({
  label: 'First Menu',
  submenu: submenu
}));

// 把菜单栏赋值给 `window.menu`，这样就能在窗口上显示出来了
nw.Window.get().menu = menu;
```

参考 [自定义菜单栏](../For Users/Advanced/Customize Menubar.md) 查看更多使用说明。

!!! warning "页面导航与菜单的使用"
    在页面中创建的菜单，在页面刷新或跳转后会失效，是因为菜单乃至网页在导航后都会被 JS 引擎垃圾回收，以防止内存泄漏。所以建议在 **后台页面** 中使用菜单，可以存在于应用的整个生命周期。参考 [`bg-script`](Manifest Format.md#bg-script) 和 [`main`](Manifest Format.md#main) 查看如何在后台页面中执行脚本。

## new Menu([option])

* `option` `{Object}` _Optional_
    - `type` `{String}` _Optional_ 有 2 个可选类型："menubar" 或 "contextmenu"。默认为 "contextmenu"。

创建一个 `Menu` 实例。

## menu.items

获取所有的菜单项。每个菜单项都是一个 `MenuItem` 实例。参考 [MenuItem](MenuItem.md)。

## menu.append(item)

* `item` `{MenuItem}` 要添加到菜单尾部的菜单项

添加 `item` 到菜单尾部。

## menu.insert(item, i)

* `item` `{MenuItem}` 要插入的菜单项
* `i` `{Integer}` 菜单中的索引

在菜单的第 `i` 个位置插入 `item`。`i` 为从 0 开始的索引值。

## menu.remove(item)

* `item` `{MenuItem}` 要移除的菜单项

从菜单中移除 `item`。您需要在 `Menu` 外部保留有 `MenuItem` 的引用才能使用该方法。

## menu.removeAt(i)

* `i` `{Integer}` 菜单中的索引

移除菜单中的第 `i` 个菜单项。

## menu.popup(x, y)

* `x` `{Integer}` 菜单的 x 位置
* `y` `{Integer}` 菜单的 y 位置

在当前窗口的 (`x`, `y`) 位置弹出上下文菜单。只对 `contextmenu` 类型的菜单有效。

通常您需要侦听 DOM 元素的 `contextmenu` 事件，然后手动弹出菜单：

```javascript
document.body.addEventListener('contextmenu', function(ev) { 
  ev.preventDefault();
  menu.popup(ev.x, ev.y);
  return false;
});
```

通过该方式，您可以为不同的元素选择适当的菜单进行显示，也可以在菜单弹出之前更新菜单元素。

## menu.createMacBuiltin(appname, [options]) (Mac)

* `appname` `{String}` 应用名
* `options` `{Object}` _Optional_
    - `hideEdit` `{Boolean}` _Optional_ 是否隐藏 Edit 菜单
    - `hideWindow` `{Boolean}` _Optional_ 是否隐藏 Window 菜单

在 Mac 的菜单栏中创建内建菜单（*App*、*Edit* 和 *Window*）。菜单项可以通过 `items` 属性进行操作。`appname` 参数用于 *App* 菜单的标题。

您也可以和其它菜单项一起使用内建菜单，如给菜单添加或插入菜单项仍是有效的。

参考 [自定义菜单栏](../For Users/Advanced/Customize Menubar.md#mac-os-x) 查看更多使用说明。

## 译者注：实用代码片段

这个是译者常用的一个代码片段：

```javascript
function createMenu(items, option = { type: "contextmenu" }) {
    const menu = new nw.Menu(option);
    items.forEach((item) => {
        if (item.submenu) item.submenu = createMenu(item.submenu);
        menu.append(new nw.MenuItem(item));
    });
    return menu;
}
```

创建菜单：

```
const menu = createMenu([
    {
        label: "菜单项 1",
        submenu: [
            { label: "子菜单项 1-1" },
            { label: "子菜单项 1-2" },
        ]
    },
    {
        label: "菜单项 2",
        submenu: [
            { label: "子菜单项 2-1" },
            { label: "子菜单项 2-2" },
        ]
    },
]);
window.addEventListener("contextmenu", e => {
    e.preventDefault();
    menu.popup(e.clientX, e.clientY);
});
```

