# hbase 安装

***

>**安装hbase之前必须保证zookeeper集群和Hadoop集群正常运行**
>
>**将Hbase的安装包解压到任意目录**
>
>## 配置hbase的环境变量
>
>```shell
>#在/etc/profile.d/目录下任意创建一个以.sh结尾的文件并在其内添加以下内容
>
>#HBASE_HOME
>export HBASE_HOME=/opt/module/hbase
>export PATH=$PATH:$HBASE_HOME/bin
>```
>
>## source环境变量
>
>```shell
>source /etc/profile
>```
>
>## 修改hbase的配置文件
>
>**1.修改hbase-env.sh**
>
>```shell
>export HBASE_MANAGES_ZK=false
>```
>
>**2.修改hbase-site.xml文件**
>
>```xml
><configuration>
>    <!--设置hbase在hdfs上的根路径地址-->
>    <property>
>        <name>hbase.rootdir</name>
>        <value>hdfs://hadoop102:8020/hbase</value>
>    </property>
>    <!--以集群模式运行-->
>    <property>
>        <name>hbase.cluster.distributed</name>
>        <value>true</value>
>    </property>
>	<!--配置zookeeper主机地址-->
>    <property>
>        <name>hbase.zookeeper.quorum</name>
>        <value>hadoop102,hadoop103,hadoop104</value>
>    </property>    
></configuration>
>
>```
>
>**3.修改regionservers文件(添加集群主机地址)**
>
>```xml
>hadoop102
>hadoop103
>hadoop104
>```
>
>**4.同步集群文件**
>
>>
>
>## HBase服务的启动
>
>>**1.单点启动**
>>
>>```shell
>>bin/hbase-daemon.sh start master
>>```
>>
>>```shell
>>bin/hbase-daemon.sh start regionserver
>>```
>>
>>**提示：如果集群之间的节点时间不同步，会导致regionserver无法启动，抛出ClockOutOfSyncException异常。**
>>
>>**修复提示：**
>>
>>`a、同步时间服务`
>>
>>`b、属性：hbase.master.maxclockskew设置更大的值`
>>
>>```xml
>><!--增大集群允许的时间误差范围--> 
>><property>      
>>     <name>hbase.master.maxclockskew</name>      
>>     <value>180000</value>      
>>     <description>Time difference of  regionserver from master</description>  </property>  
>>```
>>
>>**2.群启**
>>
>>```shell
>>bin/start-hbase.sh
>>```
>>
>>对应的停止服务：
>>
>>```shell
>> bin/stop-hbase.sh
>>```
>
>## 查看HBase页面
>
>>启动成功后，可以通过“host:port”的方式来访问HBase管理页面，例如：
>>
>>[http://hadoop102:16010](http://linux01:16010)
>
>## 高可用
>
>>**首先停止hbase集群**
>>
>>**在conf目录下创建backup-masters文件**
>>
>>```shell
>>touch conf/backup-masters
>>```
>>
>>**在backup-masters文件中填入高可用从机节点的主机地址**
>>
>>```shell
>>echo hadoop103 > conf/backup-masters
>>```
>>
>>**将修改后的配置文件同步到集群的其它节点**
>>
>>**启动hbase即可完成高可用**
>
>