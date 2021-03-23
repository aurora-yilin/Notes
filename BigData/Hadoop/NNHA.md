# Hadoop NameNode HA

## 1.集群规划

***

| hadoop102   | hadoop103       | hadoop104   |
| ----------- | --------------- | ----------- |
| NameNode    | NameNode        | NameNode    |
| JournalNode | JournalNode     | JournalNode |
| DataNode    | DataNode        | DataNode    |
| ZKFC        | ZKFC            | ZKFC        |
| ZK          | ZK              | ZK          |
|             | ResourceManager |             |
| NodeManager | NodeManager     | NodeManager |

***

## 2.配置Zookeeper集群

#### (1)解压Zookeeper安装包到/opt/module/目录下

```shell
tar -zxvf zookeeper-3.5.7.tar.gz -C /opt/module/
```

#### (2)在zookeeper根目录下创建zkData

#### (3)重命名zookeeper根目录下conf文件夹中的zoo_sample.cfg为zoo.cfg

#### (4)配置zoo.cfg文件

```shell
1）具体配置
dataDir=/opt/module/zookeeper-3.5.7/zkData
增加如下配置
#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
（2）配置参数解读
Server.A=B:C:D。
A是一个数字，表示这个是第几号服务器；
B是这个服务器的IP地址；
C是这个服务器与集群中的Leader服务器交换信息的端口；
D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。
集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。

```

#### (5)在zkData文件夹下创建一个myid文件

```
该文件中的内容只有一个数字 例如 1
```

#### (6)使用xsync脚本分发zookeeper

```shell
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
  echo Not Enough Arguement!
  exit;
fi
#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
  echo ====================  $host  ====================
  #3. 遍历所有目录，挨个发送
  for file in $@
  do
    #4. 判断文件是否存在
    if [ -e $file ]
    then
      #5. 获取父目录
      pdir=$(cd -P $(dirname $file); pwd)
      #6. 获取当前文件的名称
      fname=$(basename $file)
      ssh $host "mkdir -p $pdir"
      rsync -av $pdir/$fname $host:$pdir
    else
      echo $file does not exists!
    fi
  done
done

```

#### (7)使用脚本启动zookeeper

```shell
#!/bin/bash

if (($# != 1))
then
    echo "No Args Input"
fi


case $1 in
"start")
    for i in hadoop102 hadoop103 hadoop104
    do
        echo "-------------------start $i zookeeper------------------"
        ssh $i /opt/module/zookeeper-3.5.7/bin/zkServer.sh start
    done

    sleep 2

    for i in hadoop102 hadoop103 hadoop104
    do
        echo "-------------------status $i zookeeper---------------------"
        ssh $i /opt/module/zookeeper-3.5.7/bin/zkServer.sh status
    done

    echo "over"
;;
"status")
    for i in hadoop102 hadoop103 hadoop104
    do
        echo "-------------------status $i zookeeper---------------------"
        ssh $i /opt/module/zookeeper-3.5.7/bin/zkServer.sh status
    done

    echo "over"
;;
"stop")
    for i in hadoop102 hadoop103 hadoop104
    do
        echo "-------------------stop $i zookeeper---------------------"
        ssh $i /opt/module/zookeeper-3.5.7/bin/zkServer.sh stop
    done

    echo "over"
;;
"restart")
    for i in hadoop102 hadoop103 hadoop104
    do
        echo "-------------------restart $i zookeeper---------------------"
        ssh $i /opt/module/zookeeper-3.5.7/bin/zkServer.sh restart
    done

;;
*)
    echo "Args Error"
;;

```

## 3.配置HDFS-HA

#### 1.在core-site.xml文件中添加如下：

***

```xml
<!-- 指定NameNode的地址以及集群的文件系统 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://HACluster</value>
    </property>
<!-- 指定hadoop数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/HA/hadoop-3.1.3/data</value>
    </property>
```

