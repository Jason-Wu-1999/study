# 二、NoSQL数据库简介

## 1.技术发展

技术的分类：

1. 解决功能性的问题：Java、Jsp、RDBMS、Tomcat、HTML、Linux、JDBC、SVN
2. 解决扩展性的问题：Struts、Spring、SpringMVC、Hibernate、Mybatis
3. 解决性能的问题：NoSQL、Java线程、Hadoop、Nginx、MQ、ElasticSearch

### 1.1 Web1.0时代

Web1.0的时代，数据访问量很有限，用一夫当关的高性能的单点服务器可以解决大部分问题。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420153042580.png)

### 1.2 Web2.0时代

随着Web2.0的时代的到来，用户访问量大幅度提升，同时产生了大量的用户数据。加上后来的智能移动设备的普及，所有的互联网平台都面临了巨大的性能挑战。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210420153100849.png)

### 1.3 解决CPU及内存压力

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210420153116276.png)

### 1.4 解决IO压力

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210420153131712.png)

## 2.NoSQL数据库

### 2.1 NoSQL数据库概述

NoSQL(Not Only SQL)，意即“不仅仅是SQL”，泛指非关系型的数据库。

NoSQL 不依赖业务逻辑方式存储，而以简单的key-value模式存储。因此大大的增加了数据库的扩展能力。

- 不遵循SQL标准
- 不支持ACID
- 远超于SQL的性能

### 2.2 NoSQL适用场景

- 对数据高并发的读写
- 海量数据的读写
- 对数据高可扩展性的

### 2.3 NoSQL不适用场景

- 需要事务支持
- 基于sql的结构化查询存储，处理复杂的关系,需要即席查询。
- 用不着sql的和用了sql也不行的情况，请考虑用NoSql

### 2.4 常见的NoSQL数据库

> Memcache
> ![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210420152231999.png)

> Redis
> ![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210420152307510.png)

> MongoDB
> ![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20210420152401914.png)

## 3.行式存储数据库（大数据时代）

### 3.1 行级数据库

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420153159517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjU5NDc5Ng==,size_16,color_FFFFFF,t_70)

### 3.2 列级数据库

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420153217549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjU5NDc5Ng==,size_16,color_FFFFFF,t_70)

> 常见的列级数据库包括：Hbase、Cassandra等…这里就不做过多介绍
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/2021042015323582.png)
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420153243757.png)

## 4.图关系型数据库

> 说到图关系型数据库，首先应该想到的就是：**Neo4j**
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420153311816.png)
> 主要应用：社会关系，公共交通网络，地图及网络拓谱(n*(n-1)/2)
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420153338579.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjU5NDc5Ng==,size_16,color_FFFFFF,t_70)

## 5.DB-Engines 数据库排名

排名链接：http://db-engines.com/en/ranking
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420153457688.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjU5NDc5Ng==,size_16,color_FFFFFF,t_70)

# 三、Redis概述及安装

## 1.Redis概述

> 1. Redis是一个开源的key-value存储系统。
> 2. 和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted
>    set --有序集合)和hash（哈希类型）。
> 3. 这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。
> 4. 在此基础上，Redis支持各种不同方式的排序。
> 5. 与memcached一样，为了保证效率，数据都是缓存在内存中。
> 6. 区别的是Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。
> 7. 并且在此基础上实现了master-slave(主从)同步。

## 2.应用场景

### 2.1 配合关系型数据库做高速缓存

- 高频次，热门访问的数据，降低数据库IO
- 分布式架构，做session共享

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420154051187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjU5NDc5Ng==,size_16,color_FFFFFF,t_70)

### 2.2 多样的数据结构存储持久化数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420154137843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjU5NDc5Ng==,size_16,color_FFFFFF,t_70)

### 3.4 后台启动（推荐）

1. 将redis.conf复制到其他目录下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420202655211.png)

2. 修改redis.conf(128行)文件将里面的daemonize no 改成 yes，让服务在后台启动
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420205654374.png)

3. 启动Redis：redis-server /etc/redis.conf
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420221959136.png)

4. 用客户端访问：redis-cli
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420222550688.png)

5. 测试验证： ping
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420223003701.png)

   

6. Redis关闭
   单实例关闭：redis-cli shutdown
   也可以进入终端后再关闭：shutdown
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420223828624.png)