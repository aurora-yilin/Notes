# Taildir Source-HDFS Sink

***

## Taildir Source官方案例

>Note：**This source is provided as a preview feature. It does not work on Windows.**
>
>Watch the specified files, and tail them in nearly real-time once detected new lines appended to the each files. If the new lines are being written, this source will retry reading them in wait for the completion of the write.
>
>This source is reliable and will not miss data even when the tailing files rotate. It periodically writes the last read position of each files on the given position file in JSON format. If Flume is stopped or down for some reason, it can restart tailing from the position written on the existing position file.
>
>In other use case, this source can also start tailing from the arbitrary position for each files using the given position file. When there is no position file on the specified path, it will start tailing from the first line of each files by default.
>
>Files will be consumed in order of their modification time. File with the oldest modification time will be consumed first.
>
>This source does not rename or delete or do any modifications to the file being tailed. Currently this source does not support tailing binary files. It reads text files line by line.
>
>| Property Name                       | Default                        | Description                                                  |
>| :---------------------------------- | :----------------------------- | :----------------------------------------------------------- |
>| **channels**                        | –                              |                                                              |
>| **type**                            | –                              | The component type name, needs to be `TAILDIR`.              |
>| **filegroups**                      | –                              | Space-separated list of file groups. Each file group indicates a set of files to be tailed. |
>| **filegroups.<filegroupName>**      | –                              | Absolute path of the file group. Regular expression (and not file system patterns) can be used for filename only. |
>| positionFile                        | ~/.flume/taildir_position.json | File in JSON format to record the inode, the absolute path and the last position of each tailing file. |
>| headers.<filegroupName>.<headerKey> | –                              | Header value which is the set with header key. Multiple headers can be specified for one file group. |
>| byteOffsetHeader                    | false                          | Whether to add the byte offset of a tailed line to a header called ‘byteoffset’. |
>| skipToEnd                           | false                          | Whether to skip the position to EOF in the case of files not written on the position file. |
>| idleTimeout                         | 120000                         | Time (ms) to close inactive files. If the closed file is appended new lines to, this source will automatically re-open it. |
>| writePosInterval                    | 3000                           | Interval time (ms) to write the last position of each file on the position file. |
>| batchSize                           | 100                            | Max number of lines to read and send to the channel at a time. Using the default is usually fine. |
>| maxBatchCount                       | Long.MAX_VALUE                 | Controls the number of batches being read consecutively from the same file. If the source is tailing multiple files and one of them is written at a fast rate, it can prevent other files to be processed, because the busy file would be read in an endless loop. In this case lower this value. |
>| backoffSleepIncrement               | 1000                           | The increment for time delay before reattempting to poll for new data, when the last attempt did not find any new data. |
>| maxBackoffSleep                     | 5000                           | The max time delay between each reattempt to poll for new data, when the last attempt did not find any new data. |
>| cachePatternMatching                | true                           | Listing directories and applying the filename regex pattern may be time consuming for directories containing thousands of files. Caching the list of matching files can improve performance. The order in which files are consumed will also be cached. Requires that the file system keeps track of modification times with at least a 1-second granularity. |
>| fileHeader                          | false                          | Whether to add a header storing the absolute path filename.  |
>| fileHeaderKey                       | file                           | Header key to use when appending absolute path filename to event header. |
>
>Example for agent named a1:
>
>```properties
>a1.sources = r1
>a1.channels = c1
>a1.sources.r1.type = TAILDIR
>a1.sources.r1.channels = c1
>a1.sources.r1.positionFile = /var/log/flume/taildir_position.json
>a1.sources.r1.filegroups = f1 f2
>a1.sources.r1.filegroups.f1 = /var/log/test1/example.log
>a1.sources.r1.headers.f1.headerKey1 = value1
>a1.sources.r1.filegroups.f2 = /var/log/test2/.*log.*
>a1.sources.r1.headers.f2.headerKey1 = value2
>a1.sources.r1.headers.f2.headerKey2 = value2-2
>a1.sources.r1.fileHeader = true
>a1.sources.ri.maxBatchCount = 1000
>```
>
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
>```
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

## 案例实现

>```properties
># example.conf: A single-node Flume configuration
>
># Name the components on this agent
>a1.sources = r1
>a1.sinks = k1
>a1.channels = c1
>
># Describe/configure the source
>a1.sources.r1.type = TAILDIR
>a1.sources.r1.positionFile = /opt/logs/json/tail_dir.json
>a1.sources.r1.filegroups = f1 f2
>a1.sources.r1.filegroups.f1 = /opt/logs/file1.txt
>a1.sources.r1.filegroups.f1 = /opt/logs/file.*.txt
># Describe the sink
>a1.sinks.k1.type = hdfs
>a1.sinks.k1.hdfs.path = /tailDir/test/day=%Y-%m-%d/hour=%H
>a1.sinks.k1.hdfs.filePrefix = tailDir-test
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
>#
>
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

