# 文件变更实时重新加载 {: .doctitle}
---

[TOC]

> 原文链接：https://github.com/nwjs/nw.js/wiki/Livereload-nw.js-on-changes

在开发阶段，如果文件变更时能自动重新加载 NW.js 窗口，会有更好的开发体验。

## 嵌入式方案

[nw-dev](https://www.npmjs.com/package/nw-dev)

## 简单方案

在主文件的末尾加上下面的 script 标签：

```html
<script>
  var path = './';
  var fs = require('fs');
  
  var reloadWatcher = fs.watch(path, function() {
    location.reload();
    reloadWatcher.close();
  });
</script>
```

### 重新加载 Node 模块

通过 `require()` 加载的 Node 模块会缓存在 `require.cache` 里，如果这些模块需要和页面一起被重新加载，把下面代码添加到 reloadWatcher 侦听器里：

```javascript
    for(var i in require.cache) {
      delete require.cache[i];
    }
```

上面的代码只监视当前目录里的文件（不包含子目录）。

## 递归方案

可以在上面的示例中添加一个选项来监视所有子目录：`fs.watch(path, { recursive: true }, listener)`。`recursive` 选项目前仅支持 OS X 和 Windows。

如果需要更完善的解决方案，需要先安装 `gaze`、`chokidar` 或 `gulp`，然后更改 script 标签的内容：

### gaze

`npm install gaze`

```html
<script>
  var Gaze = require('gaze').Gaze;
  var gaze = new Gaze('**/*');

  gaze.on('all', function(event, filepath) {
    if (location)
      location.reload();
  });
</script>
```

### chokidar

`npm install chokidar`

```html
<script>
  var chokidar = require('chokidar');
  var watcher  = chokidar.watch('.', {ignored: /[\/\\]\./});
  watcher.on('all', function(event, path) {
    if (location && event == "change")
      location.reload();
  });
</script>
```

### gulp

`npm install gulp`

```html
<script>
  var gulp = require('gulp');
  gulp.task('reload', function () {
    if (location) location.reload();
  });

  gulp.watch('**/*', ['reload']);
</script>
```

或者，如果您不想在编辑样式时重新加载整个页面，可以在 gulp 中为 css 文件单独附加一个样式任务。

```html
<script>
  var gulp = require('gulp');
  
  gulp.task('html', function () {
    if (location) location.reload();
  });

  gulp.task('css', function () {
    var styles = document.querySelectorAll('link[rel=stylesheet]');

    for (var i = 0; i < styles.length; i++) {
      // 重新加载样式
      var restyled = styles[i].getAttribute('href') + '?v=' + Math.random(0, 10000);
      styles[i].setAttribute('href', restyled);
    };
  });

  gulp.watch(['**/*.css'], ['css']);
  gulp.watch(['**/*.html'], ['html']);
</script>
```

### `<script>` `src` 格式

允许 `file:` URL：https://github.com/livereload/livereload-js#using-livereloadjs

```js
[
  'http://',
  (location.host || 'localhost').split(':')[0],
  ':35729/livereload.js'
].join('')
```

## 清除 Node 模块缓存

如果想要重新加载 Node 模块，需要清除全局的 require 缓存（`global.require.cache`），参考 [这个 StackOverflow 提问](http://stackoverflow.com/questions/25143532/node-webkit-clear-cache)。注意，这会移除缓存中的所有模块，可能包括 `node-main` 模块：

```javascript
// 在 Gulp 中定义一个新的 `reload` 任务，简单的刷新当前页面
gulp.task('reload', function() {
	// 重新加载时在控制台输出信息
	console.log('Reloading app...');
	// 清除 NW.js 的全局 require 模块缓存
	// 参考：http://stackoverflow.com/q/25143532/
	for (let module_name in global.require.cache) {
		delete global.require.cache[module_name];
  }
	// 刷新页面
	window.location.reload();
});

// 设置 Gulp 监视所有文件，并在监测到变更时重新加载页面
gulp.watch('**/*', ['reload'])
```

> **译者注：**其实，如果应用比较复杂，一般会使用脚手架来开发，这些脚手架本身就自带 **热加载** 功能，如 vite、vue-cli 等。开发时将应用入口设置为本地服务器地址，打包时再替换为构建后的相对路径。