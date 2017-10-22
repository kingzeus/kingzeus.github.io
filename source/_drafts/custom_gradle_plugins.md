---
title: 使用 Android Studio 定制 Gradle 插件
date: 2016-05-18 20:40:00
updated: 2016-05-18 20:40:00
tags:
- Android Studio
- Android
- Gradle
categories:
- Android
---

- 最后更新: 2016-05-18
- 修改记录:
	* 2014-05-18 初稿

----


## 缘由
之前一直使用了 Gradle 来构建 Android 应用，也尝试过一些简单的代码来定制 Gradle 脚本。直到最近正好要做热修复相关工作，于是研究了下 Nuwa 和 AndFix，以及 Android Studio 的 Instant Run。这三个工具在构建过程中使用了自定义的 Gradle 插件。所以学习一下 Gradle 插件的编写还是一件十分有意义的事。

## 插件类型
Gradle的插件一般有这么几种：

*	一种是直接在项目中的gradle文件里编写，这种方式的缺点是无法复用插件代码，在其他项目中还得复制一遍代码。
*	另一种是在独立的项目里编写插件，然后发布到中央仓库，之后直接引用就可以了，优点就是可复用。

## Gradle 语法

	Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化建构工具。它使用一种基于Groovy的特定领域语言(DSL)来声明项目设置，抛弃了基于XML的各种繁琐配置。
	当前其支持的语言限于Java、Groovy和Scala，计划未来将支持更多的语言。
										-- 维基百科

如果要学习 Gradle 相关的东西，请查看 [Gradle for Android](https://segmentfault.com/a/1190000004229002)

## Gradle 插件
Gradle 插件可以使用 java，因此我们正好可以使用 Android Studio 来开发。

1. 新建一个 Android 工程
2. 新增一个 Android Module 项目，类型选择 Android Library
3. 修改 Module 目录里的文件结构
	![](/images/14635811281941.jpg)
 4. 修改 Module 目录下的 build.gradle
 	
```
apply plugin: 'groovy'
apply plugin: 'maven'

dependencies {
    compile gradleApi()
    compile localGroovy()
}

repositories {
    mavenCentral()
}
```

