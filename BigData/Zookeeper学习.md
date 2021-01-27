### Zookeeper学习

#### 工作机制

Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应，从而实现集群中类似Master/Slave管理模式

#### 工作流程

1、服务端集群向Zookeeper集群进行注册

2、客户端通过zookeeper进行读取在线服务器的列表并注册监听

3、当某个服务器下线后 Zookeeper对客户端进行下线通知

4、客户端刷新服务器列表并再次进行注册监听



#### zookeeper特点

![image-20200808080211948](D:\MarkDown\BigData\img\image-20200808080211948.png)

1）Zookeeper：一个领导者（leader），多个跟随者（follower）组成的集群。
2）Leader负责进行投票的发起和决议，更新系统状态
3）Follower用于接收客户请求并向客户端返回结果，在选举Leader过程中参与投票
4）集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。
5）全局数据一致：每个server保存一份相同的数据副本，client无论连接到哪个server，数据都是一致的。
6）更新请求顺序进行，来自同一个client的更新请求按其发送顺序依次执行。
7）数据更新原子性，一次数据更新要么成功，要么失败。
8）实时性，在一定时间范围内，client能读到最新数据。

#### 数据结构

ZooKeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一棵树，每个节点称做一个ZNode。每一个ZNode默认能够存储1MB的数据，每个ZNode都可以通过其路径唯一标识。

![image-20200808080434954](D:\MarkDown\BigData\img\image-20200808080434954.png)

#### 应用场景

提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等

#### 安装zookeeper

下载地址

https://zookeeper.apache.org/releases.html

这里以最新版的3.6.1为例

下载好后上传到服务器上，这里使用的时centos7 Java环境已经配置好了 为Java8

![image-20200808082641068](D:\MarkDown\BigData\img\image-20200808082641068.png)

将上传的zookeeper压缩包解压到一个目录 ($\color{red}{确保当前用户有操作权限}$)这里放在了/opt/modules里

复制zookeeper目录下的conf目录里面的zoo_sample.cfg为zoo.cfg 并修改里面的

dataDir配置为一个你想要放zk的数据的目录 这里我存放为地址为 dataDir=/opt/modules/apache-zookeeper-3.6.1-bin/zkdata



启动服务端 进入bin目录

![image-20200808083328455](D:\MarkDown\BigData\img\image-20200808083328455.png)

启动客户端

![image-20200808083451634](D:\MarkDown\BigData\img\image-20200808083451634.png)

退出客户端

![image-20200808083608166](D:\MarkDown\BigData\img\image-20200808083608166.png)

停止服务端

![image-20200808083658705](D:\MarkDown\BigData\img\image-20200808083658705.png)



配置文件 zoo.cfg

```properties
# The number of milliseconds of each tick
# 心跳时间 单位毫秒
tickTime=2000
# 集群中follower与leader初始连接时能够忍受的最多心跳数
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement

#集群中Leader与Follower之间的最大响应时间单位，#假如响应超过syncLimit * tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。

syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.

# 数据文件目录+数据持久化路径 不要使用默认的/tmp下的目录

dataDir=/opt/modules/apache-zookeeper-3.6.1-bin/zkdata

# the port at which the clients will connect
# 客户端连接的端口
clientPort=2181

```







#### 选举机制

1）半数机制：集群中半数以上机器存活，集群可用。所以Zookeeper适合安装奇数台服务器。

2）Zookeeper虽然在配置文件中并没有指定Master和Slave。但是，Zookeeper工作时，是有一个节点为Leader，其他则为Follower，Leader是通过内部的选举机制临时产生的。

3）以一个简单的例子来说明整个选举的过程。

假设有五台服务器组成的Zookeeper集群，它们的id从1-5，同时它们都是最新启动的，也就是没有历史数据，在存放数据量这一点上，都是一样的。假设这些服务器依序启动，来看看会发生什么

​                               

![image-20200808080211948](D:\MarkDown\BigData\img\image-20200808080211948.png)

（1）服务器1启动，此时只有它一台服务器启动了，它发出去的报文没有任何响应，所以它的选举状态一直是LOOKING状态。

（2）服务器2启动，它与最开始启动的服务器1进行通信，互相交换自己的选举结果，由于两者都没有历史数据，所以id值较大的服务器2胜出，但是由于没有达到超过半数以上的服务器都同意选举它(这个例子中的半数以上是3)，所以服务器1、2还是继续保持LOOKING状态。

（3）服务器3启动，根据前面的理论分析，服务器3成为服务器1、2、3中的老大，而与上面不同的是，此时有三台服务器选举了它，所以它成为了这次选举的Leader。

（4）服务器4启动，根据前面的分析，理论上服务器4应该是服务器1、2、3、4中最大的，但是由于前面已经有半数以上的服务器选举了服务器3，所以它只能接收当小弟的命了。

（5）服务器5启动，同4一样当小弟。



#### 监听器原理



首先创建一个main()线程

在main线程中创建Zookeeper客户端，这时会创建两个线程一个负责网络通信（connect线程）一个负责监听（Listener线程）

通过connect线程将注册的监听事件发送个Zookeeper

显示Zookeeper的注册监听器列表中将注册的监听事件添加到列表中

Zookeeper监听到有数据或路径变化会将这个消息发送给Listener线程。

listener线程内部调用process方法

#### 常见的监听

监听节点间数据的变化

get path [watch]

监听子节点增减的变化

ls path [watch]



#### 写数据的流程

![image-20200808120022188](D:\MarkDown\BigData\img\image-20200808120022188.png)





#### 命令行操作

