#### 1、创建javaee项目

在idea中选择 File->Project->Maven 

![image-20200802181437009](D:\MarkDown\Java\javeee\img\image-20200802181437009.png)

引入pom文件

```xml

 <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
      
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.3</version>
            <scope>provided</scope>
        </dependency>

       <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>


    </dependencies>

```

web.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0"
         metadata-complete="true">

</web-app>
```

创建java/resources目录

![image-20200802182136256](D:\MarkDown\Java\javeee\img\image-20200802182136256.png)

#### 2、HelloWorld

如果要实现一个Servlet就必须继承HttpServlet重写里面的方法。一般为网络的集中请求方式方法Get、Post、put、delete。

![image-20200802182402656](D:\MarkDown\Java\javeee\img\image-20200802182402656.png)



这里以Get请求为例，重写doGet方法。

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 设置ContentType 以网页格式显示 字符串编码为uft8 中文不会乱码
        response.setContentType("text/html; charset=UTF-8");
        PrintWriter writer = response.getWriter();
        writer.write("你好，世界");
    }
}

```



配置web.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0"
         metadata-complete="true">
    <servlet>
        <!--配置Servlet名称-->
        <servlet-name>helloServlet</servlet-name>
        <!--Servlet对应的实现类-->
        <servlet-class>com.liao.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <!--对应的名称-->
        <servlet-name>helloServlet</servlet-name>
        <!--浏览器请求的路径-->
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

</web-app>
```

配置Tomcat

![image-20200802183257006](D:\MarkDown\Java\javeee\img\image-20200802183257006.png)

![image-20200802183438095](D:\MarkDown\Java\javeee\img\image-20200802183438095.png)

配置好之后就可以运行了

访问http://localhost:8080/servlet_demo_war/hello 



#### 3、tomcat乱码问题

找到tomcat安装的位置，在conf文件夹下，找到`logging.properties`

```properties
# java.util.logging.ConsoleHandler.encoding = UTF-8 修改为GBK编码

java.util.logging.ConsoleHandler.encoding = GBK
```



#### 4、java配置实现映射

如果你不是很喜欢xml配置，而特别喜欢注释， Servlets API也有这样的功能。您可以使用@WebServlet ，如下的例子，那么你不需要在web.xml中做任何事情。在运行时会自动将`Servlet`注册到容器，并像往常一样处理它。

==注意 Java配置与xml只能选择一个 如果同时存在只能解析出xml的路径映射==

```java

@WebServlet(name = "helloServlet", urlPatterns = {"/hello"})
public class HelloServlet extends HttpServlet {...}
```



#### 5、Servlet生命周期

Servlet 生命周期可被定义为从创建直到毁灭的整个过程。以下是 Servlet 遵循的过程：

- Servlet 通过调用 **init ()** 方法进行初始化。
- Servlet 调用 **service()** 方法来处理客户端的请求。
- Servlet 通过调用 **destroy()** 方法终止（结束）。
- 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

###### init() 方法

init 方法被设计成只调用一次。它在第一次创建 Servlet 时被调用，在后续每次用户请求时不再调用。

Servlet 创建于用户第一次调用对应于该 Servlet 的 URL 时，但是您也可以指定 Servlet 在服务器第一次启动时被加载。

当用户调用一个 Servlet 时，就会创建一个 Servlet 实例，每一个用户请求都会产生一个新的线程，适当的时候移交给 doGet 或 doPost 方法。init() 方法简单地创建或加载一些数据，这些数据将被用于 Servlet 的整个生命周期。

调用的顺序时 在第一次访问时调用` init(ServletConfig config) `初始化方法，该方法会调用` init(ServletConfig() ` 调用方法见 `GenericServlet`

###### service() 方法

service() 方法是执行实际任务的主要方法。Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。

每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。

==service() 方法由容器调用==，service 方法在适当的时候调用 doGet、doPost、doPut、doDelete 等方法。所以，您不用对 service() 方法做任何动作，您只需要根据来自客户端的请求类型来重写 doGet() 或 doPost() 即可。

###### destroy() 方法

destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用。destroy() 方法可以让您的 Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动。在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。




#### 6、HttpServletResponse

用于返回给客户端服务器的数据

* 设置返回数据的编码和Content Type
* 向页面返回数据

```java
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        Integer viewNumber = (Integer) this.getServletContext().getAttribute("viewNumber");
        // 设置ContentType
        response.setContentType("text/html; charset=UTF-8");
        // 向页面写数据
        PrintWriter writer = response.getWriter();

        writer.write("访问人数为：");
        writer.write(viewNumber.toString());
    }
