---
title: Hexo网站优化之搜索引擎优化
date: 2017-04-19 15:21:08
updated: 2016-04-20 20:40:00
tags:
- hexo
- seo
- 博客
categories:
- hexo
- 博客
---

- 最后更新: 2016-04-19
- 修改记录:
	* 2014-04-19 初稿




## 0. 前言

SEO (Search Engine Optimization)，即搜索引擎优化。对网站做SEO优化，有利于提高搜索引擎的收录速度及网页排名。下面讲解一些简单的SEO优化方法，主要针对Hexo网站。

* * *

## 1. url 永久化

hexo 默认的永久链接（Permalinks）规则是 `网站名称／年／月／日／文章名称`。

这种链接格式因为url深度超过3层，对搜索引擎不太友好。另外，如果文章名称使用中文的话，会因为编码问题，造成url失效。

### 1.1 最简单的方法

修改 `_config.yml` 中的 `permalink`

```yaml
#permalink: :year/:month/:day/:title/
# 改为下面这样
permalink: :year-:month-:day-:title.html
```


参数 | 结果
----|----
:year/:month/:day/:title/	| 2013/07/14/hello-world
:year-:month-:day-:title.html | 2013-07-14-hello-world.html
:category/:title | foo/bar/hello-world

### 1.2 hexo-abbrlink

* 安装 hexo-abbrlink

```
npm install hexo-abbrlink --save
```

* 配置_config.yml

```
# permalink: :title/  将之前的注释掉
permalink: archives/:abbrlink.html
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: hex    # 进制：dec(default) and hex
```


## 1. SEO优化之title

编辑站点目录下的`themes/{模板}/layout/index.swig`文件，

将下面的代码

```
{% block title %} {{ config.title }} {% endblock %}
```  

改成
    
```
{% block title %} {{ config.title }} - {{ config.subtitle }} -  {{theme.keywords}}    {% endblock %}
```

这时将网站的描述及关键词加入了网站的`title`中，更有利于详细地描述网站。 

## 2. 添加sitemap 

Sitemap即网站地图，它的作用在于便于搜索引擎更加智能地抓取网站。最简单和常见的sitemap形式，是XML文件，在其中列出网站中的网址以及关于每个网址的其他元数据（上次更新时间、更新的频率及相对其他网址重要程度等）。 

**Step 1**: 安装sitemap生成插件 
    
    npm install hexo-generator-sitemap --save
    npm install hexo-generator-baidu-sitemap --save
    

**Step 2**: 编辑站点目录下的_config.yml，添加 
    

```
# auto sitemap
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml

```
    
**Step 3**: 在robots.txt文件中添加 
    

```
Sitemap: http://kingzeus.github.io/sitemap.xml
Sitemap: http://kingzeus.github.io/baidusitemap.xml
```
    

## 3. 百度站长工具
### 3.1 站点验证
* 登录 [百度站长平台](http://zhanzhang.baidu.com)，在站点管理 里面新建站点，验证类型选择 html 标签。
* 在站点目录下的`_config.yml`中增加:

```
baidu_site_verification: XXXX 
```

### 3.2 sitemap 提交
回到 [百度站长平台](http://zhanzhang.baidu.com) 的链接提交页面，在自动提交 -> sitemap 对应的文本编辑框中输入 <你的域名地址>/sitemap.xml，点击提交按钮后即完成了Sitemap文件的提交。百度搜索引擎会定期抓取该文件，从而获得博客网站中可供抓取的网页的链接地址。

![](/images/14610006885838.png)

### 3.3 自动推送
在 theme 目录的 `_config.yml` 中,开启

```
baidu_push: true
```

## 4. 添加robots.txt 

robots.txt是一种存放于网站根目录下的ASCII编码的文本文件，它的作用是告诉搜索引擎此网站中哪些内容是可以被爬取的，哪些是禁止爬取的。robots.txt应该放在站点目录下的source文件中，网站生成后在网站的根目录(`站点目录/public/`)下。 


我的 robots.txt 文件内容如下 
 
```
User-agent: *
Allow: /
Allow: /archives/
Allow: /categories/
Allow: /tags/

Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/

Sitemap: http://kingzeus.github.io/sitemap.xml
Sitemap: http://kingzeus.github.io/baidusitemap.xml
```   


