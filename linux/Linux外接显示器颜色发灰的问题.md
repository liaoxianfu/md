### Linux使用HDMI外接显示器颜色发灰

主要原因是HDMI线的色彩空间为16-255。在ArchWiki上解释

> It may occur that the [Intel](https://wiki.archlinux.org/index.php/Intel) driver will not configure correctly the output of the HDMI monitor. It will set a limited color range (16-235) using the [Broadcast RGB property](https://patchwork.kernel.org/patch/1972181/), and the black will not look black, it will be grey.

解决方法为

> ```
> xrandr --output HDMI名称 --set "Broadcast RGB" "Full"
> ```

HDMI的名称可以通过

> xrandr

获取对应的外接显示器的名称

![image-20210330190827558](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/image-20210330190827558.png)

![image-20210330190856633](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/image-20210330190856633.png)

可以看显示了`eDP-1-1`这是笔记本的显示器，`HDMI-1-1`就是外接显示器的名称，所以执行的命令为

> xrandr --output HDMI-1-1 --set "Broadcast RGB" "Full"

但是这种每次关机后就会失效，所以需要设置开启自启动。具体的可以根据自己linux发行版进行设置。



