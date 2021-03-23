# 配置YARN-HA集群

## 1）环境准备

（1）修改IP

（2）修改主机名及主机名和IP地址的映射

（3）关闭防火墙

（4）ssh免密登录

（5）安装JDK，配置环境变量等

（6）配置Zookeeper集群

## 2）规划集群

| hadoop102       | hadoop103       | hadoop104   |
| --------------- | --------------- | ----------- |
| NameNode        | NameNode        | NameNode    |
| JournalNode     | JournalNode     | JournalNode |
| DataNode        | DataNode        | DataNode    |
| ZKFC            | ZKFC            | ZKFC        |
| ZK              | ZK              | ZK          |
| ResourceManager | ResourceManager |             |
| NodeManager     | NodeManager     | NodeManager |

## 3）具体配置

（1）yarn-site.xml

```xml
<configuration>

        <!-- yarn容器允许分配的最大最小内存 -->
    <property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>512</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>4096</value>
    </property>
    <!-- yarn容器允许管理的物理内存大小 -->
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>4096</value>
    </property>
    <!-- 关闭yarn对物理内存和虚拟内存的限制检查 -->
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>

    <!-- 开启日志聚集功能 -->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <!-- 设置日志聚集服务器地址 -->
    <property>
        <name>yarn.log.server.url</name>
        <value>http://hadoop102:19888/jobhistory/logs</value>
    </property>
    <!-- 设置日志保留时间为7天 -->
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>
	<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- 启用resourcemanager ha -->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>

    <!-- 声明两台resourcemanager的地址 -->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>cluster-yarn1</value>
    </property>
    <!--指定resourcemanager的逻辑列表-->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
	</property>
	<!-- ========== rm1的配置 ========== -->
	<!-- 指定rm1的主机名 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>hadoop102</value>
	</property>
	<!-- 指定rm1的web端地址 -->
	<property>
         <name>yarn.resourcemanager.webapp.address.rm1</name>
         <value>hadoop102:8088</value>
	</property>
	<!-- 指定rm1的内部通信地址 -->
	<property>
         <name>yarn.resourcemanager.address.rm1</name>
         <value>hadoop102:8032</value>
    </property>
    <!-- 指定AM向rm1申请资源的地址 -->
    <property>
         <name>yarn.resourcemanager.scheduler.address.rm1</name>
         <value>hadoop102:8030</value>
    </property>
    <!-- 指定供NM连接的地址 -->
    <property>
         <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
         <value>hadoop102:8031</value>
    </property>
    <!-- ========== rm2的配置 ========== -->
        <!-- 指定rm2的主机名 -->
    <property>
          <name>yarn.resourcemanager.hostname.rm2</name>
          <value>hadoop103</value>
    </property>
    <property>
         <name>yarn.resourcemanager.webapp.address.rm2</name>
         <value>hadoop103:8088</value>
    </property>
    <property>
         <name>yarn.resourcemanager.address.rm2</name>
         <value>hadoop103:8032</value>
    </property>
    <property>
         <name>yarn.resourcemanager.scheduler.address.rm2</name>
         <value>hadoop103:8030</value>
    </property>
    <property>
         <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
         <value>hadoop103:8031</value>
    </property>

    <!-- 指定zookeeper集群的地址 -->
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>
    </property>

    <!-- 启用自动恢复 -->
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>

    <!-- 指定resourcemanager的状态信息存储在zookeeper集群 -->
    <property>
        <name>yarn.resourcemanager.store.class</name>   
        <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
	</property>
	<!-- 环境变量的继承 -->
 	<property>
        <name>yarn.nodemanager.env-whitelist</name>
 	<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>

</configuration>

```

（2）同步更新其他节点的配置信息，分发配置文件

[

```
atguigu@hadoop102 etc]$ xsync hadoop/
```

4）启动hdfs 

```
[atguigu@hadoop102 ~]$ start-dfs.sh
```

5）启动YARN 

（1）在hadoop102或者hadoop103中执行：

```
[atguigu@hadoop102 ~]$ start-yarn.sh
```

（2）查看服务状态

```
[atguigu@hadoop102 ~]$ yarn rmadmin -getServiceState rm1
```

（3）可以去zkCli.sh客户端查看ResourceManager选举锁节点内容：

```
[atguigu@hadoop102 ~]$ zkCli.sh
```

```shell
[zk: localhost:2181(CONNECTED) 16] get -s /yarn-leader-election/cluster-yarn1/ActiveStandbyElectorLock

 

cluster-yarn1rm1

cZxid = 0x100000022

ctime = Tue Jul 14 17:06:44 CST 2020

mZxid = 0x100000022

mtime = Tue Jul 14 17:06:44 CST 2020

pZxid = 0x100000022

cversion = 0

dataVersion = 0

aclVersion = 0

ephemeralOwner = 0x30000da33080005

dataLength = 20

numChildren = 0
```

## （4）web端查看

hadoop102:8088和hadoop103:8088的YARN的状态，和NameNode对比，查看区别

区别：从机的访问请求会重定向到主机



# 总结