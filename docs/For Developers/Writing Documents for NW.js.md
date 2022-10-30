# 为 NW.js 编写文档 {: .doctitle}

[TOC]

---

## 编写新文档之前

Github 自带的 GFM 语法（Github 风格 Markdown）为线上 Markdown 提供了很好的展示效果，也有很多的开发者直接在 GitHub 上查看文档，为了所有开发者都能有最好的阅读体验，**在提交 PR 前，请确保文档在 GitHub 上的可读性**。

NW.js 文档站点由 [MkDocs](http://www.mkdocs.org/) 生成 , MkDocs 的 Markdown 语法与 GFM 略有不同，一些 MkDocs 支持的写法在 GitHub 上不支持，反之亦然。
比如 `[Build NW.js](Build NW.js)` 这个不带 `.md` 后缀的内链，在 MkDocs 中可以正常跳转到 `Build NW.js.md`，但在 GitHub 上则是一个无效链接，会跳转到不存在 `Build NW.js`。内链请始终带上 `.md` 后缀。

## 离线查看文档
要在本地设备上查看文档，您需要 [Python](https://www.python.org/) 环境，并安装以下依赖：
```bash
pip install mkdocs pygments pymdown-extensions
```
在 NW.js 源码根目录运行以下指令：
```bash
mkdocs serve
```
然后在浏览器中打开 http://localhost:8000/。

### 译者注
译者按照上面的步骤试了下，出现了一些问题，应该是部分依赖的新版本不兼容导致（上面的指令默认会安装最新版本的依赖）。下面是解决方法：

#### python2
查看官方文档最新的 [构建日志](https://readthedocs.org/api/v2/build/17627167.txt) 可以看到用的是 `python2`，`MkDocs` 版本是 `0.17.3`，再参考 NW.js 源码根目录下的 `requirements.txt` 获得 `pymdown-extensions` 版本是 `6.2.1`，所以安装依赖的指令改为：
```bash
pip install mkdocs==0.17.3 pygments pymdown-extensions==6.2.1
```
后面步骤一样。

#### python3
打开 NW.js 源码根目录的 `mkdocs.yml`，按下面的修改（新版本的 MkDocs 配置格式发生了一些变化）：
```yml
site_name: NW.js Documentation
# theme: readthedocs
# theme_dir: 'custom_theme'
theme:
    name: readthedocs
    custom_dir: custom_theme
...
# pages:
nav:
...
```
后面按官方说明操作即可。

> - 如果电脑上同时安装了 python2 和 python3，可通过 `pip3 install mkdocs pygments pymdown-extensions` 来强制使用 python3 安装 MkDocs
> - 如果之前已经在 python2 上安装过 MkDocs，执行 `pip uninstall mkdocs` 卸载掉，不然 `mkdocs serve` 运行的可能是 python2 的版本。可以执行 `mkdocs -V` 查看一下 MkDocs 的路径。

## API 页面模板

下面是一个最简单的 NW.js API 页面模板：
```markdown
# 模块名 {: .doctitle}

---

[TOC]

这里是模块的说明和其他细节。

## Module.staticMethod(arg1, arg2)

* `arg1` `{String}` 参数描述
* `arg2` `{Object}` 参数描述
  * `isSet` `{Boolean}` 子字段描述
* Returns `{Type}` 返回值描述

方法的详细说明。

## inst.method(arg1, arg2)

## Event: eventName(arg1, arg2)
```

* **[必须]** 使用 `#` 作为页面标题，使用 `##` 和 `###` 作为段落的头部 **而不是** `=======` 或 `-------`。
* **[必须]** 页面标题后面添加 `{: .doctitle}`，这是文档标题的 CSS 类名。
* **[必须]** 在页面标题后面用 `---` 添加一条水平分隔线。
* **[必须]** 在水平面分隔线后面添加一个 `[TOC]`。
* 代码 **[必须]** 用反单引号（`` ` ``）括起来，除非是标题里的代码。
* 代码块 **[必须]** 指定具体的语言。
* 标题中的实例方法或属性 **[必须]** 使用小写的实例名，比如应该写为 `win.resizeTo(x,y)`，而不是 `Window.resizeTo(x,y)`。
* 标题中的事件 **[必须]** 写为 **Event: eventName(args...)**
* 方法 **[必须]** 必须提供参数和返回值说明，包括参数名、用大括号包裹的参数类型（{类型}）、以斜体表示是否是可选参数（_Optional_）。

## 提交前的检查清单

* [ ] 符合 [API 页面模板](#api) 规范
* [ ] 确保新页面有被添加到 `mkdocs.yml`
* [ ] 确保新页面在 GitHub 上的可读性
* [ ] 确保文档里的链接都是有效的
* [ ] 把您的名称和邮件添加到 [文档贡献者](Contributors of Documents.md)

## Markdown 扩展
NW.js 文档站点启用了以下扩展用于生成文档。参考根目录下的 `mkdocs.yml` 查看详细配置。

* markdown.extensions.toc
* markdown.extensions.admonition
* markdown.extensions.smarty
* markdown.extensions.nl2br
* markdown.extensions.codehilite
* pymdownx.extra
* pymdownx.inlinehilite
* pymdownx.magiclink
* pymdownx.tilde
* pymdownx.caret
* pymdownx.smartsymbols
* pymdownx.githubemoji
* pymdownx.tasklist
* pymdownx.progressbar
* pymdownx.headeranchor
* pymdownx.arithmatex
* pymdownx.mark
* pymdownx.critic


