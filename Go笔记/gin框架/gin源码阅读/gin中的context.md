# gin牛逼的context

Gin封装的最好的地方就是context和对response的处理. github的README的介绍,基本就是对这两个东西的解释. 本篇文章主要解释context的使用方法, 以及其设计原理

### 1.1.1. 为什么要将Request的处理封装到Context中

在阅读gin的源码时, 请求的处理是使用type HandlerFunc func(*Context)来处理的. 也就是

```go
func(context *gin.Context) {
    context.String(http.StatusOK, "some post")
}
```

参数是gin.Context, 但是查看源码发现其实gin.Context在整个框架处理的地方只有下面这段:

```go
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    c := engine.pool.Get().(*Context)
    c.writermem.reset(w)
    c.Request = req
    c.reset()
    engine.handleHTTPRequest(c)
    engine.pool.Put(c)
}
```

那为什么还要利用Context来处理呢. gin的context实现了的context.Context Interface.

经过查看context.Context相关资料, Context的最佳运用场景就是对Http的处理. 封装成Conetxt另外的好处就是WithCancel, WithDeadline, WithTimeout, WithValue这些context包衍生的子Context就可以直接来使用. 目前我能想到的地方就这么多, 以后发现gin.Context其他的优点再补充.

### 1.1.2. gin.Context的设计

gin.Context主要由下面几部分组成(这里沿用源代码里面的注释)

### 1.1.3. Metadata Management (我自己叫法:Key-Value)

这个模块比较简单, 就是从gin.Context中Set Key-Value, 以及各种个样的Get方法, 如GetBool, GetString等

实现这些功能也很简单, 其实就是一个map

```go
// Keys is a key/value pair exclusively for the context of each request.
Keys map[string]interface{}
```

### 1.1.4. Input Data

这个模块相当重要了, gin的README基本上都在介绍这个模块的用法.

### 1.1.5. Param (我自己的叫法: 路由变量)

gin的标准叫法是Parameters in path. restful风格api如/user/john, 这个路由在gin里面是/user/:name, 要获取john就需要使用Param函数

```go
name := c.Param("name")
```

这个方法实现也很简单, 就是在tree.go里面根据路由相关规则解析出来然后赋值给gin.Context的Params.

```go
handlers, params, tsr := root.getValue(path, c.Params, unescape)
```

### 1.1.6. Query

/welcome?firstname=Jane&lastname=Doe这样一个路由, first, last即是Querystring parameters, 要获取他们就需要使用Query相关函数.

```go
c.Query("first") // Jane
c.Query("last") // Doe
```

当然还有其他相关函数:

- QueryMap
- DefaultQuery 这个默认值的实现更加简单, 当QueryString中不包含这个值, 直接返回填入的值

这些方法是的实现是利用net/http的Request的方法实现的

### 1.1.7. PostForm

对于POST, PUT等这些能够传递参数Body的请求, 要获取其参数, 需要使用PostForm

```go
POST /user/1

{
    "name":manu,
    "message":this_is_great
}
name := c.PostForm("name")
message := c.PostForm("message")
```

其他相关函数

- DefaultPostForm

这些相关的方法是实现还是利用net/http的Request的方法实现的

### 1.1.8. FormFile

对于文件相关的操作, 一般生产情况下不建议这样使用, 因为把文件上传到服务器磁盘, 还得磁盘相关的监控. 我觉得最好利用云服务商相关的对象存储, 如:阿里云OSS, 七牛云对象存储, AWS的对象存储等来做文件的相关操作

### 1.1.9. Bind

内置的有json, xml, protobuf, form, query, yaml. 这些Bind极大的减少我们自己去解析各种个样的数据格式, 提高我们的开发速度

Bind的实现都在gin/binding里面. 这些内置的Bind都实现了Binding接口, 主要是Bind()函数.

- context.BindJSON() 支持MIME为application/json的解析
- context.BindXML() 支持MIME为application/xml的解析
- context.BindYAML() 支持MIME为application/x-yaml的解析
- context.BindQuery() 只支持QueryString的解析, 和Query()函数一样
- context.BindUri() 只支持路由变量的解析
- Context.Bind() 支持所有的类型的解析, 这个函数尽量还是少用(当QueryString, PostForm, 路由变量在一块同时使用时会产生意想不到的效果), 目前测试Bind不支持路由变量的解析, Bind()函数的解析比较复杂, 这部分代码后面再看

### 1.1.10. Response

### 1.1.11. 对Header的支持

- Header
- GetHeader

这里的Header是写到Response里面的Header. 对于客户端发的请求的Header可以通过context.Request.Header.Get("Content-Type")获取

### 1.1.12. Cookie

提供对session, cookie的支持

### 1.1.13. render

做api常用到的其实就是gin封装的各种render. 目前支持的有:

- func (c *Context) JSON(code int, obj interface{})
- func (c *Context) Protobuf(code int, obj interface{})
- func (c *Context) YAML(code int, obj interface{}) ...

当然我们可以自定义渲染, 只要实现func (c *Context) Render(code int, r render.Render)即可.

这里我们常用的是一个方法是: gin.H{"error": 111}. 这个结构相当实用, 各种render都支持. 其实这个结构很简单就是type H map[string]interface{}, 当我们要从map转换各种各样结构时, 不妨参考gin这里的代码

Context说到这里基本就说完了, 这里介绍的方法都是开发中特别实用的方法. context的代码实现也特别有条理, 建议可以看看这部分代码

转自：https://www.jianshu.com/p/73bf8fe7a745