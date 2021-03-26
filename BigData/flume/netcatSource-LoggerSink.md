# flume Source TCP-Logger Sink

***

## NetCat官方案例

>#### NetCat TCP Source
>
>A netcat-like source that listens on a given port and turns each line of text into an event. Acts like `nc -k -l [host] [port]`. In other words, it opens a specified port and listens for data. The expectation is that the supplied data is newline separated text. Each line of text is turned into a Flume event and sent via the connected channel.
>
>Required properties are in **bold**.
>
>| Property Name   | Default     | Description                                   |
>| :-------------- | :---------- | :-------------------------------------------- |
>| **channels**    | –           |                                               |
>| **type**        | –           | The component type name, needs to be `netcat` |
>| **bind**        | –           | Host name or IP address to bind to            |
>| **port**        | –           | Port # to bind to                             |
>| max-line-length | 512         | Max line length per event body (in bytes)     |
>| ack-every-event | true        | Respond with an “OK” for every event received |
>| selector.type   | replicating | replicating or multiplexing                   |
>| selector.*      |             | Depends on the selector.type value            |
>| interceptors    | –           | Space-separated list of interceptors          |
>| interceptors.*  |             |                                               |
>
>Example for agent named a1:
>
>```
>a1.sources = r1
>a1.channels = c1
>a1.sources.r1.type = netcat
>a1.sources.r1.bind = 0.0.0.0
>a1.sources.r1.port = 6666
>a1.sources.r1.channels = c1
>```
>

***

***

## Logger Sink官方案例

>Logs event at INFO level. Typically useful for testing/debugging purpose. Required properties are in **bold**. This sink is the only exception which doesn’t require the extra configuration explained in the [Logging raw data](http://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html#logging-raw-data) section.
>
>| Property Name | Default | Description                                      |
>| :------------ | :------ | :----------------------------------------------- |
>| **channel**   | –       |                                                  |
>| **type**      | –       | The component type name, needs to be `logger`    |
>| maxBytesToLog | 16      | Maximum number of bytes of the Event body to log |
>
>Example for agent named a1:
>
>```properties
>a1.channels = c1
>a1.sinks = k1
>a1.sinks.k1.type = logger
>a1.sinks.k1.channel = c1
>```

***

***

## 测试案例

>```properties
># example.conf: A single-node Flume configuration
>
># Name the components on this agent
>a1.sources = r1
>a1.sinks = k1
>a1.channels = c1
>
># Describe/configure the source
>a1.sources.r1.type = netcat
>a1.sources.r1.bind = localhost
>a1.sources.r1.port = 44444
>
># Describe the sink
>a1.sinks.k1.type = logger
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