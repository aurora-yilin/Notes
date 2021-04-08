# MapReduce

***

>## MapReduce跑的慢的原因
>
>>**1. 计算机性能(CPU、内存、磁盘、网络)**
>>
>>**2. I/O操作(数据倾斜、MapReduce数设置不合理、Map运行时间太长导致Reduce等待太久、小文件过多、大量的不可切片的超大压缩文件、Spill次数过多、Merge次数过多等)**
>>
>>
>
>## MapReduce优化方向
>
>>**数据输入、Map阶段、Reduce阶段、IO传输、数据倾斜和常用的调优参数**
>
>## MapReduce优化
>
>>**数据输入阶段**
>>
>>>**(1)合并小文件：在执行MR任务前将小文件进行合并，大量的小文件会产生大量的Map任务，增大Map任务装载次数，而任务的装载比较耗时，从而导致MR运行较慢**
>>>
>>>**(2)采用CombineTextInputFormat来作为输入，解决输入端大量小文件的场景**
>>
>>**Map阶段**
>>
>>>**(1)减少溢写(Spill)次数：通过调整mapreduce.task.io.sort.mb以及mapreduce.map.sort.spill.percent参数值，增大触发Spill的内存上限，减少Spill次数，从而减少磁盘IO**
>>>
>>>**mapred-site.xml文件**
>>>
>>>```xml
>>><property>
>>>  <name>mapreduce.task.io.sort.mb</name>
>>>  <value>100</value>
>>>  <description>The total amount of buffer memory to use while sorting
>>>  files, in megabytes.  By default, gives each merge stream 1MB, which
>>>  should minimize seeks.</description>
>>></property>
>>>
>>><property>
>>>  <name>mapreduce.map.sort.spill.percent</name>
>>>  <value>0.80</value>
>>>  <description>The soft limit in the serialization buffer. Once reached, a
>>>  thread will begin to spill the contents to disk in the background. Note that
>>>  collection will not block if this threshold is exceeded while a spill is
>>>  already in progress, so spills may be larger than this threshold when it is
>>>  set to less than .5</description>
>>></property>
>>>
>>>```
>>>
>>>***
>>>
>>>**(2)减少合并(Merge)次数：通过调整mapreduce.task.io.sort.factor参数，增大Merge的文件数目，减少Merge的次数，从而缩短MR处理的时间**
>>>
>>>**mapred-site.xml文件**
>>>
>>>```xml
>>><property>
>>>  <name>mapreduce.task.io.sort.factor</name>
>>>  <value>10</value>
>>>  <description>The number of streams to merge at once while sorting
>>>  files.  This determines the number of open file handles.</description>
>>></property>
>>>```
>>>
>>>***
>>>
>>>**(3)在Map之后不影响业务逻辑的前提下，先进行Combine处理，减少I/O**
>>>
>>>***注意：Combine的开启可能影响业务结果***
>>
>>**Reduce阶段**
>>
>>>**(1)合理设置Map和Redece数：两个都不能设置太少，也不能设置太多。太少的话会导致Task等待延长处理时间；太多的话会导致Map和Reduce任务之间竞争资源造成处理超时等错误**
>>>
>>>**(2)设置Map、Reduce共存：调整mapreduce.job.reduce.slowstart.completedmaps参数，使Map运行到一定程度后，Reduce也开始运行，减少Reduce的等待时间**
>>>
>>>```xml
>>><property>
>>>  <name>mapreduce.job.reduce.slowstart.completedmaps</name>
>>>  <value>0.05</value>
>>>  <description>Fraction of the number of maps in the job which should be
>>>  complete before reduces are scheduled for the job.
>>>  </description>
>>></property>
>>>```
>>>
>>>**(3)规避使用Reduce：因为Reduce在用于连接数据集的时候将会产生大量的网络消耗**
>>>
>>>**(4)合理设置Reduce端的Buffer：默认情况下，数据达到一个阈值的时候，Buffer中的数据就会写入到磁盘，然后Reduce会从磁盘中获得所有的数据。也就是说，Buffer和Reduce是没有直接关联的，中间会有多次写磁盘->都磁盘的过程，既然有这个弊端那么就可以通过设置参数来配置，使得Buffer中的一部分数据可以直接输送到Reduce中，从而减少IO开销：mapreduce.reduce.input.buffer.percent,默认为0.0.当值大于零的时候会保留指定比例的内存中的Buffer数据将其直接拿给Reduce使用，这样虽然在一定程度上提高了MapReduce的速度，但是却提高了对内存的要求**
>>>
>>>```xml
>>><property>
>>>  <name>mapreduce.reduce.input.buffer.percent</name>
>>>  <value>0.0</value>
>>>  <description>The percentage of memory- relative to the maximum heap size- to
>>>  retain map outputs during the reduce. When the shuffle is concluded, any
>>>  remaining map outputs in memory must consume less than this threshold before
>>>  the reduce can begin.
>>>  </description>
>>></property>
>>>```
>>
>>**I/O传输**
>>
>>>**(1)采用数据压缩的方式：减少网络IO的时间，安装Snappy和Lzo压缩解压缩编码器**
>>>
>>>**core-site.xml**
>>>
>>>```xml
>>><property>
>>>        <name>io.compression.codecs</name>
>>>        <value>
>>>            org.apache.hadoop.io.compress.GzipCodec,
>>>            org.apache.hadoop.io.compress.DefaultCodec,
>>>            org.apache.hadoop.io.compress.BZip2Codec,
>>>            org.apache.hadoop.io.compress.SnappyCodec,
>>>
>>>            com.hadoop.compression.lzo.LzoCodec,
>>>            com.hadoop.compression.lzo.LzopCodec
>>>        </value>
>>>    </property>
>>>```
>>>
>>>**mapred-site.xml**
>>>
>>>```xml
>>>	#开启mr输出时的压缩
>>>    <property>
>>>        <name>mapreduce.output.fileoutputformat.compress</name>
>>>        <value>true</value>
>>>    </property>
>>>    
>>>    <property>
>>>        <name>mapreduce.output.fileoutputformat.compress.codec</name>
>>>        <value>org.apache.hadoop.io.compress.BZip2Codec</value>
>>>    </property> 
>>>    
>>>    
>>>    #开启mr中map阶段的输出压缩
>>>    <property>
>>>        <name>mapreduce.map.output.compress</name>
>>>        <value>true</value>
>>>    </property>
>>>    
>>>    #指定mr中map阶段的输出压缩为Snappy
>>>    <property>
>>>        <name>mapreduce.map.output.compress.codec</name>
>>>        <value>org.apache.hadoop.io.compress.SnappyCodec</value>
>>>    </property>
>>>```
>>>
>>>
>>>
>>>**(2)使用SequenceFile二进制文件(***不常用***)**
>>
>>**数据倾斜问题**
>>
>>>**1.数据倾斜现象**
>>>
>>>>(1)数据频率倾斜——某一个区域的数据量要远远大于其它区域
>>>>
>>>>(2)数据大小倾斜——部分记录的大小远远大于平均值
>>>
>>>**2.减少数据倾斜的方法**
>>>
>>>>**方法一：抽样和范围分区**
>>>>
>>>>​	可以通过对原始数据进行抽样得到的结果集来预设分区边界值
>>>>
>>>>**方法二：自定义分区**
>>>>
>>>>​	基于输出数据的键进行自定义分区。例如，如果Map输出键的单词来源于一本书。且其中某几个专业词汇较多。那么就可以自定义分区将这些专业词汇发送给固定的一部分Reduce实例。将其它的都发送给剩余的Reduce实例。
>>>>
>>>>**方法三：Combiner**
>>>>
>>>>​	使用Combine可以大量的减少数据倾斜。可能的情况下，Combine的目的就是聚合并精简数据。
>>>>
>>>>**方法四：采用MapJoin尽可能的避免ReduceJoin阶段**
>
>## 常用的调优参数
>
>>```xml
>><property>
>>  <name>mapreduce.map.memory.mb</name>
>>  <value>-1</value>
>>  <description>The amount of memory to request from the scheduler for each
>>    map task. If this is not specified or is non-positive, it is inferred from
>>    mapreduce.map.java.opts and mapreduce.job.heap.memory-mb.ratio.
>>    If java-opts are also not specified, we set it to 1024.(一个MapTask可使用的资源上限（单位:MB），默认为1024。如果MapTask实际使用的资源量超过该值，则会被强制杀死。)
>>  </description>
>></property>
>>
>>
>><property>
>>  <name>mapreduce.reduce.memory.mb</name>
>>  <value>-1</value>
>>  <description>The amount of memory to request from the scheduler for each
>>    reduce task. If this is not specified or is non-positive, it is inferred
>>    from mapreduce.reduce.java.opts and mapreduce.job.heap.memory-mb.ratio.
>>    If java-opts are also not specified, we set it to 1024.(一个ReduceTask可使用的资源上限（单位:MB），默认为1024。如果ReduceTask实际使用的资源量超过该值，则会被强制杀死。)
>>  </description>
>></property>
>>
>>
>><property>
>>  <name>mapreduce.map.cpu.vcores</name>
>>  <value>1</value>
>>  <description>The number of virtual cores to request from the scheduler for
>>  each map task.(每个MapTask可使用的最多cpu core数目，默认值: 1)
>>  </description>
>></property>
>>
>>
>><property>
>>  <name>mapreduce.reduce.cpu.vcores</name>
>>  <value>1</value>
>>  <description>The number of virtual cores to request from the scheduler for
>>  each reduce task.(每个ReduceTask可使用的最多cpu core数目，默认值: 1)
>>  </description>
>></property>
>>
>>
>><property>
>>  <name>mapreduce.reduce.shuffle.parallelcopies</name>
>>  <value>5</value>
>>  <description>The default number of parallel transfers run by reduce
>>  during the copy(shuffle) phase.(每个Reduce去Map中取数据的并行数。默认值是5)
>>  </description>
>></property>
>>
>>
>>
>><property>
>>  <name>mapreduce.reduce.shuffle.merge.percent</name>
>>  <value>0.66</value>
>>  <description>The usage threshold at which an in-memory merge will be
>>  initiated, expressed as a percentage of the total memory allocated to
>>  storing in-memory map outputs, as defined by
>>  mapreduce.reduce.shuffle.input.buffer.percent.(Buffer中的数据达到多少比例开始写入磁盘。默认值0.66)
>>  </description>
>></property>
>>
>>
>>
>><property>
>>  <name>mapreduce.reduce.shuffle.input.buffer.percent</name>
>>  <value>0.70</value>
>>  <description>The percentage of memory to be allocated from the maximum heap
>>  size to storing map outputs during the shuffle.(Buffer大小占Reduce可用内存的比例。默认值0.7)
>>  </description>
>></property>
>>
>>
>><property>
>>  <name>mapreduce.reduce.input.buffer.percent</name>
>>  <value>0.0</value>
>>  <description>The percentage of memory- relative to the maximum heap size- to
>>  retain map outputs during the reduce. When the shuffle is concluded, any
>>  remaining map outputs in memory must consume less than this threshold before
>>  the reduce can begin.(指定多少比例的内存用来存放Buffer中的数据，默认值是0.0)
>>  </description>
>></property>
>>
>>
>>
>><property>
>>    <description>The minimum allocation for every container request at the RM
>>    in MBs. Memory requests lower than this will be set to the value of this
>>    property. Additionally, a node manager that is configured to have less memory
>>    than this value will be shut down by the resource manager.(给应用程序Container分配的最小内存，默认值：1024)
>>    </description>
>>    <name>yarn.scheduler.minimum-allocation-mb</name>
>>    <value>1024</value>
>></property>
>>
>>
>>
>><property>
>>    <description>The maximum allocation for every container request at the RM
>>    in MBs. Memory requests higher than this will throw an
>>    InvalidResourceRequestException.(给应用程序Container分配的最大内存，默认值：8192)
>>    </description>
>>    <name>yarn.scheduler.maximum-allocation-mb</name>
>>    <value>8192</value>
>>  </property>
>>
>>
>>
>><property>
>>    <description>The maximum allocation for every container request at the RM
>>    in terms of virtual CPU cores. Requests higher than this will throw an
>>    InvalidResourceRequestException.(每个Container申请的最大CPU核数，默认值：4)
>>    </description>
>>    <name>yarn.scheduler.maximum-allocation-vcores</name>
>>    <value>4</value>
>>  </property>
>>
>>
>>
>> <property>
>>    <description>The minimum allocation for every container request at the RM
>>    in terms of virtual CPU cores. Requests lower than this will be set to the
>>    value of this property. Additionally, a node manager that is configured to
>>    have fewer virtual cores than this value will be shut down by the resource
>>    manager.(每个Container申请的最小CPU核数，默认值：1)
>>     </description>
>>    <name>yarn.scheduler.minimum-allocation-vcores</name>
>>    <value>1</value>
>>  </property>
>>
>>
>>
>>
>>
>>  <property>
>>    <description>Amount of physical memory, in MB, that can be allocated 
>>    for containers. If set to -1 and
>>    yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
>>    automatically calculated(in case of Windows and Linux).
>>    In other cases, the default is 8192MB.(给Containers分配的最大物理内存，默认值：8192)
>>    </description>
>>    <name>yarn.nodemanager.resource.memory-mb</name>
>>    <value>-1</value>
>>  </property>
>>
>>
>>
>>
>><property>
>>  <name>mapreduce.task.io.sort.mb</name>
>>  <value>100</value>
>>  <description>The total amount of buffer memory to use while sorting
>>  files, in megabytes.  By default, gives each merge stream 1MB, which
>>  should minimize seeks.(Shuffle的环形缓冲区大小，默认100m)
>>    </description>
>></property>
>>
>>
>>
>><property>
>>  <name>mapreduce.map.sort.spill.percent</name>
>>  <value>0.80</value>
>>  <description>The soft limit in the serialization buffer. Once reached, a
>>  thread will begin to spill the contents to disk in the background. Note that
>>  collection will not block if this threshold is exceeded while a spill is
>>  already in progress, so spills may be larger than this threshold when it is
>>  set to less than .5(环形缓冲区溢出的阈值，默认80%)
>>    </description>
>></property>
>>
>>
>>
>>
>><property>
>>  <name>mapreduce.map.maxattempts</name>
>>  <value>4</value>
>>  <description>Expert: The maximum number of attempts per map task.
>>  In other words, framework will try to execute a map task these many number
>>  of times before giving up on it.(每个Map Task最大重试次数，一旦重试次数超过该值，则认为Map Task运行失败，默认值：4)
>>  </description>
>></property>
>>
>>
>>
>>
>><property>
>>  <name>mapreduce.reduce.maxattempts</name>
>>  <value>4</value>
>>  <description>Expert: The maximum number of attempts per reduce task.
>>  In other words, framework will try to execute a reduce task these many number
>>  of times before giving up on it.(每个Reduce Task最大重试次数，一旦重试次数超过该值，则认为Map Task运行失败，默认值：4)
>>  </description>
>></property>
>>
>>
>>
>>
>><property>
>>  <name>mapreduce.task.timeout</name>
>>  <value>600000</value>
>>  <description>The number of milliseconds before a task will be
>>  terminated if it neither reads an input, writes an output, nor
>>  updates its status string.  A value of 0 disables the timeout.
>>      (Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个Task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该Task处于Block状态，可能是卡住了，也许永远会卡住，为了防止因为用户程序永远Block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000（10分钟）。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是：“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster.”。)
>>  </description>
>></property>
>>```
>
>## Hadoop小文件优化方案
>
>>**Hadoop小文件弊端**
>>
>>>HDFS上每个文件都要在NameNode上创建对应的元数据，这个元数据的大小约为150byte，这样当小文件比较多的时候，就会产生很多的元数据文件，一方面会大量占用NameNode的内存空间，另一方面就是元数据文件过多，使得寻址索引速度变慢。
>>>
>>>小文件过多，在进行MR计算时，会生成过多切片，需要启动过多的MapTask。每个MapTask处理的数据量小，导致MapTask的处理时间比启动时间还小，白白消耗资源。
>>
>>**Hadoop小文件解决方案**
>>
>>>**1)小文件优化方向**
>>>
>>>>（1）在数据采集的时候，就将小文件或小批数据合成大文件再上传HDFS。
>>>>
>>>>（2）在业务处理之前，在HDFS上使用MapReduce程序对小文件进行合并。
>>>>
>>>>（3）在MapReduce处理时，可采用CombineTextInputFormat提高效率。
>>>>
>>>>（4）开启uber模式，实现jvm重用
>>>
>>>**2)Hadoop Archive**
>>>
>>>>是一个高效的将小文件放入HDFS块中的文件存档工具，能够将多个小文件打包成一个HAR文件，从而达到减少NameNode的内存使用
>>>
>>>**SequenceFile**
>>>
>>>>SequenceFile是由一系列的二进制k/v组成，如果为key为文件名，value为文件内容，可将大批小文件合并成一个大文件
>>>
>>>**CombineTextInputFormat**
>>>
>>>>CombineTextInputFormat用于将多个小文件在切片过程中生成一个单独的切片或者少量的切片。
>>>
>>>**uber模式**
>>>
>>>>开启uber模式，实现jvm重用。默认情况下，每个Task任务都需要启动一个jvm来运行，如果Task任务计算的数据量很小，我们可以让同一个Job的多个Task运行在一个Jvm中，不必为每个Task都开启一个Jvm
>>>>
>>>>**mapred-site.xml**
>>>>
>>>>```xml
>>>><!--  开启uber模式 -->
>>>><property>
>>>>  <name>mapreduce.job.ubertask.enable</name>
>>>>  <value>true</value>
>>>></property>
>>>>
>>>><!-- uber模式中最大的mapTask数量，可向下修改  --> 
>>>><property>
>>>>  <name>mapreduce.job.ubertask.maxmaps</name>
>>>>  <value>9</value>
>>>></property>
>>>><!-- uber模式中最大的reduce数量，可向下修改 -->
>>>><property>
>>>>  <name>mapreduce.job.ubertask.maxreduces</name>
>>>>  <value>1</value>
>>>></property>
>>>><!-- uber模式中最大的输入数据量，默认使用dfs.blocksize 的值，可向下修改 -->
>>>><property>
>>>>  <name>mapreduce.job.ubertask.maxbytes</name>
>>>>  <value></value>
>>>></property>
>>>>
>>>>```
>>>>
>>>>

