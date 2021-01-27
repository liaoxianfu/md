

## Hadoop之MapReduce

### 介绍

MapReduce是一个分布式运算的编程框架，是用户开发基于Hadoop的数据分析应用的核心框架。核心功能是将用户编写的业务逻辑代码和自带的组件整合成一个完整的分布式运算程序，并发布在一个Hadoop集群上。

### 优缺点

**优点**

① MapReduce**易于编程**

简单的实现一些接口就可以完成一个分布式程序。可以部署到大量的廉价PC上运行

② **良好的扩展**

当计算资源不能得到满足可以通过简单的增加机器来扩展计算能力。

③ **高容错性**

MapReduce的设计初衷就是能够部署在廉价的机器上，这就要求需要很高的容错性。比如其中一台机器挂了，可以转移它的计算任务到另一个节点上运行，不至于任务失败，整个过程无需手动干预。

④ **适合PB级别的海量数据离线处理**

可以实现上千台服务器集群的并发工作，提供数据处理能力。

**缺点**（慢）

① **不擅长实时计算**

MapReduce无法像Mysql一样在毫秒或者秒级返回结果

②**不擅长流式计算**

流式计算的输入数据是动态，而MapReduce的输入数据是静态的不能动态变化，因为MapReduce自身设计特定决定了数据源必须是静态的。

不擅长DAG有向图计算

多个应用存在在依赖关系，后一个应用的输入时是前一个应用的输出，在这种情况下MapReduce可以实现但是因为计算结果需要落磁盘，因此会造成大量的磁盘IO，导致性能低下。

### 核心思想

![image-20200116180021431](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/image-20200116180021431.png)

1）分布式的运算程序往往需要分成至少2个阶段。

2）第一个阶段的MapTask并发实例，完全并行运行，互不相干。

3）第二个阶段的ReduceTask并发实例互不相干，但是他们的数据依赖于上一个阶段的所有MapTask并发实例的输出。

4）MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑

非常复杂，那就只能多个MapReduce程序，串行运行。

一个完整的MapReduce有三类实例进程

MrAppMaster 负责整个进程过程调度以及状态协调

MapTask 负责Map阶段整个数据处理流程

ReduceTask 负责Reduce阶段整个数据处理流程。

### example

创建maven工程

引入依赖

```xml
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
```

创建mapreduce需要实现mapper与reduce和一个入口的Driver

实现mapper需要继承`org.apache.hadoop.mapreduce.Mapper`并重写mapper方法实现reduce需要实现`org.apache.hadoop.mapreduce.Reducer`并重写reduce方法。

官方案例wordcount

WordCountMapper

```java
package com.liao.mapreduce.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * @author liao
 * @since 2020/1/16 19:55
 */
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    // 设置key 和value
    private Text keyOut = new Text();

    private IntWritable one = new IntWritable(1);


    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 获取一行数据
        String line = value.toString();
        // 分词
        String[] words = line.split(" ");
        // 写出数据
        for (String word : words) {
            keyOut.set(word);
            context.write(keyOut,one);
        }
    }
}

```
WordCountReduce


```java
package com.liao.mapreduce.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author liao
 * @since 2020/1/16 19:56
 */
public class WordCountReduce extends Reducer<Text, IntWritable, Text, IntWritable> {
    // 定义返回结果
    private IntWritable result = new IntWritable();

    // 返回的是key值相同的一组value值
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        // 统计结果
        AtomicInteger sum = new AtomicInteger();
        values.forEach(value -> {
            sum.addAndGet(value.get());
        });
        result.set(sum.intValue());
        // 返回给框架的计算值
        context.write(key, result);
    }
}

```

WordCountDriver

```java
package com.liao.mapreduce.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import java.io.IOException;

/**
 * @author liao
 * @since 2020/1/16 19:56
 */
public class WordCountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        // 配置
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length < 2) {
            System.err.println("Usage: wordcount <in> [<in>...] <out>");
            System.exit(2);
        }
        // 创建一个job
        Job job = Job.getInstance(conf, "word count");
        // 设置jar的运行路径
        job.setJarByClass(WordCountDriver.class);
        // 设置job的mapper类
        job.setMapperClass(WordCountMapper.class);
        // 设置job的reduce
        job.setReducerClass(WordCountReduce.class);
        // 设置map输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        // 设置最终输出的类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // 设置输入和输出类型
        for (int i = 0; i < otherArgs.length - 1; i++) {
            FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
        }
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));

        // 提交job
        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);
    }
}

```

这里运行的输入输入路径需要通过命令行参数指定，输入参数可以为多个 最后一个是输出路径，这里在Windows下使用idea设置参数为

![image-20200117113042038](img\image-20200117113042038.png)

点解如图所示的选项卡 选择`Edit Configurations`编辑Program arguments 输入你的命令行参数即可

