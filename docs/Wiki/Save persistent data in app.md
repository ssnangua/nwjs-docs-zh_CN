# 数据持久化 {: .doctitle}
---

[TOC]

> 原文链接：https://github.com/nwjs/nw.js/wiki/Save-persistent-data-in-app

在本地应用中，存储持久化数据是非常普遍的场景，通常会通过嵌入外部数据库或操作纯文本文件来实现。在 NW.js 中，您有比这些更好的选择，您可以使用 `Web SQL 数据库`、`嵌入式数据库`、`Web 存储` 或 `应用缓存`，而无需烦恼任何额外的依赖。


## 直接保存一个纯文本文件到硬盘

NW.js 提供的 [`App.dataPath`](https://github.com/nwjs/nw.js/wiki/App) API 会返回一个路径，可以用于保存应用数据，该路径取决于系统。比如，在 Windows 上一般为 `C:\Users\用户名\AppData\Local\应用名`，在 Linux 上则为 `/home/用户名/.config/应用名`。

**完整示例：**

```js
var fs = require('fs');
var path = require('path');

var helpers = {
  file: 'my-settings-file.json',
  filePath: path.join(nw.App.dataPath, this.file),
  saveSettings: function (settings, callback) {
    fs.writeFile(this.filePath, JSON.stringify(settings), function (err) {
      if (err) {
        console.info('There was an error attempting to save your settings.');
        console.warn(err.message);
        return;
      } else if (typeof(callback) === 'function') {
        callback();
      }
    });
  },
  loadSettings: function (callback) {
    fs.readFile(this.filePath, function (err, data) {
      if (err) {
        console.info('There was an error attempting to read your settings.');
        console.warn(err.message);
      } else if (typeof(callback) === 'function') {
        try {
          data = JSON.parse(data);
          callback(data);
        } catch (error) {
          console.info('There was a problem parsing the data in your settings.');
          console.warn(error);
        }
      }
    });
  }
};

var mySettings = {
  language: 'en',
  theme: 'dark'
};

helpers.saveSettings(mySettings, function () {
  console.log('Settings saved');
});

helpers.loadSettings(function (data) {
  mySettings = data;
});
```


## Web SQL 数据库

> **译者注：**Web SQL 已 [弃用](https://developer.chrome.com/blog/deprecating-web-sql/)，不建议使用。

`Web SQL 数据库` API 实际上并不是 HTML5 规范的一部分，而是一个单独的规范，它引入了一套 API，使用 SQL 来操作客户端数据库。在下面的指南中，假设您已经熟悉了基本的数据库操作和 SQL 语言。

NW.js 中的 `Web SQL数据库` API 是用 `sqlite` 实现的，操作基本相同：

* `openDatabase`：该方法使用现有数据库或创建新数据库的方式来创建数据库对象。
* `transaction`：该方法让我们能够控制事务，并根据情况执行提交或回滚。
* `executeSql`：该方法用于执行实际的 SQL 查询。

下面的代码展示了如何创建和打开一个数据库：

```javascript
var databaseName = 'mydb';
var versionNumber = '1.0';
var textDescription = 'my first database';
var estimatedSizeOfDatabase = 2 * 1024 * 1024;

var db = openDatabase(
  databaseName,
  versionNumber,
  textDescription,
  estimatedSizeOfDatabase
);
```

如果您尝试打开一个不存在的数据库，API 会自动创建它。您也无需担心关闭数据库的问题。

创建表、插入数据或查询数据，在 `transaction`（事务）中使用 `executeSql`：

```javascript
// 创建一张表并插入一行
db.transaction(function (transaction) {
  transaction.executeSql('CREATE TABLE IF NOT EXISTS foo (id unique, text)');
  transaction.executeSql('INSERT INTO foo (id, text) VALUES (1, "synergies")');
  transaction.executeSql('INSERT INTO foo (id, text) VALUES (2, "luyao")');
});

// 查询数据
db.transaction(function (transaction) {
  transaction.executeSql('SELECT * FROM foo', [], function (transaction, results) {
    var i;
    var length = results.rows.length;
    for (i = 0; i < length; i++) {
      alert(results.rows.item(i).text);
    }
  });
});
```

更多信息，可以阅读 [Introducing Web SQL Databases](http://html5doctor.com/introducing-web-sql-databases) 等教程。


## IndexedDB

[IndexedDB](https://developer.mozilla.org/en-US/docs/IndexedDB) 被视为 WebSQL 的 NoSQL 接替者，受到最新版本的 Chromium 的支持，所以 NW.js 也支持。基于键值存储 LevelDB 的实现。

IndexedDB 的 API 是异步的，且相对底层和复杂，所以您可能会更愿意使用对它进一步封装的库，如 PouchDB。


## PouchDB

[PouchDB](http://pouchdb.com) 是一个受 CouchDB 启发的嵌入式数据库引擎。在 NW.js 中，它可以作为 IndexedDB 或 WebSQL 的封装（通过 Chromium 实现）使用，或直接用于 LevelDB（通过 node.js 模块）。

与 CouchDB 一样，没有基于动态索引的查询，不过可以通过 map/reduce 动态聚合一个视图。它可以直接复制到 CouchDB 或从 CouchDB 复制，如果您正在构建一个需要云同步的应用，这将会是一个很大的优势。

**要在 NW.js 中使用 PouchDB**，可以查看 [pouchdb-nw](https://github.com/nolanlawson/pouchdb-nw)。

其他参考：

* https://github.com/pouchdb-community/relational-pouch
* https://www.npmjs.com/package/pouchdb-find


## EJDB

[EJDB](https://github.com/Softmotions/ejdb)（Embedded JSON Database engine，嵌入式 JSON 数据库引擎）是一个基于 Tokyo Cabinet 的简单快速的数据库引擎。它的用法复制了 MongoDB，您可以轻松地进行动态查询并对结果进行排序/分页。

快速查询和易于使用的 API 让它成为 NW.js 非常好的选择。

**注意：**在 NW.js 应用中使用 EJDB 需要一个额外的步骤，参阅 [使用包含 C/C++ 插件的第三方模块](../For Users/Advanced/Use Native Node Modules.md)。


## NeDB

[NeDB](https://github.com/louischatriot/nedb)（Node embedded database，Node 嵌入式数据库）是一个用于 Node.js 的纯 JavaScript 数据库（与 EJDB 不同，您不需要编译任何东西）。它实现了 MongoDB 最常见的子集，可用于持久化数据，或仅用作内存中的数据存储。虽然不是原生模块，但对于桌面应用来说，它已然足够快（读取 40k/s，写入 10k/s）。


## LinvoDB

[LinvoDB](https://github.com/Ivshti/linvodb3) 是一个用于 Node.js / NW.js 的持久化嵌入式数据库。它可以用于 LevelDB，也可以用于 Medea，无需编译任何东西。它具有类似于 MongoDB+Mongoose 的特性和 API。性能与 MongoDB 相当。


## MarsDB

[MarsDB](https://github.com/c58/marsdb) 是一个轻量的数据库，适用于任何类型的 JS 环境（浏览器、Node、NW.js），基于 Meteor 的 minimongo，使用 ES6 Promises 精心编写。它可以保存在任何类型的存储中（内存、LocalStorage、LevelUP，等等），并且很容易实现您自己的存储管理器（多亏了 Promises）。


## SQLite3

使用 [better-sqlite3](https://www.npmjs.com/package/better-sqlite3) 以兼容 NW.js。使用示例可以参考这个 [Angular 项目](https://github.com/vatsalkgor/nw-sqlite3-error)。


## LowDB

[LowDB](https://github.com/typicode/lowdb) 是一个用于 Node.js 的扁平 JSON 文件数据库。它使用 Lodash 提供的函数式编程 API。


## StoreDB

[StoreDB](https://github.com/djyde/StoreDB) 是一个 **基于 localStorage** 的本地数据库。它提供 MongoDB 风格的 API，并使用 `collection`、`document` 等概念，允许使用 localStorage 来存储复杂数据。

存储数据超级简单且友好：

```javascript
// 插入数据
var player = { name: 'Randy', sex: 'male', score: 20 };
storedb('players').insert(player, function (err, result) {
  if (err) {
    console.info('Error inserting data in StoreDB');
    console.warn(err);
  } else {
    console.log('Data was inserted into StoreDB');
  }
})

// 更新数据
var player = { name: 'Randy' };
var instructions = { '$inc': { score: 10 } };
storedb('players').update(player, instructions, function (err) {
  if (err) {
    console.info('Error updating data in StoreDB');
    console.warn(err);
  } else {
    console.log('Data was updated in StoreDB');
  }
})
```


## Web 存储

Web 存储是一种易于使用的键值数据库。您可以像普通的 JS 对象一样使用它，但所有内容都会保存到硬盘。

Web 存储有两种类型：

* `localStorage` - 存储没有过期时间的数据
* `sessionStorage` - 存储单一会话的数据

`localStorage` 对象存储没有过期时间的数据，数据不会在浏览器关闭时被删除，并且在第二天、下周甚至明年都是可用的。

```javascript
localStorage.love = 'luyao';

// 爱没有终点
console.log(localStorage.love);
```

`sessionStorage` 对象类似于 `localStorage` 对象，只是它只存储单一会话的数据，当用户关闭窗口时，数据将被删除。

```javascript
sessionStorage.life = '';

// 但生命却有尽头
console.log(sessionStorage.life);
```

**警告：**对于大型数据集，Web 存储是非常不适用的，因为 API 是同步的，没有高级索引/查询（只有一个键值存储），并且值只能是一个字符串。


## 应用缓存

HTML5 引入了应用缓存，被缓存的 Web 应用，在没有连接网络的情况下可以正常访问。

应用缓存给应用带来了三个优势：

* 离线浏览 - 用户可以在离线时使用该应用
* 加载速度 - 缓存的资源加载更快
* 降低服务器负载 - 浏览器只会从服务器下载变更的资源

不过，应用缓存是​​为浏览器而设计的，对于 NW.js 应用来说，它的用处不如其他方案，如果您想使用它，可以参考 [HTML5 Application Cache](http://www-db.deis.unibo.it/courses/TW/DOCS/w3schools/html/html5_app_cache.asp.html)。
