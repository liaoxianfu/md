# hive学习

## hive常见属性配置

Default数据仓库的最原始位置是在hdfs上的/usr/hive/warehouse路径下

在仓库目录下，没有对默认的数据库default创建文件夹。如果某张表属于default数据库，就会在数据仓库目录下创建一个文件夹。

![image-20200129140211465](img\image-20200129140211465.png)

修改default数据仓库原始位置（将hive-default.xml.template如下配置信息拷贝到hive-site.xml文件中）。

```xml
 <property>  
     <name>hive.metastore.warehouse.dir</name> 
     <value>/user/hive/warehouse</value>  			        <description>location of default database for  the warehouse</description>  
</propert>  
```

配置同组用户有执行权限

bin/hdfs dfs -chmod g+w /user/hive/warehouse

## 配置查询后信息显示配置

在hive-site.xml文件中添加如下配置信息，就可以实现显示当前数据库，以及查询表的头信息配置。

```xml
<property>
	<name>hive.cli.print.header</name>
	<value>true</value>
</property>

<property>
	<name>hive.cli.print.current.db</name>
	<value>true</value>
</property>

```

显示效果如下：

![image-20200129140903864](D:\MarkDown\BigData\img\image-20200129140903864.png)



日志配置：

hive中的日志默认放在/tmp/${USER}/hive.log下。如果需要进行修改，例如这里修改放在

`/${HIVE_HOME}/logs`目录下,需要修改配置文件 在{HIVE_HOME}/conf目录下，将`hive-log4j.properties.template`修改成为`hive-log4j.properties` 并在文件中修改日志存放的位置`/${HIVE_HOME}/logs`.

### hive参数配置方式

1、查看当前所有配置信息

```sql
hive> set;
```

2、参数配置的三种方式

（1）、默认配置文件的方式

默认配置文件：hive-default.xml

用户自定义配置文件: hive-site.xml

注意：用户自定义配置文件会覆盖默认的配置文件。另外hive也会读取Hadoop的配置文件，配置文件的设定会对本机所有hive进程有效。

（2）命令行参数方式

启动hive时可以在命令行添加-hiveconf param=vale 来设定参数，只对本次hive有效

（3）参数声明方式

在hive启动后使用set 关键字进行设定。

优先级 配置文件<命令行<参数声明。但是注意某些系统级的参数例如log4j相关的设定，必须使用前两种进行设定，因为那些参数在会话建立之前就已经完成了。

## hive的数据类型

基本数据类型 略

集合数据类型 struct map array

## hive DDL

### 创建数据库

```sql
create database db_name;
```

![image-20200129143332043](D:\MarkDown\BigData\img\image-20200129143332043.png)

![image-20200129143421956](D:\MarkDown\BigData\img\image-20200129143421956.png)

hive创建数据库默认是在hdfs的/user/hive/warehouse创建一个db_name.db的文件夹。

![image-20200129143831306](D:\MarkDown\BigData\img\image-20200129143831306.png)

使用if  not exists 判断 是否存在要创建的数据库。

### 查询数据库

与mysql类似 不再赘述。

查看数据库详情

![image-20200129144214149](D:\MarkDown\BigData\img\image-20200129144214149.png)

```sql
-- 切换数据库
use db_name;
-- 删除空数据库 可以使用if not exist 判断数据库是否存在
drop database db_name;

-- 如果数据库不存在可以使用cascade强制删除
drop database db_name cascade;

```

### 创建表 

