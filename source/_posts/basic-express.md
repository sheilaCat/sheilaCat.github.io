title: Express4.x入门
date: 2015-09-22 19:28:31
tags: [Express,Node.js]
---
## 概述
Express，基于`Node.js`平台，快速、开放、极简的web开发框架。

![img](http://thumbsnap.com/i/sEtWabU1.png?0922)

<!--more-->
性能方面，express不对Node.js已有的特性进行二次抽象，只是在它之上扩展了Web应用所需的基本功能。

本篇记录了Express 4中文API中的一些重点部分，作为Express大体的入门参考，了解Express的主要组成。

## 入门

### express应用生成器

```javascript
$ npm install express-generator -g
```

### 路由

路由（Routing）是由一个 URI（或者叫路径）和一个特定的 HTTP 方法（GET、POST 等）组成的，涉及到应用如何响应客户端对某个网站节点的访问。

```javascript
app.get('/', function (req, res) {
  res.send('Hello World!');});
```

路由路径可以用字符串、字符串模式、正则表达式来匹配路径。
对于字符串模式，字符 ?、+、* 和 () 是正则表达式的子集，- 和 . 在基于字符串的路径中按照字面值解释。

### 响应方法

响应方法

下表中响应对象（res）的方法向客户端返回响应，终结请求响应的循环。如果在路由句柄中一个方法也不调用，来自客户端的请求会一直挂起。

| 方法            | 描述                                                        |
|-----------------|-------------------------------------------------------------|
| res.download()  | 提示下载文件。                                              |
| res.end()       | 终结响应处理流程。                                          |
| res.json()      | 发送一个 JSON 格式的响应。                                  |
| res.jsonp()     | 发送一个支持 JSONP 的 JSON 格式的响应。                     |
| res.redirect()  | 重定向请求。                                                |
| res.render()    | 渲染视图模板。                                              |
| res.send()      | 发送各种类型的响应。                                        |
| res.sendFile    | 以八位字节流的形式发送文件。                                |
| res.sendStatus()| 设置响应状态代码，并将其以字符串形式作为响应体的一部分发送。|

### 路由方法

Express 定义了如下和 HTTP 请求对应的路由方法： get, post, put, head, delete, options, trace, copy, lock, mkcol, move, purge, propfind, proppatch, unlock, report, mkactivity, checkout, merge, m-search, notify, subscribe, unsubscribe, patch, search, 和 connect。

有些路由方法名不是合规的 JavaScript 变量名，此时使用括号记法，比如： app['m-search']('/', function ...

### app.route()

可使用 app.route() 创建路由路径的链式路由句柄。由于路径在一个地方指定，这样做有助于创建模块化的路由，而且减少了代码冗余和拼写错误。

### express.Router

可使用 express.Router 类创建模块化、可挂载的路由句柄。Router 实例是一个完整的中间件和路由系统，因此常称其为一个 “mini-app”。

下面的实例程序创建了一个路由模块，并加载了一个中间件，定义了一些路由，并且将它们挂载至应用的路径上。

### 静态文件

利用 Express 托管静态文件

如果你的静态资源存放在多个目录下面，你可以多次调用 express.static 中间件：
```javascript
app.use(express.static('public'));

app.use(express.static('files'));
```
访问静态资源文件时，express.static 中间件会根据目录添加的顺序查找所需的文件。

如果你希望所有通过 express.static 访问的文件都存放在一个“虚拟（virtual）”目录（即目录根本不存在）下面，可以通过为静态资源目录指定一个挂载路径的方式来实现，如下所示：
```javascript
app.use('/static', express.static('public'));
```

### 如何处理 404 ？

在 Express 中，404 并不是一个错误（error）。因此，错误处理器中间件并不捕获 404。这是因为 404 只是意味着某些功能没有实现。也就是说，Express 执行了所有中间件、路由之后还是没有获取到任何输出。你所需要做的就是在其所有他中间件的后面添加一个处理 404 的中间件。如下：
```javascript
app.use(function(req, res, next) {
  res.status(404).send('Sorry cant find that!');});
```

如何设置一个错误处理器？

错误处理器中间件的定义和其他中间件一样，唯一的区别是 `4 个`而不是 3 个参数，即 (err, req, res, next)：

```javascript
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('Something broke!');});
```
请参考错误处理章节以了解更多信息。

如何渲染纯 HTML 文件？

不需要！无需通过 res.render() 渲染 HTML。你可以通过 res.sendFile() 直接对外输出 HTML 文件。如果你需要对外提供的资源文件很多，可以使用 express.static() 中间件。

## 内置方法

### express.static(root,[options]]
`express.static`是Express内置的唯一一个中间件。是基于`serve-static`开发的，负责托管Express应用内的静态资源。

## 从Express 3迁移到Express 4

实例

运行下述命令创建一个 Express 4 应用：
```javascript
$ express app4
```

如果查看 app4/app.js 的内容，会发现应用需要的所有中间件（不包括 express.static）都作为独立模块载入，而且再不显式地加载 router 中间件。

您可能还会发现，和旧的生成器生成的应用相比， app.js 现在成了一个 Node 模块。

安装完依赖后，使用如下命令启动应用：
```javascript
$ npm start
```

如果看一看 package.json 文件中的 npm 启动脚本，会发现启动应用的真正命令是 `node ./bin/www`，在 Express 3 中则为 node app.js。

Express 4 应用生成器生成的 app.js 是一个 Node 模块，不能作为应用（除非修改代码）单独启动，需要通过一个 Node 文件加载并启动，这里这个文件就是 node ./bin/www。

创建或启动 Express 应用时，bin 目录或者文件名没有后缀的 www 文件都不是必需的，它们只是生成器推荐的做法，请根据需要修改。

如果不想保留 www，想让应用变成 Express 3 的形式，则需要删除 module.exports = app;，并在 app.js 末尾粘贴如下代码。
```javascript
app.set('port', process.env.PORT || 3000);

var server = app.listen(app.get('port'), function() {
  debug('Express server listening on port ' + server.address().port);});
```
记得在 app.js 上方加入如下代码加载 debug 模块。
```javascript
var debug = require('debug')('app4');
```
然后将 package.json 文件中的 "start": "node ./bin/www" 修改为 "start": "node app.js"。

现在就将 ./bin/www 的功能又改回到 app.js 中了。我们并不推荐这样做，这个练习只是为了帮助大家理解 ./bin/www 是如何工作的，以及为什么 app.js不能再自己启动。


参考
[express 中文API](http://www.expressjs.com.cn/)
