### ArchLinux微信、TIM安装与乱码解决方法

#### 1、微信安装

这里使用的是星火商店对基于deepin-wine5微信的打包，网上的很多教程都是基于之前的deepin-wine的深度官方的微信打包，虽然操作都差不多但是还是有些许的差距。

这里通过AUR进行安装，首先请确保自己的电脑上安装了`yay`或者其他的工具并且确保开启了multilib源。最简单的方法就是通过`sudo pacman -Syy`来查看。如果显示下面的信息就证明开启了。

![image-20210124135900489](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/image-20210124135900489.png)

如果没有开启，请编辑`/etc/pacman.conf`文件取消multilib前面的注释，打开该源。否者无法安装微信、qq等软件。

![image-20210124140152521](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/image-20210124140152521.png)



之后进行安装

```sh
yay -S com.qq.weixin.spark # 安装微信
yay -S com.qq.tim.spark # 安装tim
```





#### 2、高分屏下的DPI调整

我的电脑是联想小新pro13 是一个2k 13.3的高分辨屏幕，打开tim和微信时非常小。因此需要调整dpi。

```sh
WINEPREFIX=~/.deepinwine/Spark-WeChat deepin-wine5 winecfg # 调整微信
```

之后会弹出一个wine配置界面，配置图形选项即可

<img src="https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/1611488457.png" style="zoom:80%;" />

TIM配置为

```sh
WINEPREFIX=~/.deepinwine/Spark-TIM deepin-wine5 winecfg # 调整微信
```

同理配置即可。

#### 3、解决微信不能显示汉字的问题

参考地址  https://github.com/wszqkzqk/deepin-wine-ubuntu/issues/136

修改方法 

**先关闭微信** 所有的操作都要在软件未启动的时候进行。

在任意的装有windows电脑下拷贝C:/Windows/Fonts/msyh.ttc

```sh
#1.添加字体
cp msyh.ttc ~/.deepinwine/Deepin-WeChat/drive_c/windows/Fonts

#2.修改系统注册表
vim ~/.deepinwine/Deepin-WeChat/system.reg
#修改以下两行
"MS Shell Dlg"="msyh"
"MS Shell Dlg 2"="msyh"

#3.字体注册
vim msyh_config.reg
#内容添加
REGEDIT4
[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\FontLink\SystemLink]
"Lucida Sans Unicode"="msyh.ttc"
"Microsoft Sans Serif"="msyh.ttc"
"MS Sans Serif"="msyh.ttc"
"Tahoma"="msyh.ttc"
"Tahoma Bold"="msyhbd.ttc"
"msyh"="msyh.ttc"
"Arial"="msyh.ttc"
"Arial Black"="msyh.ttc"
#注册
WINEPREFIX=~/.deepinwine/Deepin-WeChat deepin-wine regedit msyh_config.reg
```

启动微信就正常了。