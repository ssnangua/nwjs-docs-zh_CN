# 为 NW.js 编写测试用例 {: .doctitle}
---

[TOC]

## 测试框架

NW.js 使用一个小巧的基于 Python 的测试框架，只有 3 个文件，您可以在 [`test`](https://github.com/nwjs/nw.js/tree/nw13/test) 目录中查看源代码。

NW.js 中的每个测试用例都是一个可以运行的应用，所以您可以脱离框架手动运行。

在 NW.js 中，测试用例有两种类型：`自动` 和 `远程`。参考下面 [编写测试用例](#编写测试用例) 部分的说明。

通过以下命令来运行测试用例：

```bash
python test/test.py -t 80 auto
python test/test.py -t 80 remoting
```

<span id="编写测试用例"></span>
## 编写测试用例

### 自动测试用例

**TODO**

### 远程测试用例

远程测试用例由 ChromeDriver 驱动，通常这些测试用例涉及到用户交互，参考[ChromeDriver 测试](../For Users/Advanced/Test with ChromeDriver.md)。

远程测试用例需要应用根目录中有一个 `test.py` 文件。

这是 `test.py` 文件的模板：

```python
import time
import os

from selenium import webdriver
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_argument("nwapp=" + os.path.dirname(os.path.abspath(__file__)))

driver = webdriver.Chrome(executable_path=os.environ['CHROMEDRIVER'], chrome_options=chrome_options)
time.sleep(1)
try:
    print driver.current_url
    # 测试代码写在这里（使用 `assert`）
finally:
    driver.quit()
```

真实用例是一个 NW.js 应用，通过 ChromeDriver 可以模拟鼠标点击事件和输入动作，然后获得一些 DOM 元素的内容。例如下面的代码，点击“点我”按钮，一个内容为“成功”的新 DOM 元素将被添加到 document 中。

```html
<button id="clickme" onclick="success()">点我</button>
<script>
function success() {
    var el = document.createElement('div');
    el.id = 'result';
    el.innerHTML = '成功';
    document.body.appendChild(el);
}
</script>
```

然后您可以在 `test.py` 脚本中进行测试：

```python
driver.implicitly_wait(10) # 查找元素超时为 10 秒

clickme = driver.find_element_by_id('clickme')
clickme.click() # 点击按钮

result = driver.find_element_by_id('result')
assert("成功" in result.get_attribute('innerHTML')) # 断言 "成功" 在元素中
```
## 运行 Nightly 测试用例

测试之前需要做的准备：

1.下载 NW SDK 版本。

2.从 git 下载用于 NW 的 selenium python 库。
仓库地址和 git commit id 在根目录的 `DEPS` 文件 [管理的依赖](https://github.com/nwjs/nw.js/blob/nw35/DEPS) 里（如果测试的不是 0.35 版本，把 `nw35` 替换为对应的分支名）。
在文件的 deps 部分查找 'third_party/webdriver/pylib'，找到下面这种格式：
```js
 'src/third_party/webdriver/pylib':
    Var('chromium_git') + '/external/selenium/py.git' + '@' + 'expected commit id',
``` 
然后在文件的 vars 部分查找 Var（如 `chromium_git`）的值。
克隆 pylib 仓库后，确保 commit id 与 expected commit id 相同。

3.设置环境变量。
设置 `CHROMEDRIVER` 为 NW SDK 中 chromedriver.exe 的路径，设置 `PYTHONPATH` 为克隆下来的 'py' 文件夹的路径。

4.安装 Python 2.7（如果未安装的话）

5.克隆 nw 源码仓库：[nwjs](https://github.com/nwjs/nw.js).

然后进入 `nw仓库目录/test`，运行：
```bash
python test.py -d <path to NW SDK> -t 60 sanity
```
