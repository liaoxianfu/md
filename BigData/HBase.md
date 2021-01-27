## HBase特点

海量存储、列式存储、极易扩展、高并发、稀疏。

## HBase架构

![image-20200813151111083](D:\MarkDown\BigData\img\image-20200813151111083.png)

### client

Client包含了访问Hbase的接口，另外Client还维护了对应的cache来加速Hbase的访问，比如cache的.META.元数据的信息。

### Zookeeper

Zookeeper的高可用、RegionServer的监控、元数据的入口以及集群配置的维护等工作。具体工作如下：

通过Zoopkeeper来保证集群中只有1个master在运行，如果master异常，会通过竞争机制产生新的master提供服务

通过Zoopkeeper来监控RegionServer的状态，当RegionSevrer有异常的时候，通过回调的形式通知Master RegionServer上下线的信息

通过Zoopkeeper存储元数据的统一入口地址

### Hmaster

HmastergionServer分配Region
 维护整个集群的负载均衡
 维护集群的元数据信息
 发现失效的Region，并将失效的Region分配到正常的RegionServer上
 当RegionSever失效的时候，协调对应Hlog的拆分

### HregionServer

HregionServer直接对接用户的读写请求，是真正的“干活”的节点。它的功能概括如下：
 管理master为其分配的Region
 处理来自客户端的读写请求
 负责和底层HDFS的交互，存储数据到HDFS
 负责Region变大以后的拆分
 负责Storefile的合并工作

### HDFS

HDFS为Hbase提供最终的底层数据存储服务，同时为HBase提供高可用（Hlog存储在HDFS）的支持，具体功能概括如下：
 提供元数据和表数据的底层分布式存储服务
 数据多副本，保证的高可靠和高可用性



put 'student' ,'1002','info:name','liao'

scan 'student',{STARTROW=>'1001'}

put 'student','1002','info:name','wang'

get 'student','1002'

delete 'student','1002','info:name'



deleteall 'student','1001'



truncate 'student'