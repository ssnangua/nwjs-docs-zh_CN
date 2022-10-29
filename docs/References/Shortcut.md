# Shortcut（快捷键） {: .doctitle}

---

[TOC]

`Shortcut` 是全局的快捷键（系统级热键），成功注册后，即使您的应用 *没有* 获得焦点，也仍然会生效。

`Shortcut` 继承自 [`EventEmitter`](https://nodejs.org/api/events.html#events_class_events_eventemitter)，每次用户按下注册的快捷键，该快捷键的实例就会接收到一个 `active` 事件。

## 示例

```js
var option = {
  key : "Ctrl+Shift+A",
  active : function() {
    console.log("全局桌面快捷键：" + this.key + "被触发"); 
  },
  failed : function(msg) {
    // :(, |key| 注册失败，或无法解析 |key|
    console.log(msg);
  }
};

// 使用 |option| 创建一个快捷键
var shortcut = new nw.Shortcut(option);

// 注册一个全局桌面快捷键，即使应用没有获得焦点也能生效
nw.App.registerGlobalHotKey(shortcut);

// 如果 |shortcut| 注册成功，当用户按下 "Ctrl+Shift+A" 时，|shortcut| 就会接收到一个 "active" 事件

// 您也可以侦听快捷键的触发和失败事件
shortcut.on('active', function() {
  console.log("全局桌面快捷键：" + this.key + "被触发"); 
});

shortcut.on('failed', function(msg) {
  console.log(msg);
});

// 注销全局桌面快捷键
nw.App.unregisterGlobalHotKey(shortcut);
```

## new Shortcut(option)

* `option` `{Object}`
    - `key` `{String}` 代表快捷键的串联字符串，如 `"ctrl+shift+a"`。参考 [shortcut.key](#shortcutkey) 属性。
    - `active` `{Function}` _Optional_ 快捷键被成功触发时的回调函数。参考 [shortcut.active](#shortcutactive) 属性。
    - `failed` `{Function}` _Optional_ 快捷键注册失败时的回调函数。参考 [shortcut.failed](#shortcutfailed) 属性。

创建一个新的快捷键实例，`option` 是一个包含快捷键初始设置的对象。

## shortcut.key

获取快捷键的 `key`。这是一个代表快捷键的串联字符串，由任意个修饰键和一个主键组成，如 `"Ctrl+Alt+A"`。 可以只有一个主键。该值大小写不敏感。

支持的修饰键：

* `Ctrl`
* `Alt`
* `Shift`
* `Command`: `Command` 在 Mac 上为 Apple 键 (<kbd>&#8984;</kbd>)，在 Windows 和 Linux 上为 Windows 键。

支持的主键：

* 字母：`A`-`Z`
* 数字：`0`-`9`
* 功能键：`F1`-`F24`
* `Comma`
* `Period`
* `Tab`
* `Home` / `End` / `PageUp` / `PageDown` / `Insert` / `Delete`
* `Up` / `Down` / `Left` / `Right`
* `MediaNextTrack` / `MediaPlayPause` / `MediaPrevTrack` / `MediaStop`
* `Comma` 或 `,`
* `Period` 或 `.`
* `Tab` 或 `\t`
* `Backquote` 或 `` ` ``
* `Enter` 或 `\n`
* `Minus` 或 `-`
* `Equal` 或 `=`
* `Backslash` 或 `\`
* `Semicolon` 或 `;`
* `Quote` 或 `'`
* `BracketLeft` 或 `[`
* `BracketRight` 或 `[`
* `Escape`
* [DOM Level 3 W3C KeyboardEvent Code Values](http://www.w3.org/TR/DOM-Level-3-Events-code/)

!!! warning "不带修饰键的单键快捷键"
    `App.registerGlobalHotKey()` 支持注册只有一个主键的单键快捷键（如 `{ key: "A" }`），但在注销该快捷键之前，用户将无法正常使用 "A" 键。当然，这在某些场景下会很有用，如监听多媒体按键。
    **在使用不带修饰键的快捷键之前，请确保这就是您需要的**

## shortcut.active

获取或设置快捷键的 `active` 回调函数。该函数会在用户按下快捷键时调用。

## shortcut.failed

*获取或设置快捷键的 `failed` 回调函数。该函数会在解析快捷键的 `key` 失败或快捷键注册失败时调用。

## 事件：active

同 [`shortcut.active`](#shortcutactive)

## 事件：failed

同 [`shortcut.failed`](#shortcutfailed)
