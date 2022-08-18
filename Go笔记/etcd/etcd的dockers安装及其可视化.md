## dockers安装etcd

### 拉去镜像

```
docker pull bitnami/etcd:latest
```

### 启动服务端

```
docker run -d --name etcd-server \
    --network app-tier \
    --p 2379:2379 \
    --env ALLOW_NONE_AUTHENTICATION=yes \
    --env ETCD_ADVERTISE_CLIENT_URLS=http://etcd-server:2379 \
    bitnami/etcd:latest
```

### 启动etcd客户端实例

```
docker run -it --rm \
    --network app-tier \
    --env ALLOW_NONE_AUTHENTICATION=yes \
    bitnami/etcd:latest etcdctl --endpoints http://etcd-server:2379 set /message Hello
```





### 安装etcdkeeper可视化

```
# 下载etcdkeeper
cd /Install/
#https://github.com/evildecay/etcdkeeper/releases
wget https://github.com/evildecay/etcdkeeper/releases/download/v0.7.6/etcdkeeper-v0.7.6-linux_x86_64.zip


unzip etcdkeeper-v0.7.6-linux_x86_64.zip
cd etcdkeeper
chmod +x etcdkeeper
```



### 后台启动

```
##### 服务配置文件
cat > /lib/systemd/system/etcdkeeper.service << EOF
[Unit]
Description=etcdkeeper service
After=network.target
[Service]
Type=simple
ExecStart=/Install/etcdkeeper/etcdkeeper  -p 8800
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
PrivateTmp=true
[Install]
WantedBy=multi-user.target
EOF
```

##### 
```
systemctl stop etcdkeeper
systemctl daemon-reload
systemctl start etcdkeeper
systemctl enable etcdkeeper.service
systemctl status etcdkeeper
```



访问http://101.132.168.64:8800/etcdkeeper/

![image-20220611201253206](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img//img/20220611201253.png)