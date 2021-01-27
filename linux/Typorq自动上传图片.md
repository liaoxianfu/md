# Linux下使用Typora与Picgo实现GitHub图床



最新版本的typora实现了利用PicGo自动上传图片的功能，但是基本上都是介绍在Windows下如何使用的，很少介绍如何在Linux下使用。经过摸索实现了typora+picgo+github通过代理实现图床的功能。

## 1、Typora

不同的发行版有不同的安装方法，也可以在官网直接下载二进制文件使用。官网地址https://typora.io/。

## 2、Picgo

官网提供的`picgo-appimage`在我的Archlinux下无法正常运行，无法打开界面。而且看起来不太好与Typora整合，因此选择了node.js下的picgo包。

### 2.1 node环境安装

首先安装nodejs，下载地址https://nodejs.org/en/。安装方法就是解压配置好环境变量即可（具体可自行百度）。接着配置国内的`cnpm`。参考地址 https://developer.aliyun.com/mirror/NPM。

### 2.2 全局安装picgo

执行以下命令

```sh
cnpm install picgo -g #需要配置cnpm
```

安装好后，重新打开一个终端 能够找到`picgo`命令即可。

## 3 配置GitHub

a、创建一个新的仓库

**注意必须初始化**

![](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611491794.png)

b、创建私钥

在你的账户下的设置里选择`Developer settings`->`Personal access tokens`新建一个

<img src="https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611491890.png" style="zoom:50%;" />





<img src="https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611492053.png" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611492159.png" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611492358.png" style="zoom:50%;" />

这样会生成一段token 这个值只显示一次所以必须保存好，下面会用到。



c. 配置picgo

在终端下执行,会出现选项卡，通过键盘的方向建选择Github，回车确定

```sh
picgo set uploader
```

![](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611492765.png)

依次填入信息即可![](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611492879.png)

**（1）repo**

repo就是你`用户名/仓库名` 例如刚才创建了demo_img 我的用户名为`liaoxianfu`那么就是`liaoxianfu/demo_img`

![](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611491794.png) 

（2）branch为main （之前的是master，但是从2021年GitHub改成了main）

（3）token就是之前让你创建token时保存下来的那一串token

（4）path 就是你上传图片在你仓库保存的位置

（5）customUrl 国内是不能访问github的文件服务的，需要进行CDN加速访问，这里使用`https://cdn.jsdelivr.net/gh/用户名/仓库名`配置

配置好后使用github作为图床

```sh
picgo use uploader
```

选择GitHub即可。

在终端下使用上传命令测试以下能不能正常上传，正常上传口会出现PicGo SUCCESS 的标志和返回的访问url

```sh
picgo upload 图片地址
```

![](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611493714.png)

4、整合Typora和Picgo

在Typora的文件选项卡的偏好设置的图像选项中将插入图片时选择上传图片，上传服务选择`Custom Command`命令设置为`picgo upload`即可

![](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611493860.png)

