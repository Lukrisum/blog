---
title: Write Me A Koa
tags: 
  - source code
categories:
  - [Tech,FrontEnd,Node.js]
---

前两天有空，跟着B站上的视频，实现了一个[简易的 koa 框架](https://github.com/Lukrisum/koaoak)（不包括 `koa-router`）
<!--more-->

## 手写过程
> 手写源码前我们看看用到了哪些API，这些就是我们手写的目标。   ——[DennisJiang](https://github.com/dennis-jiang)

为方便了解 koa 的 API，这是一个 `Hello World` 的例子，随便请求一个路径（3000端口）都返回 `Hello World` 以及一个 `logger` ，就是记录下处理当前请求花了多长时间，并附带错误处理的用例

```js
const Koa = require("koa")

const app = new Koa()

// usage of middleware
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
});

app.use(async (ctx, next) => {
  // use cases for ctx
  ctx.body = "Hello World";
});

// koa's error handling
app.on('error',(e)=>{
  console.log(e)
})

// listen on port 3000
app.listen(3000, () => {
  console.log(`Server is running on http://127.0.0.1:${port}/`);
});
```
参考源码，分为以下文件，分别实现：
1. **application.js** ：`require('koa')` 引入的 koa 类
2. **context.js**：`ctx` 对象
3. **request.js**：`ctx.request` 对象
4. **response.js**：`ctx.response` 对象
5. **package.json**：配置入口文件等

### 配置入口文件为 application.js：
```json
// package.json
{
  "main":"./application.js"
}
```

### 实现 `app.use()`，`app.listen()`，`app.on()` 等
```js
// application.js
const http = require('http')
const context = require('./context')
const request = require('./request')
const response = require('./response')
const EventEmmiter = require('events')

class Application extends EventEmmiter {

  constructor() {
    super()
    this.context = Object.create(context)
    this.request = Object.create(request)
    this.response = Object.create(response)

    this.middlewares = []
  }

  use(middleware) {
    this.middlewares.push(middleware)
  }

  createContext(req, res) {
    const ctx = Object.create(this.context)
    const request = Object.create(this.request)
    const response = Object.create(this.response)
    ctx.request = request
    ctx.request.req = ctx.req = req
    ctx.response = response
    ctx.response.res = ctx.res = res
    return ctx
  }

  compose(ctx) {
    let index = -1
    const dispatch = (i) => {
      if (index >= i) {
        return Promise.reject('next() called multiple times')
      }
      index = i
      if (this.middlewares.length == i) return Promise.resolve()
      let middleware = this.middlewares[i]
      try {
        return Promise.resolve(middleware(ctx, () => dispatch(i + 1)))
      } catch (e) {
        return Promise.reject(e)
      }
    }
    return dispatch(0)
  }

  handleRequest = (req, res) => {
    let ctx = this.createContext(req, res)
    res.statusCode = 404

    this.compose(ctx).then(() => {
      const body = ctx.body
      if (body) {
        res.end(body)
      } else {
        res.end('NOT FOUND')
      }
    }).catch((e) => {
      this.emit('error', e)
    })

  }

  listen() {
    let server = http.createServer(this.handleRequest)
    server.listen(...arguments)
  }
}

module.exports = Application
```

### `ctx.request` 对象的封装
```js
// request.js
const url = require('url')

const request = {
  get url() {
    return this.req.url
  },
  get path() {
    let { pathname } = url.parse(this.req.url)
    return pathname
  },
  get query() {
    let { query } = url.parse(this.req.url, true)
    return query
  }
}

module.exports = request
```

### `ctx.response` 对象的封装
```js
// response.js
const response = {
  _body: undefined,
  get body() {
    return this._body
  },
  set body(content) {
    this.res.statusCode = 200
    this._body = content
  }
}

module.exports = response
```

### `ctx` 对象的封装（同时将 `request`，`response` 对象挂载上去）
```js
// context.js
const context = {
}

function defineGetter(target, key) {
  context.__defineGetter__(key, function () {
    return this[target][key]
  })
}

function defineSetter(target, key) {
  context.__defineSetter__(key, function (value) {
    this[target][key] = value
  })
}

defineGetter('request', 'path')
defineGetter('request', 'url')
defineGetter('request', 'query')

defineGetter('response', 'body')
defineSetter('response', 'body')


module.exports = context
```

## 总结
koa 的核心功能
1. 提供了功能丰富的 ctx（即 koa 文档说的“上下文”）
2. 特有的中间件流程（洋葱模型）
3. 更好的错误处理

第一次尝试实现一个库/框架，虽然借助了视频，过程也比较磕磕绊绊，但是现在再去看 koa 官方文档中的介绍，就会有一种恍然大悟的感觉，这也许就是所谓“阅读源码”的好处

## 相关链接
> **手写 Koa 源码**：http://dennisgo.cn/Articles/Node/Koa.html
> **B 站视频**：https://www.bilibili.com/video/BV19M4y1G71B/?spm_id_from=333.337.search-card.all.click&vd_source=8583748192f1bc61f6d8bf9d41576b98