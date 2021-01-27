## Hadoop之HDFS

### 介绍

HDFS是一个文件系统，用于存储文件，通过目录树来定位文件，HDFS的使用场景，适合一次写入多次读取的场景，不支持文件的修改。适合用来做数据分析，不适合和作为网盘使用。

#### 优点

高容错性，适合大数据处理，可以构建在廉价的机器上。

#### 缺点

不适合低延迟的数据访问，比如毫秒级别的存储数据，无法高效的对小文件进行存储，不支持并发的写入，文件随机修改（一个文件只能有一个写操作，仅支持数据追加操作不支持随机修改操作）。

### 组成

HDSF由NN（NameNode）和DN（DataNode）组成，

NN作为管理者也就是master，管理着 HDFS的名称空间，配置副本的策略，数据块的映射信息，客户端的读写请求。

DN作为salve 由NNx下发命令，DN执行操作。具有 存储实际的数据块和执行数据块的读写操作。

Client 负责文件的切分，在上传的时候按照block的大小分解成不同的块进行上传。与NameNode交互获取文件的位置信息，与DataNode交互进行文件的读写。通过命令实现HDFS的格式化，文件的增删查改。

HDFS在1.x的时候默认的block大小是64MB 2.x的时候默认是128.

![image-20200115131828090](img\image-20200115131828090.png)

### 常见的命令行操作



![image-20200115132154602](img\image-20200115132154602.png)

![image-20200115132237814](img\image-20200115132237814.png)



### java api 操作

创建maven工程

pom.xml

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>big-data-learning</artifactId>
        <groupId>com.liao</groupId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>hdfs-learning</artifactId>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.8.2</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.2</version>
        </dependency>

    </dependencies>

</project>
```



```java
package com.liao.hdfs;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.junit.Test;
import org.mortbay.util.IO;

import java.io.*;
import java.net.URI;
import java.net.URISyntaxException;

/**
 * @author liao
 * @since 2020/1/14 20:30
 */
public class HDFSTest {

    @Test
    public void testMkDirs() throws URISyntaxException, IOException, InterruptedException {

        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "root");
        boolean exists = fileSystem.exists(new Path("/mr/test1"));
        System.out.println(exists);
        boolean mkdirs = fileSystem.mkdirs(new Path("/user/liao"));
        System.out.println(mkdirs);
        fileSystem.close();
    }

    @Test
    public void testRmDirs() throws Exception {
        Configuration configuration = new Configuration();
        URI uri = new URI("hdfs://hadoop102:9000");
        String user = "root";
        FileSystem fileSystem = FileSystem.get(uri, configuration, user);
        boolean b = fileSystem.deleteOnExit(new Path("/user/liao"));
        System.out.println(b);
        fileSystem.close();
    }

    // E:\Java\big-data-learning\hdfs-learning\src\main\resources\log4j.properties


    /**
     * 上传文件
     * @throws Exception
     */

    @Test
    public void testUploadFile() throws Exception{
        Configuration configuration = new Configuration();
        URI uri = new URI("hdfs://hadoop102:9000");
        String user = "root";
        FileSystem fileSystem = FileSystem.get(uri, configuration, user);
        String srcPath = "E:\\Java\\big-data-learning\\hdfs-learning\\src\\main\\resources\\log4j.properties";
        fileSystem.copyFromLocalFile(new Path(srcPath),new Path("/user"));
        fileSystem.close();
    }

    @Test
    public void testDownloadFile() throws Exception{
        Configuration configuration = new Configuration();
        URI uri = new URI("hdfs://hadoop102:9000");
        String user = "root";
        FileSystem fileSystem = FileSystem.get(uri, configuration, user);

        String srcPath = "E:\\Java\\big-data-learning\\hdfs-learning\\src\\main\\resources\\test\\log4j.properties";
        fileSystem.copyToLocalFile(new Path("/user/log4j.properties"),new Path(srcPath));
        fileSystem.close();
    }

    /**
     * 流式拷贝上传
     * @throws Exception
     */

    @Test
    public void testUploadFileWithStream() throws Exception{

        Configuration configuration = new Configuration();
        URI uri = new URI("hdfs://hadoop102:9000");
        String user = "root";
        FileSystem fileSystem = FileSystem.get(uri, configuration, user);
        String srcPath = "E:\\Java\\big-data-learning\\hdfs-learning\\src\\main\\resources\\test\\log4j.properties";
        // 创建文件的输入流
        InputStream inputStream = new FileInputStream(srcPath);

        // 创建文件的输出流
        FSDataOutputStream outputStream = fileSystem.create(new Path("/user/log4j.properties"));
        // 拷贝数据
        IOUtils.copyBytes(inputStream,outputStream,configuration);

        IOUtils.closeStream(inputStream);
        IOUtils.closeStream(outputStream);
        fileSystem.close();
    }

    /**
     * 流式拷贝下载
     */

    @Test
    public void testDownloadFileWithStream() throws Exception{
        Configuration configuration = new Configuration();
        URI uri = new URI("hdfs://hadoop102:9000");
        String user = "root";
        FileSystem fileSystem = FileSystem.get(uri, configuration, user);
        String srcPath = "E:\\Java\\big-data-learning\\hdfs-learning\\src\\main\\resources\\test\\log4j2.properties";

        // 创建文件输入流 来自hdfs
        FSDataInputStream inputStream = fileSystem.open(new Path("/user/log4j.properties"));
        // 创建输出流
        OutputStream outputStream = new FileOutputStream(srcPath);

        IOUtils.copyBytes(inputStream,outputStream,1024);

        IOUtils.closeStream(inputStream);
        IOUtils.closeStream(outputStream);
        fileSystem.close();
    }


}

```







