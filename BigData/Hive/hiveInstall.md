# Hive安装并设置将元数据存储到mysql中

***

>## 解压hive到某个地方并为hive添加环境变量
>
>>
>
>***
>
>## 将mysql的jdbc连接jar包放到hive的根目录下的lib目录中
>
>>
>
>***
>
>## 在hive的根目录下的conf目录下创建hive-site.xml文件
>
>>```xml
>><?xml version="1.0"?>
>><?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
>><configuration>
>>    <!-- jdbc连接的URL -->
>>    <property>
>>       <name>javax.jdo.option.ConnectionURL</name>
>>       <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
>>    </property>
>>    <!-- jdbc连接的Driver-->
>>    <property>
>>       <name>javax.jdo.option.ConnectionDriverName</name>
>>       <value>com.mysql.jdbc.Driver</value>
>>    </property>
>>    <!-- jdbc连接的username-->
>>    <property>
>>       <name>javax.jdo.option.ConnectionUserName</name>
>>       <value>root</value>
>>    </property>
>>    <!-- jdbc连接的password -->
>>    <property>
>>       <name>javax.jdo.option.ConnectionPassword</name>
>>       <value>123456</value>
>>    </property>
>>    <!-- Hive默认在HDFS的工作目录 -->
>>    <property>
>>       <name>hive.metastore.warehouse.dir</name>
>>       <value>/user/hive/warehouse</value>
>>    </property>
>>    <!-- Hive元数据存储版本的验证 -->
>>    <property>
>>       <name>hive.metastore.schema.verification</name>
>>       <value>false</value>
>>    </property>
>>    <!-- 指定hiveserver2连接的端口号 -->
>>    <property>
>>    <name>hive.server2.thrift.port</name>
>>    <value>10000</value>
>>    </property>
>>    <!-- 指定hiveserver2连接的host -->
>>    <property>
>>       <name>hive.server2.thrift.bind.host</name>
>>       <value>hadoop102</value>
>>    </property>
>>    <!--元数据存储授权-->
>>    <property>
>>       <name>hive.metastore.event.db.notification.api.auth</name>
>>       <value>false</value>
>>    </property>
>>    <!--设置hive命令行操作的时候输出表头-->
>>    <property>
>>       <name>hive.cli.print.header</name>
>>       <value>true</value>
>>    </property>
>>    <property>
>>       <name>hive.cli.print.current.db</name>
>>       <value>true</value>
>>    </property>
>>	<!-- hiveserver2的高可用参数，开启此参数可以提高hiveserver2的启动速度 -->
>>	<property>
>>	<name>hive.server2.active.passive.ha.enable</name>
>>	<value>true</value>
>>	</property>
>>    <!-- 指定存储元数据要连接的地址 -->
>>    <property>
>>       <name>hive.metastore.uris</name>
>>       <value>thrift://hadoop102:9083</value>
>>    </property>
>></configuration>
>>```
>>
>>
>>
>>上面的配置只是一个MetaStoreServer，存在单点问题，但我们完全可以配置两个或者多个MetaStoreServer，就实现了负载均衡与容错的功能了，如下面的配置
>>
>>```xml
>><property>
>><name>hive.metastore.uris</name>
>><value>thrift://dw1:9083,thrift://dw2:9083</value>
>><description>A comma separated list of metastore uris on which metastore service       is running
>></description>
>></property>
>>```
>>
>>
>
>***
>
>## 初始化元数据库
>
>>```sql
>>--在mysql中创建数据库metastore作为hive的元数据库
>>create database if not exists metastore;
>>```
>>
>>```shell
>>#初始化hive的元数据库
>>schematool -initSchema -dbType mysql -verbose
>>```
>
>***
>
>## 启动metastore
>
>>```shell
>> hive --service metastore
>>```
>>
>>**注意: 启动后窗口不能再操作，需打开一个新的shell窗口做别的操作**
>
>***
>
>## 启动hiveserver2
>
>>```shell
>>hive --service hiveserver2
>>```
>
>***
>
>## 可通过hive 或 beeline来连接hive
>
>>```shell
>>bin/beeline -u jdbc:hive2://hadoop102:10000 -n 用户名
>>```
>>
>>```shell
>>hive
>>```

>***
>
>## 编写脚本来启动或关闭hiveserver2服务和metastore服务(该脚本在为hive配置了环境变量后可直接使用)
>
>>```shell
>>#!/bin/bash
>>HIVE_LOG_DIR=$HIVE_HOME/logs
>>if [ ! -d $HIVE_LOG_DIR ]
>>then
>>	mkdir -p $HIVE_LOG_DIR
>>fi
>>#检查进程是否运行正常，参数1为进程名，参数2为进程端口
>>function check_process()
>>{
>>   pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print $2}')
>>   ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -d '/' -f 1)
>>   echo $pid
>>   [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
>>}
>>
>>function hive_start()
>>{
>>   metapid=$(check_process HiveMetastore 9083)
>>   cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 &"
>>   [ -z "$metapid" ] && eval $cmd || echo "Metastroe服务已启动"
>>   server2pid=$(check_process HiveServer2 10000)
>>   cmd="nohup hive --service hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
>>   [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2服务已启动"
>>}
>>
>>function hive_stop()
>>{
>>metapid=$(check_process HiveMetastore 9083)
>>   [ "$metapid" ] && kill $metapid || echo "Metastore服务未启动"
>>   server2pid=$(check_process HiveServer2 10000)
>>   [ "$server2pid" ] && kill $server2pid || echo "HiveServer2服务未启动"
>>}
>>
>>case $1 in
>>"start")
>>   hive_start
>>   ;;
>>"stop")
>>   hive_stop
>>   ;;
>>"restart")
>>   hive_stop
>>   sleep 2
>>   hive_start
>>   ;;
>>"status")
>>   check_process HiveMetastore 9083 >/dev/null && echo "Metastore服务运行正常" || echo "Metastore服务运行异常"
>>   check_process HiveServer2 10000 >/dev/null && echo "HiveServer2服务运行正常" || echo "HiveServer2服务运行异常"
>>   ;;
>>*)
>>   echo Invalid Args!
>>   echo 'Usage: '$(basename $0)' start|stop|restart|status'
>>   ;;
>>esac
>>```
>
>