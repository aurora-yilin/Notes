# Exec Source-HDFS Sink

***

>## Exec source官方案例
>
>Exec source runs a given Unix command on start-up and expects that process to continuously produce data on standard out (stderr is simply discarded, unless property logStdErr is set to true). If the process exits for any reason, the source also exits and will produce no further data. This means configurations such as `cat [named pipe]` or `tail -F [file]` are going to produce the desired results where as `date` will probably not - the former two commands produce streams of data where as the latter produces a single event and exits.
>
>Required properties are in **bold**.
>
>| Property Name   | Default     | Description                                                  |
>| :-------------- | :---------- | :----------------------------------------------------------- |
>| **channels**    | –           |                                                              |
>| **type**        | –           | The component type name, needs to be `exec`                  |
>| **command**     | –           | The command to execute                                       |
>| shell           | –           | A shell invocation used to run the command. e.g. /bin/sh -c. Required only for commands relying on shell features like wildcards, back ticks, pipes etc. |
>| restartThrottle | 10000       | Amount of time (in millis) to wait before attempting a restart |
>| restart         | false       | Whether the executed cmd should be restarted if it dies      |
>| logStdErr       | false       | Whether the command’s stderr should be logged                |
>| batchSize       | 20          | The max number of lines to read and send to the channel at a time |
>| batchTimeout    | 3000        | Amount of time (in milliseconds) to wait, if the buffer size was not reached, before data is pushed downstream |
>| selector.type   | replicating | replicating or multiplexing                                  |
>| selector.*      |             | Depends on the selector.type value                           |
>| interceptors    | –           | Space-separated list of interceptors                         |
>| interceptors.*  |             |                                                              |
>
>Warning：The problem with ExecSource and other asynchronous sources is that the source can not guarantee that if there is a failure to put the event into the Channel the client knows about it. In such cases, the data will be lost. As a for instance, one of the most commonly requested features is the `tail -F [file]`-like use case where an application writes to a log file on disk and Flume tails the file, sending each line as an event. While this is possible, there’s an obvious problem; what happens if the channel fills up and Flume can’t send an event? Flume has no way of indicating to the application writing the log file that it needs to retain the log or that the event hasn’t been sent, for some reason. If this doesn’t make sense, you need only know this: Your application can never guarantee data has been received when using a unidirectional asynchronous interface such as ExecSource! As an extension of this warning - and to be completely clear - there is absolutely zero guarantee of event delivery when using this source. For stronger reliability guarantees, consider the Spooling Directory Source, Taildir Source or direct integration with Flume via the SDK.
>
>Example for agent named a1:
>
>```properties
>a1.sources = r1
>a1.channels = c1
>a1.sources.r1.type = exec
>a1.sources.r1.command = tail -F /var/log/secure
>a1.sources.r1.channels = c1
>```
>
>The ‘shell’ config is used to invoke the ‘command’ through a command shell (such as Bash or Powershell). The ‘command’ is passed as an argument to ‘shell’ for execution. This allows the ‘command’ to use features from the shell such as wildcards, back ticks, pipes, loops, conditionals etc. In the absence of the ‘shell’ config, the ‘command’ will be invoked directly. Common values for ‘shell’ : ‘/bin/sh -c’, ‘/bin/ksh -c’, ‘cmd /c’, ‘powershell -Command’, etc.
>
>```properties
>a1.sources.tailsource-1.type = exec
>a1.sources.tailsource-1.shell = /bin/bash -c
>a1.sources.tailsource-1.command = for i in /path/*.txt; do cat $i; done
>```
>

***

***

## HDFS Sink官方案例

