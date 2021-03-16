# Hadoop使用LZO压缩

## core-site.xml

```xml
	<property>
        <name>io.compression.codecs</name>
        <value>
            org.apache.hadoop.io.compress.GzipCodec,
            org.apache.hadoop.io.compress.DefaultCodec,
            org.apache.hadoop.io.compress.BZip2Codec,
            org.apache.hadoop.io.compress.SnappyCodec,

            com.hadoop.compression.lzo.LzoCodec,
            com.hadoop.compression.lzo.LzopCodec
        </value>
    </property>
```

## mapred-site.xml

```xml
#开启mr输出时的压缩
    <property>
        <name>mapreduce.output.fileoutputformat.compress</name>
        <value>true</value>
    </property>
    
    <property>
        <name>mapreduce.output.fileoutputformat.compress.codec</name>
        <value>org.apache.hadoop.io.compress.BZip2Codec</value>
    </property> 
    
    
    #开启mr中map阶段的输出压缩
    <property>
        <name>mapreduce.map.output.compress</name>
        <value>true</value>
    </property>
    
    #指定mr中map阶段的输出压缩为Snappy
    <property>
        <name>mapreduce.map.output.compress.codec</name>
        <value>org.apache.hadoop.io.compress.SnappyCodec</value>
    </property>
```

[参考文章](https://www.jianshu.com/p/10fc6c511761)