#### 2.在hdfs-site.xml文件中添加

***

```xml
<configuration>
<!-- NameNode数据存储目录 -->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file://${hadoop.tmp.dir}/name</value>
  </property>
<!-- DataNode数据存储目录 -->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file://${hadoop.tmp.dir}/data</value>
  </property>
<!-- JournalNode数据存储目录 -->
  <property>
    <name>dfs.journalnode.edits.dir</name>
    <value>${hadoop.tmp.dir}/jn</value>
  </property>
<!-- 完全分布式集群名称 -->
  <property>
    <name>dfs.nameservices</name>
    <value>HACluster</value>
  </property>
<!-- 集群中NameNode节点都有哪些 -->
  <property>
    <name>dfs.ha.namenodes.HACluster</name>
    <value>nn1,nn2,nn3</value>
  </property>
<!-- NameNode的RPC通信地址 -->
  <property>
    <name>dfs.namenode.rpc-address.HACluster.nn1</name>
    <value>hadoop102:8020</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.HACluster.nn2</name>
    <value>hadoop103:8020</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.HACluster.nn3</name>
    <value>hadoop104:8020</value>
  </property>
<!-- NameNode的http通信地址 -->
  <property>
    <name>dfs.namenode.http-address.HACluster.nn1</name>
    <value>hadoop102:9870</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.HACluster.nn2</name>
    <value>hadoop103:9870</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.HACluster.nn3</name>
    <value>hadoop104:9870</value>
  </property>
  <!-- 指定NameNode元数据在JournalNode上的存放位置 -->
  <property>
<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://hadoop102:8485;hadoop103:8485;hadoop104:8485/HACluster</value>
  </property>
<!-- 访问代理类：client用于确定哪个NameNode为Active -->
  <property>
    <name>dfs.client.failover.proxy.provider.HACluster</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
<!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
  <property>
    <name>dfs.ha.fencing.methods</name>
    <value>sshfence</value>
  </property>
<!-- 使用隔离机制时需要ssh秘钥登录-->
  <property>
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/home/atguigu/.ssh/id_rsa</value>
  </property>
</configuration>
```

#### 3.启动集群中所有journalnode服务

***

```shell
hdfs --daemon start journalnode
```

#### 4.格式化其中一个namenode并启动

```shell
hdfs namenode -fromat
hdfs --daemon start namenode
```

#### 5.在另外的namenode上同步格式化后的namenode的信息

```shell
hdfs namenode -bootstrapStandby
```

#### 6.启动剩余的namenode

```shell
hdfs --daemon start namenode
```







# 完整版

## 4.3 HDFS-HA集群配置

### 4.3.1 环境准备

（1）修改IP

（2）修改主机名及主机名和IP地址的映射

（3）关闭防火墙

（4）ssh免密登录

（5）安装JDK，配置环境变量等

### 4.3.2 规划集群

| hadoop102   | hadoop103       | hadoop104   |
| ----------- | --------------- | ----------- |
| NameNode    | NameNode        | NameNode    |
| JournalNode | JournalNode     | JournalNode |
| DataNode    | DataNode        | DataNode    |
| ZKFC        | ZKFC            | ZKFC        |
| ZK          | ZK              | ZK          |
|             | ResourceManager |             |
| NodeManager | NodeManager     | NodeManager |

### 4.3.3 配置Zookeeper集群

1）集群规划

在hadoop102、hadoop103和hadoop104三个节点上部署Zookeeper。

2）解压安装

（1）解压Zookeeper安装包到/opt/module/目录下

[atguigu@hadoop102 software]$ tar -zxvf zookeeper-3.5.7.tar.gz -C /opt/module/

（2）在/opt/module/zookeeper-3.5.7/这个目录下创建zkData

[atguigu@hadoop102 zookeeper-3.5.7]$ mkdir -p zkData