![image-20200117113133883](img\image-20200117113133883.png)

注意 **输出的文件夹需要不存在，例如我这里是output文件夹 但是在实际的磁盘中是不能存在该文件夹的，由Hadoop自动生成**

运行成功后，会产生output 结果如下

![image-20200117113600277](img\image-20200117113600277.png)

**打包到集群运行**

![image-20200118094926633](img\image-20200118094926633.png)

点击package进行打包 打成jar包后上传至集群

![image-20200118095713793](img\image-20200118095713793.png)

### 原理

#### InputFormat数据输入

**切片与MapTask并行度决定机制**

在HDFS中的Block是物理上将数据分成不同的块。数据切片知识在逻辑上对输入进行分片，并不会在磁盘上将其切片进行存储。切片的时候对每个文件进行考虑切片。例如有一个300M，一个30M，一个50M的总计380M的文件，会分成128 128 44 30 50 共计5个切片。

提交Job**的执行流程**

在Driver类中

```java
  boolean b = job.waitForCompletion(true);

// Submit the job to the cluster and wait for it to finish.


```

使用`waitForCompletion`方法提交任务，api给出的信息是`Submit the job to the cluster and wait for it to finish.`也就是利用这个方法将任务提交到集群，参数`verbose`是否打印进度信息

```java
  public boolean waitForCompletion(boolean verbose) throws IOException, InterruptedException,ClassNotFoundException {
    if (state == JobState.DEFINE) {
      submit();
    }
 ........ 
 }
```

如果状态是` JobState.DEFINE`就提交任务使用`submit()`方法提交任务方法的描述为

`Submit the job to the cluster and return immediately.`也就是将任务立即提交到集群。

```java
  public void submit() throws IOException, InterruptedException, ClassNotFoundException {
    ensureState(JobState.DEFINE); // 再次确保任务已经定义了
    setUseNewAPI(); // 将旧的Api转换成新的APi  兼容Hadoop1.x版本
    connect(); // 连接
    final JobSubmitter submitter = 
        getJobSubmitter(cluster.getFileSystem(), cluster.getClient()); // 获取任务提交者
    status = ugi.doAs(new PrivilegedExceptionAction<JobStatus>() {
      public JobStatus run() throws IOException, InterruptedException,  
      ClassNotFoundException {
        return submitter.submitJobInternal(Job.this, cluster);
      }// 运行任务 返回状态
    });
    state = JobState.RUNNING;
    LOG.info("The url to track the job: " + getTrackingURL());
   }
```

connect 创建一个新的集群进行工作

```java
  private synchronized void connect() {
    if (cluster == null) {
      cluster = 
        ugi.doAs(new PrivilegedExceptionAction<Cluster>() {
                   public Cluster run()
                          throws IOException, InterruptedException, 
                                 ClassNotFoundException {
                     return new Cluster(getConfiguration());
                   }
                 });
    }
  }
```



**切片规则源码解析**

在Driver类中有

```java
  FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
```

其中

```java
public abstract class InputFormat<K, V> {

   // 将切片进行逻辑切片划分 并不进行物理划分
  public abstract 
    List<InputSplit> getSplits(JobContext context
                               ) throws IOException, InterruptedException;
  
// 创建一个记录者
  public abstract 
    RecordReader<K,V> createRecordReader(InputSplit split,
                                         TaskAttemptContext context
                                        ) throws IOException, 
                                                 InterruptedException;

}
```

FileInputFormat继承了InputFormat这个抽象类

```
public abstract class FileInputFormat<K, V> extends InputFormat<K, V> 
```

继承关系

![image-20200118145454952](img\image-20200118145454952.png)

InputFormat的切片机制是 简单的按照文件内容长度进行切片，切片的大小默认为Block的大小，切片不考虑数据集的整体，而是针对每一个文件进行单独切片。

`FileInputFormat`实现了`InputFormat`抽象类，实现了抽象方法

![image-20200118150846215](img\image-20200118150846215.png)

`FileInputFormat`也是抽象类 实现类为![image-20200118151014659](img\image-20200118151014659.png)

其中`TextInputFormat`是默认的`FileInputFormat`实现类，按行读取每条记录 key是从整个文件中的字节偏移量LongWriteable类型。值是这行的内容，不包括任何终止符，是Text类型。

`KeyValueInputFormat` 也是每一行一条记录，被分隔符分成key，value 可以通过驱动类中设置分隔符的类型 默认的分隔符是tab(\t)

`NLineInputFormat`代表每个map进程中处理的InputSplit不再按照Block进行划分而是按照行数进行划分，也就是按照总文件的行数/N得到切片数目 如果没有整除就是切片数=商+1

`CombineTextInputFormat`用于小文件过多的场景，它可以将多个小文件从逻辑上规划成一个切片文件，这样，多个小文件就可以交给一个MapTask处理。



