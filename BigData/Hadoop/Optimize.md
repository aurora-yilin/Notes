# hadoop优化

***

```xml
<property>
  <name>mapreduce.map.memory.mb</name>
  <value>-1</value>
  <description>The amount of memory to request from the scheduler for each
    map task. If this is not specified or is non-positive, it is inferred from
    mapreduce.map.java.opts and mapreduce.job.heap.memory-mb.ratio.
    If java-opts are also not specified, we set it to 1024.(一个MapTask可使用的资源上限（单位:MB），默认为1024。如果MapTask实际使用的资源量超过该值，则会被强制杀死。)
  </description>
</property>


<property>
  <name>mapreduce.reduce.memory.mb</name>
  <value>-1</value>
  <description>The amount of memory to request from the scheduler for each
    reduce task. If this is not specified or is non-positive, it is inferred
    from mapreduce.reduce.java.opts and mapreduce.job.heap.memory-mb.ratio.
    If java-opts are also not specified, we set it to 1024.(一个ReduceTask可使用的资源上限（单位:MB），默认为1024。如果ReduceTask实际使用的资源量超过该值，则会被强制杀死。)
  </description>
</property>


<property>
  <name>mapreduce.map.cpu.vcores</name>
  <value>1</value>
  <description>The number of virtual cores to request from the scheduler for
  each map task.(每个MapTask可使用的最多cpu core数目，默认值: 1)
  </description>
</property>


<property>
  <name>mapreduce.reduce.cpu.vcores</name>
  <value>1</value>
  <description>The number of virtual cores to request from the scheduler for
  each reduce task.(每个ReduceTask可使用的最多cpu core数目，默认值: 1)
  </description>
</property>


<property>
  <name>mapreduce.reduce.shuffle.parallelcopies</name>
  <value>5</value>
  <description>The default number of parallel transfers run by reduce
  during the copy(shuffle) phase.(每个Reduce去Map中取数据的并行数。默认值是5)
  </description>
</property>



<property>
  <name>mapreduce.reduce.shuffle.merge.percent</name>
  <value>0.66</value>
  <description>The usage threshold at which an in-memory merge will be
  initiated, expressed as a percentage of the total memory allocated to
  storing in-memory map outputs, as defined by
  mapreduce.reduce.shuffle.input.buffer.percent.(Buffer中的数据达到多少比例开始写入磁盘。默认值0.66)
  </description>
</property>



<property>
  <name>mapreduce.reduce.shuffle.input.buffer.percent</name>
  <value>0.70</value>
  <description>The percentage of memory to be allocated from the maximum heap
  size to storing map outputs during the shuffle.(Buffer大小占Reduce可用内存的比例。默认值0.7)
  </description>
</property>


<property>
  <name>mapreduce.reduce.input.buffer.percent</name>
  <value>0.0</value>
  <description>The percentage of memory- relative to the maximum heap size- to
  retain map outputs during the reduce. When the shuffle is concluded, any
  remaining map outputs in memory must consume less than this threshold before
  the reduce can begin.(指定多少比例的内存用来存放Buffer中的数据，默认值是0.0)
  </description>
</property>



<property>
    <description>The minimum allocation for every container request at the RM
    in MBs. Memory requests lower than this will be set to the value of this
    property. Additionally, a node manager that is configured to have less memory
    than this value will be shut down by the resource manager.(给应用程序Container分配的最小内存，默认值：1024)
    </description>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>1024</value>
</property>



<property>
    <description>The maximum allocation for every container request at the RM
    in MBs. Memory requests higher than this will throw an
    InvalidResourceRequestException.(给应用程序Container分配的最大内存，默认值：8192)
    </description>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>8192</value>
  </property>



<property>
    <description>The maximum allocation for every container request at the RM
    in terms of virtual CPU cores. Requests higher than this will throw an
    InvalidResourceRequestException.(每个Container申请的最大CPU核数，默认值：4)
    </description>
    <name>yarn.scheduler.maximum-allocation-vcores</name>
    <value>4</value>
  </property>



 <property>
    <description>The minimum allocation for every container request at the RM
    in terms of virtual CPU cores. Requests lower than this will be set to the
    value of this property. Additionally, a node manager that is configured to
    have fewer virtual cores than this value will be shut down by the resource
    manager.(每个Container申请的最小CPU核数，默认值：1)
     </description>
    <name>yarn.scheduler.minimum-allocation-vcores</name>
    <value>1</value>
  </property>





  <property>
    <description>Amount of physical memory, in MB, that can be allocated 
    for containers. If set to -1 and
    yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
    automatically calculated(in case of Windows and Linux).
    In other cases, the default is 8192MB.(给Containers分配的最大物理内存，默认值：8192)
    </description>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>-1</value>
  </property>




<property>
  <name>mapreduce.task.io.sort.mb</name>
  <value>100</value>
  <description>The total amount of buffer memory to use while sorting
  files, in megabytes.  By default, gives each merge stream 1MB, which
  should minimize seeks.(Shuffle的环形缓冲区大小，默认100m)
    </description>
</property>



<property>
  <name>mapreduce.map.sort.spill.percent</name>
  <value>0.80</value>
  <description>The soft limit in the serialization buffer. Once reached, a
  thread will begin to spill the contents to disk in the background. Note that
  collection will not block if this threshold is exceeded while a spill is
  already in progress, so spills may be larger than this threshold when it is
  set to less than .5(环形缓冲区溢出的阈值，默认80%)
    </description>
</property>




<property>
  <name>mapreduce.map.maxattempts</name>
  <value>4</value>
  <description>Expert: The maximum number of attempts per map task.
  In other words, framework will try to execute a map task these many number
  of times before giving up on it.(每个Map Task最大重试次数，一旦重试次数超过该值，则认为Map Task运行失败，默认值：4)
  </description>
</property>




<property>
  <name>mapreduce.reduce.maxattempts</name>
  <value>4</value>
  <description>Expert: The maximum number of attempts per reduce task.
  In other words, framework will try to execute a reduce task these many number
  of times before giving up on it.(每个Reduce Task最大重试次数，一旦重试次数超过该值，则认为Map Task运行失败，默认值：4)
  </description>
</property>




<property>
  <name>mapreduce.task.timeout</name>
  <value>600000</value>
  <description>The number of milliseconds before a task will be
  terminated if it neither reads an input, writes an output, nor
  updates its status string.  A value of 0 disables the timeout.
      (Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个Task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该Task处于Block状态，可能是卡住了，也许永远会卡住，为了防止因为用户程序永远Block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000（10分钟）。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是：“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster.”。)
  </description>
</property>
```



