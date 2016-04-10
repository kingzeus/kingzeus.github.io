---
title: hexo 的 cdn 加速
date: 2016-03-31 15:21:08
updated: 2016-04-10 20:40:00
tags:
- hexo
- cdn
- 博客
categories:
- hexo
- 博客
---


- 最后更新: 2016-03-31
- 修改记录:
	* 2014-03-31 初稿
	* 2014-04-10 增加效果对比

最近用 hexo 搞了个博客，非常好用。不过，作为不折腾不舒服斯基，还是有很多可以瞎折腾的。

## 为啥要加速
1. 目前的博客是依托于 github 的 pages 的，所以，页面访问速度不甚理想，图片的访问更甚。
2. 另外，使用了 Next 的主题，里面使用了谷歌的字体，基于某种神秘的力量，天朝无法正常访问。
3. 博客页面使用了很多第三方的开源库，这些 css 或者 js 位于世界的各个角落，所以访问速度也是各不相同。

总的一句话，页面呈现的速度太慢，需要优化。

## 啥是 CDN

	内容分发网络（Content delivery network或Content distribution network，缩写：CDN）是指一种通过互联网互相连接的电脑网络系统，利用最靠近每位用户的服务器，更快、更可靠地将音乐、图片、视频、应用程序及其他文件发送给用户，来提供高性能、可扩展性及低成本的网络内容传递给用户。
						--来自 不存在的百科网站

![](/images/14594892390920.jpg)

左边的是单服务器模式，右边的就是 CDN 模式

## 实现
废话不多说，直接上方法。

### 主题的 CDN 静态文件加速
主要使用 cdn 来加速主题中使用的前端库。目前我使用的是 next 主题，就拿这个主题动手，其他的主题也是类似，可以参照修改。


#### 国内常用的 CDN 服务
![](/images/14594917399605.jpg)

##### 1. 百度CDN公共库
百度公共CDN为站长的应用程序提供稳定、可靠、高速的服务，包含全球所有最流行的开源JavaScript库。

* [官网：http://cdn.code.baidu.com/](http://cdn.code.baidu.com/)

##### 2. 360网站卫士
提供了由360网站卫士CDN驱动的常用前端公共库以及和谐使用Google公共库&字体库的调用方法

* [官网：http://libs.useso.com/](http://libs.useso.com/)

##### 3. BootCDN
稳定、快速、免费的开源项目 CDN 服务

* [官网：http://www.bootcdn.cn](http://www.bootcdn.cn)

##### 4. 又拍云

* [官网：http://jscdn.upai.com/](http://jscdn.upai.com/)

##### 5. 七牛

* [官网: http://jscdn.upai.com/](http://jscdn.upai.com/)

##### 6. 新浪云
新浪云计算是新浪研发中心下属的部门，主要负责新浪在云计算领域的战略规划，技术研发和平台运营工作。主要产品包括 应用云平台Sina App Engine（简称SAE）。

SAE的CDN节点覆盖全国各大城市的多路（电信、联通、移动、教育）骨干网络，使开发者能够方便的使用高质量的CDN服务。

* [官网: http://lib.sinaapp.com/](http://lib.sinaapp.com/)


#### 增加主题配置
在主题的配置文件 `_config.yml` 中增加 CDN 配置:

``` yaml
# cdn
cdn:
  enable: true
# bootcdn support: font-awesome,fancybox
  server: bootcdn
# 360cdn support:font_lato,fancybox,font-awesome(only 4.2.0)
# server: 360cdn
```

其中：

* ==cdn.enable== 控制是否启用 CDN
* ==cdn.server== 定义了使用的 CDN 服务商

#### 文件修改
修改 `head.swig`，目录是：![](/images/14602867734715.jpg)

根据上面的参数，设置下对应的地址就可以了。修改完的代码：
```html
{% if theme.fancybox %}
  {% if theme.cdn.enable %}
    {% if theme.cdn.server == "360cdn" %}
      <link href="http://libs.useso.com/js/fancybox/2.1.5/jquery.fancybox.css" rel="stylesheet" type="text/css"/>
    {% elif theme.cdn.server == "bootcdn" %}
      <link href="//cdn.bootcss.com/fancybox/2.1.5/jquery.fancybox.css" rel="stylesheet" type="text/css"/>
    {% endif %}  
  {% else %}  
    <link href="{{ url_for(theme.vendors) }}/fancybox/source/jquery.fancybox.css?v=2.1.5" rel="stylesheet" type="text/css"/>
  {% endif %}
{% endif %}
```

其他的代码也类似。



### 博客内容的 CDN 加速
主要是使用 cdn 来加速博客文章中使用的图片

#### 使用七牛云存储来加速图片

* 登录七牛网站，上传图片
* 复制图片的外链，替换对应图片



## 效果对比
使用 cdn 之前
![](/images/14602916261355.jpg)

使用 CDN 之后
![](/images/14602916431704.jpg)

* bg.jpg 加载时间从 4.17 秒减到了 264.7 毫秒 ( 文件越大,效果越明显)

## 后记
使用了 CDN 可以明显改善访问速度，但是博客页面本身访问还是存在一定问题的，github 网站也有可能 短时间或者永久的 “不存在”，接下来，会说说如何增加国内的镜像。


