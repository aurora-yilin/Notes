# mysql主从复制的步骤

前言----因为物理原因的限制，本文的主从复制采用的是一主一从的模式进行搭建的，主机在centos7.8从机是和主机一台机器，区别是，从机是通过docker创建的一个mysql容器。

# 配置：

​	操作系统：centos7.8

​	主机MySQL5.7.31

​	从机MySQL5.7.29

# 一、主机操作：

## 	1.首先通过wget下载mysql软件源

​		

```shell
wget http://repo.mysql.com/mysql57-community-release-el7-10.noarch.rpm
```



## 	2.安装软件源

​		

```shell
rpm -Uvh mysql57-community-release-el7-10.noarch.rpm
```



## 	3.安装mysql服务端

​		 

```shell
yum install  -y  mysql-community-server
```



## 	4.检查并启动mysql

​		

```shell
systemctl status mysqld

systemctl start mysqld
```



## 	5.修改临时密码



### 		5.1获取mysql的临时密码

​		为了加强安全性，MySQL5.7为root用户随机生成了一个密码，在error log中，关于error log的位置，如果安装的是RPM包，则默认是/var/log/mysqld.log。

只有启动过一次mysql才可以查看临时密码

```shell
grep 'temporary password' /var/log/mysqld.log
```

### 		5.2登录并修改密码

​			

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root123';
```

​			修改密码的时候如果新密码不满足mysql内配置的密码限制则会报错ERROR 1819 (HY000): Your password does not satisfy the current policy requirements，此时需要修改mysql关于密码的限制策略

​			必须修改两个全局参数：
首先，修改validate_password_policy参数的值

```sql
mysql> set global validate_password_policy=0; 
```

再修改密码的长度

```sql
set global validate_password_length=1;
```

再次执行修改密码就可以了

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root123';
```

### 	5.3授权其它机器登录

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION; FLUSH  PRIVILEGES;
```

## 6.修改主机的配置文件

在主机的配置文件中添加以下内容

server-id=1
log-bin=mysql-bin
binlog-ignore-db=mysql

## 7.重启主机服务

```shell
systemctl restart mysqld
```

## 8.在master数据库创建数据同步用户，授予用户 slave REPLICATION SLAVE权限和REPLICATION CLIENT权限，用于在主从库之间同步数据

```sql
CREATE USER 'slave'@'%' IDENTIFIED BY '@#$Rfg345634523rft4fa';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```

语句中的`%`代表所有服务器都可以使用这个用户，如果想指定特定的ip，将`%`改成ip即可。

## 9.连接主机mysql并查看master状态

show master status;

记录下`File`和`Position`的值，并且不进行其他操作以免引起`Position`的变化。

# 二、从机操作：

## 	1.从docker上pull一个mysql镜像

​		

```shell
docker pull 5.7.29
```



## 	2.启动一个mysql容器

```shell
docker run -p 3307:3306 -d --name mysqlSlave -e MYSQL_ROOT_PASSWORD="mysqlpassword" -v /home/Lcf/mysql/database:/var/lib/mysql  -v /home/Lcf/mysql/config/conf.d/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf   5d9483f9a7b2 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

## 	3.修改容器mysql的配置文件

​		

```
#opyright (c) 2014, 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License, version 2.0,
# as published by the Free Software Foundation.
#
# This program is also distributed with certain software (including
# but not limited to OpenSSL) that is licensed under separate terms,
# as designated in a particular file or component or in included license
# documentation.  The authors of MySQL hereby grant you an additional
# permission to link the program and your derivative works with the
# separately licensed software that they have included with MySQL.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License, version 2.0, for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
#log-error      = /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address   = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0






server-id=3
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=mysql-slave-bin
## relay_log配置中继日志
relay_log=edu-mysql-relay-bin
```

## 4.重启从机mysql容器

```shell
docker restart 容器id
```

## 5.进入从机mysql容器

```shell
docker exec -it 容器id /bin/bash
```

## 6.从机认主

```sql

change master to master_host='',master_user='slave',master_password='',master_port=3306,master_log_file='mysql-bin.000001',master_log_pos=617,master_connect_retry=30;
```

**master_host** ：Master的地址

**master_port**：Master的端口号

**master_user**：用于数据同步的用户

**master_password**：用于同步的用户的密码

**master_log_file**：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值

**master_log_pos**：从哪个 Position 开始读，即上文中提到的 Position 字段的值

**master_connect_retry**：如果连接失败，重试的时间间隔，单位是秒，默认是60秒

## 7.查看主从同步信息

```sql
show slave status \G;
```

此时Slave_IO_Running: NO   Slave_SQL_Running: NO  均为NO因为我们还没有开启主从复制过程

## 8.开启主从复制过程

```sql
start slave;
```

## 9.再次查看主从同步信息

```sql
show slave status \G;
```

SlaveIORunning 和 SlaveSQLRunning 都是Yes说明主从复制已经开启

# 三、踩坑总结

## 第一种错误

Last_IO_Error: Fatal error: The slave I/O thread stops because master and slave have equal MySQL server ids; these ids must be different for replication to work (or the --replicate-same-server-id option must be used on slave but this does not always make sense; please check the manual before using it). 

出现这种错误的原因是因为从机是docker镜像创建出来的容器，所以从机的配置文件不能够被成功加载，导致配置文件中的server-id参数恰巧和主机的是一样的，想看看主机和从机的server-id是否一样可以通过在sql中执行以下命令来查看

```sql
show variables like 'server_id'; 
```

若发现从主机的server_id为同一个数值的时候可以通过执行以下命令来设置，但是我没有这样设置，而是选择了从根源上解决问题，让docker容器从机顺利加载配置文件(根据上文的说明往从主机中添加配置信息还出现了这种问题根本原因还是mysql的配置文件未被正常加载，以上的配置文件和过程都是经过踩坑修改过的，在此只是记录一下踩坑)。

在此加上解决错误的参考博客，感谢各位师傅们的无私奉献https://www.jb51.net/article/27242.htm

## 第二种错误

在配置的过程中还出现的另一种错误是进入从机mysql容器执行

```sql
mysql -u root -p
```

进行连接的时候出现了一个警告

Warning:World-writable config file is ignored

出现这个警告的时候，我意识到估计这是配置文件又没被加载成功，经过查询，才知道引起这种原因可能是因为配置文件的权限问题，因为从机是docker的容器跑起来的然后将从机的配置文件映射到docker宿主机的一个配置文件上，这个时候就会产生容器读取配置文件的时候产生权限不足的问题，此时只需要找到从机容器配置文件在docker 宿主机上的映射文件，修改一下文件的权限即可。。。

问题解决参考博客，https://stackoverflow.com/questions/53741107/mysql-in-docker-on-ubuntu-warning-world-writable-config-file-is-ignored

# 四、总结

总参考博客

https://juejin.im/post/6844903921677238285

https://blog.csdn.net/daicooper/article/details/79905660



docker起的mysql容器真坑。。。。。