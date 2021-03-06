
# ETL
```

一条最小的生产线一分钟内可以达到100M的生产数据量，经过ORC列式存储和LZO压缩过后，只有约30M的数据量在HDFS上，

一条日志记录涵盖有50多个字段，由于业务需求是进行多个维度的进行统计(统计请求次数，请求耗费的流量，带宽的统计，

状态码的统计，Top URL的统计，用户区域统计等等)，需要提取有用的字段，还会添加一些分区时间字段，最小的批次是5分钟一个批次，

设置一个分区进行存储，按照年月日时分分区，每5分钟存储一次，修改文件名称，清洗数据时特别是对IP的解析，由于公司会采用IP库，

每个月都会实时的更新，只管调用接口即可解析相应的字段。按照日志中的logtime时间字段来存放对应的分区中，时间解析过程中要采用

线程安全的方式进行解析，否则的话，Spark是基于线程的计算引擎，生产环境中必乱套，

作业一分钟提交一次，如何保证作业的自动提交，以及数据重跑机制的策略，数据不完整时，短信预警提示的功能，

当采用Spark-Submit方式提交，每个批次的作业都独立提交，其中启动SparkContext，由于是Spark on YARN的每

个Spark Application在启动过程中要占用大量时间，大约40~50秒：每个批次的作业都是单独的Application，会在YARN作

业列表上显示很多个Application；会频繁的向YARN提交作业和资源的申请；当YARN发生主备切换时，会有作业提交不到集群

上；当YARN RM HA都挂了时，无法提交作业到YARN上运行，数据的SLA无法满足；自行开发一个ThriftServer服务器来获

取SparkContext资源提交给NodeManager执行。 

  A）将每个批次的作用统一发送到一个SparkContext上，共同占用集群资源；

  B）在YARN上只有一个Application；

  C）即使发生了主备切换或者YARN RM HA挂了，都没关系，因为作业所需要的资源在Server启动时就已经申请完毕了，只要

  NM正常运行那么作业都能提交上来并执行，SLA有很好的保证。

  D）作业提交时只需要发送一个HTTP请求即可，并不需要通过Spark-Submit进行重量级的作业提交；

各种不同维度的计算采用Hive/Spark SQL/Impala执行引擎，数据展示采用Echarts、Zeppelin、HUE等工具。