>This sink writes events into the Hadoop Distributed File System (HDFS). It currently supports creating text and sequence files. It supports compression in both file types. The files can be rolled (close current file and create a new one) periodically based on the elapsed time or size of data or number of events. It also buckets/partitions data by attributes like timestamp or machine where the event originated. The HDFS directory path may contain formatting escape sequences that will replaced by the HDFS sink to generate a directory/file name to store the events. Using this sink requires hadoop to be installed so that Flume can use the Hadoop jars to communicate with the HDFS cluster. Note that a version of Hadoop that supports the sync() call is required.
>
>The following are the escape sequences supported:
>
>| Alias        | Description                                                  |
>| :----------- | :----------------------------------------------------------- |
>| %{host}      | Substitute value of event header named “host”. Arbitrary header names are supported. |
>| %t           | Unix time in milliseconds                                    |
>| %a           | locale’s short weekday name (Mon, Tue, ...)                  |
>| %A           | locale’s full weekday name (Monday, Tuesday, ...)            |
>| %b           | locale’s short month name (Jan, Feb, ...)                    |
>| %B           | locale’s long month name (January, February, ...)            |
>| %c           | locale’s date and time (Thu Mar 3 23:05:25 2005)             |
>| %d           | day of month (01)                                            |
>| %e           | day of month without padding (1)                             |
>| %D           | date; same as %m/%d/%y                                       |
>| %H           | hour (00..23)                                                |
>| %I           | hour (01..12)                                                |
>| %j           | day of year (001..366)                                       |
>| %k           | hour ( 0..23)                                                |
>| %m           | month (01..12)                                               |
>| %n           | month without padding (1..12)                                |
>| %M           | minute (00..59)                                              |
>| %p           | locale’s equivalent of am or pm                              |
>| %s           | seconds since 1970-01-01 00:00:00 UTC                        |
>| %S           | second (00..59)                                              |
>| %y           | last two digits of year (00..99)                             |
>| %Y           | year (2010)                                                  |
>| %z           | +hhmm numeric timezone (for example, -0400)                  |
>| %[localhost] | Substitute the hostname of the host where the agent is running |
>| %[IP]        | Substitute the IP address of the host where the agent is running |
>| %[FQDN]      | Substitute the canonical hostname of the host where the agent is running |
>
>Note: The escape strings %[localhost], %[IP] and %[FQDN] all rely on Java’s ability to obtain the hostname, which may fail in some networking environments.
>
>The file in use will have the name mangled to include ”.tmp” at the end. Once the file is closed, this extension is removed. This allows excluding partially complete files in the directory. Required properties are in **bold**.
>
>Note：For all of the time related escape sequences, a header with the key “timestamp” must exist among the headers of the event (unless `hdfs.useLocalTimeStamp` is set to `true`). One way to add this automatically is to use the TimestampInterceptor.
>
>| Name                   | Default      | Description                                                  |
>| :--------------------- | :----------- | :----------------------------------------------------------- |
>| **channel**            | –            |                                                              |
>| **type**               | –            | The component type name, needs to be `hdfs`                  |
>| **hdfs.path**          | –            | HDFS directory path (eg hdfs://namenode/flume/webdata/)      |
>| hdfs.filePrefix        | FlumeData    | Name prefixed to files created by Flume in hdfs directory    |
>| hdfs.fileSuffix        | –            | Suffix to append to file (eg `.avro` - *NOTE: period is not automatically added*) |
>| hdfs.inUsePrefix       | –            | Prefix that is used for temporal files that flume actively writes into |
>| hdfs.inUseSuffix       | `.tmp`       | Suffix that is used for temporal files that flume actively writes into |
>| hdfs.emptyInUseSuffix  | false        | If `false` an `hdfs.inUseSuffix` is used while writing the output. After closing the output `hdfs.inUseSuffix` is removed from the output file name. If `true` the `hdfs.inUseSuffix` parameter is ignored an empty string is used instead. |
>| hdfs.rollInterval      | 30           | Number of seconds to wait before rolling current file (0 = never roll based on time interval) |
>| hdfs.rollSize          | 1024         | File size to trigger roll, in bytes (0: never roll based on file size) |
>| hdfs.rollCount         | 10           | Number of events written to file before it rolled (0 = never roll based on number of events) |
>| hdfs.idleTimeout       | 0            | Timeout after which inactive files get closed (0 = disable automatic closing of idle files) |
>| hdfs.batchSize         | 100          | number of events written to file before it is flushed to HDFS |
>| hdfs.codeC             | –            | Compression codec. one of following : gzip, bzip2, lzo, lzop, snappy |
>| hdfs.fileType          | SequenceFile | File format: currently `SequenceFile`, `DataStream` or `CompressedStream` (1)DataStream will not compress output file and please don’t set codeC (2)CompressedStream requires set hdfs.codeC with an available codeC |
>| hdfs.maxOpenFiles      | 5000         | Allow only this number of open files. If this number is exceeded, the oldest file is closed. |
>| hdfs.minBlockReplicas  | –            | Specify minimum number of replicas per HDFS block. If not specified, it comes from the default Hadoop config in the classpath. |
>| hdfs.writeFormat       | Writable     | Format for sequence file records. One of `Text` or `Writable`. Set to `Text` before creating data files with Flume, otherwise those files cannot be read by either Apache Impala (incubating) or Apache Hive. |
>| hdfs.threadsPoolSize   | 10           | Number of threads per HDFS sink for HDFS IO ops (open, write, etc.) |
>| hdfs.rollTimerPoolSize | 1            | Number of threads per HDFS sink for scheduling timed file rolling |
>| hdfs.kerberosPrincipal | –            | Kerberos user principal for accessing secure HDFS            |
>| hdfs.kerberosKeytab    | –            | Kerberos keytab for accessing secure HDFS                    |
>| hdfs.proxyUser         |              |                                                              |
>| hdfs.round             | false        | Should the timestamp be rounded down (if true, affects all time based escape sequences except %t) |
>| hdfs.roundValue        | 1            | Rounded down to the highest multiple of this (in the unit configured using `hdfs.roundUnit`), less than current time. |
>| hdfs.roundUnit         | second       | The unit of the round down value - `second`, `minute` or `hour`. |
>| hdfs.timeZone          | Local Time   | Name of the timezone that should be used for resolving the directory path, e.g. America/Los_Angeles. |
>| hdfs.useLocalTimeStamp | false        | Use the local time (instead of the timestamp from the event header) while replacing the escape sequences. |
>| hdfs.closeTries        | 0            | Number of times the sink must try renaming a file, after initiating a close attempt. If set to 1, this sink will not re-try a failed rename (due to, for example, NameNode or DataNode failure), and may leave the file in an open state with a .tmp extension. If set to 0, the sink will try to rename the file until the file is eventually renamed (there is no limit on the number of times it would try). The file may still remain open if the close call fails but the data will be intact and in this case, the file will be closed only after a Flume restart. |
>| hdfs.retryInterval     | 180          | Time in seconds between consecutive attempts to close a file. Each close call costs multiple RPC round-trips to the Namenode, so setting this too low can cause a lot of load on the name node. If set to 0 or less, the sink will not attempt to close the file if the first attempt fails, and may leave the file open or with a ”.tmp” extension. |
>| serializer             | `TEXT`       | Other possible options include `avro_event` or the fully-qualified class name of an implementation of the `EventSerializer.Builder` interface. |
>| serializer.*           |              |                                                              |
>
>Deprecated Properties
>
>Name Default Description ====================== ============ ====================================================================== hdfs.callTimeout 30000 Number of milliseconds allowed for HDFS operations, such as open, write, flush, close. This number should be increased if many HDFS timeout operations are occurring. ====================== ============ ======================================================================
>
>Example for agent named a1:
>
>```properties
>a1.channels = c1
>a1.sinks = k1
>a1.sinks.k1.type = hdfs
>a1.sinks.k1.channel = c1
>a1.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H%M/%S
>a1.sinks.k1.hdfs.filePrefix = events-
>a1.sinks.k1.hdfs.round = true
>a1.sinks.k1.hdfs.roundValue = 10
>a1.sinks.k1.hdfs.roundUnit = minute
>```
>
>The above configuration will round down the timestamp to the last 10th minute. For example, an event with timestamp 11:54:34 AM, June 12, 2012 will cause the hdfs path to become `/flume/events/2012-06-12/1150/00`.

***

***

## 自定义案例

>```properties
># example.conf: A single-node Flume configuration
>
># Name the components on this agent
>a1.sources = r1
>a1.sinks = k1
>a1.channels = c1
>
># Describe/configure the source
>a1.sources.r1.type = exec
>a1.sources.r1.command = tail -F /opt/module/hive-3.1.2/logs/hive.log
># Describe the sink
>a1.sinks.k1.type = hdfs
>#设置hdfs路径
>a1.sinks.k1.hdfs.path = /flume/events/day=%Y-%m-%d/hour=%H
>#设置文件前缀
>a1.sinks.k1.hdfs.filePrefix = hive_logs
>#是否对时间戳取整
>a1.sinks.k1.hdfs.round = true
>#多少时间单位创建一个新的文件夹
>a1.sinks.k1.hdfs.roundValue = 1
>##重新定义时间单位
>a1.sinks.k1.hdfs.roundUnit = hour
>##是否使用本地时间戳
>a1.sinks.k1.hdfs.useLocalTimeStamp = true
>##积攒多少个Event才flush到HDFS一次
>a1.sinks.k1.hdfs.batchSize = 100
>##设置文件类型，可支持压缩
>a1.sinks.k1.hdfs.fileType = CompressedStream
>##设置文件的压缩类型
>a1.sinks.k1.hdfs.codeC = snappy
>##多久生成一个新的文件
>a1.sinks.k1.hdfs.rollInterval = 10
>##设置每个文件的滚动大小
>a1.sinks.k1.hdfs.rollSize = 134217700
>##文件的滚动与Event数量无关
>a1.sinks.k1.hdfs.rollCount = 0
># Use a channel which buffers events in memory
>a1.channels.c1.type = memory
>a1.channels.c1.capacity = 1000
>a1.channels.c1.transactionCapacity = 100
>
># Bind the source and sink to the channel
>a1.sources.r1.channels = c1
>a1.sinks.k1.channel = c1
>
>```
>
>
>
>