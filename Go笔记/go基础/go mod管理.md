# 1、Go 包管理详解

## 1.1包管理简介

为了解决 Golang 依赖问题，类似于 Rust 的 Cargo、Node.js 的 NPM、Python 的 Pip、Ruby 的 Boundler 等，Golang 最原始的依赖管理是 go get，执行命令后会拉取代码放入 GOPATH/src 下面，但是它是作为 GOPATH 下全局的依赖，并且 go get 还不能版本控制，以及隔离项目的包依赖。

对于以上这些问题，在go mod出现之前，有dep，govendor等包管理工具的出现，但都多多少少存在缺陷。

从 Go 1.11 版本开始，官方已内置了更为强大的 Go modules 来一统多年来 Go 包依赖管理混乱的局面(Go 官方之前推出的 dep 工具也几乎胎死腹中)，并且将在 1.13 版本中正式默认开启，目前已受到社区的看好和强烈推荐，建议新项目采用 Go modules。作为新入局go的同学，可以跳过旧的包管理，直接了解和使用go modules机制（即go mod系列命令）来管理包。

## 1.2、GO111MODULE

golang提供了一个环境变量“GO111MODULE”，默认值为auto，如果当前目录里有 go.mod 文件，就使用 go modules，否则使用旧的 GOPATH 和 vendor 机制，因为在modules机制下go get只会下载go modules。

modules和传统GOPATH不同，不需要包含src，bin这样的子目录，一个源代码目录甚至是空目录都可以作为module，只要其中包含go.mod文件。

除了go.mod之外，go命令还维护一个名为go.sum的文件，其中包含特定模块版本内容的预期加密哈希，go命令使用go.sum文件确保这些模块的未来下载检索与第一次下载相同的位，以确保项目所依赖的模块不会出现意外更改，无论是出于恶意、意外还是其他原因。 go.mod和go.sum都应检入版本控制。

go.sum不需要手工维护，但是也可以了解一下，下面会进行介绍。

## 1.3、包查找顺序总结

- 如果GO111MODULE=off，那么go命令行将不会使用新的module功能，相反的，它将会在vendor目录下和GOPATH目录中查找依赖包。也把这种模式叫GOPATH模式。
- 如果GO111MODULE=on，那么go命令行就会使用modules功能，而不会访问GOPATH。也把这种模式称作module-aware模式，这种模式下，GOPATH不再在build时扮演导入的角色，但是尽管如此，它还是承担着存储下载依赖包的角色。它会将依赖包放在GOPATH/pkg/mod目录下。
- 如果GO111MODULE=auto，这种模式是默认的模式，也就是说在你不设置的情况下，就是auto。这种情况下，go命令行会根据当前目录来决定是否启用module功能。只有当当前目录在GOPATH/src目录之外而且当前目录包含go.mod文件或者其子目录包含go.mod文件才会启用。

通俗的讲：
GO111MODULE=auto，即默认情况，当然=on与=off也包含在下述情况中：

1. 没go.mod文件时，属于GOPATH模式，则使用 vendor 特性
2. 有go.mod文件时，此时默认启用 modules特性
   - 找当前目录，不找GOPATH/src目录
   - 当前目录下有vendor目录，则查找当前目录下vendor是否有此包；
   - 当前目录下没有vendor目录，则查找GOROOT/src下是否有此包；
   - 如果未找到，则启动GOPROXY特性，到仓库下载此包；
   - 如果未下载到则提示包不存在；

包顺序：标准库 -> 项目内包 -> 内部第三方 -> 外部第三方包

# 2、具体配置方式

## 方式一、使用gopath（不推荐）

最原始的方式，也就是将项目文件创建在环境变量中配置的GOPATH中

<img src="https://gitee.com/the-clock-that-stays/image/raw/master/img/image-20210701163851590.png" alt="image-20210701163851590" style="zoom: 50%;" />

<img src="https://gitee.com/the-clock-that-stays/image/raw/master/img/image-20210701164110862.png" alt="image-20210701164110862" style="zoom:50%;" />

在项目文件下创建 bin src pkg文件

src下面放置源文件



> 注意点
> ==如果使用如上方式必须保证GO111MODULE=off==



## 方式二、配置go module

使用go mod 管理项目，就不需要非得把项目放到GOPATH指定目录下，你可以在你磁盘的任何位置新建一个项目,包含go.mod文件的目录也被称为模块根，也就是说，go.mod 文件的出现定义了它所在的目录为一个模块

### 1、通过new project方式

若是通过new project方式则为普通项目想要使用go mod管理包需要进行如下操作

首先在终端执行如下代码

