# sqoop import语法简单使用

***

>## import主要参数介绍
>
>>```sql
>>bin/sqoop import \
>>--connect jdbc:mysql://hadoop102:3306/gmall \ 连接地址
>>--username root \ 用户名
>>--password 123456 \ 密码
>>--table user_info \ 表名
>>--columns id,login_name \ 列名
>>--where "id>=10 and id<=30" \ 过滤条件
>>--target-dir /test \ hdfs的目标目录
>>--delete-target-dir \ 删除目标目录的原有文件
>>--fields-terminated-by '\t' \ 写入的行切分
>>--num-mappers 2 \ map个数
>>--split-by id  \ 切分map的条件
>>```
>
>***
>
>## import案例
>
>>```sql
>>import_data(){
>>$sqoop import \
>>--connect jdbc:mysql://hadoop122:3306/$APP \
>>--username root \
>>--password 'Natural;follow' \
>>#导入HDFS的目标路径
>>--target-dir /origin_data/$APP/db/$1/$do_date \
>>#sqoop导入导出数据的过程是只有M的MR过程，要保证输出路径不存在该属性的含义就是删除target-dir指定的路径
>>--delete-target-dir \
>>#执行sql查询语句
>>--query "$2 and  \$CONDITIONS" \
>>#设置Map数量
>>--num-mappers 1 \
>>#以指定的符号分割字段
>>--fields-terminated-by '\t' \
>>#对输出的数据采用压缩
>>--compress \
>>#指定输出的数据的压缩格式
>>--compression-codec lzop \
>>#对于string类型的null替换为指定值
>>--null-string '\\N' \
>>#对于非string类型的null替换为指定值
>>--null-non-string '\\N'
>>
>>#对上传到hdfs目录下的lzo文件创建索引
>>hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /origin_data/$APP/db/$1/$do_date
>>}
>>```
>
>

