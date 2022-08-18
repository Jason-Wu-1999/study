因为使用的是**宝塔面板安装**

**安装目录**在  

```
/www/server/redis
```

**配置文件**在

```
/www/server/redis/redis.conf
```

**redis-server目录**在

```
/www/server/redis/src/redis-server
```



## 启动

进入src目录执行

```
redis-server  /www/server/redis/redis.conf 
```

![image-20211210174019765](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20211210174019765.png)



# window连接Linux中的redis

## 需要更改配置文件

1. **注释掉bind 127.0.0.1**     允许其他ip连接
2. 将**daemonize  yes**改为no    允许后台启动
3. **关掉保护模式 protected-mode:no** 允许其他ip连接
4. 设置密码（可有可无）   requirepass  密码

重新启动redis





## windows中使用redis-cli连接

![image-20211210175107573](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20211210175107573.png)

==**如果连接失败记得关闭防火墙和开放端口**==

​	