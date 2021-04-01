# Hadoop安装过程

***

## 配置文件

>### core-site.xml
>
>```xml
><?xml version="1.0" encoding="UTF-8"?>
><?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
>
><configuration>
>	<!-- 指定NameNode的地址 -->
>    <property>
>        <name>fs.defaultFS</name>
>        <value>hdfs://hadoop102:8020</value>
></property>
><!-- 指定hadoop数据的存储目录 -->
>    <property>
>        <name>hadoop.tmp.dir</name>
>        <value>/opt/module/hadoop-3.1.3/data</value>
></property>
>
><!-- 配置HDFS网页登录使用的静态用户为atguigu -->
>    <property>
>        <name>hadoop.http.staticuser.user</name>
>        <value>lyl</value>
></property>
>
><!-- 配置该atguigu(superUser)允许通过代理访问的主机节点 -->
>    <property>
>        <name>hadoop.proxyuser.lyl.hosts</name>
>        <value>*</value>
></property>
><!-- 配置该atguigu(superUser)允许通过代理用户所属组 -->
>    <property>
>        <name>hadoop.proxyuser.lyl.groups</name>
>        <value>*</value>
></property>
><!-- 配置该atguigu(superUser)允许通过代理的用户-->
>    <property>
>        <name>hadoop.proxyuser.lyl.users</name>
>        <value>*</value>
></property>
></configuration>
>
>```
>
>### hdfs-site.xml
>
>```xml
><?xml version="1.0" encoding="UTF-8"?>
><?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
>
><configuration>
>	<!-- nn web端访问地址-->
>	<property>
>        <name>dfs.namenode.http-address</name>
>        <value>hadoop102:9870</value>
>    </property>
>    
>	<!-- 2nn web端访问地址-->
>    <property>
>        <name>dfs.namenode.secondary.http-address</name>
>        <value>hadoop104:9868</value>
>    </property>
>    
>    <!-- 测试环境指定HDFS副本的数量1 -->
>    <property>
>        <name>dfs.replication</name>
>        <value>3</value>
>    </property>
></configuration>
>
>```
>
>### yarn-site.xml
>
>```xml
><?xml version="1.0" encoding="UTF-8"?>
><?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
>
><configuration>
>	<!-- 指定MR走shuffle -->
>    <property>
>        <name>yarn.nodemanager.aux-services</name>
>        <value>mapreduce_shuffle</value>
>    </property>
>    
>    <!-- 指定ResourceManager的地址-->
>    <property>
>        <name>yarn.resourcemanager.hostname</name>
>        <value>hadoop103</value>
>    </property>
>    
>    <!-- 环境变量的继承 -->
>    <property>
>        <name>yarn.nodemanager.env-whitelist</name>
>        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
>    </property>
>    
>    <!-- yarn容器允许分配的最大最小内存 -->
>    <property>
>        <name>yarn.scheduler.minimum-allocation-mb</name>
>        <value>512</value>
>    </property>
>    <property>
>        <name>yarn.scheduler.maximum-allocation-mb</name>
>        <value>4096</value>
>    </property>
>    
>    <!-- yarn容器允许管理的物理内存大小 -->
>    <property>
>        <name>yarn.nodemanager.resource.memory-mb</name>
>        <value>4096</value>
>    </property>
>    
>    <!-- 关闭yarn对物理内存和虚拟内存的限制检查 -->
>    <property>
>        <name>yarn.nodemanager.pmem-check-enabled</name>
>        <value>false</value>
>    </property>
>    <property>
>        <name>yarn.nodemanager.vmem-check-enabled</name>
>        <value>false</value>
>    </property>
>    <!-- 开启日志聚集功能 -->
><property>
>    <name>yarn.log-aggregation-enable</name>
>    <value>true</value>
></property>
>
><!-- 设置日志聚集服务器地址 -->
><property>  
>    <name>yarn.log.server.url</name>  
>    <value>http://hadoop102:19888/jobhistory/logs</value>
></property>
>
><!-- 设置日志保留时间为7天 -->
><property>
>    <name>yarn.log-aggregation.retain-seconds</name>
>    <value>604800</value>
></property>
>
></configuration>
>
>```
>
>### mapred-site.xml
>
>```xml
><?xml version="1.0" encoding="UTF-8"?>
><?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
>
><configuration>
>	<!-- 指定MapReduce程序运行在Yarn上 -->
>    <property>
>        <name>mapreduce.framework.name</name>
>        <value>yarn</value>
>    </property>
></configuration>
>
><!-- 历史服务器端地址 -->
><property>
>    <name>mapreduce.jobhistory.address</name>
>    <value>hadoop102:10020</value>
></property>
>
><!-- 历史服务器web端地址 -->
><property>
>    <name>mapreduce.jobhistory.webapp.address</name>
>    <value>hadoop102:19888</value>
></property>
>
>```
>
>

***

## 项目经验之Hadoop参数调优

>### 1）HDFS参数调优hdfs-site.xml
>
>The number of Namenode RPC server threads that listen to requests from clients. If dfs.namenode.servicerpc-address is not configured then Namenode RPC server threads listen to requests from all nodes.
>
>NameNode有一个工作线程池，用来处理不同DataNode的并发心跳以及客户端并发的元数据操作。
>
>对于大集群或者有大量客户端的集群来说，通常需要增大参数dfs.namenode.handler.count的默认值10。
>
>```xml
><property>
>  <name>dfs.namenode.handler.count</name>
>  <value>10</value>
></property>
>```
>
>**dfs.namenode.handler.count=20×log (Cluster Size)**，比如集群规模为8台时，此参数设置为41。可通过简单的python代码计算该值，代码如下。
>
>```python
>[atguigu@hadoop102 ~]$ python
>
>Python 2.7.5 (default, Apr 11 2018, 07:36:10) 
>
>[GCC 4.8.5 20150623 (Red Hat 4.8.5-28)] on linux2
>
>Type "help", "copyright", "credits" or "license" for more information.
>
>\>>> import math
>
>\>>> print int(20*math.log(8))
>
>41
>
>\>>> quit()
>```
>
>### 2）YARN参数调优yarn-site.xml
>
>**（1）情景描述：总共7台机器，每天几亿条数据，数据源->Flume->Kafka->HDFS->Hive**
>
>面临问题：数据统计主要用HiveSQL，没有数据倾斜，小文件已经做了合并处理，开启的JVM重用，而且IO没有阻塞，内存用了不到50%。但是还是跑的非常慢，而且数据量洪峰过来时，整个集群都会宕掉。基于这种情况有没有优化方案。
>
>**（2）解决办法：**
>
>内存利用率不够。这个一般是Yarn的2个配置造成的，单个任务可以申请的最大内存大小，和Hadoop单个节点可用内存大小。调节这两个参数能提高系统内存的利用率。
>
>**（a）yarn.nodemanager.resource.memory-mb**
>
>表示该节点上YARN可使用的物理内存总量，默认是8192（MB），注意，如果你的节点内存资源不够8GB，则需要调减小这个值，而YARN不会智能的探测节点的物理内存总量。
>
>**（b）yarn.scheduler.maximum-allocation-mb**
>
>单个任务可申请的最多物理内存量，默认是8192（MB）。

