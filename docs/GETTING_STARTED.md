# 入门指南

这个快速入门指南将向你展示如何使用Node.js和MongoDB编写一个简单的应用。它的功能范围仅包括如何设置一个驱动程序并且运行简单的<abbr title="增加 查看 修改 删除">crud</abbr>操作。

安装 monk
---------------------------
使用 **NPM** 来安装 `monk`.

```
npm install --save monk
```

启动一个MongoDB服务
---------------------------
让我们启动一个MongoDB服务实例。从 [MongoDB](http://www.mongodb.org) 下载正确版本的MongoDB，打开一个新的Shell或者命令行窗口，并且确保 **mongod** 命令在shell或者命令行的路径里面。现在，让我们启动一个数据库目录（在这个的例子中，放在 **/data** 目录下）。

```
mongod --dbpath=/data --port 27017
```

你应该看到 **mongod** 正在启动，并且打印出了一些状态信息。

连接到MongoDB
---------------------
让我们创建一个名为 **app.js** 的文件，我们将使用它来展示monk如何进行基本的CRUD（增删改查）操作。

首先，让我们添加一写代码，连接到服务和数据库 **myproject**。

```js
const monk = require('monk')

// 连接 URL
const url = 'localhost:27017/myproject';

const db = monk(url);

db.then(() => {
  console.log('正确的连接到了服务')
})
```

假设你在此之前已经启动了 **mongod** 进程，这个应用应该连接成功，并且在控制台里打印出 **正确的连接到了服务**。

如果你不了解 `then` 和 `Promises`，你可能需要先阅读相关的指南。

让我们添加一些代码，展示不同的CRUD操作都可以成功运行。

插入文档（documents，类似关系型数据库中的行）
--------------------
让我们在 `documents` 集合（collection，类似关系型数据库中的表）里插入一些文档。

```js
const url = 'localhost:27017/myproject'; // 连接到URL
const db = require('monk')(url);

const collection = db.get('document')

collection.insert([{a: 1}, {a: 2}, {a: 3}])
  .then((docs) => {
    // docs包含了刚才插入（insert）的文档，这些文档已经被自动加入了 **_id** 字段
    // 往document集合里插入了三条文档
  }).catch((err) => {
    // 插入的时候发生了错误
  }).then(() => db.close())
```

你可以看到，我们在操作之前没有等待连接打开。这是因为在后台，monk会对所有操作进行排队，在连接打开后，才会发送它们。

我们现在可以运行更新后的 **app.js** 文件。

```
node app.js
```

**app.js** 文件运行后，你应该看到下面的输出信息：

```
Inserted 3 documents into the document collection
```

更新文档
-------------------
让我们看看其中，字段 **a** 的值为 **2** 的文档，如何增加一个新的字段 **b**：

```js
const url = 'localhost:27017/myproject'; // 连接到 URL
const db = require('monk')(url);

const collection = db.get('document')

collection.insert([{a: 1}, {a: 2}, {a: 3}])
  .then((docs) => {
    // 往document集合里插入了三条文档
  })
  .then(() => {

    return collection.update({ a: 2 }, { $set: { b: 1 } })

  })
  .then((result) => {
    // 字段a值为2的文档已经更新了
  })
  .then(() => db.close())
```

这个方法将会更新第一条字段 **a** 值为 **2** 的文档，在其中加入了新字段 **b**，并设为 **1**。

删除文档
-----------------
下一步，让我们删除字段 **a** 值为 **3** 的那一条文档：

```js
const url = 'localhost:27017/myproject'; // 连接到 URL
const db = require('monk')(url);

const collection = db.get('document')

collection.insert([{a: 1}, {a: 2}, {a: 3}])
  .then((docs) => {
    // 往document集合里插入了三条文档
  })
  .then(() => collection.update({ a: 2 }, { $set: { b: 1 } }))
  .then((result) => {
    // 字段a值为2的文档已经更新了
  })
  .then(() => {

    return collection.remove({ a: 3})

  }).then((result) => {
    // 删除字段a值为3的那一条文档
  })
  .then(() => db.close())
```

这会删除字段 **a** 值为 **3** 的第一条文档.

查找所有文档
------------------
我们将运行一个简单的查询，返回所有符合查询条件的文档，来完成这样一个CRUD的方法：

```js
const url = 'localhost:27017/myproject'; // 连接到 URL
const db = require('monk')(url);

const collection = db.get('document')

collection.insert([{a: 1}, {a: 2}, {a: 3}])
  .then((docs) => {
    // 往document集合里插入了三条文档
  })
  .then(() => collection.update({ a: 2 }, { $set: { b: 1 } }))
  .then((result) => {
    // 字段a值为2的文档已经更新了
  })
  .then(() => collection.remove({ a: 3}))
  .then((result) => {
    // 删除字段a值为3的那一条文档
  })
  .then(() => {

    return collection.find()

  })
  .then((docs) => {
    // docs === [{ a: 1 }, { a: 2, b: 1 }]
  })
  .then(() => db.close())
```

这个查询讲返回 **documents** 集合中的所有文档。由于我们删除了一条文档，因此返回了两条文档。

这就是使用monk来连接和运行相关基本操作的入门指南。
