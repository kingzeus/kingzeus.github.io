---
title: 【无所不能的脚本】mac下的ramdisk
date: 2014-01-19 11:26:38
updated: 2016-03-30 11:26:38
tags:
- osx
- mac
- ramdisk
- 脚本
categories:
- osx
---


- 最后更新: 2016-03-30
- 修改记录:
	* 2014-01-19 初稿，发布于 51cto
	* 2015-01-21 增加大小保护，发布于 51cto
	* 2016-03-30 合并之前的文章
	
## 0. 前言

现在流行苹果，mbp，mba，iphone，ipad，……关键是windows上各种流氓软件越来越多，加上工作需要，于是转投了 mac。好吧，入手的 mbp 看起来很美，实际上各种卡，整天看到小菊花转啊转。好吧，屌丝买不起高配，用的是最低端的，说起来，满满的都是泪，但是生活还得继续，只能自己想办法了。

优化第一步，加内存。不要问我为什么能加，老机器了，不说了，满满的都是泪。有钱的孩纸买机器的时候就加内存吧，普罗大众还是不要自己折腾吧。当然大神们可以自己动手换内存，这个也不说了。

第二步，上 SSD 吧。这个真心不错，但是好东西不便宜啊，能把ssd和hdd一样随便放东西的同学，略过这篇文章吧。我的mbp是用来开发的，乱七八糟的东西占了好大的空间，于是ssd就得不停的整理空间，删除缓存文件，就为了那几个 G 的容量。最近，还碰到过因为 ssd 剩余空间是0，无法开机的悲剧。

神圣的红领巾一直在前面引领着我们，生命不息，折腾不止……重点来了，我们可以利用内存来提速啊，这个在windows上已经烂大街了，不就是ramdisk么。拿出谷哥度娘，一番OOXX之后，发现没啥好用，倒是发现mac本身就能实现。身为屌丝IT男，必须自己折腾了。当然，折腾这个的前提是你的内存足够大。

@#！@￥#￥@%#￥%（一星期过去了……………………）

## 1. 实现
### 1. ramdisk
不说折腾的过程了，直接上结果吧。
先来说下大致的工作流程：

1. 开机的时候，自动调用脚本创建内存盘，然后载入需要的数据。
2. 关机的时候，自动调用脚本把内存盘数据回写硬盘，然后执行关机。

不复杂哈，一说就明白了吧，下面来点复杂的：

1. 先来说，怎么再开机的时候运行脚本。
	先在你顺眼的地方建个目录，用来作为ramdisk的脚本和备份数据的工作目录。![](images/14593149822393.jpg)
	
	我的目录，在/etc/下面建了个ramdisk的目录，小伙伴照着做的话，注意下权限。

2.	接下来，就创建一个 login.sh 的脚本，用来再开机的时候自动运行。

	```
	#!/bin/sh
	 
	# 设置内存盘的名称
	DISK_NAME=RamDisk
	MOUNT_PATH=/Volumes/$DISK_NAME
	# 设置备份文件的保存路径, 注意修改下面路径中的用户名称
	BAK_PATH=/etc/ramdisk/$DISK_NAME.tar.gz
	# 设置分配给内存盘的空间大小(MB)
	DISK_SPACE=1024
	 
	if [ ! -e $MOUNT_PATH ]; then
		diskutil erasevolume HFS+ $DISK_NAME `hdiutil attach -nomount ram://$(($DISK_SPACE*1024*2))`
	fi
	#  隐藏分区
	chflags hidden $MOUNT_PATH
	
	if [ -s $BAK_PATH ]; then
	    tar -zxf $BAK_PATH -C $MOUNT_PATH
	fi
	```
		
	* ==DISK_NAME== 用来指定内存盘的名字，随便改
	* ==DISK_SPACE== 用来指定内存盘大小，这是上限值，一般情况下使用多少占多少的内存，所以放心使用吧
	* ==BAK_PATH== 用来指定内存盘的保存路径
3.	再创建一个logout.sh，用来在关机的时候调用
	
	```
	#!/bin/sh
	 
	DISK_NAME=RamDisk
	MOUNT_PATH=/Volumes/$DISK_NAME
	WORK_PATH=/etc/ramdisk
	BAK_PATH=$WORK_PATH/$DISK_NAME.tar.gz
	LISTFILE=$WORK_PATH/list
	
	
	if [ -e $MOUNT_PATH ] ; then
		cd $MOUNT_PATH
		tar --exclude-from $LISTFILE -czf $BAK_PATH .
	fi
	```
	
	* ==DISK_NAME== 用来指定内存盘的名字，要和之前的脚本一致
	* ==BAK_PATH== 用来指定内存盘的保存路径，一样要和上面的一致
	* ==LISTFILE== 忽略的文件列表

	
### 1.2 运行
ramdisk搞定，然后呢？当然是怎么使用了。

先在终端里面把 `login.sh`跑起来啊，跑起来。权限有问题的，可以用sudo方式。然后呢，去finder里面看看吧！

什么都没有？为什么？好吧，其实再脚本里面我把内存盘隐藏了，为啥要加这么帅的操作呢？里面有没有日本的爱情动画片。因为很多小伙伴都手贱，把ramdisk卸载了之后，就会各种悲剧了。所以，事先隐藏了之后，麻麻就再也不用担心了。

那怎么访问呢？有个神奇的办法，在finder的菜单里面找到前往，选择前往文件夹
![](images/14593152549913.jpg)
 然后，输入内存盘的加载路径，也就是脚本里面的`MOUNT_PATH`，接下来就是见证奇迹的时候了……
 ![](images/14593153601092.jpg)


### 1.3 使用
有了内存盘，里面放什么呢？爱情动作片就算了，可以把cache放在里面，这样可以加速运行速度，也可以把经常需要频繁操作的目录放进去，比如chrome的配置啊什么的……

#### 1.3.1 cache 目录
先来处理cache：

* 先把系统的cache目录移动到内存盘上去
* 然后通过符号链接把内存盘上的目录映射回原来的目录
![](/images/14593153742813.jpg)
对应了
![](/images/14593153824224.jpg)

不会的，给个脚本

```shell
#!/bin/sh

