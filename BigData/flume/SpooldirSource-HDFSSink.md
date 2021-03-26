# Spooling Directory Source-HDFS Sink

***

## Spooling Directory Source官方案例

>This source lets you ingest data by placing files to be ingested into a “spooling” directory on disk. This source will watch the specified directory for new files, and will parse events out of new files as they appear. The event parsing logic is pluggable. After a given file has been fully read into the channel, completion by default is indicated by renaming the file or it can be deleted or the trackerDir is used to keep track of processed files.
>
>Unlike the Exec source, this source is reliable and will not miss data, even if Flume is restarted or killed. In exchange for this reliability, only immutable, uniquely-named files must be dropped into the spooling directory. Flume tries to detect these problem conditions and will fail loudly if they are violated:
>
>1. If a file is written to after being placed into the spooling directory, Flume will print an error to its log file and stop processing.
>2. If a file name is reused at a later time, Flume will print an error to its log file and stop processing.
>
>To avoid the above issues, it may be useful to add a unique identifier (such as a timestamp) to log file names when they are moved into the spooling directory.
>
>Despite the reliability guarantees of this source, there are still cases in which events may be duplicated if certain downstream failures occur. This is consistent with the guarantees offered by other Flume components.
>
>| Property Name            | Default     | Description                                                  |
>| :----------------------- | :---------- | :----------------------------------------------------------- |
>| **channels**             | –           |                                                              |
>| **type**                 | –           | The component type name, needs to be `spooldir`.             |
>| **spoolDir**             | –           | The directory from which to read files from.                 |
>| fileSuffix               | .COMPLETED  | Suffix to append to completely ingested files                |
>| deletePolicy             | never       | When to delete completed files: `never` or `immediate`       |
>| fileHeader               | false       | Whether to add a header storing the absolute path filename.  |
>| fileHeaderKey            | file        | Header key to use when appending absolute path filename to event header. |
>| basenameHeader           | false       | Whether to add a header storing the basename of the file.    |
>| basenameHeaderKey        | basename    | Header Key to use when appending basename of file to event header. |
>| includePattern           | ^.*$        | Regular expression specifying which files to include. It can used together with `ignorePattern`. If a file matches both `ignorePattern` and `includePattern`regex, the file is ignored. |
>| ignorePattern            | ^$          | Regular expression specifying which files to ignore (skip). It can used together with `includePattern`. If a file matches both `ignorePattern` and `includePattern` regex, the file is ignored. |
>| trackerDir               | .flumespool | Directory to store metadata related to processing of files. If this path is not an absolute path, then it is interpreted as relative to the spoolDir. |
>| trackingPolicy           | rename      | The tracking policy defines how file processing is tracked. It can be “rename” or “tracker_dir”. This parameter is only effective if the deletePolicy is “never”. “rename” - After processing files they get renamed according to the fileSuffix parameter. “tracker_dir” - Files are not renamed but a new empty file is created in the trackerDir. The new tracker file name is derived from the ingested one plus the fileSuffix. |
>| consumeOrder             | oldest      | In which order files in the spooling directory will be consumed `oldest`, `youngest` and `random`. In case of `oldest` and `youngest`, the last modified time of the files will be used to compare the files. In case of a tie, the file with smallest lexicographical order will be consumed first. In case of `random`any file will be picked randomly. When using `oldest` and `youngest` the whole directory will be scanned to pick the oldest/youngest file, which might be slow if there are a large number of files, while using `random` may cause old files to be consumed very late if new files keep coming in the spooling directory. |
>| pollDelay                | 500         | Delay (in milliseconds) used when polling for new files.     |
>| recursiveDirectorySearch | false       | Whether to monitor sub directories for new files to read.    |
>| maxBackoff               | 4000        | The maximum time (in millis) to wait between consecutive attempts to write to the channel(s) if the channel is full. The source will start at a low backoff and increase it exponentially each time the channel throws a ChannelException, upto the value specified by this parameter. |
>| batchSize                | 100         | Granularity at which to batch transfer to the channel        |
>| inputCharset             | UTF-8       | Character set used by deserializers that treat the input file as text. |
>| decodeErrorPolicy        | `FAIL`      | What to do when we see a non-decodable character in the input file. `FAIL`: Throw an exception and fail to parse the file. `REPLACE`: Replace the unparseable character with the “replacement character” char, typically Unicode U+FFFD. `IGNORE`: Drop the unparseable character sequence. |
>| deserializer             | `LINE`      | Specify the deserializer used to parse the file into events. Defaults to parsing each line as an event. The class specified must implement `EventDeserializer.Builder`. |
>| deserializer.*           |             | Varies per event deserializer.                               |
>| bufferMaxLines           | –           | (Obselete) This option is now ignored.                       |
>| bufferMaxLineLength      | 5000        | (Deprecated) Maximum length of a line in the commit buffer. Use deserializer.maxLineLength instead. |
>| selector.type            | replicating | replicating or multiplexing                                  |
>| selector.*               |             | Depends on the selector.type value                           |
>| interceptors             | –           | Space-separated list of interceptors                         |
>| interceptors.*           |             |                                                              |
>
>Example for an agent named agent-1:
>
>```properties
>a1.channels = ch-1
>a1.sources = src-1
>
>a1.sources.src-1.type = spooldir
>a1.sources.src-1.channels = ch-1
>a1.sources.src-1.spoolDir = /var/log/apache/flumeSpool
>a1.sources.src-1.fileHeader = true
>```

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

## 实现案例

>```properties
># example.conf: A single-node Flume configuration
>
># Name the components on this agent
>a1.sources = r1
>a1.sinks = k1
>a1.channels = c1
>
># Describe/configure the source
>a1.sources.r1.type = spooldir
>a1.sources.r1.spoolDir = /opt/logs
>a1.sources.r1.fileSuffix = .COMPLETED
>#忽略所有以.tmp结尾的文件，不上传
>a1.sources.r1.ignorePattern = ([^ ]*\.tmp)
># Describe the sink
>a1.sinks.k1.type = hdfs
>a1.sinks.k1.hdfs.path = /hive/logs/day=%Y-%m-%d/hour=%H
>a1.sinks.k1.hdfs.filePrefix = hive_logs
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
>```
>
>
>
>