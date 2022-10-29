# ChromeDriver 自动化测试 {: .doctitle}
---

[TOC]

引用自 [ChromeDriver 项目主页](https://sites.google.com/a/chromium.org/chromedriver/)：

> WebDriver 是一个开源工具，用于对 web 应用进行跨浏览器的自动化测试。它提供了页面跳转、用户输入、执行 JavaScript 等功能。
> ChromeDriver 是一个独立的服务，为 Chromium 实现 WebDriver 的线协议。ChromeDriver 适用于安卓版 Chrome 和桌面版 Chrome（Mac、Linux、Windows 和 ChromeOS）。

NW.js 提供了一个定制的 ChromeDriver，用于对基于 NW.js 的应用进行自动化测试。可以和 [selenium](http://docs.seleniumhq.org/) 等工具配合使用。

## 开始

下面的工作流使用 [selenium-python](http://selenium-python.readthedocs.org/) 来驱动测试，您也可以使用 Selenium 的其他语言版本来调用 `chromedriver`。

### 安装

* 从 NW.js 官网下载 ChromeDriver，它包含在 SDK 版本中。
* 解压程序包，将 `chromedriver` 放在 NW.js 的可执行文件同目录中（NW.js 的可执行文件：Linux 为 `nw`，Windows 为 `nw.exe`，Mac 为 `node-webkit.app`）
* 在项目中安装 `selenium-python`：
```bash
pip install selenium
```

### 运行

假设你的应用显示了一个表单，用于从远程查询内容，页面的基本内容类似于：
```html
<form action="http://mysearch.com/search" method="GET">
    <input type="text" name="q"><input type="submit" value="Submit">
</form>
```

编写一个 Python 脚本，以自动填写搜索框并提交表单：
```python
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_argument("nwapp=/path/to/your/app")

driver = webdriver.Chrome(executable_path='/path/to/nwjs/chromedriver', chrome_options=chrome_options)

time.sleep(5) # 等待 5 秒以显示页面
search_box = driver.find_element_by_name('q')
search_box.send_keys('ChromeDriver')
search_box.submit()
time.sleep(5) # 等待 5 秒以查看查询结果
driver.quit()
```

参考 http://selenium-python.readthedocs.org/ 查看 `selenium-python` 的详细文档。

## 对 chromedriver 的定制

* NW.js 定制的 chromedriver 默认会查找同目录下的 NW 可执行文件

* 想要向命令行传递非开关类的参数，可以添加一个额外的 `nwargs` 选项：
```python
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_argument("nwapp=/path/to/your/app")
chrome_options.add_experimental_option("nwargs", ["arg1", "arg2"])

driver = webdriver.Chrome(executable_path='/path/to/nwjs/chromedriver', chrome_options=chrome_options)
```