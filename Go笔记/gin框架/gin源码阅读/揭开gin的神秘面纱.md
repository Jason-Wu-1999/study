# 1. 揭开gin的神秘面纱

### 1.1.1. 数据如何在gin中流转

```go
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    r.Run() // listen and serve on 0.0.0.0:8080
}
```

这段代码的大概流程:

1.r := gin.Default()初始化了相关的参数 2./ping将路由及处理handler注册到路由树中 3.启动服务

r.Run()其实调用的是err = http.ListenAndServe(address, engine), 结合上一篇文章可以看出来, gin其实利用了net/http的处理过程

### 1.1.2. ServeHTTP的作用

上一篇文章有提到DefaultServeMux, 其实DefaultServeMux实现了ServeHTTP(ResponseWriter, *Request), 在request执行到server.go的serverHandler{c.server}.ServeHTTP(w, w.req)这一行的时候, 从DefaultServeMux取到了相关路由的处理handler.

因此, gin框架的Engine最重要的函数就是func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request). Engine实现了Handler(server.go#L84-86), 让net/http请求数据最终流回到gin中, 从gin的route tree中取到相关的中间件及handler, 来处理客户端的request

### 1.1.3. Engine

在整个gin框架中最重要的一个struct就是Engine, 它包含路由, 中间件, 相关配置信息等. Engine的代码主要就在gin.go中

Engine中比较重要的几个属性, 其他的属性暂时全部省略掉

```go
type Engine struct {
    RouterGroup // 路由
    pool             sync.Pool  // context pool
    trees            methodTrees // 路由树
    // html template及其他相关属性先暂时忽略
}
```

Engine有几个比较主要的函数:

### 1.1.4. New(), Default()

```go
func New() *Engine {
    // ...
    engine := &Engine{
        RouterGroup: RouterGroup{
            Handlers: nil,
            basePath: "/",
            root:     true,
        },
        // ...
        trees: make(methodTrees, 0, 9),
    }
    engine.RouterGroup.engine = engine
    engine.pool.New = func() interface{} {
        return engine.allocateContext()
    }
    return engine
}
```

New()主要干的事情:

1. 初始化了Engine
2. 将RouterGroup的Handlers(数组)设置成nil, basePath设置成/ 
3. 为了使用方便, RouteGroup里面也有一个Engine指针, 这里将刚刚初始化的engine赋值给了RouterGroup的engine指针
4. 为了防止频繁的context GC造成效率的降低, 在Engine里使用了sync.Pool, 专门存储gin的Context

```go
func Default() *Engine {
    debugPrintWARNINGDefault()
    engine := New()
    engine.Use(Logger(), Recovery())
    return engine
}
```

Default()跟New()几乎一模一样, 就是调用了gin内置的Logger(), Recovery()中间件.

### 1.1.5. Use()

```go
func (engine *Engine) Use(middleware ...HandlerFunc) IRoutes {
    engine.RouterGroup.Use(middleware...)
    engine.rebuild404Handlers()
    engine.rebuild405Handlers()
    return engine
}
```

Use()就是gin的引入中间件的入口了. 仔细分析这个函数, 不难发现Use()其实是在给RouteGroup引入中间件的. 具体是如何让中间件在RouteGroup上起到作用的, 等说到RouteGroup再具体说.

```go
engine.rebuild404Handlers()
engine.rebuild405Handlers()
```

这两句函数其实在这里没有任何用处. 我感觉这里是给gin的测试代码用的. 我们在使用gin的时候, 要想在404, 405添加处理过程, 可以通过NoRoute(), NoMethod()来处理.

### 1.1.6. addRoute()

```go
func (engine *Engine) addRoute(method, path string, handlers HandlersChain) {
    ...
    root := engine.trees.get(method)
    if root == nil {
        root = new(node)
        engine.trees = append(engine.trees, methodTree{method: method, root: root})
    }
    root.addRoute(path, handlers)
}
```

这段代码就是利用method, path, 将handlers注册到engine的trees中. 注意这里为什么是HandlersChain呢, 可以简单说一下, 就是将中间件和处理函数都注册到method, path的tree中了.

### 1.1.7. Run系列函数

Run, RunTLS, RunUnix, RunFd 这些函数其实都是最终在调用net/http的http服务.

### 1.1.8. ServeHTTP

这个函数相当重要了, 主要有这个函数的存在, 才能将请求转到gin中, 使用gin的相关函数处理request请求.

```go
...

t := engine.trees

for i, tl := 0, len(t); i < tl; i++ {
    if t[i].method != httpMethod {
        continue
    }
    root := t[i].root

    handlers, params, tsr := root.getValue(path, c.Params, unescape)
    if handlers != nil {
        c.handlers = handlers
        c.Params = params
        c.Next()
        c.writermem.WriteHeaderNow()
        return
    }
    ...
}
```

利用request中的path, 从Engine的trees中获取已经注册的handler

```go
func (c *Context) Next() {
    c.index++
    for c.index < int8(len(c.handlers)) {
        c.handlers[c.index](c)
        c.index++
    }
}
```

在Next()执行handler的操作. 其实也就是下面的函数

```go
func(c *gin.Context) {
    c.JSON(200, gin.H{
        "message": "pong",
    })
}
```

如果在trees中没有找到对应的路由, 则会执行serveError函数, 也就是404相关的.

转自：https://www.jianshu.com/p/a1d328e368bd

