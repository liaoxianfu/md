### Flume学习



flume安装

1） Flume官网地址

http://flume.apache.org/

2）文档查看地址

http://flume.apache.org/FlumeUserGuide.html

3）下载地址

http://archive.apache.org/dist/flume/

这里选择的是1.7.0版本

1）将apache-flume-1.7.0-bin.tar.gz上传到linux的/opt/software目录下

2）解压apache-flume-1.7.0-bin.tar.gz到/opt/module/目录下

```sh
tar -zxf apache-flume-1.7.0-bin.tar.gz -C /opt/module/
```

3）修改apache-flume-1.7.0-bin的名称为flume
```sh
mv apache-flume-1.7.0-bin flume
```
4） 将flume/conf下的flume-env.sh.template文件修改为flume-env.sh，并配置flume-env.sh文件
```sh
mv flume-env.sh.template flume-env.sh
vi flume-env.sh
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

Flume组成

![image-20200823093638594](D:\MarkDown\BigData\img\image-20200823093638594.png)