# NW.js 中文文档

> **非官方中文文档**，当前版本：nw69。
> 中文文档源码：https://github.com/ssnangua/nwjs-docs-zh_CN
> 官方文档：https://nwjs.readthedocs.io/
> 官方仓库：https://github.com/nwjs/nw.js

## 离线查看文档

1. 安装 [Python](https://www.python.org/) 环境
2. 安装依赖
    - Python 2: `pip install mkdocs==0.17.3 pygments pymdown-extensions==6.2.1`
    - Python 3: `pip install mkdocs pygments pymdown-extensions`
3. 如果安装的是 Python 3，按下面的 [说明](Python-3-说明) 修改 `mkdocs.yml`，Python 2 无需处理
4. 文档源码根目录运行 `mkdocs serve`
5. 在浏览器中打开 http://localhost:8000/

### Python 3 说明
打开 `mkdocs.yml`，按下面的修改：
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