```go
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

是否成功可以使用go env查询

<img src="https://gitee.com/the-clock-that-stays/image/raw/master/img/image-20210701170017593.png" alt="image-20210701170017593" style="zoom: 67%;" />

初始化go mod 在终端执行

```go
go mod init text
```

执行完成之后会形成一个go.mod文件

#### 注意点

- 保证gopath中没有配置路径

<img src="https://gitee.com/the-clock-that-stays/image/raw/master/img/image-20210701171027311.png" alt="image-20210701171027311" style="zoom: 33%;" />

- go module中下图地方必须勾上

<img src="C:/Users/Jason/AppData/Roaming/Typora/typora-user-images/image-20210701171145154.png" style="zoom:33%;" />

### 2、使用 Go modules创建项目

<img src="https://gitee.com/the-clock-that-stays/image/raw/master/img/image-20210701171629345.png" alt="image-20210701171629345" style="zoom:50%;" />

什么也不用干，软件会帮你全部弄好



# 3、使用go mod导入本地包

假设我们现在有`moduledemo`和`mypackage`两个包，其中`moduledemo`包中会导入`mypackage`包并使用它的`New`方法。

`mypackage/mypackage.go`内容如下：

```go
package mypackage//不能写main

import "fmt"

func New(){
	fmt.Println("mypackage.New")
}
```

我们现在分两种情况讨论：

## 3.1、在同一个项目下

**注意**：在一个项目（project）下我们是可以定义多个包（package）的。

### 目录结构

现在的情况是，我们在`moduledemo/main.go`中调用了`mypackage`这个包。

```bash
moduledemo
├── go.mod
├── main.go
└── mypackage
    └── mypackage.go
```

### 导入包

这个时候，我们需要在`moduledemo/go.mod`中按如下定义：

```go
module moduledemo

go 1.14
```

然后在`moduledemo/main.go`中按如下方式导入`mypackage`

```go
package main

import (
	"fmt"
	"moduledemo/mypackage"  // 导入同一项目下的mypackage包
)
func main() {
	mypackage.New()
	fmt.Println("main")
}
```

> ### ==**易出错点：**==
>
> 1. mypackage.go(被导入包下的go文件)  第一行需要写==package 文件名==，不能写main
> 2. main.go（在导入的go文件中）import中**根目录（go mod init 的文件名）开始写起**，**直到被调用文件的上一级**

### 举个例子

举一反三，假设我们现在有文件目录结构如下：

```bash
└── bubble
    ├── dao
    │   └── mysql.go
    ├── go.mod
    └── main.go
```

其中`bubble/go.mod`内容如下：

```go
module github.com/q1mi/bubble

go 1.14
```

`bubble/dao/mysql.go`内容如下：

```go
package dao

import "fmt"

func New(){
	fmt.Println("mypackage.New")
}
```

`bubble/main.go`内容如下：

```go
package main

import (
	"fmt"
	"github.com/q1mi/bubble/dao"//从go.mod init的文件名开始
)
func main() {
	dao.New()
	fmt.Println("main")
}
```

## 3.2、不在同一个项目下

### 目录结构

```bash
├── moduledemo
│   ├── go.mod
│   └── main.go
└── mypackage
    ├── go.mod
    └── mypackage.go
```

### 导入包

这个时候，`mypackage`也需要进行module初始化，即拥有一个属于自己的`go.mod`文件，内容如下：

```go
module mypackage

go 1.14
```

然后我们在`moduledemo/main.go`中按如下方式导入：

```go
import (
	"fmt"
	"mypackage"
)
func main() {
	mypackage.New()
	fmt.Println("main")
}
```

> 注意点：==因为这两个包不在同一个项目路径下，你想要导入本地包，并且这些包也没有发布到远程的github或其他代码仓库地址。这个时候我们就需要在`go.mod`文件中使用`replace`指令。==

==在调用方也就是`moduledemo/go.mod`中按如下方式指定使用相对路径来寻找`mypackage`这个包。==

```go
module moduledemo

go 1.14


require "mypackage" v0.0.0
replace "mypackage" => "../mypackage"
```

### 举个例子

最后我们再举个例子巩固下上面的内容。

我们现在有文件目录结构如下：

```bash
├── p1
│   ├── go.mod
│   └── main.go
└── p2
    ├── go.mod
    └── p2.go
```

`p1/main.go`中想要导入`p2.go`中定义的函数。

`p2/go.mod`内容如下：

```go
module liwenzhou.com/q1mi/p2

go 1.14
```

`p1/main.go`中按如下方式导入

```go
import (
	"fmt"
	"liwenzhou.com/q1mi/p2"
)
func main() {
	p2.New()
	fmt.Println("main")
}
```

因为我并没有把`liwenzhou.com/q1mi/p2`这个包上传到`liwenzhou.com`这个网站，我们只是想导入本地的包，这个时候就需要用到`replace`这个指令了。

`p1/go.mod`内容如下：

```go
module github.com/q1mi/p1

go 1.14


require "liwenzhou.com/q1mi/p2" v0.0.0
replace "liwenzhou.com/q1mi/p2" => "../p2"
```

此时，我们就可以正常编译`p1`这个项目了。