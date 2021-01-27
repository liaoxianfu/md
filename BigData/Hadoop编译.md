



### 虚拟机centos7.8  hadoop3.2.1 源码编译

大家好！ 我将带大家编译Hadoop3.2.1，本次Linux环境为Centos7.8 在网上找到的大部分是基于centos6.x 和Hadoop2.7.x的版本 在最新的Hadoop3.x中有些是不适用的。

#### 1、环境要求

这是Hadoop编译要求的环境

```xml

* Unix System
* JDK 1.8 
* Maven 3.3 or later
* ProtocolBuffer 2.5.0
* CMake 3.1 or newer (if compiling native code)
* Zlib devel (if compiling native code)
* Cyrus SASL devel (if compiling native code)
* One of the compilers that support thread_local storage: GCC 4.8.1 or later, Visual Studio,
  Clang (community version), Clang (version for iOS 9 and later) (if compiling native code)
* openssl devel (if compiling native hadoop-pipes and to get the best HDFS encryption performance)
* Linux FUSE (Filesystem in Userspace) version 2.6 or above (if compiling fuse_dfs)
* Doxygen ( if compiling libhdfspp and generating the documents )
* Internet connection for first build (to fetch all Maven and Hadoop dependencies)
* python (for releasedocs)
* bats (for shell code testing)
* Node.js / bower / Ember-cli (for YARN UI v2 building)

```

* Linux 环境  Centos 7

  清华源地址：https://mirrors.tuna.tsinghua.edu.cn/centos/7.8.2003/isos/x86_64/

  我选择的是Minimal 

  直达下载地址： https://mirrors.tuna.tsinghua.edu.cn/centos/7.8.2003/isos/x86_64/CentOS-7-x86_64-Minimal-2003.iso

* Hadoop3.2.1 版本

  下载地址：https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.2.1/hadoop-3.2.1-src.tar.gz

* JDK 1.8  华为镜像地址 https://repo.huaweicloud.com/java/jdk/

  我选择的是https://repo.huaweicloud.com/java/jdk/8u172-b11/jdk-8u172-linux-x64.tar.gz

* Maven 3.3 or later 

  官网下载地址：https://maven.apache.org/download.cgi

  直达下载地址：https://mirror.bit.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz

* ==ProtocolBuffer 2.5.0 必须是这个 不能高也不能低 而且需要自己安装 不是解压后就能用==

  下载地址：https://github.com/protocolbuffers/protobuf/releases/tag/v2.5.0

  直达下载地址： https://github.com/protocolbuffers/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz

* Zlib devel (if compiling native code)  

  直接yum安装 `yum install zlib-devel` 

* ==CMake 3.1 or newer (if compiling native code) 需要手动安装 因为centso7的仍然是2.8.5 不能使用==

  官网下载地址：https://cmake.org/download/

  我选择的是3.17.5

  直达下载地址：https://github.com/Kitware/CMake/releases/download/v3.17.5/cmake-3.17.5-Linux-x86_64.sh

* Cyrus SASL devel (if compiling native code)  

  直接yum安装： ` yum install cyrus-sasl cyrus-sasl-devel`

* GCC 4.8.1以上

  直接yum安装   `yum install glibc-headers gcc gcc-c++`

* openssl devel  

  直接yum安装  `yum install openssl-devel`

* Linux FUSE (Filesystem in Userspace) version 2.6 or above 

  直接yum安装 `yum install fuse`

  其他

* yum install ncurses-devel

* yum install doxygen

* yum install fuse

  阿里云maven源

  ```xml
   <mirror>
                  <id>nexus-aliyun</id>
                  <mirrorOf>central</mirrorOf>
                  <name>Nexus aliyun</name>
                  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
          </mirror>
  
  ```

  

  

  ## 2、安装步骤

2.1 虚拟机环境准备

虚拟安装 选择自定义 centos7 内存和CPU根据你的电脑进行选择 但是内存最好不要小于4G 我会将这个步骤整理成图文放在我的博客上

https://www.democpp.cn



等待出现安装界面。

将Kdump关闭 打开网络连接

点击安装位置 选择自动分区即可 设置一下root密码 如果你的这台虚拟机不用来做其他的就不需要创建用户了。

等待安装完成。。



centos7.8的最小化安装没有ifconfig 需要安装net-tools

`yum install net-tools`

使用ifconfig查询虚拟机的ip地址 使用xshell进行连接 接下来的操作就不需要在使用虚拟机直接进行交互了。

安装rz 可以将文件直接拖进xshell进行上传 比较方便

`yum install lrzsz -y`

将事先准备好的软件上传到虚拟机中



#### 安装java环境

解压到/opt/module 

`tar -zxvf jdk-8u172-linux-x64.tar.gz -C /opt/module/`

安装 vim

`yum isntall vim -y`

建立文件

vim /etc/profile.d/hadoop_env.sh

写入

```sh
export JAVA_HOME=/opt/module/jdk8
export PATH=$PATH:$JAVA_HOME/bin

```



![image-20201112171641315](D:\MarkDown\BigData\img\image-20201112171641315.png)



安装maven

![image-20201112172012777](D:\MarkDown\BigData\img\image-20201112172012777.png)

![image-20201112172137341](D:\MarkDown\BigData\img\image-20201112172137341.png)

更换阿里云源

vim /opt/module/maven/conf/settings.xml 

```xml
 <mirror>
                <id>nexus-aliyun</id>
                <mirrorOf>central</mirrorOf>
                <name>Nexus aliyun</name>
                <url>http://maven.aliyun.com/nexus/content/groups/public</url>
        </mirror>
```

将第一部分的带yum的都安装好



安装Cmake

1、授予cmake执行权限

2、运行后会解压 将文件移动到/opt/module路径下

3、修改文件夹名 配置环境变量



ProtocolBuffer 2.5.0 

1、解压文件 到/opt/module下

2、进入到ProtocolBuffer 2.5.0 文件夹下

执行./configure

执行 make & make install

等待编译成功即可



配置环境变量

```sh
export LD_LIBRARY_PATH=/opt/module/protobuf-2.5.0
export PATH=$PATH:$LD_LIBRARY_PATH
```

环境准备完毕！

打包

```sh
mvn package -Pdist,native -DskipTests -Dtar
```

等待安装完成即可



如果再Hadoop-commons包下出现错误

请查看protoc是否配置成功

```
protoc --version
# 输出为libprotoc 2.5.0

```

如果没有成功 

请按以下配置

进入到/opt/module/protobuf-2.5.0目录下

```sh
./configure
make 
make install
ldconfig
```

再 vim /etc/profile.d/hadoop_env.sh

添加

```sh
#LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/opt/module/protobuf-2.5.0
export PATH=$PATH:$LD_LIBRARY_PATH

```

执行 source /etc/profile.d/hadoop_env.sh



出现BUILD SUCCESS 证明编译成功

编译成功的tar包在/opt/module/hadoop-3.2.1-src/hadoop-dist/target 下

将这个包拷贝出来就可以使用了