```



#### filter过滤器

Servlet 过滤器可以动态地拦截请求和响应，以变换或使用包含在请求或响应中的信息。

可以将一个或多个 Servlet 过滤器附加到一个 Servlet 或一组 Servlet。Servlet 过滤器也可以附加到 JavaServer Pages (JSP) 文件和 HTML 页面。调用 Servlet 前调用所有附加的 Servlet 过滤器。

Servlet 过滤器是可用于 Servlet 编程的 Java 类，可以实现以下目的：

- 在客户端的请求访问后端资源之前，拦截这些请求。
- 在服务器的响应发送回客户端之前，处理这些响应。

根据规范建议的各种类型的过滤器：

- 身份验证过滤器（Authentication Filters）。
- 数据压缩过滤器（Data compression Filters）。
- 加密过滤器（Encryption Filters）。
- 触发资源访问事件过滤器。
- 图像转换过滤器（Image Conversion Filters）。
- 日志记录和审核过滤器（Logging and Auditing Filters）。
- MIME-TYPE 链过滤器（MIME-TYPE Chain Filters）。
- 标记化过滤器（Tokenizing Filters）。
- XSL/T 过滤器（XSL/T Filters），转换 XML 内容。

使用Filter

通过实现Filter接口

| 1    |                             方法                             |
| ---- | :----------------------------------------------------------: |
| 1    | **public void init(FilterConfig filterConfig)** web 应用程序启动时，web 服务器将创建Filter 的实例对象，并调用其init方法，读取web.xml配置，完成对象的初始化功能，从而为后续的用户请求作好拦截的准备工作（filter对象只会创建一次，init方法也只会执行一次）。开发人员通过init方法的参数，可获得代表当前filter配置信息的FilterConfig对象。 |
| 2    | public void doFilter (ServletRequest, ServletResponse, FilterChain) 该方法完成实际的过滤操作，当客户端请求方法与过滤器设置匹配的URL时，Servlet容器将先调用过滤器的doFilter方法。FilterChain用户访问后续过滤器。 |
| 3    | **public void destroy()** Servlet容器在销毁过滤器实例前调用该方法，在该方法中释放Servlet过滤器占用的资源。 |



实现文件上传下载功能

配置文件指定文件存放位置

```properties
filePath=D:/file # 文件存放位置
tmpPath=D:/tmp # 缓存存放位位置
```

增加pom文件

```xml
 <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.7</version>
        </dependency>
```

添加配置文件类

```java
public class ServletConf {
    public static String filePath = "";
    public static String tmpPath = "";
    public static final String confName = "/config.properties";
    public static final String file_pro = "filePath";
    public static final String tmp_pro = "tmpPath";

}
```



创建Filter 在初始化的是否检查配置文件

```java
@WebFilter(urlPatterns = {"/*"})
public class HelloFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // 初始化获取数据
        Properties properties = new Properties();

        InputStream inputStream = this.getClass().getResourceAsStream(ServletConf.confName);
        try {
            properties.load(inputStream);
            ServletConf.filePath = properties.getProperty(ServletConf.file_pro);
            checkDir(ServletConf.filePath);
            ServletConf.tmpPath = properties.getProperty(ServletConf.tmp_pro);
            checkDir(ServletConf.tmpPath);
        } catch (IOException e) {
            // 抛出异常 停止运行
            throw new RuntimeException(e);
        }

        System.out.println("filter init.....");
    }


    private void checkDir(String path) {
        File file = new File(path);
        if (!file.exists()) {
            file.mkdirs();
        }
    }


    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        response.setContentType("text/html; charset=UTF-8");
        chain.doFilter(request, response);

    }

    @Override
    public void destroy() {
        System.out.println("filter destroy....");
    }
}
```

上传

```java
@WebServlet(name = "uploadServlet",value = "/upload")
public class UploadServlet extends HttpServlet {

    // 上传配置
    private static final int MEMORY_THRESHOLD = 1024 * 1024 * 3;
    // 3MB
    private static final int MAX_FILE_SIZE = 1024 * 1024 * 40;
    // 40MB
    private static final int MAX_REQUEST_SIZE = 1024 * 1024 * 50;
    // 50MB

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        if (!ServletFileUpload.isMultipartContent(req)) {
            resp.getWriter().write("Error: 表单必须包含 enctype=multipart/form-data");
            resp.flushBuffer();
            return;
        }
        DiskFileItemFactory factory = new DiskFileItemFactory();
        factory.setSizeThreshold(MEMORY_THRESHOLD);
        factory.setRepository(new File(ServletConf.tmpPath));
        ServletFileUpload upload = new ServletFileUpload(factory);
        upload.setSizeMax(MAX_FILE_SIZE);
        upload.setSizeMax(MAX_REQUEST_SIZE);
        upload.setHeaderEncoding("utf-8");

        try {
            // 解析请求的内容提取文件数据
            List<FileItem> formItems = upload.parseRequest(req);
            System.out.println(formItems);
            if (formItems != null && formItems.size() > 0) {
                // 迭代表单数据
                for (FileItem item : formItems) {
                    // 处理不在表单中的字段
                    if (!item.isFormField()) {
                        String fileName = new File(item.getName()).getName();
                        String filePath = ServletConf.filePath + "/" + fileName;
                        File storeFile = new File(filePath);
                        // 在控制台输出文件的上传路径
                        System.out.println(filePath);
                        // 保存文件到硬盘
                        item.write(storeFile);
                        req.setAttribute("message",
                                "文件上传成功!");
                        req.setAttribute("fileName",fileName);
                    }
                }
            }
        } catch (Exception ex) {
            req.setAttribute("message",
                    "错误信息: " + ex.getMessage());
        }
        // 跳转到 message.jsp
        req.getServletContext().getRequestDispatcher("/message.jsp").forward(
                req, resp);

    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.sendRedirect("index.jsp");
    }
}
```

下载

```java
@WebServlet(value = {"/down"})
public class DownloadServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String fileName = req.getParameter("file");
        System.out.println(fileName);
        String s = ServletConf.filePath + "/" + fileName;
        InputStream inputStream = new FileInputStream(s);
        resp.setHeader("Content-Disposition", "attachment; filename=" + fileName);
        ServletOutputStream outputStream = resp.getOutputStream();
        byte[] bs = new byte[1024];
        int len = -1;
        while ((len = inputStream.read(bs)) != -1) {
            outputStream.write(bs, 0, len);
        }
        outputStream.close();
        inputStream.close();
    }


}
```

index.jsp

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<html>
<body>

<form action="${pageContext.request.contextPath}/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="uploadFile">
    <input type="submit" value="上传文件">
</form>

</body>
</html>

```

message.jsp

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h2>${message}</h2>
<a href="${pageContext.request.contextPath}/down?file=${fileName}">下载</a>
</body>
</html>

```



