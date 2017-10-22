---
title: android studio 经验分享
date: 2016-04-15 20:40:00
updated: 2016-04-15 20:40:00
tags:
- Android Studio
- Android
- IDE
categories:
- Android
---


- 最后更新: 2016-04-15
- 修改记录:
	* 2014-04-15 初稿

----


自从 Google 放弃 Eclipse 转投 Android Studio 之后，我们的项目也从 Eclipse 迁移到了 Android Studio。使用期间踩坑无数，随着使用的时间越来越长，已经离不开 Android Studio 了，Eclipse 的时代终于过去了。

最近 Android Studio 也迎来了 2.0 正式版，正好分享下使用经验。

## 1. 配置 

Android Studio 提供了一个非常方便的功能帮助我们 **导入/导出** 设置。但是，升级 2.0 的过程中，不小心丢失了之前版本的设置。悲剧啊，不过正好来看看新版本的变化。

> 我的建议：立刻备份你的设置文件到安全的地方。 

当我在配置我的 Android Studio 的时候，下面的一些配置或许对你有一定的帮助。 

### 1.1 显示行号 

当我首次启动我的 Android Studio 的时候，我想做的第一件事就是希望能看到文件中的行号，我一直很奇怪这个基本的配置为毛不是默认开启的？!  

有人不需要么? 

* 未显示行号
![](/images/14618250074314.png)


* 显示行号
![](/images/14618250223192.png)


**配置方法**

  * File | Settings 打开设置
  * 选择 Editor | General | Appearance
  * 勾选 Show line numbers

![](/images/14618255934610.png)


**注意：** 在编辑区域最左侧右键选中 **Show line numbers** 也可以让当前打开的文件显示行号，不过这是一个临时设置，当前文件关闭后便失效。 
![](/images/14618261456427.jpg)


### 1.2 驼峰选择 

Android 开发中，我们通常会使用 `驼峰命名法` 对变量进行命名，但是当我们通过 Ctrl + Left / Right 键改变字符选择区域的时候 Android Studio 默认不支持 `驼峰` 单词的选择。 

* 不支持驼峰选择
![](/images/14618259214448.gif)

* 支持驼峰选择
![](/images/14618259544875.gif)


**配置方法**

  * File | Settings 打开设置
  * 选择 Editor | General | Smart Keys
  * 选中 Use “CamelHumps” words
![](/images/14618260869755.png)


**注意：** 如果你仍然希望当鼠标在单词上双击之后选中整个单词，需要作如下设置： 

  * File | Settings 打开设置
  * 选择 Editor | General
  * 取消选中 ‘Honor Camel Humps words settings when selecting on double click’
![](/images/14618263315533.jpg)



### 1.3 命名前缀 

我们通常会遵循 Android 官方关于编码风格的指导来进行字段命名。在 Android 源码中我们可以看到通常成员变量都是以‘m’开始。其实 Android Studio 可以自动在帮我们生成字段名称的时候加上自定义的前缀，如: 

  * 非共有，非静态的成员变量以’m’开始
  * 静态成员变量以’s’开始
  
![](/images/14618268759799.gif)

**配置方法**

  * File | Settings 打开设置
  * 选择 Editor | Code Style | Java
  * 选择 Code Generation 标签
  * 给普通 `Field` 添加一个’m’前缀，给 `Static filed` 添加一个’s’前缀
![](/images/14618270737167.png)


### 1.4 快速导包 

在 Android Studio 中，我们可以通过 Alt + Enter 和 Control + Alt + O 进行导包和清除无用导包，但这些事情应当快速自动完成。 

* 未开启imports on the fly
![](/images/14618271815542.gif)

  
* 开启imports on the fly
![](/images/14618271929685.gif)


**配置方法**

  * File | Settings 打开设置
  * 选择 Editor | General | Auto Import
  * 勾选 Optimize imports on the fly
  * 勾选 Add unambiguous imports on the fly

  ![](/images/14618272612459.png)


### 1.5 Log 颜色 

`Darcula` 主题中 `Logcat` 的默认配色只有红白两种颜色，不太便于我们区分 Log 的类型。 

* Darcula 主题配色
![](/images/14618273332588.png)

但是为了更加的直观，建议大家采用之前 `Android Holo` 主题那种鲜明的配色 

![](/images/14618274211487.png)

**配置方法**

  * File | Settings 打开设置
  * 选择 Editor | Color & Fonts | Android Logcat
  * 点击 Click on Save As… 按钮创建一个新的配色 Scheme
  * 按照下面的表格修改对应的颜色(修改之前需要取消勾选 Use inherited attributes)

