### dubbo-admin监控台配置



1、安装Zookeeper

[安装参考]: https://blog.csdn.net/liaoxianfu/article/details/107889925	"安装参考"



2、下载dubbo-admin

地址: https://github.com/apache/dubbo-admin.git

3、安装

由于官方提供的安装方法总是报错，这里提供提供手动打包前后端项目的方法进行安装。

（1）、克隆项目

这里选择的是develop项目 master项目已经很久没有更新了，默认的分支也是devlop 采用前后端分离进行开发。

```sh
https://github.com/apache/dubbo-admin.git
```

或者直接下载zip压缩包

进入下载好的文件夹下的dubbo-admin-ui文件夹下，执行

```sh
npm install
# 建议使用cnpm
cnpm install
```

下载好依赖之后执行

```sh
npm run build
```

会产生target文件夹



![2](D:\MarkDown\Java\dubbo\img\2.png)

将里面的所有内容进行复制

![image-20200810161242252](D:\MarkDown\Java\dubbo\img\image-20200810161242252.png)





再进入dubbo-admin-server文件夹下进入src/main/resources目录下创建static目录

将上面复制的内容粘贴进static文件夹下

打开cmd命令行进入dubbo-admin-server文件夹

执行

```sh
mvn package -Dmaven.test.skip=true
```

![image-20200810161739852](D:\MarkDown\Java\dubbo\img\image-20200810161739852.png)

打包没有报错就证明打包完成

在dubbo-admin-develop\dubbo-admin-server\target下找到jar并将其拷贝到一个空白的文件夹中

==由于默认的端口为8080可能呗占用，因此需要动态配置==

将dubbo-admin-develop\dubbo-admin-server\src\main\resources下的application.properties拷贝到jar报所在的文件夹下，额可以根据自己的实际情况进行修改

运行jar报即可

![image-20200810162249807](D:\MarkDown\Java\dubbo\img\image-20200810162249807.png)

==默认的用户名密码是root==  可以在配置文件中进行修改





