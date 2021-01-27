## rabbitmq入门

### 安装

使用docker-compose安装

docker-compose.yml

```yml
version: '3.1'
services:
  rabbitmq:
    restart: always
    image: rabbitmq:3.8.2-management
    container_name: rabbit-mq
    ports:
      - 5672:5672
      - 15672:15672
    volumes:
      - ./data:/var/lib/rabbitmq

```

访问IP:15672 用户名和密码均为`guest`访问成功出现页面

![image-20200204154910533](D:\MarkDown\Java\java基础知识\img\image-20200204154910533.png)

### 基本模式

![image-20200204155009937](D:\MarkDown\Java\java基础知识\img\image-20200204155009937.png)

这里三种模式分别为`direct`,`topic`,`fanout`

#### direct模式（点对点模式）

direct模式为点对点模式，也就是exchange发送数据时的routing key与queue的routing key必须严格相等。

添加一个`direct`

![image-20200204160343025](D:\MarkDown\Java\java基础知识\img\image-20200204160343025.png)

创建queue

![image-20200204160453505](D:\MarkDown\Java\java基础知识\img\image-20200204160453505.png)

同理创建多个queue

![image-20200204160706259](D:\MarkDown\Java\java基础知识\img\image-20200204160706259.png) 

测试direct模式

点击exchange创建的antguigu.direct 的exchange

![image-20200204161006230](D:\MarkDown\Java\java基础知识\img\image-20200204161006230.png)

绑定queue

![image-20200204160941886](D:\MarkDown\Java\java基础知识\img\image-20200204160941886.png)

同理绑定其他的queue

![image-20200204161216667](D:\MarkDown\Java\java基础知识\img\image-20200204161216667.png)

测试数据

![image-20200204161322989](D:\MarkDown\Java\java基础知识\img\image-20200204161322989.png)

到queue中查询

![image-20200204161354905](D:\MarkDown\Java\java基础知识\img\image-20200204161354905.png)

发现atguigu队列中有数据 其他队列没有

![image-20200204161452112](D:\MarkDown\Java\java基础知识\img\image-20200204161452112.png)

获取数据成功。

#### fanout模式（广播模式）

创建一个广播模式的exchange

![image-20200204161735124](D:\MarkDown\Java\java基础知识\img\image-20200204161735124.png)

同理绑定queue，因为是广播模式可以不用绑定routing key 绑定也无影响。

![image-20200204161839544](D:\MarkDown\Java\java基础知识\img\image-20200204161839544.png)

同理绑定其他的

![image-20200204162017639](D:\MarkDown\Java\java基础知识\img\image-20200204162017639.png)

测试发送数据

![image-20200204162057849](D:\MarkDown\Java\java基础知识\img\image-20200204162057849.png)

到queue中查看，发现所有的queue都接到了数据

![image-20200204162153116](D:\MarkDown\Java\java基础知识\img\image-20200204162153116.png)

![image-20200204162323683](D:\MarkDown\Java\java基础知识\img\image-20200204162323683.png)

#### topic模式（指定规则）

创建topic的exchange

![image-20200204162453544](D:\MarkDown\Java\java基础知识\img\image-20200204162453544.png)

指定规则绑定

![image-20200204162552711](D:\MarkDown\Java\java基础知识\img\image-20200204162552711.png)

atguigu.# 表示匹配为`atguigu.`开头的routing key ,我们将atguigu开头的queue都绑定为该routing key

![image-20200204162813632](D:\MarkDown\Java\java基础知识\img\image-20200204162813632.png)

![image-20200204162841054](D:\MarkDown\Java\java基础知识\img\image-20200204162841054.png)

![image-20200204162935078](D:\MarkDown\Java\java基础知识\img\image-20200204162935078.png)

可以发现atguigu开通的都获取到数据了。

![image-20200204163036004](D:\MarkDown\Java\java基础知识\img\image-20200204163036004.png)

也可以指定以什么结尾的 利用指定为以.news结尾的`*.news`