Log类型 | 颜色
------ | ----
Assert | #AA66CC 
Debug | #33B5E5 
Error | #FF4444 
Info | #99CC00 
Verbose | #FFFFFF 
Warning | #FFBB33 


### 1.6 代码配色 

Android Studio 中默认的代码配色已经很和谐，但这个东西仁者见仁。比如有的朋友会觉得 java 代码中局部变量的默认的白色不太便于快速与其它代码进行区分，这时候就需要自定义 java 代码颜色，这里以局部变量为例。 

* 默认配色
![](/images/14618278134767.png)

* 自定义配色
![](/images/14618278258226.png)


**配置方法**

  * File | Settings 打开设置
  * 选择 Editor | Color & Fonts | Java
  * 点击 Click on Save As… 按钮创建一个新的配色 Scheme
  * 展开下方的 Variables 选择 Local variable
  * 设置右侧的 Foreground 颜色

![](/images/14618279086360.png)


### 1.7 工程模板 

Android Studio 创建 Module 时并没有将 Android 开发中常用的文件目录全部生成，比如默认只生成了一个 `drawable` 文件夹，常用的 `drawable-hdpi` 等文件夹需要我们自己创建。这些事情应该自动完成，毕竟我们都很‘懒’！ 

* 默认结构
![](/images/14618345830242.png)

* 自定义结构

![](/images/14618346245922.png)


**配置方法**

  * 进入 Android Studio 安装目录
  * 依次进入 plugins | android | lib | templates | gradle-projects | NewAndroidModule
  * 用编辑器打开 recipe.xml.ftl 文件，并加入以下配置

  ![](/images/14618350920038.png)



当然，通过类似的方式我们还可以在创建 Module 的时候做很多事情，比如： 

  * 在 colors.xml 文件中生成常用颜色
  * 在 build.gradle 文件中生成自定义配置
  * 在 .gitignore 文件中生成自定义忽略配置
  * 等等…

### 1.8 活动模板 

Android Studio 中默认提供了很多非常方便的活动模板(Live Templates)，例如，我们输入 `sout` 后按回车键， Android Studio 会自动帮我们写入 `System.out.println();`

![](/images/14618353662673.gif)

其实 `sout` 就是 AS 自带的一个活动模板。 

![](/images/14618353841421.png)

由此可以看出，活动模板就是我们常用代码的一个**缩写**。开发中有很多代码都会重复出现，因此自定义合适的活动模板能很大程度上避免我们很多重复的体力劳动。

那么如何自定义？
  
这里我们以 `Handler` 为例。下面是在 `Activity` 中一个合格的 `Handler` ： 
    
  
```java
private static class MyHandler extends Handler {
            private WeakReference<MainActivity> activityWeakReference;
    
            public MyHandler(MainActivity activity) {
                activityWeakReference = new WeakReference<MainActivity>(activity);
            }
    
            @Override
            public void handleMessage(Message msg) {
                MainActivity activity = activityWeakReference.get();
                if (activity != null) {
    
                }
            }
        }

```  
    
    

现在如果我只希望输入一个 `psh` 就自动出现上面这段代码的话，我应该这么做： 

**配置方法**

  * File | Settings 打开设置
  * 选择 Editor | Code Style | Live Templates
  * 点击最右侧的加号并选择 Template Group
  * 在弹出的对话框中输入一个活动模板分组的名称，如 custom
  * 在左侧选中上一步中创建的 custom 分组，点击右边的加号
  * 选择 Live Template ，在 Abbreviation 中对输入 `psh`
  * 在 Description 中输入这个活动模板的描述
  * 在 Template text 中输入以下代码

  ![](/images/14618356008642.png)
  * 点击下方的 Define 按钮，选中 java 表示这个模板用于 java 代码
  * 点击右侧的 Edit variables
  * 选择 Expression 下拉框中的 className 并勾选 Skip if…

> 这个操作的作用是，AS会自动将我们在上一步中用’$’符包裹的 `className` 自动替换为当前类不含包名的类名 

  * 点击 Apply 和 Ok 让设置生效。

至此，一个我们自定义的 custom 模板组中的 psh 活动模板就定义完成了。

![](/images/14618357184926.gif)

## 2. 插件
Android Studio 另一个优势也体现在插件系统上，你能方便找到别人做好的插件来加速工作。

**安装方法**

* File | Settings 打开设置
* 打开 Plugins
![](/images/14618377035606.jpg)

下面是一些好用的插件推荐

## 2.1 FindBugs
java 的静态代码分析工具，可以通过静态代码检查提升代码质量。
![](/images/14618382458895.jpg)

## 2.2 checkstyle