（3）重命名/opt/module/zookeeper-3.4.14/conf这个目录下的zoo_sample.cfg为zoo.cfg

[atguigu@hadoop102 conf]$ mv zoo_sample.cfg zoo.cfg

3）配置zoo.cfg文件

（1）具体配置

dataDir=/opt/module/zookeeper-3.5.7/zkData

增加如下配置

```shell
#######################cluster##########################

server.2=hadoop102:2888:3888

server.3=hadoop103:2888:3888

server.4=hadoop104:2888:3888
```

（2）配置参数解读

Server.A=B:C:D。

A是一个数字，表示这个是第几号服务器；

B是这个服务器的IP地址；

C是这个服务器与集群中的Leader服务器交换信息的端口；

D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。

4）集群操作

（1）在/opt/module/zookeeper-3.5.7/zkData目录下创建一个myid的文件

[atguigu@hadoop102 zkData]$ touch myid

添加myid文件，注意一定要在linux里面创建，在notepad++里面很可能乱码

（2）编辑myid文件

[atguigu@hadoop102 zkData]$ vi myid

在文件中添加与server对应的编号：如2

（3）拷贝配置好的zookeeper到其他机器上

```
[atguigu@hadoop102 module]$ scp -r zookeeper-3.5.7/ [atguigu@hadoop103:/opt/module/](mailto:root@hadoop103.atguigu.com:/opt/app/)

[atguigu@hadoop102 module]$ scp -r zookeeper-3.5.7/ [atguigu@hadoop104:/opt/module/
```

mailto:root@hadoop104.atguigu.com:/opt/app/)

 并分别修改myid文件中内容为3、4

（4）分别启动zookeeper

```shell
[atguigu@hadoop102 zookeeper-3.5.7]$ bin/zkServer.sh start

[atguigu@hadoop103 zookeeper-3.5.7]$ bin/zkServer.sh start

[atguigu@hadoop104 zookeeper-3.5.7]$ bin/zkServer.sh start
```

（5）查看状态

```shell
[atguigu@hadoop104 zookeeper-3.5.7]$ bin/zkServer.sh status

JMX enabled by default

Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg

Mode: follower

[atguigu@hadoop104 zookeeper-3.5.7]$ bin/zkServer.sh status

JMX enabled by default

Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg

Mode: leader

[atguigu@hadoop104 zookeeper-3.5.7]$ bin/zkServer.sh status

JMX enabled by default

Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg

Mode: follower
```



### 4.3.4 配置HDFS-HA集群

1）官方地址：http://hadoop.apache.org/

2）在opt目录下创建一个ha文件夹

```shell
[atguigu@hadoop102 ~]$ cd /opt

[atguigu@hadoop102 opt]$ sudo mkdir ha

[atguigu@hadoop102 opt]$ sudo chown atguigu:atguigu /opt/ha
```

3）将/opt/module/下的 hadoop-3.1.3拷贝到/opt/ha目录下（记得删除data 和 log目录）

```shell
[atguigu@hadoop102 opt]$ cp -r /opt/module/hadoop-3.1.3 /opt/ha/
```

4）配置hadoop-env.sh

```shell
export JAVA_HOME=/opt/module/jdk1.8.0_212s
```

5）配置core-site.xml

```xml
<configuration>

<!-- 把多个NameNode的地址组装成一个集群mycluster -->

 <property>

  <name>fs.defaultFS</name>

  <value>hdfs://mycluster</value>

 </property>

<!-- 指定hadoop运行时产生文件的存储目录 -->

 <property>

  <name>hadoop.tmp.dir</name>

  <value>/opt/ha/hadoop-3.1.3/data</value>

 </property>

</configuration>
```

6）配置hdfs-site.xml

