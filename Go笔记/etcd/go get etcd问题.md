go sdk版本在go1.13和go1.14版本使用go mod管理依赖包中有etcd时会出现异常，无法正常拉取etcd包。

 

错误如下：

```Go
go.etcd.io/etcd/clientv3 tested by

        go.etcd.io/etcd/clientv3.test imports
        github.com/coreos/etcd/auth imports
        github.com/coreos/etcd/mvcc/backend imports
		github.com/coreos/bbolt: github.com/coreos/bbolt@v1.3.5: parsing go.mod:
        module declares its path as: go.etcd.io/bbolt
              but was required as: github.com/coreos/bbolt
```

引起以上的原因主要是etcd中使用的bbolt和grpc版本冲突引起

 

### **解决方法**

删除原来已生成得go.mod和go.sum

```Go
go mod init
go mod edit -replace github.com/coreos/bbolt@v1.3.4=go.etcd.io/bbolt@v1.3.4
go mod edit -replace google.golang.org/grpc@v1.29.1=google.golang.org/grpc@v1.26.0
go mod tidy
```

 

