## 网络编程

本文将介绍使用Go语言来开发网络程序。Go语言标准库里提供的net包，支持基于IP层、TCP/UDP层以及应用层（HTTP，FTP等）的网络操作。其中用于操作IP层的称为Raw Socket

### 1、Socket编程

在Go语言编程中编写网络，我们将看不到传统的编码形式、以前使用Socket进行编程时会进行如下步骤的操作

创建Socket-> 绑定Socket->监听-> 接受连接-> 接收或者发送。

Go语言标准库对此进行了抽象和封装。无论我们期望使用什么协议建立什么形式的连接都只需要调用net.Dial()

#### 1.1、dial() 函数

函数原型为

```qi
func Dial(net,addr string)(Conn,error)
```

其中net参数为网络协议的名字，addr参数为IP地址或者域名，端口号以`:`分割 例如addr = `127.0.0.1:8080`端口为可选。如果连接成功返回连接对象否者返回error。

常见的连接方式

```go
// TCP
conn,err := net.Dial("tcp","127.0.0.1:2000")
// UDP
conn,err := net.Dial("udp","127.0.0.1:8080")
// ICMP(使用协议名称)
conn,err := net.Dial("ip4:icmp","127.0.0.1:8080")
// ICMP（使用协议编号）
conn,err := net.Dial("ip4:1","127.0.0.1:8080")
```

>百度百科
>
>ICMP（Internet Control Message Protocol）Internet控制[报文](https://baike.baidu.com/item/报文/3164352)协议。它是[TCP/IP协议簇](https://baike.baidu.com/item/TCP%2FIP协议簇)的一个子协议，用于在IP[主机](https://baike.baidu.com/item/主机/455151)、[路由](https://baike.baidu.com/item/路由)器之间传递控制消息。控制消息是指[网络通](https://baike.baidu.com/item/网络通)不通、[主机](https://baike.baidu.com/item/主机/455151)是否可达、[路由](https://baike.baidu.com/item/路由/363497)是否可用等网络本身的消息。这些控制消息虽然并不传输用户数据，但是对于用户数据的传递起着重要的作用。



TODO

>  TCP和UDP还没有弄懂 





### 2、HTTP编程



Go语言标准库中提供了net/http包用于实现HTTP客户端和服务端编程。



#### 2.1、HTTP客户端

Go语言中的`net/http`包提供了最简洁的HTTP客户端实现，我们无需借助第三方网络通信库就可以直接使用HTTP中的GET和POST请求。