```xml
<configuration>

<!-- NameNode数据存储目录 -->

 <property>

  <name>dfs.namenode.name.dir</name>

  <value>file://${hadoop.tmp.dir}/name</value>

 </property>

<!-- DataNode数据存储目录 -->

 <property>

  <name>dfs.datanode.data.dir</name>

  <value>file://${hadoop.tmp.dir}/data</value>

 </property>

<!-- JournalNode数据存储目录 -->

 <property>

  <name>dfs.journalnode.edits.dir</name>

  <value>${hadoop.tmp.dir}/jn</value>

 </property>

<!-- 完全分布式集群名称 -->

 <property>

  <name>dfs.nameservices</name>

  <value>mycluster</value>

 </property>

<!-- 集群中NameNode节点都有哪些 -->

 <property>

  <name>dfs.ha.namenodes.mycluster</name>

  <value>nn1,nn2,nn3</value>

 </property>

<!-- NameNode的RPC通信地址 -->

 <property>

  <name>dfs.namenode.rpc-address.mycluster.nn1</name>

  <value>hadoop102:8020</value>

 </property>

 <property>

  <name>dfs.namenode.rpc-address.mycluster.nn2</name>

  <value>hadoop103:8020</value>

 </property>

 <property>

  <name>dfs.namenode.rpc-address.mycluster.nn3</name>

  <value>hadoop104:8020</value>

 </property>

<!-- NameNode的http通信地址 -->

 <property>

  <name>dfs.namenode.http-address.mycluster.nn1</name>

  <value>hadoop102:9870</value>

 </property>

 <property>

  <name>dfs.namenode.http-address.mycluster.nn2</name>

  <value>hadoop103:9870</value>

 </property>

 <property>

  <name>dfs.namenode.http-address.mycluster.nn3</name>

  <value>hadoop104:9870</value>

 </property>

<!-- 指定NameNode元数据在JournalNode上的存放位置 -->

 <property>

<name>dfs.namenode.shared.edits.dir</name>

<value>qjournal://hadoop102:8485;hadoop103:8485;hadoop104:8485/mycluster</value>

 </property>

<!-- 访问代理类：client用于确定哪个NameNode为Active -->

 <property>

  <name>dfs.client.failover.proxy.provider.mycluster</name>

  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>

 </property>

<!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->

 <property>

  <name>dfs.ha.fencing.methods</name>

  <value>sshfence</value>

 </property>

<!-- 使用隔离机制时需要ssh秘钥登录-->

 <property>

  <name>dfs.ha.fencing.ssh.private-key-files</name>

  <value>/home/atguigu/.ssh/id_rsa</value>

 </property>

</configuration>
```

7）分发配置好的hadoop环境到其他节点

### 4.3.5 启动HDFS-HA集群

1）将HADOOP_HOME环境变量更改到HA目录(三台机器)

```xml
[atguigu@hadoop102 ~]$ sudo vim /etc/profile.d/my_env.sh

将HADOOP_HOME部分改为如下

\##HADOOP_HOME

export HADOOP_HOME=/opt/ha/hadoop-3.1.3

export PATH=$PATH:$HADOOP_HOME/bin

export PATH=$PATH:$HADOOP_HOME/sbin
```

2）在各个JournalNode节点上，输入以下命令启动journalnode服务

```
[atguigu@hadoop102 ~]$ hdfs --daemon start journalnode

[atguigu@hadoop103 ~]$ hdfs --daemon start journalnode

[atguigu@hadoop104 ~]$ hdfs --daemon start journalnode
```

3）在[nn1]上，对其进行格式化，并启动

```
[atguigu@hadoop102 ~]$ hdfs namenode -format

[atguigu@hadoop102 ~]$ hdfs --daemon start namenode
```

4）在[nn2]和[nn3]上，同步nn1的元数据信息

```
[atguigu@hadoop103 ~]$ hdfs namenode -bootstrapStandby

[atguigu@hadoop104 ~]$ hdfs namenode -bootstrapStandby
```

5）启动[nn2]和[nn3]

