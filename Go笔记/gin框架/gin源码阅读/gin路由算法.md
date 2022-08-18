# 1. gin的路由算法

gin的是路由算法其实就是一个Trie树(也就是前缀树). 有关数据结构的可以自己去网上找相关资料查看.

### 1.1.1. 注册路由预处理

我们在使用gin时通过下面的代码注册路由

### 1.1.2. 普通注册

```go
router.POST("/somePost", func(context *gin.Context) {
    context.String(http.StatusOK, "some post")
})
```

### 1.1.3. 使用中间件

```go
router.Use(Logger())
```

### 1.1.4. 使用Group

```go
v1 := router.Group("v1")
{
    v1.POST("login", func(context *gin.Context) {
        context.String(http.StatusOK, "v1 login")
    })
}
```

这些操作, 最终都会在反应到gin的路由树上

### 1.1.5. 具体实现

```go
// routergroup.go:L72-77
func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
    // <---把group中的basepath和relativepath 合并在一起  计算出绝对路径
    absolutePath := group.calculateAbsolutePath(relativePath) 
    // <---//合并handler 把group中添加的中间件和传入的handlers合并起来
    handlers = group.combineHandlers(handlers) 
      //调用addRoute 添加router
    group.engine.addRoute(httpMethod, absolutePath, handlers)
    return group.returnObj()
}
```

==在调用POST, GET, HEAD等路由HTTP相关函数时, 会调用handle函数==

如果调用了==中间件==的话, 会调用下面函数

```go
func (group *RouterGroup) Use(middleware ...HandlerFunc) IRoutes {
    group.Handlers = append(group.Handlers, middleware...)
    return group.returnObj()
}
```

如果==使用了Group==的话, 会调用下面函数

```go
func (group *RouterGroup) Group(relativePath string, handlers ...HandlerFunc) *RouterGroup {
    return &RouterGroup{
        Handlers: group.combineHandlers(handlers),
        basePath: group.calculateAbsolutePath(relativePath),
        engine:   group.engine,
    }
}
```

重点关注下面两个函数:

```go
// routergroup.go:L208-217
func (group *RouterGroup) combineHandlers(handlers HandlersChain) HandlersChain {
    finalSize := len(group.Handlers) + len(handlers)
    if finalSize >= int(abortIndex) {
        panic("too many handlers")
    }
    mergedHandlers := make(HandlersChain, finalSize)
    copy(mergedHandlers, group.Handlers)
    copy(mergedHandlers[len(group.Handlers):], handlers)
    return mergedHandlers
}
func (group *RouterGroup) calculateAbsolutePath(relativePath string) string {
    return joinPaths(group.basePath, relativePath)
}

func joinPaths(absolutePath, relativePath string) string {
    if relativePath == "" {
        return absolutePath
    }

    finalPath := path.Join(absolutePath, relativePath)
    appendSlash := lastChar(relativePath) == '/' && lastChar(finalPath) != '/'
    if appendSlash {
        return finalPath + "/"
    }
    return finalPath
}
```

在joinPaths函数里面有段代码, 很有意思, 我还以为是写错了. 主要是path.Join的用法.

```go
finalPath := path.Join(absolutePath, relativePath)
appendSlash := lastChar(relativePath) == '/' && lastChar(finalPath) != '/'
```

在当路由是/user/这种情况就满足了lastChar(relativePath) == '/' && lastChar(finalPath) != '/'. 主要原因是path.Join(absolutePath, relativePath)之后, finalPath是user

综合来看, 在预处理阶段

1.在调用中间件的时候, 是将某个路由的handler处理函数和中间件的处理函数都放在了Handlers的数组中 

2.在调用Group的时候, 是将路由的path上面拼上Group的值. 也就是/user/:name, 会变成v1/user:name

### 1.1.6. 真正注册

```go
// routergroup.go:L72-77
func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
    absolutePath := group.calculateAbsolutePath(relativePath) // <---
    handlers = group.combineHandlers(handlers) // <---
    group.engine.addRoute(httpMethod, absolutePath, handlers)
    return group.returnObj()
}
```

调用group.engine.addRoute(httpMethod, absolutePath, handlers)将预处理阶段的结果注册到gin Engine的trees上

### 1.1.7. gin路由树简单介绍

gin的路由树算法是一棵前缀树. 不过并不是只有一颗树, 而是每种方法(POST, GET ...)都有自己的一颗树

```go
func (engine *Engine) addRoute(method, path string, handlers HandlersChain) {
    assert1(path[0] == '/', "path must begin with '/'")
    assert1(method != "", "HTTP method can not be empty")
    assert1(len(handlers) > 0, "there must be at least one handler")

    debugPrintRoute(method, path, handlers)
    root := engine.trees.get(method) // <-- 看这里
    if root == nil {
        root = new(node)
        engine.trees = append(engine.trees, methodTree{method: method, root: root})
    }
    root.addRoute(path, handlers)
}
```

gin 路由最终的样子大概是下面的样子

![img](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/3.png)

```go
type node struct {
    path      string
    indices   string
    children  []*node
    handlers  HandlersChain
    priority  uint32
    nType     nodeType
    maxParams uint8
    wildChild bool
}
```

其实gin的实现不像一个真正的树, children []*node所有的孩子都放在这个数组里面, 利用indices, priority变相实现一棵树

### 1.1.8. 获取路由handler

当服务端收到客户端的请求时, 根据path去trees匹配到相关的路由, 拿到相关的处理handlers

```go
...
t := engine.trees
for i, tl := 0, len(t); i < tl; i++ {
    if t[i].method != httpMethod {
        continue
    }
    root := t[i].root
    // Find route in tree
    handlers, params, tsr := root.getValue(path, c.Params, unescape) // 看这里
    if handlers != nil {
        c.handlers = handlers
        c.Params = params
        c.Next()
        c.writermem.WriteHeaderNow()
        return
    }
    if httpMethod != "CONNECT" && path != "/" {
        if tsr && engine.RedirectTrailingSlash {
            redirectTrailingSlash(c)
            return
        }
        if engine.RedirectFixedPath && redirectFixedPath(c, root, engine.RedirectFixedPath) {
            return
        }
    }
    break
}
...
```

主要在下面这个函数里面调用程序注册的路由处理函数

```go
func (c *Context) Next() {
    c.index++
    for c.index < int8(len(c.handlers)) {
        c.handlers[c.index](c)
        c.index++
    }
}
```

转自：https://www.jianshu.com/p/5c0f1925f5db