语法

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
[(col_name data_type [COMMENT col_comment], ...)] 
[COMMENT table_comment] 
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
[CLUSTERED BY (col_name, col_name, ...) 
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
[ROW FORMAT row_format] 
[STORED AS file_format] 
[LOCATION hdfs_path]
```

解释 create table 创建一个指定名字的。如果相同名字的表已经存在，就会抛出异常；用户可以使用 if not exists 选项忽略这个异常

external 关键字可以让用户创建一个外部表，在建表的同事指定一个指向实际数据的路径。Hive创建内部表的时候回将数据换移动到数据仓库指定的路径，若创建外部表仅记录数据所在的路径，不会对数据位置做任何改变。在删除表的时候内部数据表会将表格数据一同删除，外部表只会删除表不会删除数据。

comment：为表和列添加注释。

pratitionned bt 创建分区表

STORED AS指定存储文件类型

常用的存储文件类型：SEQUENCEFILE（二进制序列文件）、TEXTFILE（文本）、RCFILE（列式存储格式文件）

如果文件数据是纯文本，可以使用STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCEFILE。

（9）LOCATION ：指定表在HDFS上的存储位置。

（10）LIKE允许用户复制现有的表结构，但是不复制数据。

### 管理表

1．理论

默认创建的表都是所谓的管理表，有时也被称为内部表。因为这种表，Hive会（或多或少地）控制着数据的生命周期。Hive默认情况下会将这些表的数据存储在由配置项hive.metastore.warehouse.dir(例如，/user/hive/warehouse)所定义的目录的子目录下。 当我们删除一个管理表时，Hive也会删除这个表中数据。管理表不适合和其他工具共享数据。·

**创建普通表**

```sql
create table if not exists student2(
id int, name string
)
-- 通过\t区分数据字段
row format delimited fields terminated by '\t'
-- 作为文本存储
stored as textfile
-- 存储位置
location '/user/hive/warehouse/student2';

```

根据查询结果创建表（查询结果会存放在新的表中）

```sql
create table if not exists student3 as select id, name from student;
```

![image-20200129151425038](D:\MarkDown\BigData\img\image-20200129151425038.png)

![image-20200129151526513](D:\MarkDown\BigData\img\image-20200129151526513.png)

**根据已经存在的表创建结构表**（不会移动数据）

```sql
create table if not exists student4 like student; 
```

**查询表的类型**

```sql
desc formatted student2;
```

### 外部表

1．理论

因为表是外部表，所以Hive并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。

2．管理表和外部表的使用场景

每天将收集到的网站日志定期流入HDFS文本文件。在外部表（原始日志表）的基础上做大量的统计分析，用到的中间表、结果表使用内部表存储，数据通过SELECT+INSERT进入内部表。

```sql
create external table if not exists dept(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';
```

```sql
create external table if not exists emp(
empno int,
ename string,
job string,
mgr int,
hiredate string, 
sal double, 
comm double,
deptno int)
row format delimited fields terminated by '\t';
```

导入数据

```sql
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept;
hive (default)> load data local inpath '/opt/module/datas/emp.txt' into table default.emp;

```

### 管理表与外部表的相互转化

```sql
--（1）查询表的类型
hive (default)> desc formatted student2;
Table Type:             MANAGED_TABLE
--（2）修改内部表student2为外部表
alter table student2 set tblproperties('EXTERNAL'='TRUE');
--（3）查询表的类型
hive (default)> desc formatted student2;
Table Type:             EXTERNAL_TABLE
--（4）修改外部表student2为内部表
alter table student2 set tblproperties('EXTERNAL'='FALSE');
--（5）查询表的类型
hive (default)> desc formatted student2;
Table Type:             MANAGED_TABLE

```

注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写！

### 分区表

分区表实际上就是对应一个HDFS文件系统上的独立文件夹，该文件夹下是该分区的所有数据文件。HIve的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据集。在查询时通过WHERE子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。

创建分区表sql

```sql
create table dept_partition(
deptno int,
dname string,
loc string
)
partitioned by (month string)
row format delimited fields terminated by '\t';
```

加载数据到分区表

```sqsl
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201709');
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201708');
hive (default)> load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201707’);
```

![image-20200129164204337](D:\MarkDown\BigData\img\image-20200129164204337.png)



![image-20200129164232089](D:\MarkDown\BigData\img\image-20200129164232089.png)

### 查询分区表的数据

#### 查询单区数据

![image-20200129164437780](D:\MarkDown\BigData\img\image-20200129164437780.png)

#### 多分区联合查询

![image-20200129164908170](D:\MarkDown\BigData\img\image-20200129164908170.png)



#### 增加分区

添加单个分区

```sql
alter table dept_partition add partition(month=202004)
```

添加多个分区

```sql
alter table dept_partition add partition(month=202005)
partition(month=202006);
```

![image-20200129165318858](D:\MarkDown\BigData\img\image-20200129165318858.png)

#### 删除分区

删除单个分区

```sql
alter table dept_partition drop partition(month=202004)
```

删除多个分区

```sql
alter table dept_partition drop partition(month=202005),
partition(month=202006);
```

#### 创建二级分区

```sql
create table dept_partition2(
               deptno int, dname string, loc string
               )
               partitioned by (month string, day string)
               row format delimited fields terminated by '\t';
```

![image-20200129170929497](D:\MarkDown\BigData\img\image-20200129170929497.png)

![image-20200129171016887](D:\MarkDown\BigData\img\image-20200129171016887.png)



## 数据导入

### 向表中装载数据

```sql
load data [local] inpath 'filepath' [overwrite] into table student [partition (partcoll=value1,value2...)]
```

（1）load data:表示加载数据

（2）local:表示从本地加载数据到hive表；否则从HDFS加载数据到hive表

（3）inpath:表示加载数据的路径

（4）overwrite:表示覆盖表中已有数据，否则表示追加

（5）into table:表示加载到哪张表

（6）student:表示具体的表

（7）partition:表示上传到指定分区



实战：

从HDFS长传文件

在本地创建文件 stu

```txt
1	demo
2	test
```

上传到HDFS

```sh
hadoop fs -copyFrommLocal stu /
```

从HDFS加载数据

```sql
create table student(
    id int,
    name string
)
row format delimited  fields terminated by '\t';

show tables ;
desc student;
load data inpath '/stu' into table student;
select * from student;
```

```sql
-- 在hive中使用dfs上传数据
-- 创建student5表
create table student5(
    id int,
    name string
)
row format delimited fields terminated by '\t'
location '/user/hive/warehouse/demo.db/student5';

show tables ;
-- 上传本地数据到指定文件表的文件夹
dfs -put /root/stu /user/hive/warehouse/demo.db/student5;

select * from student5;

```



## 表连接操作

### 内连接

只有进行连接的两个表中都存在与连接条件相匹配的数据才能保留下来

### 左外连接

JOIN操作左边表中符合了WHERE子句的所有记录都会被返回

### 右外连接

右外连接：JOIN操作符右边表中符合WHERE子句的所有记录将会被返回。

### 满外连接

​     满外连接：将会返回所有表中符合WHERE语句条件的所有记录。如果任一表的指定字段没有符合条件的值的话，那么就使用NULL值替代。



### 排序

全局排序

使用order进行排序，可以进行多列表排序