```
[atguigu@hadoop103 ~]$ hdfs --daemon start namenode

[atguigu@hadoop104 ~]$ hdfs --daemon start namenode
```

6）查看web页面显示

![img](file:///C:/Users/lyl/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

图 hadoop102(standby)

![img](file:///C:/Users/lyl/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

图 hadoop103(standby)

![img](file:///C:/Users/lyl/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

图 hadoop104(standby)

7）在所有节点上，启动datanode

```
[atguigu@hadoop102 ~]$ hdfs --daemon start datanode

[atguigu@hadoop103 ~]$ hdfs --daemon start datanode

[atguigu@hadoop104 ~]$ hdfs --daemon start datanode
```

8）将[nn1]切换为Active

```
[atguigu@hadoop102 ~]$ hdfs haadmin -transitionToActive nn1
```

9）查看是否Active

```
[atguigu@hadoop102 ~]$ hdfs haadmin -getServiceState nn1
```



### 4.3.6 配置HDFS-HA自动故障转移

1）具体配置

（1）在hdfs-site.xml中增加

```
<!-- 启用nn故障自动转移 -->

<property>

  <name>dfs.ha.automatic-failover.enabled</name>

  <value>true</value>

</property>

（2）在core-site.xml文件中增加

<!-- 指定zkfc要连接的zkServer地址 -->

<property>

  <name>ha.zookeeper.quorum</name>

  <value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>

</property>
```

（3）修改后分发配置文件

```
[atguigu@hadoop102 etc]$ pwd

/opt/ha/hadoop-3.1.3/etc

[atguigu@hadoop102 etc]$ xsync hadoop/
```

2）启动

（1）关闭所有HDFS服务：

```
[atguigu@hadoop102 ~]$ stop-dfs.sh
```

（2）启动Zookeeper集群：

```
[atguigu@hadoop102 ~]$ zkServer.sh start

[atguigu@hadoop103 ~]$ zkServer.sh start

[atguigu@hadoop104 ~]$ zkServer.sh start
```

（3）启动Zookeeper以后，然后再初始化HA在Zookeeper中状态：

```
[atguigu@hadoop102 ~]$ hdfs zkfc -formatZK
```

（4）启动HDFS服务：

```
[atguigu@hadoop102 ~]$ start-dfs.sh
```

（5）可以去zkCli.sh客户端查看Namenode选举锁节点内容：

```shell
[zk: localhost:2181(CONNECTED) 7] get -s /hadoop-ha/mycluster/ActiveStandbyElectorLock

 

  myclusternn2  hadoop103 �>(�>

cZxid = 0x10000000b

ctime = Tue Jul 14 17:00:13 CST 2020

mZxid = 0x10000000b

mtime = Tue Jul 14 17:00:13 CST 2020

pZxid = 0x10000000b

cversion = 0

dataVersion = 0

aclVersion = 0

ephemeralOwner = 0x40000da2eb70000

dataLength = 33

numChildren = 0
```

3）验证

（1）将Active NameNode进程kill，查看网页端三台Namenode的状态变化

[atguigu@hadoop102 ~]$ kill -9 namenode的进程id

### 4.3.7 解决NN连接不上JN的问题

自动故障转移配置好以后，然后使用start-dfs.sh群起脚本启动hdfs集群，有可能会遇到NameNode起来一会后，进程自动关闭的问题。查看NameNode日志，报错信息如下：

```
2020-08-17 10:11:40,658 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:40,659 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:40,659 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:41,660 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:41,660 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:41,665 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:42,661 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:42,661 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:42,667 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:43,662 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:43,662 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:43,668 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:44,663 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:44,663 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:44,670 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:45,467 INFO org.apache.hadoop.hdfs.qjournal.client.QuorumJournalManager: Waited 6001 ms (timeout=20000 ms) for a response for selectStreamingInputStreams. No responses yet.

2020-08-17 10:11:45,664 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 5 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:45,664 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 5 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:45,672 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 5 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:46,469 INFO org.apache.hadoop.hdfs.qjournal.client.QuorumJournalManager: Waited 7003 ms (timeout=20000 ms) for a response for selectStreamingInputStreams. No responses yet.

2020-08-17 10:11:46,665 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 6 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:46,665 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 6 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:46,673 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 6 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:47,470 INFO org.apache.hadoop.hdfs.qjournal.client.QuorumJournalManager: Waited 8004 ms (timeout=20000 ms) for a response for selectStreamingInputStreams. No responses yet.

2020-08-17 10:11:47,666 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 7 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:47,667 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 7 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:47,674 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 7 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:48,471 INFO org.apache.hadoop.hdfs.qjournal.client.QuorumJournalManager: Waited 9005 ms (timeout=20000 ms) for a response for selectStreamingInputStreams. No responses yet.

2020-08-17 10:11:48,668 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:48,668 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:48,675 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:49,669 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop102/192.168.6.102:8485. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:49,673 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop104/192.168.6.104:8485. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:49,676 INFO org.apache.hadoop.ipc.Client: Retrying connect to server: hadoop103/192.168.6.103:8485. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

2020-08-17 10:11:49,678 WARN org.apache.hadoop.hdfs.server.namenode.FSEditLog: Unable to determine input streams from QJM to [192.168.6.102:8485, 192.168.6.103:8485, 192.168.6.104:8485]. Skipping.

org.apache.hadoop.hdfs.qjournal.client.QuorumException: Got too many exceptions to achieve quorum size 2/3. 3 exceptions thrown:

192.168.6.103:8485: Call From hadoop102/192.168.6.102 to hadoop103:8485 failed on connection exception: java.net.ConnectException: 拒绝连接; For more details see: http://wiki.apache.org/hadoop/ConnectionRefused

192.168.6.102:8485: Call From hadoop102/192.168.6.102 to hadoop102:8485 failed on connection exception: java.net.ConnectException: 拒绝连接; For more details see: http://wiki.apache.org/hadoop/ConnectionRefused

192.168.6.104:8485: Call From hadoop102/192.168.6.102 to hadoop104:8485 failed on connection exception: java.net.ConnectException: 拒绝连接; For more details see: http://wiki.apache.org/hadoop/ConnectionRefused
```

查看报错日志，可分析出报错原因是因为NameNode连接不上JournalNode，而利用jps命令查看到三台JN都已经正常启动，为什么NN还是无法正常连接到JN呢？这是因为start-dfs.sh群起脚本默认的启动顺序是先启动NN，再启动DN，然后再启动JN，并且默认的rpc连接参数是重试次数为10，每次重试的间隔是1s，也就是说启动完NN以后的10s中内，JN还启动不起来，NN就会报错了。

core-default.xml里面有两个参数如下：

```xml
<!-- NN连接JN重试次数，默认是10次 -->

<property>

 <name>ipc.client.connect.max.retries</name>

 <value>10</value>

</property>

<!-- 重试时间间隔，默认1s -->

<property>

 <name>ipc.client.connect.retry.interval</name>

 <value>1000</value>

</property>
```

   解决方案：遇到上述问题后，可以稍等片刻，等JN成功启动后，手动启动下三台NN：

```shell
[atguigu@hadoop102 ~]$ hdfs --daemon start namenode

[atguigu@hadoop103 ~]$ hdfs --daemon start namenode

[atguigu@hadoop104 ~]$ hdfs --daemon start namenode
```

  也可以在core-site.xml里面适当调大上面的两个参数：

```xml
<!-- NN连接JN重试次数，默认是10次 -->

<property>

 <name>ipc.client.connect.max.retries</name>

 <value>20</value>

</property>

<!-- 重试时间间隔，默认1s -->

<property>

 <name>ipc.client.connect.retry.interval</name>

 <value>5000</value>

</property>
```