# 设置内存盘的名称
DISK_NAME=RamDisk
MOUNT_PATH=/Volumes/$DISK_NAME

#Caches =》 ~/Library/Caches
mv ~/Library/Caches $MOUNT_PATH
ln -s /$MOUNT_PATH/Caches ~/Library/Caches
```

#### 1.3.2 chrome 目录
同样处理下
![](/images/14593154100049.jpg)
对应的是
![](/images/14593154195716.jpg)

#### 1.3.3 其他目录
照着上面的方法,可以把你想要搬的目录全部移动到 ramdisk 上去

再打开内存盘就能看到目录了![](/images/14593154301396.jpg)

### 1.4 配置自动运行
脚本有了，关键是要能随着系统启动、关闭，自动运行起来。

在终端里面运行

```shell
	defaults write com.apple.loginwindow LoginHook /etc/ramdisk/login.sh
	defaults write com.apple.loginwindow LogoutHook /etc/ramdisk/logout.sh
```

可以通过

```shell
	defaults read com.apple.loginwindow
```

来检查设置是否成功。

最后，重启机器看看。
## 2. 脚本高级配置
有了ramdisk以后，mac是不是不一样了？没感觉，好吧，ramdisk不是救世主，不能减少pm2.5的，只能提高部分文件的io效率，要想那啥的，换机器+ssd才是正道。

用了一阵子之后，发现ramdisk越来越大了没有？赶紧看下，果然啊，cache目录占了好多容量。

很大有没有，用软件清理么？不符合懒人哲学啊。作为有追求的屌丝it男，必须用脚本。

￥%￥#……%……&%……（再次省略无数的探索过程……）


```shell
#!/bin/sh
 
DISK_NAME=RamDisk
MOUNT_PATH=/Volumes/$DISK_NAME
WORK_PATH=/etc/ramdisk
BAK_PATH=$WORK_PATH/$DISK_NAME.tar.gz
LISTFILE=$WORK_PATH/list
#设置最大的cache大小(M)
MAX_CACHE_SIZE=50

# 
cd $MOUNT_PATH


declare -a fa
i=0
# Caches 目录存在
if [ -d "$myPath"]; then 
	for file in $(du -s Caches/* | sort -n)
	do
	  fa[$i]=$file
	  let i=i+1
	done
fi


size=$((i/2))
echo "Caches file number:"$size

cd $WORK_PATH
echo ".?*">$LISTFILE
for((i=0;i<$size;i++))
do
  if ((${fa[$((i*2))]}<(($MAX_CACHE_SIZE*1024*2)) ));then
  	echo "add:"${fa[$((i*2+1))]}
  else
  	echo ${fa[$((i*2+1))]}>>$LISTFILE
  fi
  
done


if [ -e $MOUNT_PATH ] ; then
	cd $MOUNT_PATH
	tar --exclude-from $LISTFILE -czf $BAK_PATH .
fi
```

简单说明下，就是每次关机会把`cache`目录下超过50M的目录直接清掉。

* ==MAX_CACHE_SIZE== 可以指定最大的cache目录大小

重启机器，再来看下

于是世界终于和平了…………








