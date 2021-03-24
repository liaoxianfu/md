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



#### 2.0、web的工作方式

什么是http协议？

> HTTP是一种无状态、由文本构成的请求-响应(request-response)协议,这种协议使用 的是客户端-服务器( client-server)计算模型 。

我们平时浏览网页的时候,会打开浏览器，输入网址后按下回车键，然后就会显示出你想要浏览的内容。在这个看似简单的用户行为背后，到底隐藏了些什么呢？

对于普通的上网过程，系统其实是这样做的：浏览器本身是一个客户端，当你输入URL的时候，首先浏览器会去请求DNS服务器，通过DNS获取相应的域名对应的IP，然后通过IP地址找到IP对应的服务器后，要求建立TCP连接，等浏览器发送完HTTP Request（请求）包后，服务器接收到请求包之后才开始处理请求包，服务器调用自身服务，返回HTTP Response（响应）包；客户端收到来自服务器的响应后开始渲染这个Response包里的主体（body），等收到全部的内容随后断开与该服务器之间的TCP连接。

Web服务器的工作原理可以简单地归纳为：

- 客户机通过TCP/IP协议建立到服务器的TCP连接
- 客户端向服务器发送HTTP协议请求包，请求服务器里的资源文档
- 服务器向客户机发送HTTP协议应答包，如果请求的资源包含有动态语言的内容，那么服务器会调用动态语言的解释引擎负责处理“动态内容”，并将处理得到的数据返回给客户端
- 客户机与服务器断开。由客户端解释HTML文档，在客户端屏幕上渲染图形结果



URL 与DNS解析

```
scheme://host[:port#]/path/.../[?query-string][#anchor]
scheme         指定低层使用的协议(例如：http, https, ftp)
host           HTTP服务器的IP地址或者域名
port#          HTTP服务器的默认端口是80，这种情况下端口号可以省略。如果使用了别的端口，必须指明，例如 http://www.cnblogs.com:8080/
path           访问资源的路径
query-string   发送给http服务器的数据
anchor         锚
```

HTTP 请求包（浏览器信息）

```
GET /domains/example/ HTTP/1.1      //请求行: 请求方法 请求URI HTTP协议/协议版本
Host：www.iana.org             //服务端的主机名
User-Agent：Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.4 (KHTML, like Gecko) Chrome/22.0.1229.94 Safari/537.4            //浏览器信息
Accept：text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8    //客户端能接收的mine
Accept-Encoding：gzip,deflate,sdch     //是否支持流压缩
Accept-Charset：UTF-8,*;q=0.5      //客户端字符编码集
//空行,用于分割请求头和消息体
//消息体,请求资源参数,例如POST传递的参数
```







#### 2.1、HTTP客户端

Go语言中的`net/http`包提供了最简洁的HTTP客户端实现，我们无需借助第三方网络通信库就可以直接使用HTTP中的GET和POST请求。

#### 基本方法

```go
func (c *Client) Get(url string)(r *Response,err error)
func (c *Client) Post(url string,bodyType string,body io.Reader)(r *Response,err error)
func (c *Client) PostForm(url string,data url.Values)(r *Response,err error)
func (c *Client) Head(url string)(r *Response，err error)
func (c *Client) Do(req *Request)(r *Response, err error)
```

Get方法

```go
func GetDemo() {
	resp, err := http.Get("http://www.baidu.com")
	if err != nil {
		fmt.Println(err.Error())
		return
	}
	defer func() {
		_ = resp.Body.Close()
	}()
	_, _ = io.Copy(os.Stdout, resp.Body)
}
```





Post方法

```go
func PostDemo(){
	imgFile, err := ioutil.ReadFile("a,jpg")
	if err != nil {
		fmt.Println("ioutil.ReadFile(\"a,jpg\") failed ",err)
		return
	}
	reader := bytes.NewReader(imgFile)
	resp, err := http.Post("http://example.com/upload", "image/jpeg", reader)
	if err != nil {
		fmt.Println(" http.Post(...) error ",err)
		return
	}
	if resp.StatusCode != http.StatusOK {
		// 请求失败
		fmt.Println("request error，Response code is ",resp.StatusCode)
		return
	}
	// do something
}
```



