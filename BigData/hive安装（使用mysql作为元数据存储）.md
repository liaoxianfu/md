# hive安装（使用mysql作为元数据存储）

## 前提

hive操作需要Hadoop的支持，这里默认已经安装好了Hadoop。

## 下载安装文件

hive  http://hive.apache.org/downloads.html

我这里安装Hadoop版本是2.7.7 属于2.x.y 

![image-20200127164707972](img\image-20200127164707972.png)

所以根据官网的介绍，我下载的版本是2.3.6

下载mysql的Java驱动

https://mvnrepository.com/artifact/mysql/mysql-connector-java/5.1.26

![image-20200127165010963](img\image-20200127165010963.png)

上传到任意一台部署有Hadoop机器上。

## 规划

**软件存储的路径在/opt/software 安装的路径在/opt/module，使用的Linux版本是Ubuntu18.04LTS版本。**

将下载的hive和mysql驱动包上传到服务器上。

![image-20200127165444322](D:\MarkDown\BigData\img\image-20200127165444322.png)

解压文件到/opt/module文件夹下

```sh
tar -C /opt/module -zxvf apache-hive-2.3.6-bin.tar.gz
```

![image-20200127165804033](D:\MarkDown\BigData\img\image-20200127165804033.png)

为了方便，重命名文件夹

```sh
mv apache-hive-2.3.6-bin/ hive
```

```sh
export HADOOP_HOME=/opt/module/hadoop-2.7.7
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${HIVE_HOME}/bin
```

执行source /etc/profiles 使的配置文件生效

将mysql驱动包放到hive的lib文件夹下

```sh
 cp mysql-connector-java-5.1.26.jar /opt/module/hive/lib/
```

配置配置文件

在hive/conf下创建hive-site.xml

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
	  <name>javax.jdo.option.ConnectionURL</name>
	  <value>jdbc:mysql://hadoop102:3306/metastore?createDatabaseIfNotExist=true</value>
	  <description>JDBC connect string for a JDBC metastore</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionDriverName</name>
	  <value>com.mysql.jdbc.Driver</value>
	  <description>Driver class name for a JDBC metastore</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionUserName</name>
	  <value>username</value>
	  <description>username to use against metastore database</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionPassword</name>
	  <value>password</value>
	  <description>password to use against metastore database</description>
	</property>
</configuration>

```

启动hdfs和yarn

```sh
start-hdfs.sh
start-yarn.sh
```

初始化元数据库

```sh
$HIVE_HOME/bin/schematool -dbType mysql -initSchema
```

![image-20200127173924397](img\image-20200127173924397.png)

出现上图所示的信息，证明初始化成功。

输入hive启动交互命令行

![image-20200127174112959](img\image-20200127174112959.png)

执行show databases;

显示如上图，证明成功。

## 启动hiveserver2与beeline

注意：如果使用的root作为用户的话，是无法连接的。因此需要对Hadoop的core-site.xml进行修改。修改如下

```xml
<property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>root</value>
        <description>Allow the superuser oozie to impersonate any members of the group group1 and group2</description>
</property>


<property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
        <description>The superuser can connect only from host1 and host2 to impersonate a user</description>
</property>

```



### 出现的问题



**初始化元数据库报错**

1、检查${HIVE_HOME}有没有mysql驱动

2、检查mysql驱动jar包有没有可执行权限

3、检查hive-site.xml有没有配置正确

4、删除metasore_db文件夹。

**beeline无法连接**

修改hadoop配置文件core-site.xml

```xml
<property>
           <name>hadoop.proxyuser.liao.groups</name>
           <value>*</value>
        </property>
        <property>
          <name>hadoop.proxyuser.liao.hosts</name>
          <value>*</value>
 </property>

```

liao设置为你用户名即可。

**解决日志jar冲突**

删除${HIVE}/lib下的log4j jar包即可。



与hbbase整合的时候 Phoenix jar冲突

```java
Exception in thread "main" java.lang.NoSuchMethodError: com.ibm.icu.impl.ICUBinary.getRequiredData(Ljava/lang/String;)Ljava/nio/ByteBuffer;
    at com.ibm.icu.charset.UConverterAlias.haveAliasData(UConverterAlias.java:131)
    at com.ibm.icu.charset.UConverterAlias.getCanonicalName(UConverterAlias.java:525)
    at com.ibm.icu.charset.CharsetProviderICU.getICUCanonicalName(CharsetProviderICU.java:126)
    at com.ibm.icu.charset.CharsetProviderICU.charsetForName(CharsetProviderICU.java:62)
    at java.nio.charset.Charset$2.run(Charset.java:412)
    at java.nio.charset.Charset$2.run(Charset.java:407)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.nio.charset.Charset.lookupViaProviders(Charset.java:406)
```



https://www.jianshu.com/p/1501e59420e4

或者在Hadoop初始化的时候进行排除

```sh
for f in $HBASE_HOME/lib/*; do
# 排除slf4j jar和phoenix jar
  if [[ $f != $HBASE_HOME/lib/slf4j*  &&  $f != $HBASE_HOME/lib/phoenix* ]];then
      export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$f
  fi
done

```



**tez 引擎BUG**

在整合tez后使用hive与hbase关联表进行count查询 插入数据会出现bug



整合hbase

```xml

        <!--整合hbase-->
        <property>
          <name>hive.zookeeper.quorum</name>
          <value>hadoop102,hadoop103,hadoop104</value>
          <description>The list of ZooKeeper servers to talk to. This is only needed for read/write locks.</description>
       </property>
       <property>
          <name>hive.zookeeper.client.port</name>
          <value>2181</value>
          <description>The port of ZooKeeper servers to talk to. This is only needed for read/write locks.</description>
       </property>
        <!--设置允许yarn获取的用户名为设置的而不是获取的-->
       <property>
          <name>hive.server2.enable.doAs</name>
          <value>false</value>
       </property>

```

设置hive执行引擎

```sh
set hive.execution.engine=mr;

```

debug启动

```sh
hive --hiveconf hive.root.logger=DEBUG,console 




Inconsistent constant pool data in classfile for class org/apache/hadoop/hbase/client/Row. Method 'int lambda$static$28(org.apache.hadoop.hbase.client.Row, org.apache.hadoop.hbase.client.Row)' at index 57 is CONSTANT_MethodRef and should be CONSTANT_InterfaceMethodRef.
```