| 命令基本语法      | 功能描述                                           |
| ----------------- | -------------------------------------------------- |
| help              | 显示所有操作命令                                   |
| ls path [watch]   | 使用 ls 命令来查看当前znode中所包含的内容          |
| ls2 path  [watch] | 查看当前节点数据并能看到更新次数等数据             |
| create            | 普通创建  -s 含有序列  -e 临时（重启或者超时消失） |
| get path  [watch] | 获得节点的值                                       |
| set               | 设置节点的具体值                                   |
| stat              | 查看节点状态                                       |
| delete            | 删除节点                                           |
| rmr               | 递归删除节点                                       |



#### Zookeeper ACL权限控制

ACL 权限控制，使用：schema: id :permission 来标识，主要涵盖 3 个方面：

1. 权限模式（Schema）：鉴权的策略
2. 授权对象（ID）
3. 权限（Permission）

其特性如下：

1. ZooKeeper的权限控制是基于每个znode节点的，需要对每个节点设置权限
2. 每个znode支持设置多种权限控制方案和多个权限
3. 子节点不会继承父节点的权限，客户端无权访问某节点，但可能可以访问它的子节点



1、schema：

ZooKeeper内置了一些权限控制方案，可以用以下方案为每个节点设置权限：

| 方案   | 描述                                     |
| :----- | :--------------------------------------- |
| world  | 只有一个用户：anyone，代表所有人（默认） |
| ip     | 使用IP地址认证                           |
| auth   | 使用已添加认证的用户认证                 |
| digest | 使用“用户名:密码”方式认证                |

2、id：

授权对象ID是指，权限赋予的用户或者一个实体，例如：IP 地址或者机器。授权模式 schema 与 授权对象 ID 之间关系

![img](D:\MarkDown\BigData\img\20180227181722352)

3、权限permission：

| 权限   | ACL简写 | 描述                             |
| :----- | :------ | :------------------------------- |
| CREATE | c       | 可以创建子节点                   |
| DELETE | d       | 可以删除子节点（仅下一级节点）   |
| READ   | r       | 可以读取节点数据及显示子节点列表 |
| WRITE  | w       | 可以设置节点数据                 |
| ADMIN  | a       | 可以设置节点访问控制列表权限     |

二、权限相关命令：

| 命令    | 使用方式                | 描述         |
| :------ | :---------------------- | :----------- |
| getAcl  | getAcl <path>           | 读取ACL权限  |
| setAcl  | setAcl <path> <acl>     | 设置ACL权限  |
| addauth | addauth <scheme> <auth> | 添加认证用户 |



设置digest模式

![image-20200808173808475](D:\MarkDown\BigData\img\image-20200808173808475.png)

![image-20200808173836160](D:\MarkDown\BigData\img\image-20200808173836160.png)



#### api应用

pom.xml

```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.13.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.6.1</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>RELEASE</version>
        </dependency>
    </dependencies>
```

log4j.properties

```properties
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n

```

创建客户端

```java
 private static final String CONNECT_STRING =
            "hadoop102:2181,hadoop103:2181,hadoop104:2181";
    private static final int sessionTimeout = 2000;
    private ZooKeeper zkClient = null;

    @Before
    public void init() throws IOException {
        // 创建zk客户端
        zkClient = new ZooKeeper(CONNECT_STRING, sessionTimeout, watchedEvent -> {
        });
    }
```

创建子节点

```java
    @Test
    public void creatChildNode() {
        // CreateMode 枚举类 设置节点为 持久节点 还是临时节点
        try {
            String s = zkClient.create("/mmd", "model".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            System.out.println("创建了路径" + s);

        } catch (KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
    }
```

监听子节点并获取数据

```java
 @Test
    public void testData() throws InterruptedException {
        getData();
        while (true){
            Thread.sleep(5000);
        }
    }



    public void getData() {
        try {
            byte[] data = zkClient.getData("/node3", event -> {
                System.out.println(event.getPath() + "---" + event.getType());
                getData();
            }, new Stat());
            System.out.println(new String(data, StandardCharsets.UTF_8));
        } catch (KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
    }
```

设置ACL

```java
    @Test
    public void create() {
        try {

            zkClient.addAuthInfo("digest", "liao:1234".getBytes());
            Id  authId = new Id("auth","liao");
            ArrayList<ACL> acls = new ArrayList<>(Collections.singletonList(new ACL(ZooDefs.Perms.ALL, authId)));
            zkClient.create("/mdd","mdd".getBytes(),acls,CreateMode.PERSISTENT);

        } catch (KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
    }

```

```java
    @Test
    public void testData() throws InterruptedException {
        getData();
        while (true){
            Thread.sleep(5000);
        }
    }



    public void getData() {
        zkClient.addAuthInfo("digest", "liao:1234".getBytes());
        try {
            byte[] data = zkClient.getData("/mdd", event -> {
                System.out.println(event.getPath() + "---" + event.getType());
                getData();
            }, new Stat());
            System.out.println(new String(data, StandardCharsets.UTF_8));
        } catch (KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
    }
```





#### 集群搭建

修改zoo.cfg



```sh
dataDir=/opt/modules/apache-zookeeper-3.6.1-bin/zkdata
```

在/opt/modules/apache-zookeeper-3.6.1-bin/zkdata目录下创建myid文件

在文件中添加与server对应的编号：

```sh
102 # ip最后的一位
```



修该zoo.cfg 添加如下内容

```sh
#######################cluster##########################

server.102=hadoop102:2888:3888 

server.103=hadoop103:2888:3888

server.104=hadoop104:2888:3888

```

将zookeeper分发到其他的集群

分别修改myid文件为对应的数据（每个都不同）

在集群上启动即可。





问题

```sh
[liao@hadoop201 bin]$ ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Error contacting service. It is probably not running.

```

出现这种是需要全部启动即可