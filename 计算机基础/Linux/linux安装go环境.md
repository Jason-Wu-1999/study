# 安装Go环境

## **1.获取linux位数**

```
getconf LONG_BIT
```

## **2.下载地址：**

https://studygolang.com/dl

根据Linux 的位数选择下载安装包

![image-20211208174451167](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20211208174451167.png)

## **3.解压安装包**

```
tar -xzf go1.17.4.linux-amd64.tar.gz -C /usr/local/INSTALL
```

## **4.设置环境变量**

```
 vim /etc/profile 
 增加如下代码
export GOROOT=/usr/local/INSTALL/go
export GOBIN=$GOROOT/bin
export GOPATH=/home/wjs/prj_go1
export PATH=$PATH:$GOBIN:$GOPATH
```

刷新

```
source /etc/profile
```



```
5.go env  查看go环境
6.go version 查看go版本
```

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20211208175040256.png" alt="image-20211208175040256" style="zoom:50%;" />

# 交叉编译在Linux下运行go文件

## 1、设置

在Windows中的Goland中设置如下

```
SET CGO_ENABLED=0 
SET GOOS=linux 
SET GOARCH=amd64 
```

## 2、编译

```go
go build main.go
```

生成的go文件 放到linux64位，下面的gopath下，在命令行输入 main，就可以运行了。

```
./main
```





# 在Linux下编译、运行go文件

**同样选择go mod管理**

配置如下

```
go env -w GO111MODULE="on"
go env -w GOPROXY="https://goproxy.cn,direct"
```

## 创建项目文件

```
mkdir project01
```

![image-20211210105004258](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20211210105004258.png)

## 创建go文件、写入代码

```
vim demo.go
```

初始化go mod文件

```
go mod init project01
go mod tidy
```

![image-20211210105409844](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20211210105409844.png)

## 在项目下创建bin目录

（可以不创建，只是为了更规范，让编译后的文件放在bin目录下）

## 编译

在bin目录下编译demo.go 这样生成可执行文件就这bin目录下了

## 执行

![image-20211210105810287](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20211210105810287.png)

