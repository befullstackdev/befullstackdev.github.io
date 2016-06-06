---
layout: post
title: MySQL 的日常使用
---



# MySQL 的日常使用

### *nix os 下配置文件一般存放位置

> Default options are read from the following files in the given order:
/etc/my.cnf, /etc/mysql/my.cnf, /usr/local/etc/my.cnf, ~/.my.cnf
>
> 默认设置按一下顺序读取：
> 
/etc/my.cnf, /etc/mysql/my.cnf, /usr/local/etc/my.cnf, ~/.my.cnf

---

### Ubuntu 下的启动，停止与重启

**启动方式：**

 - 方式一：`sudo /etc/init.d/mysql start`
 - 方式二：`sudo start mysql`
 - 方式三：`sudo service mysql start`

**停止mysql：**

 - 方式一：`sudo /etc/init.d/mysql stop`
 - 方式二：`sudo stop mysql`
 - 方式三：`sudo service mysql stop`

**重启mysql：**

 - 方式一：`sudo/etc/init.d/mysql restart`
 - 方式二：`sudo restart mysql`
 - 方式三：`sudo service mysql restart`

### Mac 下的启动，停止与重启

这里记录通过 homebrew 安装的操作

* 启动: `mysql.server start`
* 停止: `mysql.server stop`
* 重启: `mysql.server restart 或者 reload 或者 force-reload`

---
### 权限管理
**查看mysql状态**

 - 方式一：`service mysql status` (输出类似mysql start/running, process 810)
 - 方式二：登录mysql client, 执行命令：`show status`;
 - 方式三：Mac 如果是通过 homebrew 安装的，则可以直接使用 `mysql.server status`查看

**增加用户及权限**

{% highlight mysql %}
GRANT ALL ON *.* TO 'username'@'hostname' IDENTIFIED BY 'username' WITH GRANT OPTION;
# 然后刷新权限
flush privileges;
{% endhighlight %}

**删除用户权限**
{% highlight mysql %}
REVOKE ALL ON *.* FROM 'username'@'hostname';
# 然后刷新权限
flush privileges;
{% endhighlight %}
