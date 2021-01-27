集群规划

| 机器名    | 机器iP         | 机器角色 |
| --------- | -------------- | -------- |
| Hadoop201 | 192.168.77.201 | HDFS     |
| Hadoop202 | 192.168.77.202 | YARN     |
| Hadoop203 | 192.168.77.203 | 2NN      |



#### 修改主机名(全部机器)

```sh
hostnamectl set-hostname hadoop201 # 192.168.77.201机器
hostnamectl set-hostname hadoop202 # 192.168.77.202机器
hostnamectl set-hostname hadoop203 # 192.168.77.203机器
```

修改完毕后重启机器

![image-20200901182533461](/home/liao/Documents/md/BigData/img\image-20200901182533461.png)

查看到主机名更改成功即可

#### 修改hosts（全部机器）

需要使用==sudo==进行修改

在每一台机器上都执行

```sh
sudo vim /etc/hosts
```

在后面追加

```sh
192.168.77.201 hadoop201
192.168.77.202 hadoop202
192.168.77.203 hadoop203
```

重启之后在所有机器上执行如下命令 机器之间能够ping通即可。



#### 集群免密登陆

在==hadoop201==机器上执行(用户为liao)

```sh
ssh-keygen -t rsa
```

一路回车生成公私钥，然后将公钥拷贝到hadoop201、hadoop202、hadoop203上

```sh
ssh-copy-id hadoop201
ssh-copy-id hadoop202
ssh-copy-id hadoop203
```

在hadoop201机器上执行

```ssh
ssh hadoop201
ssh hadoop202
ssh hadoop203
```

如果都能在==不要用密码下==登录成功即可

再以==root==用户执行以上操作

在hadoop202上执行同样的操作。

```sh
ssh-keygen -t rsa
```

一路回车生成公私钥，然后将公钥拷贝到hadoop201、hadoop202、hadoop203上

```sh
ssh-copy-id hadoop201
ssh-copy-id hadoop202
ssh-copy-id hadoop203
```

在hadoop202机器上执行

```ssh
ssh hadoop201
ssh hadoop202
ssh hadoop203
```

如果都能在==不要用密码下==登录成功即可

#### 安装常用组件(全部机器)

```sh
sudo yum install -y psmisc nc net-tools rsync vim lrzsz ntp libzstd openssl-static
```

安装JDk

我这里使用的是jdk1.8

新建两个目录(全部机器)

```sh
cd /opt
sudo mkdir software module
# 修改权限
sudo chown liao:liao software module
```

将下载好的jdk包上传到/opt/software下(hadoop201机器)

```sh
# 解压jdk
tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/
# 进入解压目录
cd /opt/module/
# 修改名称
mv jdk1.8.0_212/ jdk
```

配置环境变量

```sh
 cd /etc/profile.d/
 sudo vim bigdata_env.sh
```

添加

```sh
# JDK
export JAVA_HOME=/opt/module/jdk
export PATH=$PATH:$JAVA_HOME/bin

```

重启，执行`java -version`

有类似输出即可

![image-20200901190028023](/home/liao/Documents/md/BigData/img\image-20200901190028023.png)

分发

1、分发jdk

```sh
cd /opt/module
scp -r jdk/ liao@hadoop202:$PWD
scp -r jdk/ liao@hadoop203:$PWD
```



2、分发环境变量

以root用户执行

```sh
/etc/profile.d
scp bigdata-env.sh root@hadoop202:$PWD
scp bigdata-env.sh root@hadoop203:$PWD
```

重启



#### 安装Hadoop

上传Hadoop压缩包到/opt/software  这里使用的是Hadoop3.1.3

```sh
cd /opt/software
tar -zxvf hadoop-3.1.3.tar.gz -C /opt/module/
cd /opt/module
mv hadoop-3.1.3/ hadoop

sudo vim /etc/profile.d/bigdata-env.sh
# 追加如下内容
export HADOOP_HOME=/opt/module/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

修改配置文件

```sh
cd $HADOOP_HOME/etc/hadoop

```



core-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop201:8020</value>
    </property>
    <property>
        <name>hadoop.data.dir</name>
        <value>/opt/module/hadoop/data</value>
    </property>
    <property>
        <name>hadoop.proxyuser.liao.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.liao.groups</name>
        <value>*</value>
    </property>
    <!--设置http静态用户 能够使在web界面创建文件夹等-->
     <property>
        <name>hadoop.http.staticuser.user</name>
        <value>liao</value>
    </property>
</configuration>

```

hdfs-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file://${hadoop.data.dir}/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file://${hadoop.data.dir}/data</value>
  </property>
    <property>
    <name>dfs.namenode.checkpoint.dir</name>
   <value>file://${hadoop.data.dir}/namesecondary</value>
  </property>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop203:9868</value>
  </property>
</configuration>

```

yarn-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop202</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>

```

mapred-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
    
    <!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop201:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop201:19888</value>
</property>

</configuration>

```

workers

```xml
hadoop201
hadoop202
hadoop203
```

#### 格式化NameNode

在Hadoop201上执行

```sh
hdfs namenode -format
```

在Hadoop201上启动

```sh
start-dfs.sh
```

在浏览器上访问hadoop201:9870

![image-20200901203239161](/home/liao/Documents/md/BigData/img\image-20200901203239161.png)



==如果不成功 请查看防火墙是否关闭==

![image-20200901203322684](/home/liao/Documents/md/BigData/img\image-20200901203322684.png)

上图就是没有关闭防火墙，在所有机器上都要关闭防火墙

```sh
sudo systemctl stop firewalld.service   #停止firewall
sudo systemctl disable firewalld.service #禁止firewall开机启动
```

#### 启动yarn

在hadoop202上执行`start-yarn.sh`

在浏览器上访问 http://hadoop202:8042/

![image-20200901204024558](/home/liao/Documents/md/BigData/img\image-20200901204024558.png)

#### 集群时间同步设置

 时间服务器配置（必须root用户）

（1）在所有节点关闭ntp服务和自启动

```sh
systemctl stop ntpd

systemctl disable ntpd
```
这里以Hadoop203机器为ntp服务器进行设置

在Hadoop203机器上

授权192.168.77.0-192.168.77.255网段上的所有机器可以从这台机器上查询和同步时间

```sh
vim /etc/ntp.conf
#修改第17行
#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
# 以自己的网段为准
restrict 192.168.77.0 mask 255.255.255.0 nomodify notrap
```

注释21-24行 集群在局域网中，不使用其他互联网上的时间

```sh
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst

```

当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步

```sh
server 127.127.1.0
fudge 127.127.1.0 stratum 10

```

修改/etc/sysconfig/ntpd 文件增加内容如下（让硬件时间与系统时间一起同步）

```sh
SYNC_HWCLOCK=yes
```


```sh
systemctl start ntpd #重新启动ntpd服务
systemctl enable ntpd #设置ntpd服务开机启动
```

Hadoop201 Hadoop202机器同步Hadoop203时间

（必须root用户）

```sh
crontab -e
*/10 * * * * /usr/sbin/ntpdate hadoop203
```

测试 在任意一台（非Hadoop203）机器上执行

```sh
date -s "2020-9-11 11:11:11"
```

十分钟后查看机器是否与时间服务器同步

```sh
date
```



问题

TEZ 内存不够

```sh
Failing this attempt.Diagnostics: [2020-09-09 20:13:20.934]Container [pid=2345,containerID=container_1599653535802_0002_02_000001] is running 419129856B beyond the 'VIRTUAL' memory limit. Current usage: 288.8 MB of 1 GB physical memory used; 2.5 GB of 2.1 GB virtual memory used. Killing container.
```

