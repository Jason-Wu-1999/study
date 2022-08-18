# 记一次手贱改成https后无法进后台的解决方法



1.首先连接到服务器的数据库

2.将数据库中wp_option表的option_name为siteurl和home对应的option_value中的https改回http

![image-20211120221034891](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20211120221034891.png)

**对应的代码**

> updata wp_options set option_value="http://101.132.168.64" where option_name='siteurl';

